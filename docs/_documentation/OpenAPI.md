---
slug: openapi
title: OpenAPI
---

[OpenAPI](http://swagger.io/) is a specification and complete framework implementation for describing, producing, consuming, and visualizing RESTful web services. ServiceStack implements the 
[OpenAPI Spec](https://github.com/swagger-api/swagger-spec/blob/master/versions/2.0.md) back-end and embeds the OpenAPI UI front-end in a separate plugin which is available under [OpenAPI NuGet package](http://nuget.org/packages/ServiceStack.Api.OpenApi/):

    PM> Install-Package ServiceStack.Api.OpenApi

## Installation

You can enable OpenAPI by registering the `OpenApiFeature` plugin in AppHost with:

```csharp
public override void Configure(Container container)
{
    ...
    Plugins.Add(new OpenApiFeature());

    // uncomment CORS feature if it's has to be available from external sites 
    //Plugins.Add(new CorsFeature()); 
    ...
}
```

Then you will be able to view the OpenAPI UI from `/openapi-ui/`. A link to **OpenAPI UI** will also be available from your `/metadata` [Metadata Page](/metadata-page).

#### Configuring ServiceStack with MVC

If you're [Hosting ServiceStack with MVC](/mvc-integration) then you'll need to tell MVC to ignore the path where ServiceStack is hosted, e.g:

```csharp
routes.IgnoreRoute("api/{*pathInfo}"); 
```

For MVC4 projects, you'll also need to disable WebAPI:

```csharp
//WebApiConfig.Register(GlobalConfiguration.Configuration);
```

## OpenAPI Attributes

Each route could have a separate summary and description. You can set it with `Route` attribute:

```csharp
[Route("/hello", Summary = @"Default hello service.", 
    Notes = "Longer description for hello service.")]
```

You can set specific description for each HTTP method like shown below:

```csharp
[Route("/hello/{Name}", "GET", Summary="Says 'Hello' to provided Name", 
    Notes = "Longer description of the GET method which says 'Hello'")]
[Route("/hello/{Name}", "POST", Summary="Says 'Hello' to provided Name", 
    Notes = "Longer description of the POST method which says 'Hello'")]
```

You can further document your services in the OpenAPI with the new `[Api]` and `[ApiMember]` annotation attributes, e,g: Here's an example of a fully documented service:

```csharp
[Api("Service Description")]
[ApiResponse(HttpStatusCode.BadRequest, "Your request was not understood")]
[ApiResponse(HttpStatusCode.InternalServerError, "Oops, something broke")]
[Route("/swagger/{Name}", "GET", Summary = "GET Summary", Notes = "Notes")]
[Route("/swagger/{Name}", "POST", Summary = "POST Summary", Notes="Notes")]
public class MyRequestDto
{
    [ApiMember(Name="Name", Description = "Name Description",
        ParameterType = "path", DataType = "string", IsRequired = true)]
    [ApiAllowableValues("Name", typeof(Color))] //Enum
    public string Name { get; set; }
}
```

You can Exclude **properties** from being listed in OpenAPI with:

```csharp
[IgnoreDataMember]
```

Exclude **properties** from being listed in OpenAPI Schema Body with:

```csharp
[ApiMember(ExcludeInSchema=true)]
```

### Exclude Services from Metadata Pages

To exclude entire Services from showing up in OpenAPI or any other Metadata Services (i.e. Metadata Pages, Postman, NativeTypes, etc), annotate **Request DTO's** with:

```csharp
[Exclude(Feature.Metadata)]
public class MyRequestDto { ... }
```

### OpenAPI UI Route Summaries

The OpenAPI UI groups multiple routes under a single top-level route that covers multiple different 
services sharing the top-level route which can be specified using the `RouteSummary` dictionary of 
the `OpenApiFeature` plugin, e.g: 

```csharp
Plugins.Add(new OpenApiFeature {
    RouteSummary = {
        { "/top-level-path", "Route Summary" }
    }
});
```

### OpenAPI operation filters

You can override operation or parameter definitions by specifying appropriate `Action<>` in plugin configuration:

```csharp
Plugins.Add(new OpenApiFeature()
{
    OperationFilter = (verb, operation) => operation.Tags.Add("all operations")
});
```

Available configuration options:

```
ApiDeclarationFilter - allows to modify final result of returned OpenAPI json
OperationFilter - allows to modify operations
ModelFilter - allows to modify OpenAPI schema for user types
ModelPropertyFilter - allows to modify propery declarations in OpenAPI schema
```

### Properties naming conventions

You can control naming conventions of generated properties by following configuration options:

```
UseCamelCaseModelPropertyNames - generate camel case property names
UseLowercaseUnderscoreModelPropertyNames - generate underscored lower cased property names (to enable this feature UseCamelCaseModelPropertyNames must also be set) 
```

Example:

```csharp
Plugins.Add(new OpenApiFeature()
{
    UseCamelCaseModelPropertyNames = true,
    UseLowercaseUnderscoreModelPropertyNames = true
});
```

### Miscellaneous configuration options

```
DisableAutoDtoInBodyParam - disables adding `body` parameter for request DTO to operations
UseBootstrapTheme - use bootstrap for OpenAPI UI
LogoUrl - url of the logo image for OpenAPI UI
```

Example:

```csharp
Plugins.Add(new OpenApiFeature()
{
    DisableAutoDtoInBodyParam = true
});
```


## Virtual File System

The docs on the Virtual File System shows how to override embedded resources:

### Overriding OpenAPI Embedded Resources

ServiceStack's [Virtual File System](/virtual-file-system) supports multiple file source locations where you can override OpenAPI's embedded files by including your own custom files in the same location as the existing embedded files. This lets you replace built-in ServiceStack embedded resources with your own by simply copying the [/openapi-ui](https://github.com/ServiceStack/ServiceStack/tree/master/src/ServiceStack.Api.OpenApi/openapi-ui) or [/openapi-ui-bootstrap](https://github.com/ServiceStack/ServiceStack/tree/master/src/ServiceStack.Api.OpenApi/openapi-ui-bootstrap) files you want to customize and placing them in your Website Directory at:

```
/openapi-ui
  /css
  /images
  /lib
  index.html

/openapi-ui-bootstrap
  index.html
  swagger-like-template.html
```

### Basic Auth added to OpenAPI

![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/release-notes/swagger-basicauth.png)

Users can call protected Services using the Username and Password fields in OpenAPI UI. 
OpenAPI UI sends these credentials with every API request using HTTP Basic Auth, 
which can be enabled in your AppHost with:

```csharp
Plugins.Add(new AuthFeature(...,
      new IAuthProvider[] { 
        new BasicAuthProvider(), //Allow Sign-ins with HTTP Basic Auth
      }));
```

Alternatively users can login outside of OpenAPI UI, to access protected Services in OpenAPI UI.

## Demo Project

ServiceStack.UseCases project contains example [OpenApiHelloWorld](https://github.com/ServiceStack/ServiceStack.UseCases/tree/master/OpenApiHelloWorld). It demonstrates how to use and integrate [ServiceStack.Api.OpenApi](http://nuget.org/packages/ServiceStack.Api.OpenApi/). Take a look at [README.txt](https://github.com/ServiceStack/ServiceStack.UseCases/blob/master/OpenApiHelloWorld/README.txt) for more details.