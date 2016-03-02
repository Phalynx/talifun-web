# Introduction #

A module that compresses js into a single file, and css into a single file.

It also watches for changes to any of the watched css or js files and regenerates the crushed file. It generates a unique hash for the crushed file and appends it to the css url and the js url. So you are always sure to be served the correct content, regardless of caching.

# When to use it #

When you want to minimize the number of http requests and the size of js and css files.

Serving requests with an http handler that dynamically creates a crushed js or css file per a page is not a good idea. It is better to crush your entire sites js and css files into one file and rather serve that. This way the browser loads it once and its cached. All further requests just load from cache. Dynamically crushed js and css files create a file per page. So you might be re-serving the same css and js files for every page.

You can then let the web server serve the crushed css/js file. IIS implements kernel mode caching, which is the **fastest way** to serve any static file.

For maximum performance it is recommend to **not** enable the http module and rather precompile the css and js files required by the site by using the command line tool Talifun.Crusher.exe. It can easily be run as part of your build process.

This would also negate the need for the http module to have access rights to create and modify the crushed files, which could be an issue in a secure environment. It also enables easy deployment to external CDNs. Another nice side effect is that you **do not need medium trust** so it will work nicely on all shared hosting environments.

# Features #
  * Can crush and compress multiple css files into one file
  * Can crush and compress multiple js files into one file
  * CDN friendly
  * Always serves latest version of crushed file by appending hash of crushed file to url
  * Detects any changes to component js or css files and regenerates crushed file on the fly
  * Supports multiple grouping of js or css
  * Debug mode for when you need to work with the css or js

# File permissions #

In order to create the crushed files the application pool user for the website that is using the module will needs "modify" rights. Generally the user used for application pool is "NetworkService".

See http://msdn.microsoft.com/en-us/magazine/cc982153.aspx for details on how to change file permissions.

# CDN or cookieless domain usage #

To make use of CDN you will need to copy up the crushed files to the CDN.

Then for cookieless domain or CDN specify usage, specify the full url to use in the **url** configuration setting. The etag of the local version of the file will be appending to the full url. This will insure that clients will always received the latest version of content even if it is loading from external servers.

e.g.

<cssGroup name="SiteCss" debug="false" appendHash="true" outputFilePath="~/Static/Css/site.css" **url="http://www.test.com/Css/site.css"**>

<jsGroup name="SiteJs" debug="false" outputFilePath="~/Static/Js/site.js" **url="http://www.test.com/Js/site.js"**>

Remote urls are specifically not allowed as a filePath as there is no way of detecting changes to them. So it is better to copy them locally, and then copy crushed files up as we can be certain of file changes.

# Configuration #
**Crusher**
  * _string_ **QuerystringKeyName** - The name of the query string key that is appended for the hash of the file

**JsCompressionType**
  * **None** - Do not use any compression at all on js file
  * **Min** - Use jsmin to compress the js file

**JsGroupElement**
  * _string_ **Name** - the unique name of the configuration element
  * _string_ **OutputFilePath** - the file path for the output js file
  * _string_ **Url** - the url for output js file. If Debug=true it will use local resource instead of specified url.
  * _bool_ **Debug** - should the uncrushed versions of the js files be used

**CssCompressionType**
  * **None** - Do not use any compression at all on css file
  * **StockYuiCompressor** - http://yuicompressor.codeplex.com/
  * **MichaelAshRegexEnhancements** - http://yuicompressor.codeplex.com/
  * **Hybrid** - http://yuicompressor.codeplex.com/

**CssGroupElement**
  * _string_ **Name** - the unique name of the configuration element
  * _string_ **OutputFilePath** - the file path for the output css file
  * _string_ **Url** - the url for output css file. If Debug=true it will use local resource instead of specified url
  * _string_ **Media** - the url for output css file
  * _bool_ **Debug** - should the uncrushed versions of the css files be used
  * _bool_ **AppendHash** - Should a hash be appended to url of css assets

**CssFileElement**
  * _string_ **Name** - the unique name of the configuration element
  * _string_ **FilePath** - the file path for the css file
  * _CssCompressionType_ **CompressionType** - the compression type to use for the css file


Example Configuration:
```
<configuration>
	<configSections>
		<section name="Crusher" type="Talifun.Web.Crusher.Config.CrusherSection, Talifun.Web" requirePermission="false" allowDefinition="MachineToApplication"/>
	</configSections>
	<Crusher querystringKeyName="etag">
		<!-- outputFilePath is the identifier for the cssGroup, so make sure its unique for each cssGroup  -->
		<cssGroups>
			<!-- Css group to demo the crushing -->
			<cssGroup name="SiteCss" debug="false" appendHash="true" outputFilePath="~/Static/Css/site.css">
				<files>
					<file name="JQueryUI" filePath="~/Static/Css/jquery-ui-1.7.1.css" compressionType="Hybrid" />
					<file name="Default" filePath="~/Static/Css/default.css" compressionType="Hybrid" />
					<file name="Test1" filePath="~/Static/Css/test1.css" compressionType="Hybrid" />
					<file name="Test2" filePath="~/Static/Css/test2.css" compressionType="Hybrid" />
					<file name="Test3" filePath="~/Static/Css/test3.css" compressionType="Hybrid" />
					<file name="Test4" filePath="~/Static/Css/test4.css" compressionType="Hybrid" />
				</files>
			</cssGroup>
		</cssGroups>
		<!-- outputFilePath is the identifier for the jsGroup, so make sure its unique for each jsGroup  -->
		<jsGroups>
			<!-- Js group to demo the crushing  -->
			<jsGroup name="SiteJs" debug="false" outputFilePath="~/Static/Js/site.js">
				<files>
					<file name="JQuery" filePath="~/Static/Js/jquery-1.3.2.min.js" compressionType="Min"/>
					<file name="JQueryUI" filePath="~/Static/Js/jquery-ui-1.7.1.min.js" compressionType="Min"/>
					<file name="JqueryFlash" filePath="~/Static/Js/jquery.flash.min.js" compressionType="Min"/>
					<file name="JqueryValidate" filePath="~/Static/Js/jquery.validate.min.js" compressionType="Min"/>
					<file name="JqueryValidateExtra" filePath="~/Static/Js/additional-validation-methods.min.js" compressionType="Min"/>
				</files>
			</jsGroup>
		</jsGroups>
	</Crusher>	
	<system.web>
		<pages>
			<controls>
				<add tagPrefix="talifun" namespace="Talifun.Web.Crusher" assembly="Talifun.Web"/>
			</controls>
		</pages>	
		<httpModules>
			<!-- Only recommended when doing local development / rather use Talifun.Crusher.exe as part of your build script -->
			<!--
			<add name="CrusherModule" type="Talifun.Web.Crusher.CrusherModule, Talifun.Web"/>
			-->
		</httpModules>
	</system.web>		
</configuration>
```

Example usage in page:
```
<head>
    <talifun:CssControl ID="SiteCssControl" runat="server" GroupName="SiteCss" />
    <talifun:JsControl ID="SiteJsControl" runat="server" GroupName="SiteJs" />
</head>
```