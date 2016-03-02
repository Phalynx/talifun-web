# Introduction #

A module that compresses component images into a single sprite image file and generates the css required to cut the sprite image into its component images.

# When to use it #

When you want to combine images to reduce load times. Images should ideally be very similar in size. So combining all menu navigation icons is a ideally suited for this. Reducing the number of http request speeds up page load times.

# Features #

It adds a 2 pixel gap between component images as some browser don't handle css properly.

It watches for changes to any of the component images and regenerates the sprite image file and the css sprite file.

# Configuration #

**CssSpriteGroupElement**
  * _string_ **Name** - the unique name of the configuration element
  * _string_ **CssOutputFilePath** - the file path for the output css file for the sprite
  * _string_ **ImageOutputFilePath** - the file path for the output image file for the sprite
  * _string_ **CssUrl** - the url for output css file for the sprite
  * _string_ **ImageUrl** - the url for output image file for the sprite

**ImageFileElement**
  * _string_ **Name** - the unique name of the configuration element
  * _string_ **FilePath** - the file path for the image file


Example Configuration:<configuration>
	<configSections>
		<section name="CssSprite" type="Talifun.Web.CssSprite.Config.CssSpriteSection, Talifun.Web" requirePermission="false" allowDefinition="MachineToApplication"/>
	</configSections>
	<CssSprite>
		<cssSpriteGroups>
			<cssSpriteGroup name="GoodCssSprite" imageOutputFilePath="~/Static/Image/good-css-sprite.png" cssOutputFilePath="~/Static/Css/good-css-sprite.css">
				<files>
					<file name="GoodImage1" filePath="~/Static/Image/good/01.png"/>
					<file name="GoodImage2" filePath="~/Static/Image/good/02.png"/>
					<file name="GoodImage3" filePath="~/Static/Image/good/03.png"/>
					<file name="GoodImage4" filePath="~/Static/Image/good/04.png"/>
					<file name="GoodImage5" filePath="~/Static/Image/good/05.png"/>
					<file name="GoodImage6" filePath="~/Static/Image/good/06.png"/>
					<file name="GoodImage7" filePath="~/Static/Image/good/07.png"/>
					<file name="GoodImage8" filePath="~/Static/Image/good/08.png"/>
					<file name="GoodImage9" filePath="~/Static/Image/good/09.png"/>
					<file name="GoodImage10" filePath="~/Static/Image/good/10.png"/>
				</files>
			</cssSpriteGroup>
			<cssSpriteGroup name="BadCssSprite" imageOutputFilePath="~/Static/Image/bad-css-sprite.png" cssOutputFilePath="~/Static/Css/bad-css-sprite.css">
				<files>
					<file name="BadImage1" filePath="~/Static/Image/bad/background-nav-ages-11-14.gif"/>
					<file name="BadImage2" filePath="~/Static/Image/bad/logo.gif"/>
					<file name="BadImage3" filePath="~/Static/Image/bad/Main.jpg"/>
				</files>
			</cssSpriteGroup>
		</cssSpriteGroups>
	</CssSprite>
	<system.web>
		<httpModules>
			<add name="CssSpriteModule" type="Talifun.Web.CssSprite.CssSpriteModule, Talifun.Web"/>
		</httpModules>
	</system.web>
</configuration>```