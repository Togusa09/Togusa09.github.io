---
layout: post
title: Umbraco Setup and Configuration
---

Describe umbraco, summaries what we're going to do
Custom content goes in App_Plugins folder

Description of how to set up an Umbraco website using TeamCity and Octopus deploy for building and deployment.

Will cover: (Add links)
- Setup
    - Installation
    - Logging & Dependency Injection
    - AU Hacks
- Build and deployment config
    - Folder permissions
    - Virtual folders for media
    - Forcing cache refresh on deployment
- Creating new backoffice Sections
    - Create menus
    - Javascript file structure
    - Templates
    - Controllers
        - Types
        - Code snippets
    - Downloading from API controller
    - Uploading with parameters

# Umbraco Project Setup

## Logging and Dependency Injection Container Setup

Using Autofac for the IoC setup
- Autofac
- Autofac.WebApi - WeApi dependency injection
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

 [Umbraco Security settings](https://our.umbraco.org/Documentation/Reference/Security/)

# Custom content

Extensions can go in App_Plugins to provide separation from standard umbraco content/features
All plugin.manifest files are loaded, when you're in the backoffice, regardless of what page you're on. Because of this, a 'Shared' plugin is not required, but is neater.



For new section backoffice menu:

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

Page paths - By default umbraco expects all backoffice pages to be for an entity, so expects an id at the end of the URL. Umbraco is using UI router internally, so this may be customisable. After some trial and error, we determined that the mapping of urls to page html files was as follows:

Umbraco#/PluginName/PluginTreeName/PageName/0
~/App_Plugins/PluginName/backoffice/PluginTreeName/PageName.html


Template for basic layout

## List page template

## Edit page template

Page references the controller, so having a controller is optional

{% highlight html %}
<div class="custom-editor" ng-controller="Section.CustomController as vm">
    <form  val-form-manager>
        <umb-editor-view>
            <umb-editor-header name="vm.pageName"
                               hide-alias="true"
                               hide-description="true"
                               hide-icon="true"
                               name-locked="true">
            </umb-editor-header>

            <umb-editor-container>
            </umb-editor-container>
        </umb-editor-view>
        <umb-editor-footer>
            <umb-editor-footer-content-right>
                <button type="button" class="btn btn-success" ng-disabled="!clinicForm.$valid || vm.loading" ng-click="vm.save(clinicForm)" hotkey="ctrl+s">Save</button>
            </umb-editor-footer-content-right>
        </umb-editor-footer>
    </form>
</div>
{% endhighlight %}

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

Controller implementation
UmbracoAuthorizedJsonController - Authorised umbraco backoffice api controller that is autorouted to "/umbraco/backoffice/api/{controller}/{action}"
UmbracoAuthorizedController - Authorised umbraco backoffice mvc controller that is manually routed.

Code snippets:

Doing paging calculations

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

Returning a content stream as a response to a webapi request

{% highlight c# %}
public class WebApiCsvGenerator : IWebApiCsvGenerator
{
    public HttpResponseMessage GenerateCsvResponse(object[] content, string fileName)
    {
        DataTable table = Utils.CreateDataTable(content);
        return GenerateCsvResponse(table, fileName);
    }

    public HttpResponseMessage GenerateCsvResponse(DataTable dataTable, string fileName)
    {
        var stream = new MemoryStream();
        var writer = new StreamWriter(stream);

        Utils.WriteCSVFileToStream(writer, dataTable);
        stream.Flush();
        stream.Position = 0;

        var result = new HttpResponseMessage(HttpStatusCode.OK);
        result.Content = new StreamContent(stream);
        result.Content.Headers.ContentType = new MediaTypeHeaderValue("text/csv");
        result.Content.Headers.ContentDisposition = new ContentDispositionHeaderValue("attachment") { FileName = fileName };
        return result;
    }
}
{% endhighlight %}


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