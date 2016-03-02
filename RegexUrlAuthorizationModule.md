# Introduction #

A web module to provides authorization based on urls matching regular expressions.

# When to use it #

When you need to authorize requests matching a regular expression. For instance you might want only registered users downloading any pdf files. Another idea might be to only allow administrators to view jpg files.

# Features #

It uses standard .net web configuration for rules to specify authorization. So this means you can apply authorization per **users** or **roles**, and can be based on **allow** or **deny** rules.

The only think that has changed is the parent configuration node **Location**. It has been replaced with **urlMatch**, which matches a request based on a regular expression.

# Configuration #

**UrlMatchElement**
  * _string_ **Name** - the unique name of the configuration element
  * _string_ **Expression** - the regular expression, the incoming request must match, for authorization to be applied against it.
  * _AuthorizationRuleCollection_ **Rules** - the rules to apply. See http://msdn.microsoft.com/en-us/library/8d82143t.aspx for more information.

Example Configuration:
```
<configuration>
	<configSections>
		<section name="authorizationMatches" type="Talifun.Web.RegexUrlAuthorization.Config.RegexUrlAuthorizationSection, Talifun.Web" requirePermission="false" allowDefinition="MachineToApplication"/>
	</configSections>
	<authorizationMatches>
		<urlMatches>
			<!--You will need to use wildcard mappings or specifically map each type to get asp.net 
			to handle the serving of the file type. It can apply authorization when it is handling
			the serving of the file	-->
			<!--This will match file extensions. Don't forget that their might be a querystring or a bookmark on the end-->
			<urlMatch name="Pdf" expression=".+\.(pdf)([\?|\#].*)?$">
				<allow users="pdf"/>
				<deny users="?"/>
			</urlMatch>
			<urlMatch name="Txt" expression=".+\.(txt)([\?|\#].*)?$">
				<allow users="txt"/>
				<deny users="?"/>
			</urlMatch>
		</urlMatches>
	</authorizationMatches>
	<system.web>
		<httpModules>
			<add name="RegexUrlAuthorizationModule" type="Talifun.Web.RegexUrlAuthorization.RegexUrlAuthorizationModule, Talifun.Web"/>
		</httpModules>
	</system.web>
</configuration>
```