Cache-MDX2JSON
==============

RESTful web api for MDX2JSON transformation (also JSONP and XML/A). Also supports requests about Dashboards and Widgets. Supports Caché 2014.1+.

Installation
-----------

1. Download Installer.cls.xml (from MDX2JSON folder in repository or releases page) into Caché manager directory.
2. Run in terminal (any namespace) under user with %All role: 

        do ##class(%Installer.Installer).InstallFromCommandLine(##class(%File).ManagerDirectory()_"Installer.cls.xml")

 On this step installer would create (if needed) MDX2JSON namespace and corresponding database, download source code from GitHub and compile it, create required web application (named /MDX2JSON) if one does not exist (skipping web application creation process if one does exist), and map MDX2JSON package to %All namespace (which will be created if it does not exist).
3. Give the correct roles to /MDX2JSON webapplication for it to be able to query desired cubes:
  - SELECT on %DeepSee_Dashboard.Definition table (all namespaces with dashboards)
  - EXECUTE on all procedures in MDX2JSON package (all namespaces with dashboards)
  - Resource %Admin_Secure:U
  - Resource %DB_CACHESYS:RW
  - Resource Database:R - Usually %DB_Name (all namespaces with dashboards)

Installation without fs access to server
-----------

1. Download Installer.cls.xml (from MDX2JSON folder in repository or releases page) into Caché Studio (any namespace)
2. Run in terminal (any namespace) under user with %All role: 

        do ##class(MDX2JSON.Installer).setup()
        
 On this step installer would create (if needed) MDX2JSON namespace and corresponding database, download source code from GitHub and compile it, create required web application (named /MDX2JSON) if one does not exist (skipping web application creation process if one does exist), and map MDX2JSON package to %All namespace (which will be created if it does not exist).
3. Give the correct roles to /MDX2JSON webapplication for it to be able to query desired cubes.

Offline Installation
-----------

1. Download zip and unpack it.
2. Run in terminal (any namespace) under user with %All role: 

        do ##class(%Installer.Installer).InstallFromCommandLine("{SourceDir}\Installer.cls.xml","SourceDir={SourceDir}")

  where: 
    
      {SourceDir} is a directory where you unpacked zip \ MDX2JSON (see 1).

 On this step installer would create (if needed) MDX2JSON Namespace and corresponding database, import source code and compile it, create required web application (named /MDX2JSON) if one does not exist (skipping web application creation process if one does exist), and map MDX2JSON package to %All namespace (which will be created if it does not exist).
3. Give the correct roles to /MDX2JSON webapplication for it to be able to query desired cubes.


For information on how to work with this RESTful web API please refer to included documentation.

Additional installation parameters
-----------

As a second parameter to `do ##class(%Installer.Installer).InstallFromCommandLine()` you can supply a comma-separated list of additional variables. Eg: User=MDX2JSON,Password=123456

- `Namespace` is a namespace you want to install MDX2JSON to (Not namespace with dashes). If it does not exist it would be created automatically. If it does exist only MDX2JSON package would be overwritten. WebApplication would be named `/Namespace`. Strongly not recommended to use.
- `User` is a Caché user to create or modify. He will be given SELECT access to %DeepSee_Dashboard.Definition table in `Namespace`
- `Password` must be supplied alongside User parameters
- `NSList` is a comma-separated list, you want to give access to MDX2JSON. Default - "*" and evaluates to all namespaces
- `SourceDir` - all xmls from this directory would be imported and compiled

Role creation
-----------
You can also automatically create MDx2JSON role with all the required procedures. To do that run in terminal (any namespace) under user with %All role: 

    set pVars("Namespace")="comma-separated list of namespaces"
    do ##class(MDX2JSON.Installer).createrole(.pVars)
 
Update
-----------

1.  Run in terminal (namespace where you installed MDX2JSON): 

        do ##class(MDX2JSON.Installer).Update()
	  
You can also supply parameters such as fork, desired branch/commit, target namespace, authorization information. Please refer to Caché documentation of MDX2JSON.Installer class for correct syntax. 

Requests
-----------

