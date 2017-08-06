---
layout: post
title: Upgrading Umbraco from v4 to v7
---

Recently for work I've had the opportunity to upgrade a website from using version 4 of Umbraco to version 7. Due to the significant investment already made in the website, the upgrade needed to be able to  reuse as much of the existing customisations and templates as possible. Fortunately version 7 still supports the older template format that used ASP.net master pages.

Upgrading an Umbraco website is composed of two main parts. Upgrading the umbraco database, and upgrading the website content itself.

# Upgrading Database
Normally the database is upgraded the first time that the website is started after being upgraded. An administration screen is displayed on startup, prompting the user to login and authorise the upgrade.

While this is practical for upgrades between minor versions, there can be larger database changes that prevent this being done in a single step, such as the changes to the authentication system in 7.3.X. 
To handle the database upgrade process, I suggest incrementally stepping through versions to perform the upgrade. An upgrade project can be done by creating an empty mvc website, then installing the desired umbraco version through nuget, and configuring it to use your database. In version 6 the database connection string is now stored as a connection string instead of as an app setting. The umbraco upgrade process will automatically migrate the connection string if found.
 
Where configuring the upgrade projects to perform the database upgrade, there are two settings that need to be customised:
- Database connection string: For upgrading to Umbraco 6, the existing app setting can be copied across, but the Umbraco 7 upgrade will need it as a connection string.
- umbracoConfigurationStatus app setting: This tells Umbraco what the current version of the database is.

For my upgrade, I stepped through the following versions:
- 6.2.4
- 7.3.1
- 7.5.15 (Initially went to 7.6.4, but had to go to an earlier version after discovering a bug with winforms templates).

# Upgrading website
## .Net and  Nuget Packages
Moving from Umbraco 4 to 7 not only brings a number of API changes, but also raises the minimum supported version of .net to 4.5.2. When the .net version on a project is changed, nuget packages need to be reinstalled so that the correct versions of the assemblies is reference to by the project. By changing the .net version before upgrading the Umbraco libraries, most packages will be reinstalled with a newer version whe the Umbraco packages are updated.

For any other packages that weren't reinstalled as part of the upgrade, Visual Studio will provide a warning that they are targeting an old version of .net. Once identified, these can be reinstalled with the following command using the nuget console:

```Update-Package <package_name> -ProjectName <project_name> -reinstall```
[NuGet Reference](https://docs.microsoft.com/en-us/nuget/consume-packages/reinstalling-and-updating-packages)

## API Changes

- Umbraco startup now relies on global.asax inheriting from the UmbracoApplication class to correctly initialise Umbraco's application context and services. If you have no startup customisation, you can just change the class referenced in global.asax. If you have existing startup code that relies on `Application_Start`, it will need to be updated to use the `protected override void OnApplicationStarted(object sender, EventArgs e)` callback provided by UmbracoApplication, or else it will not be executed.

- If you have made use of the uComponents plugin, its functionality has now been merged into the Umbraco libraries, so this package can be removed.

- The `Document` class has been replaced by the `Content` class, which implements the `IContent` interface.

- Content properties have their values returned directly instead of using a property class. `.GetProperty("name").value` becomes  `GetValue("name")`.

- Services are  now longer static, but are provided as singletons on the Umbraco Application context.

    ```c#
    DataTypeDefinition dataType = DataTypeDefinition.GetAll().FirstOrDefault(dt => dt.Text == dataTypeName);
    ```
    Becomes
    ```c#
    var dataTypeService = ApplicationContext.Current.Services.DataTypeService;
    IDataTypeDefinition dataType = dataTypeService.GetAllDataTypeDefinitions().FirstOrDefault(dt => dt.Name == dataTypeName);
    ```

- PreValues are now keyed and handled as dictionaries instead of lists of strings. Legacy support is provided for loading existing PreValue collections as string lists, but they the method to save updated PreValues will only accept a dictionary.
    ```c#
    var values = PreValues.GetPreValues(dataType.Id).GetValueList().Cast<PreValue>();
    ```
    becomes
    ```c#
    var dataTypeService = ApplicationContext.Current.Services.DataTypeService;
    var values = dataTypeService.GetPreValuesCollectionByDataTypeId(dataType.Id);
    ```

- If you are using an IoC container to resolve controllers, you will have to register the umbraco controllers. In Autofac ```builder.RegisterApiControllers(typeof(UmbracoApplication).Assembly);```

- If upgrading Autofac to version 4, after the container is build you no longer deal with `IContainer`, but solely with `ILifetimeScope`.

## Gotchas

If you have the normal global MVC route registered (```/{controller}/{action}/{id}```), it will interfere with Umbraco and prevent it from being able to resolve templates. This was not an issue in Umbraco 4, as pages ended with .aspx, but pages in Umbraco 7 are extensionless. If you need to use custom controllers, it is recommended to recommend inherit from one of the Umbraco controller classes, such as `SurfaceController` and use the route umbraco registers for it. If custom routing is needed, it should be limited to avoid interference with umbraco.




Next up I will detail the challenges encountered when rebuilding Umbraco backoffice custom sections for Umbraco 7. 