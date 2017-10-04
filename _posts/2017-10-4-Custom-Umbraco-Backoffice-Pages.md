---
layout: post
title: Customising the Umbraco Backoffice
---

This post will go through the setup of custom backoffice pages in Umbraco 7, as well as adding Autofac and Serilog

# Umbraco Project Setup

The process of creating an new umbraco instance is documented in the official docs at https://our.umbraco.org/documentation/getting-started/setup/install/install-umbraco-with-nuget 
For this post, I am using version 7.5.14.

All the code from this example can be found on my [GitHub](https://github.com/Togusa09/UmbracoBackofficeSample)

## Logging and Dependency Injection Container Setup

Using Autofac for the IoC setup
- Autofac
- Autofac.WebApi2 - WeApi dependency injection
- Autofac.Mvc5 - MVC dependency injection
- Autofac.Web - WebForms dependency injection

Using Serilog for the logging
- Serilog
- Serilog.Sinks.Seq - For writing logs to seq
- Serilog.WebClassic - For logging web content
- Serilog.WebClassic.WebApi - For logging webapi
- Serilog.Enrichers.Process - Supplies information on the running process

As Umbraco requires its UmbracoApplication class to be instantiated as part of application startup for all of its features to work correctly. UmbracoApplication already implements many of the normally used startup callbacks preventing them from being used, but provides additional callbacks that can be overridden in an inheriting class. 

Example implementation of Global.asax, registering Autofac for dependency injection and Serilog for logging

{% highlight c# %}
public class Global : UmbracoApplication, IContainerProviderAccessor // IContentProviderAccessor is required for WebForms dependency injection
{
    protected override void OnApplicationError(object sender, EventArgs e)
    {
        var exception = Server.GetLastError();
        var httpException = exception as HttpException;
        if (httpException != null && httpException.GetHttpCode() >= 500)
        {
            Log.Error(httpException, "Application error");
        }
    }
    protected override IBootManager GetBootManager()
    {
        return new WebBootManager(this);
    }

    static IContainerProvider _containerProvider;
    public IContainerProvider ContainerProvider
    {
        get { return _containerProvider; }
    }

    // Equivalent of Application_Start
    protected override void OnApplicationStarted(object sender, EventArgs e)
    {
        base.OnApplicationStarted(sender, e);

        var runtimeEnvironment = ConfigurationManager.AppSettings["Environment"];
        var seqUrl = ConfigurationManager.AppSettings["SeqUrl"];

        var log = new LoggerConfiguration()
            .MinimumLevel.Is(LogEventLevel.Debug)
            .Enrich.WithProcessId()
            .Enrich.WithProperty("MachineName", Environment.MachineName)
            .Enrich.WithProperty("Source", "Umbraco Web")
            .Enrich.WithProperty("Environment", runtimeEnvironment)
            .Enrich.With<UserNameEnricher>()
            .Enrich.With<HttpRequestUrlEnricher>()
            .Enrich.With<HttpRequestUserAgentEnricher>()
            .Enrich.With<HttpSessionIdEnricher>()
            .Enrich.With<HttpRequestIdEnricher>()
            .Enrich.FromLogContext()
            .WriteTo.Seq(seqUrl)
            .CreateLogger();
        Log.Logger = log;

        var builder = new ContainerBuilder();
        var config = GlobalConfiguration.Configuration;

        // Register Umbraco Context, MVC Controllers and API Controllers.
        builder.Register(c => UmbracoContext.Current).AsSelf();
        builder.Register(c => ApplicationContext.Current.Services.ContentService).As<Umbraco.Core.Services.IContentService>();

        builder.RegisterControllers(Assembly.GetExecutingAssembly()); // Register our MVC controllers
        builder.RegisterApiControllers(typeof(UmbracoApplication).Assembly); // Register umbraco backoffice controllers
        builder.RegisterApiControllers(typeof(Global).Assembly); // Register our API controllers

        builder.RegisterInstance(log).As<ILogger>();
        builder.Register(c => new HttpContextWrapper(HttpContext.Current)).As<HttpContextBase>();
        //builder.RegisterType<PagingService>().As<IPagingService>();
        //builder.RegisterType<WebApiCsvGenerator>().As<IWebApiCsvGenerator>();

        var container = builder.Build();
        _containerProvider = new ContainerProvider(container);
        config.DependencyResolver = new AutofacWebApiDependencyResolver(container);

        RouteConfig.RegisterRoutes(RouteTable.Routes);
        DependencyResolver.SetResolver(new AutofacDependencyResolver(_containerProvider.ApplicationContainer));

        AppDomain.CurrentDomain.UnhandledException += (s, args) =>
        {
            var exp = (Exception)args.ExceptionObject;
            Log.Error(exp, "Unhandled exception");
        };

        Log.Information("Website Started.");
    }
}
{% endhighlight %}

## AU Language support
Note for Australian languages: Umbraco doesn't support Australian English at the time of writing, only UK English and American English. Setting UK English gives the correct date formats, but uses the pound symbol for currency, while American English gives the dollar for currency, but uses the wrong date format. To create an EN_AU locale for umbraco, copy the existing US English or UK English and rename the file to "en_au.xml" replace the language element with `<language alias="en_au" intName="English (AU)" localName="English (AU)" lcid="" culture="en-AU">`. This will allow you to set users to have Australian English as their language in umbraco.

[Sample en_au language file]({{ site.url }}/download/en_au.xml)

The moment locale did sometimes not set correctly to en-au, so I had to add "moment.locale('en-au');" to line 10682 and 11026 in "umbraco.controllers.js" after the moment script was loaded to ensure it was working correctly.

## Build and deployment
### Refreshing cached files
Umbraco uses the client dependency framework for caching resources when running in release mode. A cache refresh is triggered when the value of the version attribute of the ClientDependency element in the ClientDependency.config file. As Umbraco does not automatically update this value, it needs to be manually updated as part of the build and deployment process, or else clients will have issues with cached resources.

A simple method of resolving this is to set the content of the attribute to a unique numeric value for the file .ie "0000000000", and the a find and replace build step to perform the substitution.
As the file is XML, powershell's xml support could also be used to perform the substitution.

### Setting folder permissions
The configuration screens in umbraco require the website to be able to modify files in the config and app_data folders. This can be configured as part of the deployment process using the following powershell script in octopus deploy:

{% highlight powershell %}
$siteRoot = $OctopusParameters['Octopus.Action[Deploy Step Name].Output.Package.InstallationDirectoryPath']
$siteName = $OctopusParameters["Site name"]
$configPath = Join-Path $siteRoot 'config'
$appDataPath = Join-Path $siteRoot 'App_Data'

Write-Host $SiteName

$acl = Get-ACL $configPath
$accessRule= New-Object System.Security.AccessControl.FileSystemAccessRule(`
    "IIS APPPOOL\$siteName", `
    "full", `
    "ContainerInherit,Objectinherit", `
    "InheritOnly", `
    "Allow")
$acl.AddAccessRule($accessRule)
Set-Acl $configPath $acl

$acl2 = Get-ACL $appDataPath
$accessRule2= New-Object System.Security.AccessControl.FileSystemAccessRule(`
    "IIS APPPOOL\$siteName", `
    "full", `
    "ContainerInherit,Objectinherit", `
    "InheritOnly", `
    "Allow")
$acl.AddAccessRule($accessRule2)
Set-Acl $appDataPath $acl2
{% endhighlight %}

### Media Folder
As users can upload content into umbraco, it is recommended to set Umbraco's media directory to a directory that will be persisted between deployments. This can be done by changing the umbracoMediaPath appsetting to point to a folder outside of the deployment directory, or configuring the website to have a virtual directory for the default "~/media" path. [Umbraco App Settings](https://our.umbraco.org/Documentation/Reference/Config/webconfig/)

### Security
By default umbraco uses an implmentation of the ASP.net identity provider for internal users and external membership, but does not have very practical 
security options enabled. 

If your site is using SSL (Which it should), require it for logging into umbraco with the appsetting `<add key="umbracoUseSSL" value="true" />`

I recommend setting the following attributes on the UsersMembershipProvider:
enablePasswordRetrieval="false" 
enablePasswordReset="true" 
requiresQuestionAndAnswer="false" 
passwordFormat="Hashed" 
allowManuallyChangingPassword="true"

[ASP Membership Security settings](https://msdn.microsoft.com/en-us/library/system.web.security.membership(v=vs.110).aspx)

# Adding Custom content

Extensions can go in App_Plugins to provide separation from standard umbraco content/features
All plugin.manifest files are loaded, when you're in the backoffice, regardless of what page you're on. Because of this, a 'Shared' plugin is not required, but is neater.


## Custom backoffice sections
To create a new menu option in the Umbraco Back office:

Create new application - Provides the new menu ico

{% highlight c# %}
[Application("CommerceManagement", "CommerceManagement", "management.gif", 15)]
public class CommerceManagementApplication : IApplication
{
}
{% endhighlight %}

Create new menu tree - Defines the navigation structure within a tree

{% highlight c# %}
[PluginController("CustomSection")]
[Umbraco.Web.Trees.Tree("CustomSection", "CustomSectionTree", "Custom Section", iconClosed: "icon-doc")]
public class CommerceManagementTreeController : TreeController
{
    protected override TreeNodeCollection GetTreeNodes(string id, FormDataCollection queryStrings)
    {
        var nodes = new TreeNodeCollection();

        // Create a new node in the menu tree, that will open the specified url
        var item = this.CreateTreeNode("1a", id, queryStrings, "Find Custom Item", "developerMacro.gif", false, "CustomSection/CustomSectionTree/FindItem/0");
        nodes.Add(item);

        return nodes;
    }
    protected override MenuItemCollection GetMenuForNode(string id, FormDataCollection queryStrings)
    {
        var menu = new MenuItemCollection();
        menu.DefaultMenuAlias = ActionNew.Instance.Alias;
        return menu;
    }
}
{% endhighlight %}

Newly added menus will be disabled for users by default. They can be enabled using the user settings menu in umbraco.

Umbraco's menu structure is designed for each entity to exist a node in the tree structure, and to an edit.html for each. This doesn't let itself well to large sets of data such as a list of orders. It's possible to manipulate the route resolution to allow you to use a html file with a specific name instead of the default edit.html. This allows the addition of custom views that don't have to confirm to umbraco's expected structure, which has the upside of allowing quick and easy development for adding simple CRUD pages, but prevents proper integration with Umbraco's features.

Folder structure maps to urls as described below:
Umbraco#/PluginName/PluginTreeName/PageName/0
~/App_Plugins/PluginName/backoffice/PluginTreeName/PageName.html

Umbraco uses angular ui router for its navigation, so it may be possible to define custom routes. As I am avoiding making any changes involving the contents of the `umbraco` and `umbraco_client` folders, I haven't investigated this.

Javascript and css files used in the custom section can be listed in the package.manifest file in the root directory (~\App_Plugins\CustomSection\package.manifest)

## Custom pages

Sample umbraco resource
{% highlight javascript %}
angular.module('umbraco.resources').factory('customItemResource',
    function ($q, $http, umbRequestHelper) {
        //the factory object returned
        return {
            getItem: function () {
                return umbRequestHelper.resourcePromise(
                    $http.get("backoffice/api/CustomItem/GetItems"),
                    "Failed to retrieve Items");
            },
            getItemsCsv: function () {
                return $http.get("backoffice/api/CustomItem/GetItemsCsv");
            },
            getItem: function (id) {
                return umbRequestHelper.resourcePromise(
                    $http.get("backoffice/api/CustomItem/GetItem/" + id),
                    "Failed to retrieve CustomItem");
            },
            updateItem: function (id, updatedCustomItem) {
                return umbRequestHelper.resourcePromise(
                    $http.get("backoffice/api/Brand/CustomItem/" + id, updatedCustomItem),
                    "Failed to update CustomItem");
            },
        }
    });
{% endhighlight %}


### List page example
Example of search page that lists results with a loading indicator and paging. The `umbraco-property` element is an element I created to make managing the layout of fields.

{% highlight html %}
<div class="umb-panel umb-editor-wrapper" ng-controller="CustomSection.ListCustomItemController as vm">
    <form name="mySectionForm" novalidate ng-submit="vm.search()">
        <umb-editor-view>
            <umb-editor-header name="vm.pageName"
                               hide-alias="true"
                               hide-description="true"
                               hide-icon="true"
                               name-locked="true">
            </umb-editor-header>

            <umb-editor-container>
                <div class="form-horizontal">
                    <umbraco-property title="Search Criteria">
                        <input type="text" ng-model="vm.searchCriteria.searchProperty" />
                    </umbraco-property>
                    <button type="submit" class="btn">Search</button>
                    <button type="button" class="btn" ng-click="vm.clear()">Clear</button>
                    <record-count total-items="vm.totalItems" page-number="vm.page" page-count="vm.pageCount"></record-count>

                    <umb-load-indicator ng-if="vm.loading">
                    </umb-load-indicator>
                    <div ng-if="!vm.loading">
                        <table class="table table-striped" cellspacing="0" cellpadding="5px">
                            <thead>
                                <tr>
                                    <th class="firstCol"><a ng-click="vm.sortBy('id')">Id</a></th>
                                    <th class="big-col"><a ng-click="vm.sortBy('name')">Name</a></th>
                                    <th class="small-col">Edit</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr ng-repeat="searchResult in vm.searchResults">
                                    <td class="firstCol">{{searchResult.Id}}</td>

                                    <td>{{searchResult.Name}}</td>
                                    <td><a ng-href="#/CustomSection/CustomSectionTree/EditCustomItem/{{searchResult.Id}}">Edit</a></td>
                                </tr>
                            </tbody>
                        </table>
                        <umb-pagination ng-if="vm.pageCount"
                                        page-number="vm.currentPage"
                                        total-pages="vm.pageCount"
                                        on-go-to-page="vm.goToPage"
                                        on-next="vm.next"
                                        on-prev="vm.prev">
                        </umb-pagination>

                    </div>
                </div>
            </umb-editor-container>
        </umb-editor-view>
    </form>
</div>


{% endhighlight %}

### Edit page template

Example editing page that displays a save and back button on a menu at the bottom of the screen, matching the layout used by Umbraco on its screens. There is also a loading indicator for display while the the entity is saving and loading.

{% highlight html %}
<div class="custom-editor" ng-controller="CustomSection.EditCustomItemController as vm">
    <form name="editForm" val-form-manager novalidate>
        <umb-editor-view>
            <umb-editor-header name="vm.pageName"
                               hide-alias="true"
                               hide-description="true"
                               hide-icon="true"
                               name-locked="true">
            </umb-editor-header>
            <umb-editor-container>
                <div class="form-horizontal">
                    <umb-load-indicator ng-if="vm.loading">
                    </umb-load-indicator>
                    <div ng-if="!vm.loading">
                        <div class="well">
                            <p>Sample editing page</p>
                        </div>

                        <div class="form-horizontal">
                            <umbraco-property title="Item Id">
                                {{vm.customItem.Id}}
                            </umbraco-property>
                            <umbraco-property title="Name">
                                <input type="text" ng-model="vm.customItem.Name"/>
                            </umbraco-property>
                        </div>
                    </div>
                </div>
            </umb-editor-container>
            <umb-editor-footer>
                <umb-editor-footer-content-left>
                    <a href="#/CustomSection/CustomSectionTree/ListCustomItem/0">Back to search</a>
                </umb-editor-footer-content-left>
                <umb-editor-footer-content-right>
                    <button type="button" class="btn btn-success" ng-disabled="!editForm.$valid || vm.loading" ng-click="vm.save(editForm)" hotkey="ctrl+s">Save</button>
                </umb-editor-footer-content-right>
            </umb-editor-footer>
        </umb-editor-view>
    </form>
</div>
{% endhighlight %}

## Backoffice Api Controllers

Umbraco provides controller implementations that integrate with the Umbraco back office authentication, simplifying the process of securing the backoffice api and pages.

Umbraco backoffice Controllers:
UmbracoAuthorizedJsonController - Authorised umbraco backoffice api controller that is autorouted to "/umbraco/backoffice/api/{controller}/{action}"
UmbracoAuthorizedController - Authorised umbraco backoffice mvc controller that is manually routed.

Code snippets:

Paging service is a small utility class to handle common paging methods

{% highlight c# %}
public class PagingService : IPagingService
{
    public int GetLastPage(int totalItems, PagedRequest request)
    {
        return (int)Math.Ceiling((decimal)totalItems / request.PageSize);
    }

    public int BoundPageNumber(int totalItems, int page, int lastPage)
    {
        if (page > lastPage)
            page = lastPage;
        if (page < 1)
            page = 1;
        return page;
    }
}
{% endhighlight %}

As we're supporting paging in the search screens, it is helpful to have some base classes that can be used as base classes when there's custom parameters, or by themselves when there's no extra search parameters. This object allows page, sorting parameter, sort order and page size to be set.

{% highlight c# %}
public enum SortDirection
{
    ASC,
    DESC
}

public class PagedRequest
{
    public PagedRequest()
    {
        Page = 1;
        SortBy = string.Empty;
        SortDirection = SortDirection.ASC;
        PageSize = Constants.DefaultPageSize;
    }

    public int Page { get; set; }
    public string SortBy { get; set; }
    public SortDirection SortDirection { get; set; }
    public int PageSize { get; set; }
}
{% endhighlight %}

As with the request, we want a reusable class for returning the paging information, as well as including the list of items for the current page.

{% highlight c# %}
public class PagedListResponse<T>
{
    public PagedListResponse(List<T> list, int pageNumber, int pageCount, int totalItems)
    {
        List = list;
        PageNumber = pageNumber;
        PageCount = pageCount;
        TotalItems = totalItems;
    }

    public List<T> List { get;  } 
    public int PageNumber { get; }
    public int PageCount { get; }
    public int TotalItems { get;  }
}
{% endhighlight %}

Building the response is will be a common process for everything returning a paged list, so we can put this in a common base clase.

{% highlight c# %}
public abstract class BackofficeApiController : UmbracoAuthorizedJsonController
{
    private readonly IPagingService _pagingService;

    protected BackofficeApiController(IPagingService pagingService)
    {
        _pagingService = pagingService;
    }

    protected PagedListResponse<T> GetResponse<T>(PagedRequest request, int totalItems, IEnumerable<T> results)
    {
        var lastPage = _pagingService.GetLastPage(totalItems, request);
        var list = results.ToList();
        var pageNumber = _pagingService.BoundPageNumber(totalItems, request.Page, lastPage);

        return new PagedListResponse<T>(list, pageNumber, lastPage, totalItems);
    }
}
{% endhighlight %}

And finally we use all that code to create a stub controller and its request class, that will respond to the requests from the frontend.
{% highlight c# %}
public class CustomItemRequest : PagedRequest
    {
        public string SearchProperty { get; set; }
    }

    public class CustomItemController : BackofficeApiController
    {
        private List<CustomItem> _items = new List<CustomItem>();


        public CustomItemController(IPagingService pagingService) : base(pagingService)
        {
            for (int i = 0; i < 100; i++)
            {
                _items.Add(new CustomItem
                {
                    Id = i,
                    Name = $"Item {i}"
                });
            }
        }

        [HttpGet]
        public PagedListResponse<CustomItem> GetCustomItems([FromUri]CustomItemRequest request)
        {
            IEnumerable<CustomItem> results = _items;
            if (!string.IsNullOrWhiteSpace(request.SearchProperty))
            {
                results = results.Where(r => r.Name.Contains(request.SearchProperty));
            }
            var totalItemCount = results.Count();
            var startAt = (request.Page - 1) * request.PageSize;

            var pageResult = results.Skip(startAt).Take(request.PageSize);

            return GetResponse(request, totalItemCount, pageResult);
        }

        [HttpGet]
        public CustomItem GetCustomItem(int id)
        {
            return _items.FirstOrDefault(i => i.Id == id);
        }

        [HttpPost]
        public void UpdateCustomItem(int id, CustomItem model)
        {
            // Not actually updating anything
        }
    }
{% endhighlight %}