These are the possible requests to web application (add param ?Namespace={Desired Namespace} to query another namespace cubes

| URL                         | Type | Body (JSON)                 | Response  | Description                    |
|-----------------------------|------|-----------------------------|-----------|--------------------------------|
| MDX                         | POST | { "MDX":"QUERY" }           | JSON      | Results of MDX execution       |
| MDX2JSONP                   | POST | { "MDX":"QUERY" }           | JSONP     | Results of MDX execution       |
| MDXDrillthrough             | POST | { "MDX":"QUERY" }           | JSON      | Results of MDX execution       |
| MDX2XMLA                    | POST | { "MDX":"QUERY" }           | XMLA      | Results of MDX execution       |
| Dashboards                  | GET  |                             | JSON      | All dashboards                 |
| Dashboards                  | POST | {Folder:"FolderName"}       | JSON      | All dashboards in FolderName. Empty FolderName for root scope.|
| Dashboard                   | POST | {Dashboard:"DashboardName"} | JSON      | All widgets in a dashboard, with filters as part of dashboard|
| Widgets                     | POST | {Dashboard:"DashboardName"} | JSON      | All widgets in a dashboard, with filters as part of widgets  |
| DataSource                  | POST |{DataSource:"Pivot fullname"}| JSON      | All info about Pivot           |
| Action/:Cube/:Action        | POST |{[context object](http://docs.intersystems.com/ens20152/csp/docbook/DocBook.UI.Page.cls?KEY=D2IMP_ch_action#D2IMP_action_context_info)}             | JSON      | Execute cube action            |
| Filters                     | POST |{ "DataSource": "DataSourceName.ext", "Values":1, Search:"SearchTerm"}| JSON   | All filters for DeepSee DataSource (cube, pivot, kpi, metric) with values (if Values = 1, set to 0 or omit otherwise). If Search is not empty only filter values, containing search term would be returned.|
| Format                      | GET  |                             | JSON      | Default formatting             |
| TermList                    | POST | {"TermList":"TermListName"} | JSON      | Termlist key-value array       |
| Config                      | POST |{"Application":"AppName", "Config","value"}| JSON      | Set config for arbitrary application for current user|
| Config/:Application         | GET  | {"Application":"AppName"}   | JSON      | Get config for Application for current user|
| Favorites                   | GET  |                             | JSON      | Array of current user favorites|
| Favorites/:Item             | POST |                             | JSON      | Add favorite item              |
| Favorites/:Item             |DELETE|                             | JSON      | Remove item from favorites     |
| Test                        | GET  |                             | JSON      | Test info                      |
| Logout                      | GET  |                             | JSON      | Close session                  |

Example
-----------

Request URL: http://localhost:57772/MDX2JSON/MDX?Namespace=Samples

Request type: POST

Request body:

    {"MDX": "SELECT NON EMPTY [Product].[P1].[Product Category].Members ON 0,NON EMPTY [Outlet].[H1].[Region].Members ON 1 FROM [HoleFoods]"} 
    
[Result](https://github.com/intersystems-ru/Cache-MDX2JSON/wiki/MDX-output)

Please note that corresponding cube must be compiled and built beforehand.

Localization
-----------

If requested data or meta-information is available in several different languages, you can choose one, by supplying Accept-Language header to your HTTP request, formed in accordance with [RFC 2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.4). 

Troubleshooting
-----------

If something goes wrong, server must report an error in the following format {Error : "Error description"}, and usually that is a good indicator of what went wrong. If you received an error in another format or an error without "Error description" please file an issue [here](https://github.com/intersystems-ru/Cache-MDX2JSON/issues/8).

| Problem                     | Solution                         | 
|-----------------------------|----------------------------------|
| CSP Error                   | User does not have enough rights. Configure User or Webapplication roles  | 
| Authenticated access        | Web application must have password access, resource for database with MDX2JSON must have public RW rights enabled|
| No dashboards               | Configure roles. Change [dashboard scope](https://github.com/intersystems-ru/Cache-MDX2JSON/issues/10)|
| DeepSee errors              | Build and compile DeepSee cube(s)|
| MDX errors                  | Don't forget to escape JSON strings [here](http://json.org/string.gif) |
| Installation errors		  | Usually problems arise when installation is run under user without %All permissions. To repair the install rerun it under correct user. If the error persists then file an issue with installation log (terminal output) attached|


Debugging
-----------

Use $$$Debug macro. It would evaluate as true only if there is a "Debug" URL parameter present. Example request URL:

        http://localhost:57772/MDX2JSON/MDX?Namespace=Samples&Debug
		
Use with post conditional expressions, or other flow control statements:

		w:$$$Debug "debugging"
		if $$$Debug { w "debugging" } else { w "not debugging"}
		
Also available are $$$Public and $$$Private macros. Evaluates to true based on request port (80 and 443 are public, private otherwise).

Querying
-----------

Querying this project is done via REST web client. It may be a standalone application or a browser plug-in, a number of different solutions are available.

For Google Chrome, install [Advanced REST client](https://chrome.google.com/webstore/detail/advanced-rest-client/hgmloofddffdnphfgcellkdfbfbjeloo?hl=en) extension.
For Firefox, install [REST client](https://addons.mozilla.org/en-US/firefox/addon/restclient/) extension.
Open installed extension and set the following parameters:

- URL to required web api method, e.g.: `http://serverip:port/mdx2json/Dashboard?Namespace=Samples`
- Request type to `GET` or `POST`
- Payload to `{"Dashboard":"Listing with Filters.dashboard"}`
- Content-Type to `application/json` (only in Advanced REST client)

Press Send button to view results (depending on your server configuration you may be asked to provide valid login/password to access MDX2JSON api).

Development
-----------

To develop MDX2JSON you need:

1. Install MDX2JSON in `MDX2JSON` namespace
2. Install [Cache-Tort-Git](https://github.com/intersystems-ru/cache-tort-git)
3. In terminal, `MDX2JSON` namespace execute:

		set ^Git("settings","hook") = $lb("MDX2JSON.Tests","OnCommit")
		set ^Git("settings","groupByFolder") = 1

4. Activate Cache-Tort-Git for `MDX2JSON` namespace
5. Commit all changes via Studio