# Introduction #

An http handler that will serve static files in a cached, compressed and resumable manner.

It generates consistent etags and the correct meta tags for caching on proxies and locally. This is especially useful when you don't have control over the configuration of the webserver.

It can also serve cached requests and compressed cached requests from memory bypassing the hard drive.

# When to use it #
When you want to serve static files. For example jpg, gif, xls, pdf, mp3, avi.

Rather use IIS Kernel Mode caching when:
  * IIS version is 7 or higher
  * You only need to varyByHeaders attribute and not by varyByQuerystring
  * You do not need to support any modules or features that must be run in user mode, e.g. authentication or authorization.
  * Have control over IIS configuration

http://learn.iis.net/page.aspx/154/walkthrough-iis-70-output-caching/

Kernel Mode caching is the quickest way to serve content.

# Features #

## Resumable downloads ##
Partial requests are processed by handler, and result in a **206 partial** response.

This is very useful when sending large files or streaming media files, as only the requested bytes are served.

Adobe Reader in "Allow fast web view" mode starts downloading a pdf and soon as it has enough bytes to display the first page it aborts the connection to the web server, and displays the first page. Then it makes a "multipart/byteranges" request to retrieve the rest of the document. This is what causes most download scripts to fail when serving pdf content.

It supports the following partial http header tags:
  * Accept-Ranges
  * Range
  * If-Range

## Server side caching ##
The handler stores original and compressed content in memory, bypassing hard drive. This can really speed up web requests. Partial content can be served from memory too, unlike IIS kernel mode caching.

## Client side caching ##
**ETag** and **Expires** http headers are sent by handler. Matched cached requests result in a **304 Not Modified** response.

It supports the following caching http header tags:
  * ETag
  * Expires
  * Last-Modified
  * If-Match
  * If-None-Match
  * If-Modified-Since
  * If-Unmodified-Since
  * Unless-Modified-Since

## Compression ##
Gzip and deflate compression is supported. Deflate is probably the optimal compression type to use:

http://www.stardeveloper.com/articles/display.html?article=2008111201&page=1

http://madskristensen.net/post/Compression-and-performance-GZip-vs-Deflate.aspx

These file types will benefit from further compression:
  * css, js, htm, html, swf, xml, xslt, txt
  * doc, xls, ppt, docx, xlsx, pptx

There are some file types that will not benefit from further compression:
  * pdf (causes problems with certain versions of Adobe Reader in IE)
  * png, jpg, jpeg, gif, ico
  * wav, mp3, m4a, aac  (wav is often compressed)
  * 3gp, 3g2, asf, avi, dv, flv, mov, mp4, mpg, mpeg, wmv
  * zip, rar, 7z, arj

Compression of **multipart/byteranges** is supported too.

# Configuration #

**WebServerType**
  * **NotSet** - Auto detect web server type (**default**)
  * **Unknown** - Unable to detect web server type
  * **IIS7** - IIS 7
  * **IIS6orIIS7ClassicMode** - IIS 6 or IIS 7 in classic mode
  * **Cassini** - Cassini
_If you need to run this this handler in medium trust you will need to set this value, as it uses reflection to work out the type of web server used._

**EtagMethodType**
  * **MD5** - The etag must be a MD5 hash of the file's contents
  * **LastModified** - The etag must be the date time the file was last accessed

**FileExtensionDefaultElement**
  * _string_ **Name** - the unique name of the configuration element
  * _bool_ **ServeFromMemory** - should content be served from memory
  * _long_ **MaxMemorySize** - the maximum size a file can be before it is no longer served from memory
  * _bool_ **Compress** - is the content compressible
  * _TimeSpan_ **MemorySlidingExpiration** - The amount of time a file should be cached in memory
  * _TimeSpan_ **Expires** - the amount of time a browser request is valid for. This should be set to a value like 1 week.
  * _EtagMethodType_ **EtagMethod** - the type of method used to calculate the etag.

**FileExtensionElement**
  * _string_ **Name** - the unique name of the configuration element
  * _string_ **Extension** - a comma separated list of extensions this rule must apply too
  * _bool_ **ServeFromMemory** - should content be served from memory
  * _long_ **MaxMemorySize** - the maximum size a file can be before it is no longer served from memory
  * _bool_ **Compress** - is the content compressible
  * _TimeSpan_ **MemorySlidingExpiration** - The amount of time a file should be cached in memory
  * _TimeSpan_ **Expires** - the amount of time a browser request is valid for. This should be set to a value like 1 week.
  * _EtagMethodType_ **EtagMethod** - the type of method used to calculate the etag.


Example Configuration:
```
<configuration>
	<configSections>
		<section name="StaticFileHandler" type="Talifun.Web.StaticFile.Config.StaticFileHandlerSection, Talifun.Web" requirePermission="false" allowDefinition="MachineToApplication"/>
	</configSections>
	<StaticFileHandler webServerType="NotSet">
		<!-- The defaults to use when an extension is found that does not have a specific rule  -->
		<fileExtensionDefault name="Default" serveFromMemory="true" maxMemorySize="100000" compress="true"/>
		<!-- Specific rules for extension types -->
		<fileExtensions>
			<fileExtension name="CommonStaticContent" extension="css, js, htm, html, swf, xml, xslt, txt" serveFromMemory="true" maxMemorySize="5000000" compress="true"/>
			<fileExtension name="OfficeDocumentStaticContent" extension="doc, xls, ppt, docx, xlsx, pptx" serveFromMemory="true" maxMemorySize="100000" compress="true"/>
			<!-- Dont compress pdfs as they are already compressed and sending compressed pdf is know to cause problems -->
			<fileExtension name="PdfDocumentStaticContent" extension="pdf" serveFromMemory="true" maxMemorySize="100000" compress="false"/>			
			<!-- Dont compress images as they are already compressed -->
			<fileExtension name="ImagesStaticContent" extension="png, jpg, jpeg, gif, ico" serveFromMemory="true" maxMemorySize="500000" compress="false"/>
			<!-- Dont compress audio as they are already compressed -->
			<fileExtension name="AudioStaticContent" extension="wav, mp3, m4a, aac" serveFromMemory="true" maxMemorySize="100000" compress="false"/>
			<!-- Dont compress videos as they are already compressed -->
			<fileExtension name="VideoStaticContent" extension="3gp, 3g2, asf, avi, dv, flv, mov, mp4, mpg, mpeg, wmv" serveFromMemory="true" maxMemorySize="100000" compress="false"/>
			<!-- Dont compress compressed content -->
			<fileExtension name="CompressedStaticContent" extension="zip, rar, 7z, arj" serveFromMemory="true" maxMemorySize="100000" compress="false"/>
		</fileExtensions>
	</StaticFileHandler>
	<system.web>
		<httpHandlers>
			<!-- All the extensions that should be handled by static file handler -->
			<add verb="GET,HEAD" path="*.css,*.js,*.htm,*.html,*.xml,*.txt,*.xslt,*.swf,*.jpg,*.jpeg,*.gif,*.png,*.bmp,*.ico,*.wav,*.mp3,*.m4a,*.aac,*.3gp,*.3g2,*.asf,*.avi,*.dv,*.flv,*.mov,*.mp4,*.mpg,*.mpeg,*.wmv,*.pdf,*.xls,*.doc,*.ppt,*.xlsx,*.docx,*.pptx,*.swf,*.zip,*.rar" type="Talifun.Web.StaticFile.StaticFileHandler, Talifun.Web"/>
		</httpHandlers>
	</system.web>
<configuration>
```