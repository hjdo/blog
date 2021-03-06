---
title: .NET-.NET Core中的配置
date: 2016-02-20T13:31:43+08:00
tags: ["ASP.NET Core","转载"]
permalink: NET-NET-in-the-Core-Configuration
categories: ["cSharp"]
---
> 就在截稿时间之前，Microsoft 宣布了更改 ASP.NET 5 和相关堆叠的名称。ASP.NET 5 已更名为 ASP.NET Core 1.0。Entity Framework (EF) 7 已更名为 Entity Framework (EF) Core 1.0。虽然 ASP.NET 5 和 EF7 程序包以及命名空间将会发生变化，但新的命名法不会对本文中的课程造成任何影响。

使用 ASP.NET 5 的同仁们自然都会注意到此平台中新增了配置支持，可用于 NuGet 包的 Microsoft.Extensions.Configuration 集合。

新的配置支持名称/值对列表（可分入多层层次结构）。例如，您可以将一个设置存储在 SampleApp:Users:InigoMontoya:MaximizeMainWindow 中，将另一个设置存储在 SampleApp:AllUsers:Default:MaximizeMainWindow 中。

存储的所有值都会映射到字符串，您可以借助内置的绑定支持，将设置反序列化为自定义 POCO 对象。已经熟悉新配置 API 的同仁们可能最初是在 ASP.NET 5 注意到的。不过，此 API 绝不仅限于 ASP.NET。实际上，本文中的所有列表都是使用 Microsoft .NET Framework 4.5.1（同时引用 ASP.NET 5 RC1 中的 Microsoft.Extensions.Configuration 包）在 Visual Studio 2015 单元测试项目中创建（有关源代码，请访问 [gitHub.com/IntelliTect/Articles](http://github.com/IntelliTect/Articles)）。
<!--more-->
配置 API 支持内存中 .NET 对象、INI 文件、JSON 文件、XML 文件、命令行自变量、环境变量、加密的用户存储的配置提供程序，以及您创建的所有自定义提供程序。如果您希望对自己的配置利用 JSON 文件，只需添加 Microsoft.Extensions.Configuration.Json NuGet 包。然后，如果您想允许命令行提供配置信息，只需添加 Microsoft.Extensions.Configuration.CommandLine NuGet 包即可（可以在其他配置引用的基础上另外添加，也可以替代其他配置引用添加）。如果您对内置的所有配置提供程序都不满意，也可以创建您自己的提供程序，方法为实现 Microsoft.Extensions.Configuration.Abstractions 中的接口。

# 1. 检索配置设置
请查看图 1，自行熟悉如何检索配置设置。
图 1：使用 InMemoryConfigurationProvider 和 ConfigurationBinder 扩展方法的基本配置:
```csharp
public class Program
{
  static public string DefaultConnectionString { get; } =
    @"Server=(localdb)\\mssqllocaldb;Database=SampleData-0B3B0919-C8B3-481C-9833-
    36C21776A565;Trusted_Connection=True;MultipleActiveResultSets=true";
  static IReadOnlyDictionary<string, string> DefaultConfigurationStrings{get;} =
    new Dictionary<string, string>()
    {
      ["Profile:UserName"] = Environment.UserName,
      [$"AppConfiguration:ConnectionString"] = DefaultConnectionString,
      [$"AppConfiguration:MainWindow:Height"] = "400",
      [$"AppConfiguration:MainWindow:Width"] = "600",
      [$"AppConfiguration:MainWindow:Top"] = "0",
      [$"AppConfiguration:MainWindow:Left"] = "0",
    };
  static public IConfiguration Configuration { get; set; }
  public static void Main(string[] args = null)
  {
    ConfigurationBuilder configurationBuilder =
      new ConfigurationBuilder();
      // Add defaultConfigurationStrings
      configurationBuilder.AddInMemoryCollection(
        DefaultConfigurationStrings);
      Configuration = configurationBuilder.Build();
      Console.WriteLine($"Hello {Configuration["Profile:UserName"]}");
      ConsoleWindow consoleWindow =
        Configuration.Get<ConsoleWindow>("AppConfiguration:MainWindow");
      ConsoleWindow.SetConsoleWindow(consoleWindow);
  }
}
```
若要轻松访问配置，请先从 ConfigurationBuilder 实例入手，这是 Microsoft.Extensions.Configuration NuGet 包中的类。借助 ConfigurationBuilder 实例，您可以使用 IConfigurationBuilder 扩展方法（如 AddInMemoryCollection）直接添加提供程序，如图 1 所示。此方法提取配置名称/值对的 Dictionary<string,string> 实例，以便在将其添加到 ConifigurationBuilder 实例之前初始化配置提供程序。在“已配置”配置生成器后，您可以调用其 Build 方法来检索配置。

如前所述，配置就是名称/值对的分层列表，其中节点是由冒号分隔。因此，若要检索特定值，您只需使用相应项目的密钥访问配置索引器即可：
```csharp
Console.WriteLine($"Hello {Configuration["Profile:UserName"]}");
```
不过，访问值并不仅限于检索字符串。例如，您可以通过 ConfigurationBinder 的 Get<T> 扩展方法检索值。比如，若要检索主要窗口屏幕缓冲区大小，您可以使用：
```csharp
Configuration.Get<int>("AppConfiguration:MainWindow:ScreenBufferSize", 80);
```
这种绑定支持需要引用 Microsoft.Extensions.Configuration.Binder NuGet 包。

请注意，密钥后面有一个可选自变量，您可以用来指定在密钥不存在时返回的默认值（如果没有默认值，那么会返回所分配的 default(T)，而不是像您所预料的那样引发异常）。

配置值并不仅限于标量。您可以检索 POCO 对象或甚至整个对象图。为了检索其成员映射到 AppConfiguration:MainWindow 配置部分的 ConsoleWindow 实例，图 1 使用：
```csharp
ConsoleWindow consoleWindow =
  Configuration.Get<ConsoleWindow>("AppConfiguration:MainWindow")
```
或者，您可以定义配置图表（如 AppConfiguration），如图 2 所示。
图 2：示例配置对象图:
```csharp
class AppConfiguration
{
  public ProfileConfiguration Profile { get; set; }
   public string ConnectionString { get; set; }
  public WindowConfiguration MainWindow { get; set; }
  public class WindowConfiguration
  {
    public int Height { get; set; }
    public int Width { get; set; }
    public int Left { get; set; }
    public int Top { get; set; }
  }
  public class ProfileConfiguration
  {
    public string UserName { get; set; }
  }
}
public static void Main()
{
  // ...
  AppConfiguration appConfiguration =
    Program.Configuration.Get<AppConfiguration>(
      nameof(AppConfiguration));
  // Requires referencing System.Diagnostics.TraceSource in Corefx
  System.Diagnostics.Trace.Assert(
    600 == appConfiguration.MainWindow.Width);
}
```
有了此类对象图，您可以使用强类型对象层次结构（随后可使用此层次结构一次性检索所有设置）定义所有或部分配置。

# 2. 多个配置提供程序
InMemoryConfigurationProvider 可有效用于存储默认值或可能的已计算值。不过，如果您仅使用此提供程序，那么在使用 ConfigurationBuilder 注册配置之前，您需要检索配置并将配置加载到 Dictionary<string,string> 中，负担很重。幸运的是，您还可以使用更多内置配置提供程序，包括三个基于文件的提供程序（XmlConfigurationProvider、IniConfigurationProvider 和 JsonConfigurationProvider）、一个环境变量提供程序 (EnvironmentVariableConfigurationProvider)、一个命令行自变量提供程序 (CommandLineConfigurationProvider)。此外，您还可以根据自己的应用程序逻辑来混合使用这些提供程序。例如，您可以按以下升序优先顺序来指定配置设置：
　　● InMemoryConfigurationProvider
　　● 适用于 Config.json 的 JsonFileConfigurationProvider
　　● 适用于 Config.Production.json 的 JsonFileConfigurationProvider
　　● EnvironmentVariableConfigurationProvider
　　● CommandLineConfigurationProvider

换言之，默认配置值可能存储在代码中。接下来，Config.Production.json 前面的 config.json 文件可能会替代 InMemory 指定的值，稍后 JSON 等提供程序将优先处理所有重叠值。然后，在部署时，您可能会在环境变量中存储自定义配置值。例如，您可能会从 Windows 环境变量中检索环境设置，然后访问环境变量确定的特定文件（可能是 Config.Test.Json），而不是对 Config.Production.json 进行硬编码（“环境设置”一词有多义性：与生产、测试、预生产或开发、%USERNAME% 或 %USERDOMAIN% 等 Windows 环境变量相关）。 最后，您通过命令行指定（或替代）之前提供的所有设置（例如，可能一次性更改为启用日志记录）。

若要指定各个提供程序，请将它们添加到配置生成器中（通过扩展方法 AddX Fluent API），如图 3 所示：
图 3：添加多个配置提供程序（最后指定的提供程序优先）：
```csharp
public static void Main(string[] args = null)
{
  ConfigurationBuilder configurationBuilder =
    new ConfigurationBuilder();
  configurationBuilder
    .AddInMemoryCollection(DefaultConfigurationStrings)
    .AddJsonFile("Config.json",
      true) // Bool indicates file is optional
    // "EssentialDotNetConfiguartion" is an optional prefix for all
    // environment configuration keys, but once used,
    // only environment variables with that prefix will be found        
    .AddEnvironmentVariables("EssentialDotNetConfiguration")
    .AddCommandLine(
      args, GetSwitchMappings(DefaultConfigurationStrings));
  Console.WriteLine($"Hello {Configuration["Profile:UserName"]}");
  AppConfiguration appConfiguration =
    Configuration.Get<AppConfiguration>(nameof(AppConfiguration));
}
static public Dictionary<string,string> GetSwitchMappings(
  IReadOnlyDictionary<string, string> configurationStrings)
{
  return configurationStrings.Select(item =>
    new KeyValuePair<string, string>(
      "-" + item.Key.Substring(item.Key.LastIndexOf(':')+1),
      item.Key))
      .ToDictionary(
        item => item.Key, item=>item.Value);
}
```
对于 JsonConfigurationProvider，您可以将文件设为必有或可选；因此，需要在 AddJsonFile 上添加其他可选参数。如果您没有提供任何参数，那么文件为必有。如果找不到文件，则会触发 System.IO.FileNotFoundException。鉴于 JSON 的分层性质，配置非常适合配置 API（见图 4）。
图 4：JsonConfigurationProvider 的 JSON 配置数据:
```csharp
{
  "AppConfiguration": {
    "MainWindow": {
      "Height": "400",
      "Width": "600",
      "Top": "0",
      "Left": "0"
    },
    "ConnectionString":
      "Server=(localdb)\\\\mssqllocaldb;Database=Database-0B3B0919-C8B3-481C-9833-
      36C21776A565;Trusted_Connection=True;MultipleActiveResultSets=true"
  }
}
```

CommandLineConfigurationProvider 要求您在使用配置生成器注册它时指定自变量。通过名称/值对的字符串数组指定自变量，每一对均采用 `/<name>=<value>` 格式，其中必须有等号。前导斜线也是必须有的，但函数 AddCommandLine(string[] args, Dictionary<string,string> switchMappings) 的第二个参数允许您提供前缀必须为 - 或 -- 的别名。例如，值字典将允许将命令行“program.exe -LogFile="c:\programdata\Application Data\Program.txt”加载到 AppConfiguration:LogFile configuration 元素中：
```csharp
["-DBConnectionString"]="AppConfiguration:ConnectionString",
  ["-LogFile"]="AppConfiguration:LogFile"
```

在完成基本配置之前，您还需要另外注意以下几点：
　　● CommandLineConfigurationProvider 具有多项并非 IntelliSense 原生的特性，您需要多加注意：
　　　　○ CommandLineConfigurationProvider 的 switchMappings 只允许使用开关前缀 - 或 --。即使是斜线 (/)，也不允许用作开关参数。这样可避免您通过开关映射提供斜线开关的别名。
　　　　○ CommandLineConfigurationProvider 不允许使用基于开关的命令行自变量（即，不包含赋值的自变量）。例如，不允许指定密钥“/Maximize”。
　　　　○ 虽然您可以向新的 CommandLineConfigurationProvider 实例传递 Main 的自变量，但如果没有先删除进程名称，则无法传递 Environment.GetCommandLineArgs。（请注意，在附加调试器后，Environment.GetCommandLineArgs 的行为不同。具体而言，如果没有附加调试器，那么含空格的可执行文件名会被分成各个自变量。请参阅 itl.ty\GetCommandLineGotchas。）
　　　　○ 如果您指定的命令行开关前缀 - 或 -- 没有对应的开关映射，则会引发异常。
　　● 虽然配置可以更新 (`Configuration["Profile:UserName"]="Inigo Montoya"`)，但更新后的值不会继续保留回原始存储。例如，当您分配 JSON 提供程序配置值时，就不会更新 JSON 文件。同样，环境变量也不会在其配置项分配时进行更新。
　　● EnvironmentVariableConfigurationProvider 可以视情况允许您指定密钥前缀。在这种情况下，它只会加载具有指定前缀的环境变量。这样一来，您可以自动将配置项限定为环境变量“section”内的配置项，或从更广泛的角度来讲与您的应用程序相关的配置项。
　　● 支持包含冒号分隔符的环境变量。例如，允许在命令行上分配 SET AppConfiguration:ConnectionString=Console。
　　● 所有配置密钥（名称）均不区分大小写。
　　● 每个提供程序均位于各自的 NuGet 包内，其中 NuGet 包名称与提供程序相对应： Microsoft.Extensions.Configuration.CommandLine、Microsoft.Extensions.Configuration.EnvironmentVariables、Microsoft.Extensions.Configuration.Ini、Microsoft.Extensions.Configuration.Json 和 Microsoft.Extensions.Configuration.Xml。

# 3. 了解面向对象的结构
配置 API 的模块性和面向对象的结构均已经过认真研究，提供可检测到且可轻松扩展的模块类和接口方便您使用（见图 5）。
图 5：配置提供程序类模型 ：
![](http://ww3.sinaimg.cn/mw690/c55a7aeejw1f1hcknikwsj20m80njtgs.jpg)
每种类型的配置机制均有对应的配置提供程序类来实现 IConfigurationProvider。对于大多数的内置提供程序实现，实现的快速启动方式是从 ConfigurationBuilder 派生，而不是对所有的接口方法使用自定义实现。无法直接引用图 1 中的任何提供程序，这一点或许有些令人惊讶。这是因为每个提供程序的 NuGet 包内有静态扩展类和 IConfigurationBuilder 扩展方法，不用您手动实例化每个提供程序并使用 ConfigurationBuilder 类的 Add 方法注册提供程序（扩展类的名称通常是由后缀 ConfigurationExtensions 确定）。 借助扩展类，您可以开始直接从 ConfigurationBuilder（可实现 IConfigurationBuilder）访问配置数据，并直接调用与您的提供程序相关联的扩展方法。例如，JasonConfigurationExtensions 类向 IConfigurationBuilder 添加 AddJsonFile 扩展方法，以便您可以通过调用 Configuration­Builder.AddJsonFile(fileName, optional).Build(); 来添加 JSON 配置。
大部分情况下，只要拥有配置，您就拥有了开始检索值所需的一切。

IConfiguration 包含字符串索引器，允许您使用密钥检索任意特定的配置值，从而访问要查找的元素。您可以使用 GetSection 或 GetChildren 方法检索整个设置集（称为 section），具体取决于您是否想深入到层次结构中的另一层级。请注意，配置元素部分允许您检索以下内容：
　　● 密钥：名称的最后一个元素。
　　● 路径：从根指向当前位置的完整名称。
　　● 值：存储在配置设置中的配置值。
　　● 对象形式的值：通过 ConfigurationBinder，您可以检索与要访问的配置部分（可能包括其子部分）相对应的 POCO 对象。这就是图 3 中的示例 Configuration.Get<AppConfiguration>(nameof(App­Configuration)) 的工作原理。
　　● IConfigurationRoot 包含 Reload 函数，可允许您重新加载值，以便更新配置。ConfigurationRoot（可实现 IConfigurationRoot）包含 GetReloadToken 方法，以便您可以针对在重新加载发生（以及值可能发生变化）时发出的通知进行注册。

# 4. 加密的设置
有时，您需要检索加密的设置，而不是存储在开放文本中的设置。例如，当您要存储 OAuth 应用程序密钥或令牌时，或当您要存储数据库连接字符串的凭据时，这一点就非常重要。幸运的是，Microsoft.Extensions.Configuration 系统内置对读取加密值的支持。若要访问安全存储，您需要添加对 Microsoft.Extensions.Configuration.User­Secrets NuGet 包的引用。添加后，您将获得新的 IConfigurationBuilder.AddUserSecrets 扩展方法，用于提取称为“userSecretsId”的配置项字符串自变量（存储在您的 project.json 文件中）。正如您所期望的那样，在向配置生成器添加 UserSecrets 配置后，您便可以开始检索加密值，而这只有与设置相关联的用户才能访问。

显然，如果您无法设定设置，那么检索设置就有点多余。为此，请使用 user-secret.cmd 工具，如下所示：
```csharp
user-secret set <secretName> <value> [--project <projectPath>]
```
借助 --project 选项，您可以将设置与 project.json 文件（默认由 ASP.NET 5 新项目向导创建）中存储的 userSecretsId 值相关联。如果您没有 user-secret 工具，则需要通过开发者命令提示符并使用 DNX 实用工具（当前为 dnu.exe）来添加。

若要详细了解如何使用用户机密配置选项，请参阅 Rick Anderson 和 David Roth 撰写的“应用程序机密的安全存储”一文 ([bit.ly/1mmnG0L](http://bit.ly/1mmnG0L))。

# 5. 总结
接触 .NET 已有一段时间的同仁们可能已对通过 System.Configuration 提供的内置配置支持感到非常失望。如果您之前使用的是经典 ASP.NET，则情况更是如此。在经典 ASP.NET 中，配置仅限于 Web.Config 或 App.config 文件，且只能通过访问其中的 AppSettings 节点。幸运的是，全新的开放源代码 Microsoft.Extensions.Configuration API 在最初版本的基础上实现了很大飞跃，添加了各种新的配置提供程序，以及可方便您连接任意所需的自定义提供程序的可轻松扩展的系统。对于那些仍在使用旧版 ASP.NET 5（苦苦挣扎？）的同仁们，虽然旧版 System.Configuration API 仍可运行，但您可以慢慢开始迁移到（甚至并行运行）新版 API，只需引用新包即可。此外，您还可以在控制台等 Windows 客户端项目和 Windows Presentation Foundation 应用程序中使用 NuGet 包。因此，当您下次需要访问配置数据时，就没有理由不使用 Microsoft.Extensions.Configuration API 了。

原文来自[MSND订阅杂志](https://msdn.microsoft.com/zh-cn/magazine/mt632279)
作者：**Mark Michaelis**
