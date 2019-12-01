# ASP.NET Core Web API 最佳实践指南

> ASP.NET-Core-Web-API-Best-Practices-Guide

## 目录

## 介绍

当我们编写一个项目的时候，我们的主要目标是使它能如期运行，并尽可能地满足所有用户需求。

但是，你难道不认为创建一个能正常工作的项目还不够吗？同时这个项目不应该也是可维护和可读的吗？

事实证明，我们需要把更多的关注点放到我们项目的可读性和可维护性上。这背后的主要原因是我们或许不是这个项目的唯一编写者。一旦我们完成后，其他人也极有可能会加入到这里面来。

因此，我们应该把关注点放到哪里呢？

在这一份指南中，关于开发 .NET Core Web API 项目，我们将叙述一些我们认为会是最佳实践的方式。进而让我们的项目变得更好和更加具有可维护性。

现在，让我们开始想一些可以应用到 ASP.NET Web API 项目中的一些最佳实践。

## Startup 类 和 服务配置

> STARTUP CLASS AND THE SERVICE CONFIGURATION

在 `Startup` 类中，有两个方法：`ConfigureServices` 是用于服务注册，`Configure` 方法是向应用程序的请求管道中添加中间件。

因此，最好的方式是保持 `ConfigureServices` 方法干净，并且尽可能地具有可读性。当然，我们需要在该方法内部编写代码来注册服务，但是我们可以通过使用 `扩展方法` 来让我们的代码更加地可读和可维护。

例如，让我们看一个注册 CORS 服务的错误方式：

```C#
public void ConfigureServices(IServiceCollection services)
{
    services.AddCors(options => 
    {
        options.AddPolicy("CorsPolicy", builder => builder.AllowAnyOrigin()
            .AllowAnyMethod()
            .AllowAnyHeader()
            .AllowCredentials());
    });
}
```

尽管这种方式看起来挺好，也能没有问题地将 CORS 服务注册成功。想象一下，在注册了十几个服务之后这个方法体的长度。

这样一点也不具有可读性。

一种好的方式是通过在扩展类中创建静态方式：

```C#
public static class ServiceExtensions
{
    public static void ConfigureCors(this IServiceCollection services)
    {
        services.AddCors(options =>
        {
            options.AddPolicy("CorsPolicy", builder => builder.AllowAnyOrigin()
                .AllowAnyMethod()
                .AllowAnyHeader()
                .AllowCredentials());
        });
    }
}
```

然后，只需要调用这个扩展方法即可：

```C#
public void ConfigureServices(IServiceCollection services)
{
    services.ConfigureCors();
}
```

了解更多关于 .NET Core's 的项目配置，请查看：[.NET Core Project Configuration](https://code-maze.com/net-core-web-development-part2/)

## 项目组织

> PROJECT ORGANIZATION

我们应该尝试将我们的应该程序拆分为多个更小的项目。通过这种方式，我们可以获得最佳的项目组织方式，并能将关注点分离（SoC）.和我们的实体、契约、访问数据库、记录信息或者发送邮件的业务逻辑应该始终是在单独的 .NET Core 类库项目中。

应用程序中的每个小项目都应该包含多个文件夹用来组织业务逻辑。

这里有个简单的示例用来展示一个复杂的项目应该如何组织：

<div align="center">

![Project-Structure](images/01-Project-Structure.png)

</div>

## 基于环境的设置

> ENVIRONMENT BASED SETTINGS

当我们开发应用程序时，它处于开发环境。但是一旦我们发布我们的应用程序，它将处于生产环境。因此，将每个环境进行分隔配置往往是一种好的实践方式。

在 .NET Core 中，这一点很容易完成。

一旦我们创建好了项目，我们就已经有一个 `appsettings.json` 文件，当我们展开它时会看到 `appsettings.Development.json` 文件：

<div align="center">

![Appsettings-development](images/02-Appsettings-development.png)

</div>

此文件中的所有设置将用于开发环境。

我们应该添加另一个文件 `appsettings.Production.json`，将其用于生产环境：

<div align="center">

![Appsettings-production.png](images/03-Appsettings-production.png)

</div>

生产文件将位于开发文件下面。

设置修改后，我们就可以通过不同的 appsettings 文件来加载不同的配置，取决于我们应用程序当前所处环境，.NET Core 将会给我们提供正确的设置。更多关于这一主题，请查阅：[Multiple Environments in ASP.NET Core.](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/environments?view=aspnetcore-3.0)

## 数据访问层

> DATA ACCESS LAYER

在一些不同的示例教程中，我们可能看到 DAL 的实现在主项目中，并且每个控制器中都有实例。这是我们不应该做的事情。

当我们编写 DAL 时，我们应该将其作为一个独立的服务来创建。在 .NET Core 项目中，这一点很重要，因为当我们将 DAL 作为一个独立的服务时，我们就可以将其直接注入到 IOC（控制反转）容器中。IOC 是 .NET Core 内置的功能。通过这种方式，我们可以在任何控制器中通过构造函数注入的方式来使用。

```C#
public class OwnerController: Controller
{
    private IRepository _repository;
    public OwnerController(IRepository repository)
    {
        _repository = repository;
    }
}
```

## 控制器

> CONTROLLERS

