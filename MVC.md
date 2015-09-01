_Note: Since ASP.NET MVC 1.0 ignores .axd rules by default this Guide has become mostly irrelevant. For detailed installation instructions look at the [DotNetSlackers Article](http://code.google.com/p/elmah/wiki/DotNetSlackersArticle)._

Setting up ELMAH on MVC is really simple since most of the work is done in web.config that is more or less shared between MVC and Webforms. There are already [excellent](http://code.google.com/p/elmah/wiki/DotNetSlackersArticle) (and more detailed) articles in this Wiki on how to configure and fine tune ELMAH.
This Article is intended to provide a quick and simple tutorial on the most important steps to get ELMAH up and running on MVC.

(For advanced users seeking MVC-specific information, skip ahead to Step 4)

# Step 1: Referencing the assemblies #

First, grab the [latest binary release](http://code.google.com/p/elmah/downloads/list) of elmah from the project’s page and extract the `bin` folder.

ELMAH requires no GAC installation, so you can drop the contents of the `bin` to any location of your liking (although I usually recommend using a lib folder to store external dependencies) and reference the `Elmah.dll` from within your app.

# Step 2: Edit your web.config to call ELMAH #

First add the following code to your `<configSections>` to make ELMAH read it’s configuration from web.config:

```
<sectionGroup name="elmah">
  <section name="security" requirePermission="false" type="Elmah.SecuritySectionHandler, Elmah" />
  <section name="errorLog" requirePermission="false" type="Elmah.ErrorLogSectionHandler, Elmah" />
  <section name="errorMail" requirePermission="false" type="Elmah.ErrorMailSectionHandler, Elmah" />
  <section name="errorFilter" requirePermission="false" type="Elmah.ErrorFilterSectionHandler, Elmah" />
</sectionGroup>
```

Next go to your `<httpHandlers>` section and add the elmah file handler:

```
<add verb="POST,GET,HEAD" path="elmah.axd" type="Elmah.ErrorLogPageFactory, Elmah" />
```

This will reroute all requests to a file called elmah.axd to the ELMAH error-overview page. So when you want to look at the list of errors you’ll access `http://server/elmah.axd`. The name doesn’t matter, feel free to rename it, but be aware that the extension has to be mapped to the ASP.NET pipeline inside IIS (so naming it .html wouldn’t work if not configured correctly).

At last add the ELMAH logging module to your `<httpModules>` section:

```
<add name="ErrorLog" type="Elmah.ErrorLogModule, Elmah"/>
```

# Step 3: Configure ELMAH #

I’d suggest you read the wiki articles on how to configure ELMAH correctly, but for getting it up and running quickly we'll just log all errors to XML files by simply adding this code to the web.config:

```
<elmah>
  <errorLog type="Elmah.XmlFileErrorLog, Elmah" logPath="~/App_Data" />
</elmah>
```

This instructs ELMAH to create XML files in your `App_Data` directory (so make sure the ASP.NET process has sufficient access rights to that folder) and generate output like this:

![http://www.tigraine.at/wp-content/uploads/2009/04/image1.png](http://www.tigraine.at/wp-content/uploads/2009/04/image1.png)

# Step 4: Configure routing #

Up until now we were 100% true to the normal configuration routine for normal ASP.NET applications, there is only one slight adjustment to making it work in MVC.

You need to allow the requests to the ELMAH front-end (`elmah.axd` in this example) to pass through the MVC routing logic unchanged so that it gets handled by normal ASP.NET behind MVC. This is as trivial as adding an ignore route to your routing table in `Gobal.asax.cs`:

```
public static void RegisterRoutes(RouteCollection routes)
{
    routes.IgnoreRoute("elmah.axd");
```

Once all the above is done, you’ll see all unhandled exceptions that result in a yellowscreen-of-death be also logged into the XML files inside your `App_Data` and you can then watch them remotely by accessing `http://server/elmah.axd`. You’ll get a rather nice overview page like this one:

![http://www.tigraine.at/wp-content/uploads/2009/04/image2.png](http://www.tigraine.at/wp-content/uploads/2009/04/image2.png)

Congratulations, you have set up ELMAH on ASP.NET MVC and configured it to log all errors to XML Files.
I strongly suggest you now read [how to secure the Errorlog](http://code.google.com/p/elmah/wiki/SecuringErrorLogPages) to prevent strangers from reading your logs.

Also, since 404 Errors in ASP.NET MVC are thrown as Exceptions, ELMAH will log them too, so you may want to read on how to [filter those out](http://code.google.com/p/elmah/wiki/ErrorFiltering)