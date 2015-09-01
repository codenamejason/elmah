# Microsoft ASP.NET AJAX and ELMAH #

This article describes the `MsAjaxDeltaErrorLogModule` that was introduced with [ELMAH 1.0 BETA 3](http://code.google.com/p/elmah/downloads/detail?name=ELMAH-1.0-BETA3-bin.zip).

## The Problem ##

The first incarnation of [ASP.NET AJAX](http://www.asp.net/ajax/) written against v2.0 of the .NET Framework (v1.0.61025.0) is unfortunately not 100% compatible with ELMAH. When an unhandled exception occurs during a [Partial PostBack](http://www.asp.net/AJAX/Documentation/Live/overview/PartialPageRenderingOverview.aspx), the original code catches the exception, and unceremoniously calls `Response.End()`, preventing ELMAH from capturing it. This code can be seen from [Reflector](http://www.aisto.com/roeder/dotnet/) by looking in `System.Web.Extensions.dll` at the `System.Web.UI.PageRequestManager.OnPageError` method:

```
private void OnPageError(object sender, EventArgs e)
{
    ...
    if (flag)
    {
        ...
        this._owner.IPage.Response.End();
    }
}
```

## The Solution ##

To overcome this you can add the `MsAjaxDeltaErrorLogModule` to your application as follows:

```
<httpModules>
    <!-- 
        This entry is only to be used if you are using ASP.NET AJAX v1.0.x.x
        and want to capture errors during Partial Postbacks. It is not required 
        for the version of ASP.NET AJAX that shipped with .NET Framework v3.5!
    -->
    <add name="MsAjaxDeltaErrorLog" type="Elmah.MsAjaxDeltaErrorLogModule, Elmah"/>
</httpModules>
```

This module checks to see if the current request is a [Partial PostBack](http://www.asp.net/AJAX/Documentation/Live/overview/PartialPageRenderingOverview.aspx), and if so sets up an alternative handler to capture any unhandled exception.

## Potential Issues ##

Unfortunately, the `MsAjaxDeltaErrorLogModule` is not a perfect solution. Due to the way it works, there is an extremely small possibility that errors will get logged _twice_ within ELMAH. For this to occur, the following must happen:

  1. The unhandled exception must occur during a [Partial PostBack](http://www.asp.net/AJAX/Documentation/Live/overview/PartialPageRenderingOverview.aspx)
  1. It must occur _before_ the `OnInit` method of `System.Web.UI.PageRequestManager` is called.

This will mean that both the standard ELMAH modules and the `MsAjaxDeltaErrorModule` will capture the exception.

In reality, the chances of this happening are going to be fairly slender.

## .NET Framework 3.5 ##

An updated version of Microsoft Ajax was shipped with version 3.5 of the .NET Framework. This new version changed the way that unexpected errors in [Partial PostBacks](http://www.asp.net/AJAX/Documentation/Live/overview/PartialPageRenderingOverview.aspx) were handled, removing the call to `Response.End`. Therefore, if you continue to use the `MsAjaxDeltaErrorLogModule` with this version of the framework, you will see all unhandled errors in [Partial PostBacks](http://www.asp.net/AJAX/Documentation/Live/overview/PartialPageRenderingOverview.aspx) being logged _twice_ in ELMAH.