控制器应该始终尽量保持整洁。我们不应该将任何业务逻辑放置于内。

因此，我们的控制器应该复杂通过构造函数注入的方式接收服务实例，组织 HTTP 操作方法（GET，POST，PUT，DELETE，PATCH...）:

```C#
public class OwnerController : Controller
{
    private ILoggerManager _logger;
    private IRepository _repository;
    public OwnerController(ILoggerManager logger, IRepository repository)
    {
        _logger = logger;
        _repository = repository;
    }

    [HttpGet]
    public IActionResult GetAllOwners()
    {
    }
    [HttpGet("{id}", Name = "OwnerById")]
    public IActionResult GetOwnerById(Guid id)
    {
    }
    [HttpGet("{id}/account")]
    public IActionResult GetOwnerWithDetails(Guid id)
    {
    }
    [HttpPost]
    public IActionResult CreateOwner([FromBody]Owner owner)
    {
    }
    [HttpPut("{id}")]
    public IActionResult UpdateOwner(Guid id, [FromBody]Owner owner)
    {
    }
    [HttpDelete("{id}")]
    public IActionResult DeleteOwner(Guid id)
    {
    }
}
```

我们的 action 应该尽量保持整洁简单，它们的职责应该包括处理 HTTP 请求，验证模型，捕捉异常和返回响应。

```C#
[HttpPost]
public IActionResult CreateOwner([FromBody]Owner owner)
{
    try
    {
        if (owner.IsObjectNull())
        {
            return BadRequest("Owner object is null");
        }
        if (!ModelState.IsValid)
        {
            return BadRequest("Invalid model object");
        }
        _repository.Owner.CreateOwner(owner);
        return CreatedAtRoute("OwnerById", new { id = owner.Id }, owner);
    }
    catch (Exception ex)
    {
        _logger.LogError($"Something went wrong inside the CreateOwner action: { ex} ");
        return StatusCode(500, "Internal server error");
    }
}
```

在大多数情况下，我们的 action 应该将 `IActonResult` 作为返回类型（有时我们希望返回一个特定类型或者是 `JsonResult` ...）。通过使用这种方式，我们可以很好地使用 .NET Core 中内置方法的返回值和状态码。

使用最多的方法是：

- OK => returns the 200 status code
- NotFound => returns the 404 status code
- BadRequest => returns the 400 status code
- NoContent => returns the 204 status code
- Created, CreatedAtRoute, CreatedAtAction => returns the 201 status code
- Unauthorized => returns the 401 status code
- Forbid => returns the 403 status code
- StatusCode => returns the status code we provide as input

## 处理全局异常

> HANDLING ERRORS GLOBALLY

在上面的示例中，我们的 action 内部有一个 `try-catch` 代码块。这一点很重要，我们需要在我们的 action 方法体中处理所有的异常（包括未处理的）。一些开发者在 action 中使用 `try-catch` 代码块，这种方式明显没有任何问题。但我们希望 action 尽量保持简洁。因此，从我们的 action 中删除  `try-catch` ,并将其放在一个集中的地方会是一种更好的方式。.NET Core 给我们提供了一种处理全局异常的方式，只需要稍加修改，就可以使用内置且完善的的中间件。我们需要做的修改就是在 `Startup` 类中修改 `Configure` 方法：

```C#
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseExceptionHandler(config => 
    {
        config.Run(async context => 
        {
            context.Response.StatusCode = 500;
            context.Response.ContentType = "application/json";

            var error = context.Features.Get<IExceptionHandlerFeature>();
            if (error != null)
            {
                var ex = error.Error;
                await context.Response.WriteAsync(new ErrorModel
                {
                    StatusCode = 500,
                    ErrorMessage = ex.Message
                }.ToString());
            }
        });
    });

    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```

我们也可以通过创建自定义的中间件来实现我们的自定义异常处理：

```C#
// You may need to install the Microsoft.AspNetCore.Http.Abstractions package into your project
public class CustomExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<CustomExceptionMiddleware> _logger;
    public CustomExceptionMiddleware(RequestDelegate next, ILogger<CustomExceptionMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task Invoke(HttpContext httpContext)
    {
        try
        {
            await _next(httpContext);
        }
        catch (Exception ex)
        {
            _logger.LogError("Unhandled exception....", ex);
            await HandleExceptionAsync(httpContext, ex);
        }
    }

    private Task HandleExceptionAsync(HttpContext httpContext, Exception ex)
    {
        //todo
        return Task.CompletedTask;
    }
}

// Extension method used to add the middleware to the HTTP request pipeline.
public static class CustomExceptionMiddlewareExtensions
{
    public static IApplicationBuilder UseCustomExceptionMiddleware(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<CustomExceptionMiddleware>();
    }
}
```

之后，我们只需要将其注入到应用程序的请求管道中即可：

```C#
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseCustomExceptionMiddleware();
}
```

## 使用操作过滤器移除重复代码

> USING ACTIONFILTERS TO REMOVE DUPLICATED CODE

ASP.NET Core 的过滤器可以让我们在请求管道的特定状态之前或之后运行一些代码。因此如果我们的 action  中有重复验证的话，可以使用它来执行验证操作。

当我们在 action 方法中处理 PUT 或者 POST 请求时，我们需要验证我们的模型对象是否符合我们的预期。