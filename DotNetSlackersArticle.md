Published on November 15, 2007 on [DotNetSlackers.com](http://dotnetslackers.com/articles/aspnet/ErrorLoggingModulesAndHandlers.aspx)

Written by [Simone Busoli](http://dotnetslackers.com/community/members/simoneb.aspx)

Simone Busoli does a detailed overview of the ELMAH open source project.



# Introduction #

[ELMAH](http://elmah.googlecode.com/) is an open source project whose purpose is to log and report unhandled exceptions in ASP.NET web applications. Since its first public release back in 2004 as a sample to the MSDN article [Using HTTP Modules and Handlers to Create Pluggable ASP.NET Components](http://msdn2.microsoft.com/en-us/library/aa479332.aspx), it has gained many new features and extended its support for newer ASP.NET releases - yet without dropping compatibility with older ones.

This article provides a comprehensive overview of the project as well as a detailed guide about setting it up under different environments.

> _Note:_ This article is based on the [BETA](http://code.google.com/p/elmah/downloads/detail?name=ELMAH-1.0-BETA2-src.zip) [2](http://code.google.com/p/elmah/downloads/detail?name=ELMAH-1.0-BETA2-bin.zip) release of ELMAH 1.0 and therefore some details may possibly change by the time it is released. Once the RTM release is reached this article will be updated to reflect changes and include new features.

Back in September 2004 [Atif Aziz](http://www.raboof.com/) and [Scott Mitchell](http://www.4guysfromrolla.com/ScottMitchell.shtml) published an article on the MSDN Library whose purpose was to serve as a proof of concept that writing self-contained features with ASP.NET HTTP modules and handlers was definitely possible and mostly that such features could be plugged into existing applications with no recompilation and minor changes to configuration files. To demonstrate these concepts the article was published with a sample project whose aim was to intercept, log and notify unhandled exceptions occurring in ASP.NET applications. It was given the name of ELMAH, Error Logging Modules And Handlers.

Since then quite some things have changed; most notably its release as an open source project under the New BSD license on the [Google Code](http://code.google.com/) hosting environment. The remainder of this article provides up-to-date information about the latest release of ELMAH, its working principle and configuration details.

# Purpose #

ELMAH serves as an unobtrusive interceptor of unhandled ASP.NET exceptions, those usually manifesting with the ASP.NET [yellow screen of death](http://en.wikipedia.org/wiki/Yellow_Screen_of_Death).

> Note: The screenshots showed in the remainder of the article may vary according to the environment where the application runs. As the environment I refer to the operating system, the web server and the ASP.NET version as a whole. The environment in which the application shown in the screenshots ran consisted of Microsoft Windows XP SP2, IIS 5.1 and ASP.NET 2.0. The screenshots were taken in Mozilla Firefox 2.0 browser (Italian localization).

Figure 1 shows what happens by default when an unhandled exception is raised into an ASP.NET web application. In the sample, this result is achieved by throwing an [InvalidOperationException](http://msdn2.microsoft.com/en-us/library/system.invalidoperationexception.aspx) exception after the click of a button, as shown in the following code snippet.

```
protected void ButtonRaiseException_Click(object sender, EventArgs e)   
{   
    throw new InvalidOperationException();   
}
```

**Figure 1: The yellow screen of death**

![http://dotnetslackers.com/images/articleimages/yellow-screen-of-death-1.png](http://dotnetslackers.com/images/articleimages/yellow-screen-of-death-1.png)

Note that the _Source Error_ section shown in Figure 1 is only displayed if either the `debug` attribute of the compilation configuration section in the Web.config file is set to `true` or debug is enabled at page level - which is not by default if either declaration is omitted.

```
<system.web>     
    <compilation debug="true" />     
</system.web>
```

```
<%@ Page Language="C#" Debug="true" %>   
```

When unhandled exceptions occur, there's not much a website administrator can do if he doesn't get any notification, and furthermore reproducing the error case might not always be possible.

Moreover, to prevent security risks most production environments set the Web.config `customErrors` configuration element mode to `On`, eventually using a custom error page, to prevent the details of the exception from being displayed to end users. This further hardens the job of discovering and fixing the problem which caused the application malfunction.

> Note: The default value for the mode attribute of the `customErrors` node is `RemoteOnly`, which means that custom errors are shown to clients accessing the application from remote locations while full descriptive error messages are shown to local clients; a good compromise between hardening the system and giving developers the information they need.

```
<system.web>  
    <customErrors mode="On" />  
</system.web>  
```

Figure 2 shows what the error screen looks like when custom errors are enabled.

**Figure 2: The yellow screen of death with custom errors enabled**

![http://dotnetslackers.com/images/articleimages/yellow-screen-of-death-2.png](http://dotnetslackers.com/images/articleimages/yellow-screen-of-death-2.png)

Enabling custom errors thus prevents sensitive information from being revealed but at the same time such data from being communicated to website administrators. Besides, instead of displaying the error page with no useful message the `customErrors` section accepts an attribute which allows to specify a default error page to show whenever an unhandled exception occurs.

```
<system.web>  
    <customErrors mode="On" defaultRedirect="GenericErrorPage.htm" />  
</system.web>  
```

A sample error page to notify of an unhandled exception is shown in Figure 3.

**Figure 3: A custom error page**

![http://dotnetslackers.com/images/articleimages/custom-error.png](http://dotnetslackers.com/images/articleimages/custom-error.png)

When this happens by default there's no way to get the details of the exception, except turning off custom errors to get all the details shown in Figure 1 and trying to reproduce the error case while hoping no user is going to do the same at the same time - given that it's trivial to reproduce the issue, which is not always the case.

> Note: Additional information about the `customErrors` configuration section can be found on the official documentation at [this page](http://msdn2.microsoft.com/en-us/library/h0hfz6fc.aspx).

While malfunctions are often unpredictable and unavoidable, an unobtrusive mechanism to log and get notifications about them happening is something definitely desirable.

This is where ELMAH taps in; without affecting the normal flow of the application ELMAH seamlessly integrates with it, intercepting, logging and notifying all the details of each unhandled exception with several configuration options which make it customizable to a great extent. Above all, ELMAH is able to grab each and every detail about the exception even if custom errors are enabled, so that website maintainers are given all the details they need while users will just see a customized error message.

In the end ELMAH won't prevent unhandled exceptions from occurring, it instead provides a means to tackle them as soon as they happen.

# Features #

Before delving into each single feature offered by ELMAH, an overview of its capabilities is in order.

> Note: The features discussed in the following paragraphs are independent from each other. They can be enabled or disabled individually.

## Logging ##

The logging mechanism is the main feature, as it allows recording unhandled exceptions together with their details on one of several available persistence stores.

By default, errors are stored in memory, therefore they don't survive application restarts or server resets, but other non volatile persistence stores are available as well - and more likely to be used in real scenarios. They include one which logs exceptions to XML files, one for the open source relational database engine SQLite, and another one for Microsoft SQL Server.

Although those several alternatives are available, custom logging back ends can be implemented by simply inheriting a base abstract class.

## Reporting ##

Once uncaught exceptions are logged by means of one of the mechanisms introduced above, they need to be able to be shown to users authorized to view such data. ELMAH provides several reporting mechanisms which format the list of logged exceptions - or single exceptions - in different fashions and with different detail levels.

Without delving too much into this topic, which will be discussed and demonstrated later, ELMAH can format errors in the following ways:

  * a list of all the errors together with the main information about them displayed as an HTML page and formatted as a table;
  * both HTML and XML/raw view of all the details of a single exception, together with the server variables pertaining the request which generated the exception;
  * a RSS feed of the 15 most recent exceptions;
  * a RSS digest showing a more compact summary of recently logged exceptions;
  * a downloadable file in CSV format of all the exceptions.

> Note: ELMAH provides a switch to block access to its error reports from remote locations which can be enabled via configuration. Another mechanism for blocking access to reports is using the standard ASP.NET authorization features. These concepts will be illustrated more clearly later.

## Notification ##

In addition to silently logging and storing information about uncaught exceptions, ELMAH is capable of immediately notifying the error via email as soon as it occurs.

Email notifications can be configured to be sent either synchronously or asynchronously.

## Filtering ##

By default all uncaught exceptions are logged, and eventually notified via email, but ELMAH offers a rich filtering mechanism that can specify exactly what exceptions to record and which ones to dismiss. Filtering can be performed in code or declaratively using the web application's configuration file.

## Error Signaling ##

This is one of the newest and most requested features. By design ELMAH can intercept only unhandled exceptions, but what if one wants to log and be notified of exceptions caught by the code, or eventually directly created by developers to notify of an error condition? In the sample code above an exception was thrown explicitly, and of course ended up into an error message more or less informative being displayed to the user. If that exception was caught no one would ever know about it, while in some circumstances one might want to.

One situation in which such a scenario is more likely to occur is during Ajax or Web Service calls, where exceptions usually need to be handled in order to prevent them from bubbling up as well as to provide the invoking client code with a descriptive message of the error occurred, but always with a standard response, for example in JSON or XML format.

Error signaling consists in the ability to tap into ELMAH infrastructure using an ad-hoc API to signal manually created exceptions.

## Update Notification ##

This is another new feature. ELMAH can now check for the availability of a new version, and in case of redirecting to the download page to grab the latest bits.

# Setting it up #

Now comes the interesting part. Setting up ELMAH is very straightforward and mostly consists in editing the configuration file of the application. Moreover, ELMAH distribution comes with a well-commented sample Web.config file containing all the necessary and available configuration entries.

The first essential step consists in getting the latest release of ELMAH, which can be downloaded from the project page (see the links at the bottom), and referencing the `Elmah.dll` assembly compiled for the right version of the .NET framework from the application.

One of the obstacles one might encounter concerns the differences in the features/limitations specific to particular environments, which change according to whether the application runs on ASP.NET 1.x or 2.0 and in Full or partial Trust levels. The following paragraphs will illustrate the steps needed to configure ELMAH and will notify whenever a feature either requires configuration changes to work on a particular environment or eventually isn't designed to work there.

> Note: One important constraint to be aware of is that ELMAH isn't designed to work in ASP.NET 1.1 applications under partial trust. In such an environment it needs Full Trust to work correctly. This however usually represents a minor issue since hosting providers rarely ever used the trust levels until the release of the .NET framework 2.0.

## Declaring configuration sections ##

In order to provide a customization mechanism for the features introduced in the previous section ELMAH requires the configuration file to contain specific sections declarations under the root configuration node, as shown in the following snippet.

```
<configSections>     
  <sectionGroup name="elmah">     
    <section name="security" requirePermission="false" type="Elmah.SecuritySectionHandler, Elmah" />     
    <section name="errorLog" requirePermission="false" type="Elmah.ErrorLogSectionHandler, Elmah" />     
    <section name="errorMail" requirePermission="false" type="Elmah.ErrorMailSectionHandler, Elmah" />     
    <section name="errorFilter" requirePermission="false" type="Elmah.ErrorFilterSectionHandler, Elmah" />    
  </sectionGroup>     
</configSections>
```

There's a default section group named `elmah` which contains custom configuration sections to tune several aspects concerning security, logging, notification and filtering.

> Note: Declaring all the configuration sections is not compulsory if a specific section is not needed. For example, if no error filtering needs to be performed the corresponding section can be omitted; the same applies to the other sections as well.

Notice the presence of the `requirePermission` attribute in each section declaration.

Note: The `requirePermission` attribute is new to ASP.NET 2.0 and it's not recognized by ASP.NET 1.x, thus it must be omitted in that environment.

In ASP.NET 2.0 it serves the purpose of allowing access to configuration sections to partially trusted applications. By default, access to configuration sections is granted only under High and Full trust levels - with a couple of exceptions. If running under less permissive trust levels the `requirePermission` attribute needs to be specified and its value set to `false` for ELMAH to be able to access the corresponding configuration section.

Since all configuration sections are declared within a section group named `elmah`, all the corresponding settings, if defined, must follow the same hierarchy relationship. This will be described more accurately later.

## Setting up and testing simple error reporting ##

As its name suggests, ELMAH works on top of HTTP modules and HTTP handlers to provide its functionality. The reporting mechanism relies on HTTP handlers capable of formatting errors and their details in several formats. To set up the reporting feature a new entry in the `httpHandlers` node of the configuration file is needed, as shown in the following snippet.

```
<httpHandlers>  
  <add verb="POST,GET,HEAD" path="elmah.axd" type="Elmah.ErrorLogPageFactory, Elmah" />  
</httpHandlers>  
```

This enables requests forwarded to the `elmah.axd` file to be handled by the `ErrorLogPageFactory` class, which according to the parameters passed on the query string outputs a different view of the errors.

> Note: The `elmah.axd` file doesn't need to be a real file on the file system. This is just the default ASP.NET way to map requests to certain resources to classes implementing the `IHttpHandler` or `IHttpHandlerFactory` interfaces.

It's not important the path chosen for the handler, as well as where it is virtually placed on the application folder hierarchy, as long as the extension is mapped on the web server as an extension handled by the ASP.NET pipeline. This will be clearer when I'll talk about security.

The `.axd` extension is just a conventional name for handlers usually employed by ASP.NET to provide some kind of service, like the diagnostic trace tool `trace.axd` or the `WebResource.axd` handler which extracts resources embedded in .NET assemblies.

Now, browsing to the `elmah.axd` file in the application it's been configured for should show a page similar to the following.

**Figure 4: Default error view via the elmah.axd handler**

![http://dotnetslackers.com/images/articleimages/elmah.axd.png](http://dotnetslackers.com/images/articleimages/elmah.axd.png)

At this point, no error is displayed even if the application throws unhandled exceptions, since error logging has not yet been set up. The picture shows on the top of the page some links which bring to other views of the errors, like an RSS feed, a RSS digest, and a downloadable file containing errors in CSV (comma-separated values) format. These alternate views will be illustrated later once error logging has been enabled.

## Enabling and configuring error logging ##

So far, if an uncaught exception bubbled up in the application no error would be shown by ELMAH, since it's not been configured for intercepting errors yet.

Error logging is achieved via an HTTP module called `ErrorLogModule` which intercepts uncaught exceptions at the application level and logs them into a persistence store.

To enable this behavior, the `ErrorLogModule` needs to be declared in the `httpModules` node of the `Web.config` file, as shown in the snippet below.

```
<system.web>  
  <httpModules>  
    <add name="ErrorLog" type="Elmah.ErrorLogModule, Elmah" />  
  </httpModules>  
</system.web>  
```

Error logging is performed by means of one out of several concrete inheritors of the base `ErrorLog` class.

As introduced before, by default errors are logged in central memory via the `MemoryErrorLog` class, thus making them volatile due to application restarts, but other, non volatile storages are available, and customized providers can be created by inheriting the base `ErrorLog` class.

Once the logging module has been enabled, uncaught exceptions should end up being intercepted by ELMAH and then available for viewing in the default reporting page.

**Figure 5: Default error view containing errors**

![http://dotnetslackers.com/images/articleimages/elmah.axd-with-error.png](http://dotnetslackers.com/images/articleimages/elmah.axd-with-error.png)

ELMAH ships with four error log providers, listed in the following table, together with their compatibility with Medium Trust level and .NET version.

| **`ErrorLog` implementation** | **Description** | **Medium Trust (2.0 only)** | **1.x** | **2.0** |
|:------------------------------|:----------------|:----------------------------|:--------|:--------|
| `MemoryErrorLog`              | Logs errors in RAM. | Yes                         | Yes     | Yes     |
| `XmlFileErrorLog`             | Logs errors in multiple XML files. | Yes                         | Yes     | Yes     |
| `SQLiteErrorLog`              | Logs errors in SQLite. | No                          | No      | Yes     |
| `SqlErrorLog`                 | Logs errors in Microsoft SQL Server. | Yes                         | Yes     | Yes     |

> Note: `XmlFileErrorLog` provider works in Medium Trust as long as it is configured to store XML error files into a folder under the root folder of the application. Medium Trust, in fact, doesn't allow access to the file system except for the application directory and its subfolders.

Choosing and configuring a storage provider is accomplished with the `errorLog` child node of the main elmah section group in the `Web.config` file. The following paragraphs instruct how to enable and configure each error provider.

### Storing errors in memory ###

This is the default option, and therefore requires no additional configuration. By default, the maximum number of errors the provider will store is set to 15. This can be changed via configuration, but it's however limited to a maximum of 500 to prevent memory issues. The snippet below shows how to set the size of the error store.

```
<elmah>  
  <errorLog type="Elmah.MemoryErrorLog, Elmah" size="50" />  
</elmah>  
```

### Storing errors in XML files ###

The `XmlFileErrorLog` enables error entries to be stored into XML files. Each error gets its own file containing all of its details.

The following snippet shows how to configure the `XmlFileErrorLog` class.

```
<elmah>  
    <errorLog type="Elmah.XmlFileErrorLog, Elmah" logPath="~/App_Data" />  
</elmah>  
```

The `logPath` attribute must point to an existing location on the file system, on which the identity the ASP.NET worker process runs with needs to have write permission.

As noted before, logging to XML files works in Medium Trust level only if the folder the `logPath` points to is a subfolder of the application.

The syntax accepted by the `logPath` attribute is either an absolute path on the file system, or the new application relative syntax `~/` introduced by ASP.NET 2.0. Relative paths are not supported in ASP.NET 1.1.

Once exceptions are thrown the target folder gets populated as shown in the following figure.

**Figure 6: Xml errors**

![http://dotnetslackers.com/images/articleimages/xml-errors.png](http://dotnetslackers.com/images/articleimages/xml-errors.png)

### Storing errors in SQLite ###

[SQLite](http://www.sqlite.org/) is an open source lightweight embedded relational database engine. A managed and standalone version written for the .NET environment is available under public domain license. ELMAH has recently adopted the [managed SQLite provider](http://sqlite.phxsoftware.com/) as an alternative storage mechanism to SQL Server, although it comes with a couple of limitations. The class which implements logging via SQLite is called `SQLiteErrorLog`.

> Note: The `SQLiteErrorLog` class requires ASP.NET 2.0 and Full Trust level.

The snippet below shows how to configure ELMAH to use the SQLite provider.

```
<connectionStrings>  
  <add name="Elmah.SQLite" connectionString="data source=~/App_Data/Elmah.SQLite.db" />  
</connectionStrings>  
  
<elmah>  
  <errorLog type="Elmah.SQLiteErrorLog, Elmah" connectionStringName="Elmah.SQLite" />  
</elmah>  
```

As shown above it leverages the new `connectionStrings` section available in ASP.NET 2.0. The `data source` attribute can point to either an absolute or application relative path, on which the account running ASP.NET must have edit (which include both write and read) permissions. A database file name needs to be specified as well, by convention with the `.db` extension, but it doesn't need to exist ahead. If it doesn't, the provider automatically creates it.

### Storing errors in SQL Server ###

The last alternative for error storage is Microsoft SQL Server via the `SqlErrorLog` class. ELMAH supports both SQL Server 2000 and 2005, though it doesn't leverage new features offered by SQL Server 2005 to maintain backwards compatibility, and both ASP.NET 1.x and 2.0.

Before enabling SQL Server error logging a database needs to be created. A [SQL DDL script](http://code.google.com/p/elmah/source/browse/trunk/src/Elmah/SQLServer.sql) is supplied for this and will take care of creating a table with the correct schema where the errors will be stored, but the database should already be there.

Due to the availability of the new `connectionStrings` section in ASP.NET 2.0 the configuration varies depending on the version of ASP.NET the application is running on.

The snippet below shows how to configure the `SqlErrorLog` in ASP.NET 1.x, along with a sample connection string which, of course, needs to be adapted to reflect custom needs. It presumes that a database called ELMAH has been created and is accessible according to the parameters specified in the connection string.

```
<elmah>  
  <errorLog type="Elmah.SqlErrorLog, Elmah"    
            connectionString="Data Source=.;Initial Catalog=ELMAH;Trusted_Connection=True" />  
</elmah>  
```

The following snippet, instead, shows how the same configuration can be achieved under ASP.NET 2.0, but note that this is just an improvement, the previous syntax will work as well.

```
<connectionStrings>  
  <add name="Elmah.Sql"  
       connectionString="Data Source=.;Initial Catalog=ELMAH;Trusted_Connection=True" />  
</connectionStrings>  
  
<elmah>  
  <errorLog type="Elmah.SqlErrorLog, Elmah" connectionStringName="Elmah.Sql" />  
</elmah>  
```

### Choosing among error log providers ###

Given the several storage mechanisms provided by ELMAH, here's a brief guide which explains how to choose between them and what scenarios they were built to tackle.

Storing errors in memory is fast and suitable as long as they don't need to survive application restarts. Since application resets are often unpredictable this log provider is usually employed for testing and troubleshooting only; in cases when everything is going wrong it's good to remove dependencies on external services and the `MemoryErrorLog` might help reducing moving parts. Moreover, as the error number grows their memory occupation might grow as well at unwanted levels. For these reasons most of the times a persistent storage is preferable.

Storing errors into XML files is a nice solution in that it's compatible with both ASP.NET 1.x and 2.0 and works under Medium Trust; furthermore, it doesn't require any database engines, like SQL Server, which usually come at additional costs in hosting plans. However, although very versatile, this solution is not as performant as relational database alternatives, and might not be suitable for larger applications. In case this turned out to be the only available choice a smart way of keeping it run smoothly is limiting the growth of the number of XML files by running a scheduled task to archive the old logs and clean up the folder.

The SQLite storage is a good choice in terms of speed and cost because it is known to be very fast and doesn't require any additional service to be provided by the web application hoster. Its main limitations are that it is supported only in ASP.NET 2.0 and needs Full Trust to work.

Finally, whenever SQL Server 2000 or 2005 are available they represent definitely the best choice for every application size and complexity. Speed, performance, data resilience and many others are intrinsic features of relational database engines. In addition, the `SqlErrorLog` provider works on both ASP.NET 1.x and 2.0 as well as in Medium Trust.

Furthermore, since being the only server-based error logging provider it features the capability of storing exceptions coming from different applications. This means that several applications can point to the same database and ELMAH will be able to store their exceptions in a way that each application will only see and be notified of its own exceptions. This is accomplished by storing the name of each application, which is unique, into a field of the table where ELMAH logs the errors and consequently filtering on the same field whenever a query is performed.

This is especially useful in case an hosting provider or an operations environment within an enterprise want to provide a centralized error logging mechanism out of the box without wasting too many server resources and without requiring each user to have its own database for that. They would expose a public connection string pointing to a database every user can connect to just for error logging.

## Configuring security ##

ELMAH provides a configuration section to enable or disable remote access to its error log handler - that is, to the list of errors as shown in Figure 5. When it's disabled only local access to the ELMAH HTTP handler is allowed.

```
<elmah>  
    <security allowRemoteAccess="0" />  
</elmah>  
```

The snippet above shows how to disable remote access. To enable it, any of the following values is accepted: `1`, `yes`, `true`, `on`.

Besides the integrated security setting offered by ELMAH, ASP.NET provides its own authorization mechanism. For example, in case ELMAH error handler is mapped to the `elmah.axd` resource and access to it wants to be denied to unauthenticated users, the following configuration excerpt can be used.

```
<location path="elmah.axd">  
    <system.web>  
        <authorization>  
            <deny users="?" />  
        </authorization>  
    </system.web>  
</location>  
```

More information about how to limit access to error reports can be found on the project Wiki's [SecuringErrorLogPages](http://code.google.com/p/elmah/wiki/SecuringErrorLogPages) page.

## Configuring error notifications ##

One of the features offered by ELMAH is the ability to send error notifications via email. This feature requires an additional entry into the `httpModules` section to be enabled and can be customized via the child `errorMail` section of the main `elmah` node.

```
<elmah>  
  <errorMail from="elmah@example.com"    
             to="admin@example.com"    
             cc="carboncopy@example.com"    
             subject="..."  
             async="true|false"  
             smtpPort="25"  
             smtpServer="smtp.example.com"    
             userName="johndoe"  
             password="secret" />  
</elmah>  
  
<system.web>  
    <httpModules>  
        <!-- ...other HTTP modules -->  
        <add name="ErrorMail" type="Elmah.ErrorMailModule, Elmah" />  
    </httpModules>  
</system.web>  
```

The `errorMail` section shown above provides several options, illustrated in the following table.

| **Option name** | **Description** | **Required** | **Default** |
|:----------------|:----------------|:-------------|:------------|
| `to`            | Recipient email address. Supports multiple addresses delimited by semi-colon ";" (in 1.x) or comma "," (in 2.0). | Yes          | n/a         |
| `from`          | Sender email address. | Yes          | n/a         |
| `cc`            | Carbon copy email address. | No           | ""          |
| `subject`       | Subject of the mail. | No           | ""          |
| `async`         | Boolean value indicating whether the email is sent synchronously or asynchronously using a thread of the ASP.NET pool. | No           | true        |
| `smtpPort`      | Port of the smtp server. | No           | 25          |
| `smtpServer`    | Name of the smtp server. | Yes          | n/a         |
| `userName`      | Username to access the smtp server. Leave blank if no authentication is required. | No           | ""          |
| `password`      | Password to access the smtp server. Leave blank if no authentication is required. | No           | ""          |

## Configuring error filtering ##

The error filtering features offered by ELMAH prevent exceptions from being logged and notified. This feature was introduced to allow filtering out exceptions coming from known error conditions which don't need to be logged, like those which might be caused by crawlers and robots in Internet-facing websites, and thus avoid polluting the logs and reducing false positives.

Filtering can be carried out both declaratively and programmatically.

### Filtering programmatically ###

Both the `ErrorLogModule` and `ErrorMailModule` used to intercept uncaught exceptions expose an event called `Filtering`. Each time an unhandled exception is intercepted this event is fired, and if any of the subscribers instructs that the exception must be dismissed then it doesn't get logged/notified. Exception dismissal can be configured independently for the `ErrorLogModule` and the `ErrorMailModule`; this means that exceptions filtered programmatically can either be both logged and notified, logged but not notified, notified but not logged or neither logged nor notified.

Since the the `Filtering` event is raised by HTTP modules, the most straightforward way to subscribe to it is by writing event handlers in the `Global.asax` file, following the naming syntax that ASP.NET itself uses: `ModuleName_EventName`. Thus, given that ELMAH modules are declared as follows:

```
<httpModules>  
    <!-- ...other HTTP  modules -->    
    <add name="ErrorLog" type="Elmah.ErrorLogModule, Elmah" />  
    <add name="ErrorMail" type="Elmah.ErrorMailModule, Elmah" />  
</httpModules>  
```

The event handlers' correct syntax is the following (excerpt of the `Global.asax` file):

```
void ErrorLog_Filtering(object sender, ExceptionFilterEventArgs e)
{
    // perform filtering here   
}

void ErrorMail_Filtering(object sender, ExceptionFilterEventArgs e)
{
    // perform filtering here   
}
```

As shown above the handlers for the filtering events receive an input parameter of type `ExceptionFilterEventArgs`. This object can be used to obtain information about the exception as well as to choose between taking or dismissing it. Figure 7 shown its class diagram.

**Figure 7: ExceptionFilterEventArgs class diagram**

![http://dotnetslackers.com/images/articleimages/exceptionfiltereventargs.png](http://dotnetslackers.com/images/articleimages/exceptionfiltereventargs.png)

The `Context` property supplies the current `HttpContext` object, which contains information about the current request.

The `Dismissed` property indicates whether the exception has been dismissed or not, while the `Exception` property contains the uncaught exception.

The `Dismiss` method marks the exception for dismissal, preventing it from being either logged or notified, depending on which module has raised the event.

One thing to be aware of is that uncaught exceptions are bubbled up by ASP.NET as `HttpUnhandledException` exceptions. To get the actual type of exception you'll need to get the base exception. The following code snippet shows how to prevent all exceptions of type `FileNotFoundException` from being logged.

```
void ErrorLog_Filtering(object sender, ExceptionFilterEventArgs e)     
{     
    if(e.Exception.GetBaseException() is FileNotFoundException)   
        e.Dismiss();   
}  
```

### Filtering declaratively ###

Declarative approach to error filtering is a little more intricate since filters need to be described via the rich syntax provided by ELMAH. The benefit of this approach is that it doesn't require any application recompilation.

To enable this feature a new HTTP module needs to be registered. This module takes care of intercepting the `Filter` event of both the `ErrorLogModule` and `ErrorMailModule`, if they are registered, and filters exceptions according to conditions described in the configuration file.

```
<httpModules>  
  <!-- ...other HTTP modules -->  
  <add name="ErrorFilter" type="Elmah.ErrorFilterModule, Elmah" />  
</httpModules>  
```

> Note: The `ErrorFilterModule` should be registered after both `ErrorLogModule` and `ErrorMailModule` to avoid initialization order to prevent the first from seeing the others.

Filtering is then accomplished by means of the `errorFilter` configuration section, child of the main `elmah` section group. The snippet below shows an example of filtering out exceptions of type `FileNotFoundException` - as done before programmatically.

```
<elmah>  
  <errorFilter>  
    <test>  
      <is-type binding="BaseException" type="System.IO.FileNotFoundException" />  
    </test>  
  </errorFilter>  
</elmah>  
```

ELMAH provides a rich and extensible syntax for declarative error filtering. For a more detailed description of the available features check out the [ErrorFiltering](http://code.google.com/p/elmah/wiki/ErrorFiltering) page on the project's Wiki.

# Signaling errors #

The ability to directly tap into the mechanism for logging and notifying exceptions offered by ELMAH has been recently one of the most requested features on the project's discussion group. People were asking to be able to manually log exceptions, meaning with this that they didn't want to throw them, but instead just notify ELMAH about an error condition, which, in turn, would then either log or notify about it.

This hasn't been possible until recently since ELMAH - as should be clear by now - was designed to intercept only unhandled exceptions. Error signaling is exposed via the `ErrorSignal` class, which provides a single overloaded method called `Raise`. Simply put, exceptions raised via the `ErrorSignal` class are not thrown, therefore they don't bubble up, but instead are only sent out to ELMAH, as well as to whomever subscribes to the `Raise` event of the `ErrorSignal` class.

The code snippet below shows how to obtain an instance of the `ErrorSignal` class, which is unique per application and can be retrieved simply with the static `FromCurrentContext` method, and then use it to signal an exception.

```
protected void ButtonSignalException_Click(object sender, EventArgs e)   
{   
    ErrorSignal.FromCurrentContext().Raise(new NotSupportedException());   
}  
```

The difference between signaling and throwing an exception is that in the first case no one will ever know about the error except ELMAH.

As discussed before, this feature would be especially handy when client code is communicating with external services which need to return a well formed message regardless of whether an exception was raised on their side. Similar scenarios are represented for example by Ajax or Web Service calls.

# Getting updates and the About page #

Together with error signaling, this is the other most recent feature. ELMAH provides the ability to check for the availability of a newer version. It is available in the About page which can be reached following the About link in ELMAH main page.

**Figure 8: The about page**

![http://dotnetslackers.com/images/articleimages/about-page.png](http://dotnetslackers.com/images/articleimages/about-page.png)

The _About_ page also contains detailed information about the configuration - either debug or release - the revisions of the source files used to build the assembly in use and finally the version of the .NET framework against which the assembly was compiled.

# The HTML error table #

The report shown by default when accessing the `elmah.axd` handler has been shown in the previous paragraphs and is represented in Figure 8 below, with a number of errors.

**Figure 9: Default error view with errors**

![http://dotnetslackers.com/images/articleimages/elmah.axd-with-errors.png](http://dotnetslackers.com/images/articleimages/elmah.axd-with-errors.png)

The default view shows some details about the exceptions, like the host machine where the application was running when the error was logged, the status code of the HTTP response returned together with the response content, the type of the exception (trimmed down to gain more screen real estate), the message of the exception, the name of the user authenticated during the request, and finally the date and the time at which the exception occurred. A flexible and smart paging mechanism is employed, which prevents all the exceptions from being retrieved each time from the persistence store. Paging is performed on the server side.

> Note: Hovering over the Code, Type and Date fields of each row shows a tooltip with a user friendlier representation of the message they contain.

# Detailed exception view #

Clicking the _Details..._ link next to each exception error message shows a detailed view of the exception, containing the stack trace, links bringing to other views of the exception, and a list of all the values kept into the server variables during the request in which the exception was thrown.

**Figure 10: Detailed view**

![http://dotnetslackers.com/images/articleimages/detail-view.png](http://dotnetslackers.com/images/articleimages/detail-view.png)

The figure above spots out two links which bring to other views of the exception. Original ASP.NET error page links to the yellow screen of death that would have been shown to the user if ASP.NET custom errors were disabled.

**Figure 11: Original ASP.NET error page**

![http://dotnetslackers.com/images/articleimages/detail-html-view.png](http://dotnetslackers.com/images/articleimages/detail-html-view.png)

Note that the _Original ASP.NET error page_ shows the full detailed view about the exception irrespective of whether custom errors are enabled or not. This means that even if the user is shown a custom error page ELMAH will log the full details about the exception.

The other available data is a XML representation of the exception. In addition to the information displayed in the detailed view it contains all the information about the request which generated the exception, and therefore it is filled with the contents of the `Forms` collection in case of an HTTP POST request, for example. The reason why these details were left out from the detailed view was to avoid polluting the display with too many data.

**Figure 12: Raw/Source data in XML**

![http://dotnetslackers.com/images/articleimages/detail-xml-view.png](http://dotnetslackers.com/images/articleimages/detail-xml-view.png)

# RSS feeds #

The links on the main error page bring to other representation of the list of exceptions. The _RSS FEED_ link provides a RSS feed of the exceptions which can be subscribed to. This is more a notification than reporting feature because everyone who subscribes the feed receives an updated list of the last 15 errors.

**Figure 13: RSS feed**

![http://dotnetslackers.com/images/articleimages/rss-feed.png](http://dotnetslackers.com/images/articleimages/rss-feed.png)

The _RSS DIGEST_ link, instead, provides a daily summary of the exceptions up to the last 15 days.

**Figure 14: RSS Digest**

![http://dotnetslackers.com/images/articleimages/rss-digest.png](http://dotnetslackers.com/images/articleimages/rss-digest.png)

# Downloadable log #

Finally, the last exception report available is a downloadable file containing the log in CSV format. It can be retrieved via the DOWNLOAD LOG link on the main error page. It contains all the details about each error shown in the main error log page together with the link to the detail page.

**Figure 15: Downloadable log in .CSV format**

![http://dotnetslackers.com/images/articleimages/csv-log.png](http://dotnetslackers.com/images/articleimages/csv-log.png)

The downloadable log can be used for several purposes. For example it can be opened in Microsoft Excel to perform some quick and simple analysis, or can be programmatically accessed and analyzed by monitoring tools which can then create reports and statistics about application health, or finally it can be queried against with Microsoft's [Log Parser](http://www.microsoft.com/downloads/details.aspx?FamilyID=890cd06b-abf8-4c25-91b2-f8d975cf8c07&displaylang=%0A%0Aen) tool.

# ELMAH and ASP.NET 2.0 Health Monitoring #

The release of .NET 2.0 has brought some new acclaimed features concerning application monitoring, provided out of the box to ASP.NET 2.0 applications. It's called Health Monitoring and consists in a considerable number of classes which together set up a complex infrastructure for logging and notifying application events.

ELMAH was designed with a slightly different purpose instead, representing a simple and quickly pluggable component to log and view exceptions.

[Phil Haack](http://haacked.com/), recently member of the Microsoft ASP.NET team, talked about the comparison between ELMAH and ASP.NET 2.0 Health Monitoring in the follow up to [one of the posts on his blog](http://haacked.com/archive/2007/07/24/securely-implement-elmah-for-plug-and-play-error-logging.aspx), which I recommend reading.

# Summary #

ELMAH provides a pluggable logging and notification system equipped with many useful additional features for monitoring ASP.NET 1.x and 2.0 applications as easily as using a standard web application. It seamlessly integrates with existing applications without much effort except for minor changes to configuration files, and works on top of many storages to provide a flexible and extensible logging mechanism.

It provides developers and website maintainers with logging and notification of errors which often get ignored because they appear with meaningless error messages to users often lacking from a technical standpoint, and usually not willing to report them.

Now, ELMAH has a user and developer discussion group, a Wiki, a Subversion repository where the source is hosted and is approaching a major release.

As any other open source projects it needs support from the community, not limited to writing code, but in finding bugs, reporting issues, creating documentation and animating the discussion group. So, why waiting?