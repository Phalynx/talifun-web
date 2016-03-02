The project consists of a suite of http modules and http handlers that have documented source code and sample implementations. Performance is maximized by making extensive use of caching.

All modules and handlers are configurable via the web.config. So it is just as easy to retrofit into existing websites, as it is to implement into new ones.

# StaticFileHandler #
An http handler that will serve static files in a cached, compressed and resumable manner.

It generates consistent etags and the correct meta tags for caching on proxies and locally. This is especially useful when you don't have control over the configuration of the webserver.

It can also serve cached requests and compressed cached requests from memory bypassing the hard drive.

It supports the following http header tags:

  * Accept-Ranges
  * ETag
  * Expires
  * Last-Modified
  * Range
  * If-Range
  * If-Match
  * If-None-Match
  * If-Modified-Since
  * If-Unmodified-Since
  * Unless-Modified-Since

# CrusherModule #
A module that compresses js into a single file, and css into a single file.

It also watches for changes to any of the watched css or js files and regenerates the crushed file. It generates a unique hash for the crushed file and appends it to the css url and the js url. So you are always sure to be served the correct content, regardless of caching.

# CssSpriteModule #
A module that combines component images into a single sprite image file and generates the css sprite file required to cut the sprite image into its component images.

# RegexUrlAuthorizationModule #
A module that provides authorization based on urls matching regular expressions.

# LogUrlModule #
A very simple module that makes it easy to hook into web requests that match a regular expression.

# PageCompressionModule #
A module to compresses dynamic pages for webforms and mvc.

# WebResourceCompressionModule #
A module to compresses web resources for webforms and mvc.