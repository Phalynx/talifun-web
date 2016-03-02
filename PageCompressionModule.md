# Introduction #

A module that will compress dynamic content created by webforms or mvc.

# When to use it #

# Features #

# Configuration #

Example Configuration:
```
<configuration>
	<system.web>
		<httpModules>
			<add name="PageCompressionModule" type="Talifun.Web.Compress.PageCompressionModule, Talifun.Web"/>
		</httpModules>
	</system.web>
</configuration>
```