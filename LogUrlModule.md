# Introduction #

A module that makes it easy to hook into web requests that match a regular expression.

# When to use it #

When you want to do something when a certain url is requested. For instance it makes it easy to log all requests for pdfs using log4net.

# Features #

Exposes a matching url with an event.

# Configuration #

**UrlMatchElement**
  * _string_ **Name** - the unique name of the configuration element
  * _string_ **Expression** - the regular expression, the incoming request must match, for authorization to be applied against it.

Example Configuration:
```
<configuration>
	<configSections>
		<section name="logMatches" type="Talifun.Web.LogUrl.Config.LogUrlSection, Talifun.Web" requirePermission="false" allowDefinition="MachineToApplication"/>
	</configSections>
	<logMatches>
		<urlMatches>
			<!--This will match file extensions. Don't forget that their might be a querystring or a bookmark on the end-->
			<urlMatch name="Pdf" expression=".+\.(pdf)([\?|\#].*)?$" />
			<urlMatch name="Office" expression=".+\.(doc|xls|ppt)([\?|\#].*)?$" />
		</urlMatches>
	</logMatches>
	<system.web>
		<httpModules>
			<add name="LogModule" type="Talifun.Web.LogUrl.LogUrlModule, Talifun.Web"/>
		</httpModules>
	</system.web>
</configuration>
```

Example Implementation:
Global.asax.cs
```
public class Global : System.Web.HttpApplication
    {
        protected void Application_Start(object sender, EventArgs e)
        {
            LogUrlManager.Instance.LogUrlEvent += OnLogUrlEvent;
        }

        protected void Application_End(object sender, EventArgs e)
        {
            LogUrlManager.Instance.LogUrlEvent -= OnLogUrlEvent;
        }

        protected static void OnLogUrlEvent(object sender, LogUrlEventArgs args)
        {
            //Usually log request to database. Information to include is User, Url, Date/Time etc
            var request = args.HttpApplication.Request;

            var referrer = string.Empty;
            if (request.UrlReferrer != null)
            {
                referrer = request.UrlReferrer.PathAndQuery;
            }

            var url = request.Path;
            var querystring = request.QueryString;

            var userAgent = request.UserAgent;
        }
    }
```