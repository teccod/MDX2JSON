Cache-MDX2JSON
==============

RESTful web api for MDX2JSON transformation (also JSONP and XML/A). Also supports requests about Dashboards and Widgets.

Installation
-----------

1. Download zip and unpack it.
2. Run in terminal (any namespace): 

        do ##class(%Installer.Installer).InstallFromCommandLine("{SourceDir}\Installer.cls.xml","Namespace={Namespace},SourceDir={SourceDir}",0)

  where: 
  
      {Namespace} is a namespace you want to install to. If it does not exist it would be created automatically. If it does exist only MDX2JSON package would be overwritten.
  
      {SourceDir} is a directory where you unpacked zip \ MDX2JSON (see 1).
On this step installer would create (if needed) Namespace and corresponding database, import source code and compile it, create required web application (named /{Namespace}) if one does not exist (skipping web application creation process if one does exist), and map MDX2JSON package to %All namespace (which will be created if does not exist).
3. Give the correct roles to /{Namespace} webapplication for it to be able to query desired cubes.


For information on how to work with this RESTful web API please refer to included documentation.

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
| Widgets                     | POST | {Dashboard:"DashboardName"} | JSON      | All widgets in a dashboard     |
| DataSource                  | POST |{DataSource:"Pivot fullname"}| JSON      | All info about Pivot           |
| FilterValues/:cube          | GET  |                             | JSON      | All filters for DeepSee Cube   |
| FilterValues                | POST |{ "Cube":"Cube Name","Values":1}| JSON   | All filters for DeepSee Cube with values (if Values = 1, set to 0 or omit otherwise)|
| FilterValues/:cube/:filter  | GET  |                             | JSON      | All possible values for filter |
| Format                      | GET  |                             | JSON      | Default formatting             |
| Test                        | GET  |                             | JSON      | Test info                      |

Example
-----------

Request Url: http://localhost:57772/MDX2JSON/MDX?Namespace=Samples

Request type: POST

Request body:

    {"MDX": "SELECT NON EMPTY [Product].[P1].[Product Category].Members ON 0,NON EMPTY [Outlet].[H1].[Region].Members ON 1 FROM [HoleFoods]"} 
    
Result: http://pastebin.com/XddMUPHX

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
| Authenticated access        | Guideline to configure [here](https://github.com/intersystems-ru/Cache-MDX2JSON/issues/24)|
| No dashboards               | Configure roles. Change [dashboard scope](https://github.com/intersystems-ru/Cache-MDX2JSON/issues/10)|
| DeepSee errors              | Build and compile DeepSee cube(s)|
| MDX errors                  | Don't forget to escape JSON strings [here](http://json.org/string.gif) |

Debugging
-----------

Use $$$Debug macro. It would evaluate as true only if there is a "Debug" URL parameter present. Example request URL:

        http://localhost:57772/MDX2JSON/MDX?Namespace=Samples&Debug
		
Use with postconditional expressions, or other flow control statements

		w:$$$Debug "debugging"
		if $$$Debug { w "debugging" } else { w "not debugging"}
