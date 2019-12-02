# ASP.NET Core Web API 最佳实践

> [ASP.NET-Core-Web-API-Best-Practices-Guide](https://code-maze.us12.list-manage.com/track/click?u=9bb15645129501e5249a9a8e1&id=986d07e1f0&e=1184a539da)

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

因此，最好的方式是保持 `ConfigureServices` 方法简洁，并且尽可能地具有可读性。当然，我们需要在该方法内部编写代码来注册服务，但是我们可以通过使用 `扩展方法` 来让我们的代码更加地可读和可维护。

例如，让我们看一个注册 CORS 服务的不好方式：

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

尽管这种方式看起来挺好，也能正常地将 CORS 服务注册成功。但是想象一下，在注册了十几个服务之后这个方法体的长度。

这样一点也不具有可读性。

一种好的方式是通过在扩展类中创建静态方法：

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

我们应该尝试将我们的应用程序拆分为多个小项目。通过这种方式，我们可以获得最佳的项目组织方式，并能将关注点分离（SoC）。我们的实体、契约、访问数据库操作、记录信息或者发送邮件的业务逻辑应该始终放在单独的 .NET Core 类库项目中。

应用程序中的每个小项目都应该包含多个文件夹用来组织业务逻辑。

这里有个简单的示例用来展示一个复杂的项目应该如何组织：

<div align="center">

![Project-Structure](images/01-Project-Structure.png)

</div>

## 基于环境的设置

> ENVIRONMENT BASED SETTINGS

当我们开发应用程序时，它处于开发环境。但是一旦我们发布之后，它将处于生产环境。因此，将每个环境进行隔离配置往往是一种好的实践方式。

在 .NET Core 中，这一点很容易实现。

一旦我们创建好了项目，就已经有一个 `appsettings.json` 文件，当我们展开它时会看到 `appsettings.Development.json` 文件：

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

在一些不同的示例教程中，我们可能看到 DAL 的实现在主项目中，并且每个控制器中都有实例。我们不建议这么做。

当我们编写 DAL 时，我们应该将其作为一个独立的服务来创建。在 .NET Core 项目中，这一点很重要，因为当我们将 DAL 作为一个独立的服务时，我们就可以将其直接注入到 IOC（控制反转）容器中。IOC 是 .NET Core 内置功能。通过这种方式，我们可以在任何控制器中通过构造函数注入的方式来使用。

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

因此，我们的控制器应该通过构造函数注入的方式接收服务实例，并组织 HTTP 的操作方法（GET，POST，PUT，DELETE，PATCH...）:

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

我们的 Action 应该尽量保持简洁，它们的职责应该包括处理 HTTP 请求，验证模型，捕捉异常和返回响应。

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

## 使用过滤器移除重复代码

> USING ACTIONFILTERS TO REMOVE DUPLICATED CODE

ASP.NET Core 的过滤器可以让我们在请求管道的特定状态之前或之后运行一些代码。因此如果我们的 action  中有重复验证的话，可以使用它来简化验证操作。

当我们在 action 方法中处理 PUT 或者 POST 请求时，我们需要验证我们的模型对象是否符合我们的预期。作为结果，这将导致我们的验证代码重复，我们希望避免出现这种情况，（基本上，我们应该尽我们所能避免出现任何代码重复。）我们可以在代码中通过使用 ActionFilter 来代替我们的验证代码：

```C#
if (!ModelState.IsValid)
{
    //bad request and logging logic
}
```

我们可以创建一个过滤器：

```C#
public class ModelValidationAttribute : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        if (!context.ModelState.IsValid)
        {
            context.Result = new BadRequestObjectResult(context.ModelState);
        }
    }
}
```

然后在 `Startup` 类的 `ConfigureServices` 函数中将其注入：

```C#
services.AddScoped<ModelValidationAttribute>();
```

现在，我们可以将上述注入的过滤器应用到我们的 action 中。

## Microsoft.AspNetCore.all 元包

> MICROSOFT.ASPNETCORE.ALL META-PACKAGE

注：如果你使用的是 2.1 和更高版本的 ASP.NET Core。建议使用 Microsoft.AspNetCore.App 包，而不是 Microsoft.AspNetCore.All。这一切都是出于安全原因。此外，如果使用 2.1 版本创建新的 WebAPI 项目，我们将自动获取 AspNetCore.App 包，而不是 AspNetCore.All。

这个元包包含了所有 AspNetCore 的相关包，EntityFrameworkCore 包，SignalR 包（version 2.1） 和依赖框架运行的支持包。采用这种方式创建一个新项目很方便，因为我们不需要手动安装一些我们可能使用到的包。

当然，为了能使用 Microsoft.AspNetCore.all 元包，需要确保你的机器安装了 .NET Core Runtime。

## 路由

> ROUTING

在 .NET Core Web API 项目中，我们应该使用属性路由代替传统路由，这是因为属性路由可以帮助我们匹配路由参数名称与 Action 内的实际参数方法。另一个原因是路由参数的描述，对我们而言，一个名为 "ownerId" 的参数要比 "id" 更加具有可读性。

我们可以使用 **[Route]** 属性来在控制器的顶部进行标注：

```C#
[Route("api/[controller]")]
public class OwnerController : Controller
{
    [Route("{id}")]
    [HttpGet]
    public IActionResult GetOwnerById(Guid id)
    {
    }
}
```

还有另一种方式为控制器和操作创建路由规则：

```C#
[Route("api/owner")]
public class OwnerController : Controller
{
    [Route("{id}")]
    [HttpGet]
    public IActionResult GetOwnerById(Guid id)
    {
    }
}
```

对于这两种方式哪种会好一些存在分歧，但是我们经常建议采用第二种方式。这是我们一直在项目中采用的方式。

当我们谈论路由时，我们需要提到路由的命名规则。我们可以为我们的操作使用描述性名称，但对于 路由/节点，我们应该使用 NOUNS 而不是 VERBS。

一个较差的示例：

```C#
[Route("api/owner")]
public class OwnerController : Controller
{
    [HttpGet("getAllOwners")]
    public IActionResult GetAllOwners()
    {
    }
    [HttpGet("getOwnerById/{id}"]
    public IActionResult GetOwnerById(Guid id)
    {
    }
}
```

一个较好的示例：

```C#
[Route("api/owner")]
public class OwnerController : Controller
{
    [HttpGet]
    public IActionResult GetAllOwners()
    {
    }
    [HttpGet("{id}"]
    public IActionResult GetOwnerById(Guid id)
    {
    }
}

```

更多关于 Restful 实践的细节解释，请查阅：[Top REST API Best Practices](https://code-maze.com/top-rest-api-best-practices/)

## 日志

> LOGGING

如果我们打算将我们的应用程序发布到生产环境，我们应该在合适的位置添加一个日志记录机制。在生产环境中记录日志对于我们梳理应用程序的运行很有帮助。

.NET Core 通过继承 `ILogger` 接口实现了它自己的日志记录。通过借助依赖注入机制，它可以很容易地使用。

```C#
public class TestController: Controller
{
    private readonly ILogger _logger;
    public TestController(ILogger<TestController> logger)
    {
        _logger = logger;
    }
}
```

然后，在我们的 action 中，我们可以通过使用 _logger 对象借助不同的日志级别来记录日志。

.NET Core 支持使用于各种日志记录的 Provider。因此，我们可能会在项目中使用不同的 Provider 来实现我们的日志逻辑。

NLog 是一个很不错的可以用于我们自定义的日志逻辑类库，它极具扩展性。支持结构化日志，且易于配置。我们可以将信息记录到控制台，文件甚至是数据库中。

想了解更多关于该类库在 .NET Core 中的应用，请查阅：[.NET Core series – Logging With NLog.](https://code-maze.com/net-core-web-development-part3/)

Serilog 也是一个很不错的类库，它适用于 .NET Core 内置的日志系统。

## 加密

> CRYPTOHELPER

我们不会建议将密码以明文形式存储到数据库中。处于安全原因，我们需要对其进行哈希处理。这超出了本指南的内容范围。互联网上有大量哈希算法，其中不乏一些不错的方法来将密码进行哈希处理。

但是如果需要为 .NET Core 的应用程序提供易于使用的加密类库，CryptoHelper 是一个不错的选择。

CryptoHelper 是适用于 .NET Core 的独立密码哈希库，它是基于 PBKDF2 来实现的。通过创建 `Data Protection` 栈来将密码进行哈希化。这个类库在 NuGet 上是可用的，并且使用也很简单：

```C#
using CryptoHelper;

// Hash a password
public string HashPassword(string password)
{
    return Crypto.HashPassword(password);
}

// Verify the password hash against the given password
public bool VerifyPassword(string hash, string password)
{
    return Crypto.VerifyHashedPassword(hash, password);
}
```

## 内容协商

> CONTENT NEGOTIATION

默认情况下，.NET Core Web API 会返回 JSON 格式的结果。大多数情况下，这是我们所希望的。

但是如果客户希望我们的 Web API 返回其它的响应格式，例如 XML 格式呢？

为了解决这个问题，我们需要进行服务端配置，用于按需格式化我们的响应结果：

```C#
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers().AddXmlSerializerFormatters();
}
```

但有时客户端会请求一个我们 Web API 不支持的格式，因此最好的实践方式是对于未经处理的请求格式统一返回 406 状态码。这种方式也同样能在 ConfigureServices 方法中进行简单配置：

```C#
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers(options => options.ReturnHttpNotAcceptable = true).AddXmlSerializerFormatters();
}
```

我们也可以创建我们自己的格式化规则。

这一部分内容是一个很大的主题，如果你希望了解更对，请查阅：[Content Negotiation in .NET Core](https://code-maze.com/content-negotiation-dotnet-core/)

## 使用 JWT

> USING JWT

现如今的 Web 开发中，JSON Web Tokens (JWT) 变得越来越流行。得益于 .NET Core 内置了对 JWT 的支持，因此实现起来非常容易。JWT 是一个开发标准，它允许我们以 JSON 格式在服务端和客户端进行安全的数据传输。

我们可以在 ConfigureServices 中配置 JWT 认证：

```C#
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
        .AddJwtBearer(options => 
        {
            options.TokenValidationParameters = new TokenValidationParameters
            {
                ValidateIssuer = true,
                ValidIssuer = _authToken.Issuer,

                ValidateAudience = true,
                ValidAudience = _authToken.Audience,

                ValidateIssuerSigningKey = true,
                IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_authToken.Key)),

                RequireExpirationTime = true,
                ValidateLifetime = true,

                //others
            };
        });
}
```

为了能在应用程序中使用它，我们还需要在 Configure 中调用下面一段代码：

```C#
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseAuthentication();
}
```

此外，创建 Token 可以使用如下方式：

```C#
var securityToken = new JwtSecurityToken(
                claims: new Claim[]
                {
                    new Claim(ClaimTypes.NameIdentifier,user.Id),
                    new Claim(ClaimTypes.Email,user.Email)
                },
                issuer: _authToken.Issuer,
                audience: _authToken.Audience,
                notBefore: DateTime.Now,
                expires: DateTime.UtcNow.AddDays(_authToken.Expires),
                signingCredentials: new SigningCredentials(
                    new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_authToken.Key)),
                    SecurityAlgorithms.HmacSha256Signature));

Token = new JwtSecurityTokenHandler().WriteToken(securityToken)
```

基于 Token 的用户验证可以在控制器中使用如下方式：

```C#
var auth = await HttpContext.AuthenticateAsync();
var id = auth.Principal.Claims.FirstOrDefault(x => x.Type.Equals(ClaimTypes.NameIdentifier))?.Value;
```

我们也可以将 JWT 用于授权部分，只需添加角色声明到 JWT 配置中即可。

更多关于 .NET Core 中 JWT 认证和授权部分，请查阅：[authentication-aspnetcore-jwt-1](https://code-maze.com/authentication-aspnetcore-jwt-1/) 和 [authentication-aspnetcore-jwt-2](https://code-maze.com/authentication-aspnetcore-jwt-2/)

## 总结

在这份指南中，我们的主要目的是让你熟悉关于使用 .NET Core 开发 web API 项目时的一些最佳实践。这里面的部分内容在其它框架中也同样适用。因此，熟练掌握它们很有用。

非常感谢你能阅读这份指南，希望它能对你有所帮助。