# NET多版本变迁史，理解跨平台

## .NETCore版本说明

2019.05.06 Build2019 发布时间表 ---  一年一个大版本，两年一个LTS，一个月一个Preview 

LTS：long term support--------36个月  

GA: general availability------- 正式发布的版本 

STS：standard term support---18个月

下载地址： https://dotnet.microsoft.com/zh-cn/download 

![](.\img\企业微信截图_16728191133561.png)



## .NETFramework

 .NET Framework：描述的是上层应用框架，底层就只支持windows平台，BCL CLR都是 只有一个的 

![](.\img\企业微信截图_16728192354179.png)



## .NETCore

 .NET Core：2016年.NETCore1.0，特点是3个CLR共存的—最后个版本是.NET Core3.1— 所以就要统一，然后就是.NET5.0  

![](.\img\企业微信截图_16728192527052.png)



## .Net5/6/7

.NET5：终结3个分支，Unified统一，一个CLR，一个BCL---2020年全球疫情，导 致很多内容跳票—没完成 

![](.\img\v2-b7bb6f99cc9213e959bbe6e3a462ccc1_1440w.jpg)

.NET6：口号也是统一平台，各种该实现的都实现了一下 

![](.\img\u=2568827473,3566241140&fm=253&fmt=auto&app=120&f=PNG.webp)



.NET7：进一步完善统一，没有太大的变化，缝缝补补 

![](.\img\cefc1e178a82b901f768f1f277f2927c3b12efb0.webp)



 三者一脉相承，终结了多CLR现状 



## 理解跨平台

 运行时CLR---CoreCLR 

 能运行在不同的操作系统  

![](.\img\企业微信截图_16728197178850.png)



## .Net7源码

### 源码阅读方法目标以及步骤

#### 阅读源码方法

1. 自上而下，先整体再细节
2. 望文生义，该猜猜该过过
3. 理解设计模式，抓住核心套路

#### 阅读源码目标

1. 梳理完整流程
2. 理解核心对接
3. 手写常用扩展
4. 掌握框架心法

#### 阅读步骤

1. 基本使用：先会用，常见扩展
2. 上帝视角：自上而下，为了效率
3. 核心源码解读：只读核心，该猜猜该过过
4. 扩展和应用：根据理解，写出不一样的扩展和应用

###  启动流程 

程序启动时，执行的一系列操作---初始化配置文件、初始化日志、IOC容器 的初始化、各种IOC注册、MVC相关的IOC注册、Kestrel服务器初始化、HttpPipeline的 初始化 

####  1.创建WebApplicationBuilder

做一些信息读取 

##### WebApplicationBuilder 

1.  这是个GOF23种设计模式里面的---创建型设计模式---建造者模式 

    建造者（Builder）模式的定义： 指将一个复杂对象的构造与它的表示分离，使同样的构建过程可以创建不同的表示。 它是将一个复杂的对象分解为多个简单的对象，然后一步一步构建而成。 它将变与不变相分离，即产品的组成部分是不变的，但每一部分是可以灵活选择的。 

    接地气儿：为了创建复杂对象---对象里面可能还有多个子对象，还有不同步骤---类似 这种复杂构造，就新建一个Builder来完成 

2.  WebApplicationBuilder ， WebHostBuilder ， HostApplicationBuilder 区别

    WebApplicationBuilder是为了WebApplication 

    WebApplicationBuilder里面有个WebHost属性就是配置WebHostBuilder， 还有个Host属性，是配置HostApplicationBuilder 

    三者是不一样的，但是都差不多，其实都是讲的Web程序，只是在多版本的历史债务，大体没有多少区别 

  看源码： 里面也就是做了点初始化东西，包括rootpath、environment、configuration 等信息---还包括一些自定义配置等，都封装进去  

#### 2.配置WebApplicationBuilder

各种初始化配置 ,下面两点配置都是对Builder进行各种配置—各种初始化  

#####  属性配置Builder 

Builder是个复杂工厂，支持各种配置(也有默认实现的)： 当下是简单演示，每个组件要看源码 

builder.Configuration：读取配置文件的帮助类---可以配置新增多个数据源 

​	builder.Configuration.AddJsonFile

​	builder.Configuration.AddXmlFile

builder.Logging：配置下项目的日志组件---常用的是替换成log4net 

​	builder.Logging.AddConsole()

​	builder.Logging.AddLog4Net()

builder.Services是IOC注册

#####  Host配置Builder 

1.  容器替换得基于Host --builder.Host.UseServiceProviderFactory

2.  Kestrel配置得基于Host--- builder.WebHost.ConfigureKestrel 

3.  前面用属性配置的，其实也可以基于Host配置，如Configuration，IOC容器，但不 推荐 

   builder.Host.ConfigureAppConfiguration

   builder.Host.ConfigureLogging

   builder.Host.ConfigureServices

#### 3. IOC注册

各种IOC注册，含MVC 

##### IOC概述

Inversion of Control：控制反转，简称IOC 

Dependency Injection：依赖注入，简称DI 

Dependence Inversion Principle：依赖倒置原则： 面向对象语言程序设计时，高层模块不要依赖于低层模块，二者应该通过抽象来依赖， 简单来说就是依赖抽象，而不是依赖细节！ DIP是IOC的核心理论基础  

##### 理解IOC

依赖细节，会导致低层的变化影响高层 

依赖抽象， 低层的变化就不会影响高层，保持稳定，可扩展 

控制反转IoC是面向对象编程中的一种设计原则，可以用来减低计算机代码之间的耦合度。 

依赖倒置，倒置的什么？--高层不再直接依赖低层，而是依赖于抽象 

控制反转，反转的什么？--高层以前是直接依赖于低层，现在依赖于抽象，把控制权交 给第三方 

##### .netcore 内置IOC使用 

1.  建立抽象+实现，启动时注册 

2.  基于容器获取实例------简单，里面有依赖注入 

3.  容器实例注入和使用----也比较常用 

4.  生命周期验证---实例是容器生成的，所以容器也能管理生命周期---瞬时-单例-作 用域 

5.  多种注册方式 

   builder.Services.AddTransient

   builder.Services.AddSingleton

   builder.Services.AddScoped

   这几个的不同重载

6.  对象集合的获取  ---多次注册，获取集合 

   ServiceProvider.GetServices<ITestServiceA>()

7.  重复注册和替换---replace的方式才能替换掉 

   builder.Services.Replace(ServiceDescriptor.Transient<ITestServiceE, TestServiceEV2>())

#####  IOC基本过程 

1.  注册Register-------获取实例Resolve 

2.  依赖和注入，构造函数注入-属性注入-方法注入 

3.  生命周期管理，各种注册方式，集合注入等，其实都可以编码实现 

    依赖注入：构造A对象时，需要依赖B对象，那么就先构造B对象传入，也就是能在构造对 象时，对象的依赖初始化并注入进去，这种技术手段就叫依赖注入。 

    IOC和DI是密不可分，要实现IOC，就必须有DI  

#####  IOC意义 

1.  面向抽象编程，架构更稳定 
2.  方便扩展，新的实现+注册覆盖 
3.  注入屏蔽细节------使用者需要知道底层细节 
4.  生命周期管理等----不需要再写单例了，直接用容器 
5.  方便AOP扩展 

#####  关于IOC的开发建议 

1.  保持面向抽象编程，有接口有实现，优先构造函数注入 
2.  自上而下每一层皆为IOC，贯穿全部层 
3.  关于静态帮助方法：建议改成IOC式，静态方法一般是写扩展方法---除非是既不需 要传入参数配置，也不需要扩展升级，纯粹的工具 
4.  组件注册统一封装，AddXXXX--- RegisterNET7Service 
5.  Options+委托模式数据获取(后面再讲)  

#####  IOC三层扩展 

 基于IOC是无止境的扩展框架---下面是按照方向-层级，进行一个划分： 

1.  通过IOC注册替换，完成框架扩展 
2.  默认容器不支持属性注入-AOP等，可以扩展去支持 
3.  替换IOC容器  

#####  IOC扩展-1 

 通过IOC注册替换，完成框架扩展 

 这是ASP.NET Core最常用的框架扩展点，几乎无处不在 

 IControllerFactory--DefaultControllerFactory--CustomControllerFactory 

替换DefaultControllerFactory

builder.Services.Replace(ServiceDescriptor.Transient<IControllerFactory, CustomControllerFactory>());

 步骤： 

1.  实现接口 
2.  IOC注册/替换 

 理解： 

成功替换掉了框架的默认实现，换成了我们自己的逻辑---可以写个日志，也可以干别的 呀？----日常开发中，这种扩展是用的特别特别多的  

#####  IOC扩展-2 

 默认容器不支持属性注入-AOP等，可以扩展去支持---ASP.NETCore范围内 

 IControllerActivator--DefaultControllerActivator --CustomControllerActivator 

替换DefaultControllerActivator

builder.Services.Replace(ServiceDescriptor.Transient<IControllerActivator, CustomControllerActivator>());

步骤： 

1.  实现接口 
2.  IOC注册/替换 
3.  AddControllersAsServices() 

 理解： 

1.  这也是标准的IOC注册替换的扩展，添加自定义动作 
2.  这个也扩展了下默认IOC容器的功能---甚至搞个AOP扩展，也不难的----这个只是小 道，满足一些特殊需求  

#####  IOC扩展-3 （替换IOC-Autofac）

替换IOC容器，使用Autofac，这个是最常见的---很熟悉，但是很想懂why Autofac比较灵活，方式很多----AOP扩展方便 

 步骤： 

1.  Nuget添加Autofac.Extensions.DependencyInjection 
2.  builder.Host.UseServiceProviderFactory(new AutofacServiceProviderFactory()); 
3.  autofac式注入---N中注册方式--Module 
4.  Autofac---AOP扩展---面向切面编程 

#####  上帝视角--IOC生命周期 

理解前提：

 IOC容器的2件事儿被拆开设计：注册ServiceCollection 生成ServiceProvider---- ServiceCollection.BuildServiceProvider() 

 生命周期： 

1.  初始化Builder，ConfigBuilder时，只要有ServiceCollection就行；没有的话就保存到委托； 
2.  Build()实例化ServiceCollection，各种默认注册—都是基于默认容器 
3.  Build()基于Factory支持容器实例替换，注册关系会转换过去 
4.  请求响应时，按HttpContext分配IOC容器Scope实例  

#####  各种IOC注册 

 源码查看各种注册，系统的各种IOC注册： 

1.  都是写入ServiceCollection 
2.  没有ServiceCollection就先写入委托，后面执行委托即可 
3.  包括开发者注册 

 都保存在委托里面  

#####  IOC实例化 

 在Build()时，发生了非常多动作---只是ServiceColletion 

 核心在WebHostBuilder. BuildCommonServices 

1.  关于HostStartup 

2.  new ServiceCollection(); 实例化ServiceCollection 

3.  添加各种内置注册，注意默认的IOC提供者： 

   services.AddTransient<IServiceProviderFactory<IServiceCollection>, DefaultServiceProviderFactory>();

4.  _configureServices?.Invoke(_context, services);//前面注册的委托执行，其实 都是在build里面，而且是在默认注册之后  

#####  IOC容器替换 

Build()里面的GetProviderFromFactory(hostingServices);  

则是IOC容器替换的核心环节！--到ServiceProvider 

1.  先获取容器实例，这里一定是默认容器 
2.  通过默认容器，再获取容器实例的工厂！---扩展点就是基于这里实现的 
3.  如果已替换容器工厂，则用工厂创建个新的Provider， 
4.  以后程序里面用的Provider就是这个了---所以替换了factory，就能替换掉容器实 例  

替换容器的源码：

```c#
static IServiceProvider GetProviderFromFactory(IServiceCollection collection)
{
	var provider = collection.BuildServiceProvider();
	var factory = provider.GetService<IServiceProviderFactory<IServiceCollection>>();

	if (factory != null && factory is not DefaultServiceProviderFactory)
	{
		using (provider)
		{
			return factory.CreateServiceProvider(factory.CreateBuilder(collection));
		}
	}

	return provider;
}
```

#####  自制简易IOC容器和替换 

1.  准备给IOC容器， IElevenContainer
2.  准备容器Factory，得实现IServiceProviderFactory--- -两个职责：生成Provider；把注册的映射关系转换到自实现容器； 
3.  准备ZhaoxiContainerBuilder：真正负责完成生成和转化动作 
4.  准备ZhaoxiServiceProvider，其实就是最终提供实例获取的地方，里面包裹上内置 容器，将来获取实例就是从这里走的 
5.  初始化时指定IOC容器工厂 

 工厂---找Builder---Builder完成映射关系转换+生成Provider----Provider是包了一层 的Container 

 Factory+Builder+Provider  

#####  IOC容器总结 

1.  内置IOC容器的使用—各种细节 
2.  IOC/DI的理解 
3.  IOC的三层扩展 
4.  源码解读了IOC初始化—各种内置注册—开发者注册--- 
5.  IOC容器的替换---第三方容器的接入---源码+简易实现 
6.  请求来了，是启动完成后-----每次请求创建一个ScopeServiceProvider,保存到 base.HttpContext.RequestServices就是容器实例，所有的实例获取都基于这个 Provider----就可以实现作用域生命周期---后期再看源码 
7.  IOC容器本身的源码---思路简单，就是分享的手写IOC---内容也多，得太多时间--- 关注的是ASP.NET Core源码，过程中用的组件多了，没法都看 

#### 4. Build()

IOC容器初始化—前面配置生效 

#####  HostingStartup 

 在Build()时，有个HostStartup 

1.  找出各种dll 
2.  检测特性 
3.  反射实例，执行Configure 

 程序启动过程中，IOC容器实例化前，通过反射额外执行了一波逻辑—这就是个扩展点  



 ###### 无侵入式扩展 

1.  项目启动时额外处理逻辑---IWebHostBuilder几乎啥都能干 
2.  扩展CustomHostingStartup，项目内—自动的被执行---优先于默认流程---逼格高 一点，但是通常不用这样装---后期扩展，减少修改可以做---计数统计 
3.  如果能放在项目外----其实是有bug的---可以轻松的修改流程，破坏之前的流程--- 所以默认是不允许的----希望允许呢？设置一个environmentVariables-- "ASPNETCORE_HOSTINGSTARTUPASSEMBLIES":  "Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation;Zhaoxi.AgileFramework.WebCore" 
4.  支持无侵入式扩展----SkyAPM----链路追踪，可以把请求的各种参数/结果/时间都 收集起来做分析---每个服务都需要的，直接通过HostingStartup来扩展的，是要配 置环境变量的 

源码

```c#
// Execute the hosting startup assemblies
foreach (var assemblyName in _options.GetFinalHostingStartupAssemblies())
{
    try
    {
        var assembly = Assembly.Load(new AssemblyName(assemblyName));

        if (!processed.Add(assembly))
        {
            // Already processed, skip it
            continue;
        }

        foreach (var attribute in assembly.GetCustomAttributes<HostingStartupAttribute>())
        {
            var hostingStartup = (IHostingStartup)Activator.CreateInstance(attribute.HostingStartupType)!;
            hostingStartup.Configure(this);
        }
    }
    catch (Exception ex)
    {
        // Capture any errors that happen during startup
        exceptions.Add(new InvalidOperationException($"Startup assembly {assemblyName} failed to execute. See the inner exception for more details.", ex));
    }
}
```



#####  Options模式 

 使用IOC时，对象都可以依赖注入，但数据呢？要传值呢？ 比如DbContext里面需要获取连接字符串，该如何实现？ 

1.  DbContext自己指定连接字符串---那就不是集中配置了，有变动得修改底层 
2.  注入IConfiguration， 然后DbContext自己指定路径去获取？--- 也跟1一样 
3.  上端注入个string字符串？一是麻烦，二是如果有后期修改呢？ 

 标准用法：Options模式，跟IOC密切相关的 

######  Options使用过程 

 Options是为了用DI模式，完成数据的初始化和传递 

1.  封装实体对象，用来保存数据，无参数构造函数 
2.  向IOC注册数据(程序/配置文件) 
3.  容器DI获取使用，IOptions  

######  Options多种注册方式 

1.  Configure 
2.  ConfigureAll 
3.  PostConfigure 
4.  PostConfigureAll 
5.  AddOptions 

 注册分默认注册是名字为null，以及带名字注册 Post就是后面执行的 

 名字为null的会覆盖所有同类型的options

  // Null name is used to configure all named options.

```c#
/// <summary>
/// Invokes the registered configure <see cref="Action"/> if the <paramref name="name"/> matches.
/// </summary>
/// <param name="name">The name of the options instance being configured.</param>
/// <param name="options">The options instance to configure.</param>
public virtual void Configure(string? name, TOptions options)
{
    ThrowHelper.ThrowIfNull(options);

    // Null name is used to configure all named options.
    if (Name == null || name == Name)
    {
        Action?.Invoke(options);
    }
}
```



######  Options三种获取方式 

1.  IOptions：只能获取默认名字， 
2.  IOptionsMonitor：既可以默认名字，也可以带别名 
3.  IOptionsSnapshot ：既可以默认名字，也可以带别名 

######  Options上帝视角 

 对照前面Options的使用，其实大部分东西都能猜测了！ 

1.  启动流程中，AddOptions完成各种相关注册 
2.  基于委托完成数据注册，获取数据时执行委托 
3.  IOptions、IOptionsMonitor、IOptionsSnapshot数据的生命周期不一样，有缓存 
4.  源码也挺简单的---也没必要去扩展Options了  

######  Options源码核心 

 Options源码在OptionsServiceCollectionExtensions，源自runtime类库—不能直接调 试---提供了最新的下载 https://github.com/dotnet/runtime 

 路径：runtime-main\src\libraries\ externals.csproj 

注意不要路径太深----还有很多环节的源码，都在这里看的 

1.  IOC注册环节：Build方法的BuilderCommon—就是5个注册 
2.  实例生成，各种注入理解—不同的生命周期 
3.  下一步是执行数据注册---然后就是注入实例  

```c#
/// <summary>
/// Adds services required for using options.
/// </summary>
/// <param name="services">The <see cref="IServiceCollection"/> to add the services to.</param>
/// <returns>The <see cref="IServiceCollection"/> so that additional calls can be chained.</returns>
public static IServiceCollection AddOptions(this IServiceCollection services)
{
    ThrowHelper.ThrowIfNull(services);

    services.TryAdd(ServiceDescriptor.Singleton(typeof(IOptions<>), typeof(UnnamedOptionsManager<>)));
    services.TryAdd(ServiceDescriptor.Scoped(typeof(IOptionsSnapshot<>), typeof(OptionsManager<>)));
    services.TryAdd(ServiceDescriptor.Singleton(typeof(IOptionsMonitor<>), typeof(OptionsMonitor<>)));
    services.TryAdd(ServiceDescriptor.Transient(typeof(IOptionsFactory<>), typeof(OptionsFactory<>)));
    services.TryAdd(ServiceDescriptor.Singleton(typeof(IOptionsMonitorCache<>), typeof(OptionsCache<>)));
    return services;
}
```



######  IOptions源码解析 

 IOptions源码其实最简单，步骤详解如下：---读一次，一直用 

1.  先注册到UnnamedOptionsManager---注入生成就是UnnamedOptionsManager，----注 入一个IOptionsFactory的实例 

2.  Value在Get时，第一次是通过factory获取，并保存---后续就复用这个值--- 

3.  UnnamedOptionsManager是单例生命周期---所以只有一个实例，所以这个Value就是 缓存------这里是泛型的，其实就是泛型缓存 

   ```c#
   internal sealed class UnnamedOptionsManager<[DynamicallyAccessedMembers(Options.DynamicallyAccessedMembers)] TOptions> :
           IOptions<TOptions>
           where TOptions : class
       {
           private readonly IOptionsFactory<TOptions> _factory;
           private volatile object? _syncObj;
           private volatile TOptions? _value;
   
           public UnnamedOptionsManager(IOptionsFactory<TOptions> factory) => _factory = factory;
   
           public TOptions Value
           {
               get
               {
                   if (_value is TOptions value)
                   {
                       return value;
                   }
   
                   lock (_syncObj ?? Interlocked.CompareExchange(ref _syncObj, new object(), null) ?? _syncObj)
                   {
                       return _value ??= _factory.Create(Options.DefaultName);
                   }
               }
           }
       }
   ```

   

 Factory 

1.  生成时，会注入IConfigureOptions集合， IPostConfigureOptions集合 
2.  Create方法，其实就是创建空Options实例，然后遍历执行IConfigureOptions，再 遍历执行IPostConfigureOptions，完成数据的生成 

Configure方法就是去IOC注册，IConfigureOptions注册给new  

ConfigureNamedOptions(name, configureOptions)-----多次注册---然后注 入给Factory  

```c#
/// <summary>
/// Returns a configured <typeparamref name="TOptions"/> instance with the given <paramref name="name"/>.
/// </summary>
/// <param name="name">The name of the <typeparamref name="TOptions"/> instance to create.</param>
/// <returns>The created <typeparamref name="TOptions"/> instance with the given <paramref name="name"/>.</returns>
/// <exception cref="OptionsValidationException">One or more <see cref="IValidateOptions{TOptions}"/> return failed <see cref="ValidateOptionsResult"/> when validating the <typeparamref name="TOptions"/> instance been created.</exception>
/// <exception cref="MissingMethodException">The <typeparamref name="TOptions"/> does not have a public parameterless constructor or <typeparamref name="TOptions"/> is <see langword="abstract"/>.</exception>
public TOptions Create(string name)
{
    TOptions options = CreateInstance(name);
    foreach (IConfigureOptions<TOptions> setup in _setups)
    {
        if (setup is IConfigureNamedOptions<TOptions> namedSetup)
        {
            namedSetup.Configure(name, options);
        }
        else if (name == Options.DefaultName)
        {
            setup.Configure(options);
        }
    }
    foreach (IPostConfigureOptions<TOptions> post in _postConfigures)
    {
        post.PostConfigure(name, options);
    }

    if (_validations.Length > 0)
    {
        var failures = new List<string>();
        foreach (IValidateOptions<TOptions> validate in _validations)
        {
            ValidateOptionsResult result = validate.Validate(name, options);
            if (result is not null && result.Failed)
            {
                failures.AddRange(result.Failures);
            }
        }
        if (failures.Count > 0)
        {
            throw new OptionsValidationException(name, typeof(TOptions), failures);
        }
    }

    return options;
}
```



######  IOptionsSnapshot和IOptionsMonitor 

 IOptionsMonitor----一次读取，重复使用，有变更会自动更新 

1.  CurrentValue就是DefaultName---Get时是通过缓存GetOrAdd—通过factory读取 
2.  也是Singleton，也是一直缓存？---在构造函数加了个InvokeChanged，负责在数据 变化后，触发动作，清理缓存，把新数据读进去，下次直接用 

 IOptionsSnapshot----请求内缓存重用，不同请求分别读取 

1.  Value就是DefaultName---Get时是通过缓存GetOrAdd---通过factory读取 
2.  Scope生命周期----一次请求内，重复读取，是用的缓存；不同次请求，数据是重新 存的；  

######  Options使用建议 

1.  简单粗暴不更新，就IOptions----其实用的最多 
2.  需要更新IOptionsMonitor-------实时支持更新 
3.  IOptionsSnapshot(除非单次请求内要求保证不变，新的请求用新的数据)---请求处 理过程中，可以去Configure配置，新的请求就能用上最新的 

 总结下：Options使用细节多，源码还算简单，建议大家好好看看  

#####  Logger组件 

 日志是项目基础组件了，框架直接内置，不过也基本上都会扩展的 

1.  Log控制器，基本使用，ILogger和ILoggerFactory 
2.  Log日志过滤简单使用 
3.  log4net组件扩展(log4net+kafka+ELK)  

######  Logger上帝视角 

 Logger就是写日志---一个是日志级别；另外一个是存储介质；再一个是过滤； 

1.  启动时注册，AddLogging---里面会有LogFactory，ILogger 
2.  支持多种日志有序输出，靠的是Provider，会有多个Provider 
3.  支持Filter日志过滤 

 要扩展日志，一般就是扩展那个存储介质， 所以就不换Factory了，增加个Provider，实现自己的日志介质即可  

######  Logger源码探究 

 还是从Build()方法的BuildCommon的AddLogging开始！ 

1.  AddLogging注册了Logger和LoggerFactory，都是单例 
2.  生成：其实是通过LoggerFactory，支持注入Provider，也可以直接AddProvider 
3.  Factory去CreateLogger，一是缓存，二是把provider集合都传过去了 
4.  那还Logger里面就遍历Provider提供的Logger，然后一一执行日志输出 
5.  写日志是LoggerExtensions，执行的就是Logger.log，不同级别就是个参数 
6.  关于Filter，在Add时是配置Option，然后再Factroy注入—RefreshFilters变成通用， 保存在logger缓存，传递给Logger，使用时就过滤一下 

 要扩展日志，就是实现个Provider---Logger---注册进去就行 更细节，关心的，就自己去仔细看看  

######  自定义Logger 

 写一个自定义日志介质的方式，日志信息我自己写个处理方式 

1.  LoggerProvider对接Factory 
2.  ConsoleLogger实现接口ILogger 
3.  将LoggerProvider提供给Factory/Builder 

 这里就是典型的Factory--Provider--Worker  

######  标准组件封装 

1.  封装的组件，不可能是一成不变，需要支持上端的配置---Options式数据传递初始 化---CustomConsoleLoggerOptions 
2.  封装Add式组件注册---CustomConsoleLoggerExtensions----方便注册，隐藏注册细 节 
3.  委托式初始化---也简单， 

 以上就是标准的组件封装，几个元素都是需要的  

#####  Configuration 

######  Configuration基本操作-1 

1.  基本的路径读取---appsettings.json 
2.  Bind和Get---对象获取 
3.  命令行参数支持---框架默认添加，三种格式，---以命令行为准  

######  Configuration基本操作-2 

1.  多配置文件支持---json-xml-ini 
2.  内存数据提供---memory模式 
3.  支持多配置文件---所有的配置文件源都是有效的----但是同一个key，后面的覆盖 前面的-----命令行是最后注册

######  Configuration上帝视角 

   Configuration其实就是包装了一下对文件读取----也支持了下其他模式--- 

1.  程序启动时，初始化ConfigurationBuilder，支持添加多种配置文件的Provider 
2.  ConfigurationBuilder在Build时，完成数据的初始化，放入内存 
3.  获取的时候，Provider倒序遍历获取，第一个有效—所以后注册的有效 

 配置文件也是增加个Provider，完成扩展  

######  Configuration源码探究 

1.  初始化环节，Build—BuildCommon---new ConfigurationBuilder,Inovke执行的一 系列扩展配置 
2.  ConfigurationBuilder.Build时---遍历全部的IConfigurationSource，其实就是 AddCommand,AddJson,AddXML;---执行Build获得Provider，全部Provider传给 ConfigurationRoot----构造时遍历provider，执行Load，完成数据加载，放入到内存里 
3.  AddCommandLine源码--- CommandLineConfigurationExtensions给 ConfigurationBuilder添加CommandLineConfigurationSource----Build时执行Build得到 CommandLineConfigurationProvider---触发Load加载全部数据，保存给父类的Data 
4.  数据如何读取？--就是从ConfigurationRoot用索引获取--- GetConfiguration倒序遍 历provider去匹配key，第一个就返回------- SetConfiguration就是遍历provider，都设置 这个值 

 其实直接对着ConfigurationRoot就可以set

######  Configuration扩展 

1.  CustomConfigurationProvider----提供数据增删改查的 
2.  CustomConfigurationSource------生成CustomConfigurationProvider的 
3.  CustomConfigurationExtensions--方便注册 
4.  CustomConfigurationOptions-----额外增加的方便配置  

#####  ChangeToken 

Options和Configuration都关联到一个ChangeToken 

Configuration是如何支持的reloadOnChange？  

其实这只是组件的一个具体知识点—其实现，还是有点内容的 类似这种东西，实在太多了，后面真的没办法一一都讲到  

######  ChangeToken上帝视角 

 AddJsonFile—reloadOnChange，如果文件更新，程序的数据就更新—其实在Build时数 据都已经加载到内存---肯定是文件更新，触发了事件，完成了内存数据的刷新 

 靠的就是ChangeToken.OnChange，到底是发生了什么事儿？ 

1.  ChangeToken是啥？其实是个生产者消费者的封装，把A注册给B，B发生了，就触发 A-------- ChangeToken源码 
2.  配置文件的自动刷新，需要的就是文件监听+刷新数据，关联注册起来--- Configuration源码 
3.  文件是如何监听的？源自于System.IO的封装-----FileProvider源码 

 这其实是3个方向，需分别解读  

######  ChangeToken源码 

 先理解ChangeToken，直接看源码 

1.  ChangeToken.Onchange的2个参数分别是生产者Func 消费者Action 
2.  ChangeTokenRegistration---构造函数，先得到Token，注册回调---回调里面再注册 Token和回调(递归注册，就能持续监听和触发) 
3.  如果要看是哪个ChangeToken---这次看的就是配置文件监听时触发的—待会儿看 

 总结下： ChangeToken.Onchange是一个 生产者—消费者 关联关系的封装，支持持续生 产+消费，不是一次性----就像消息队列(不准确)，业务无关 所以不仅是可以为配置文件服务，为各种东西都可以服务，将来路由Route里面还有这个  

######  测试OnChange 

 测试下配置文件的Onchange监听的效果，基于PhysicalFileProvider做监听 

1.  PhysicalFileProvider可以完成监听---而且支持绑定消费动作—只有一次效果 

2.  OnChange就是调用Token的绑定动作，只是在触发后，会再次绑定---会持续生效 

3.  猜测下理解配置文件更新----配置文件就是初始化时，通过Onchange给 FileChangeToken绑定个更新动作---文件改动-触发Watch-触发配置文件Load-再注 册---再生效  

   ```c#
   ////生效一次
   //Task.Run(() =>
   //{
   //    string path = "D:\\zhaoxi\\Architect03\\20230117Architect03Course006.NET7Advanced-6";
   
   //    PhysicalFileProvider phyFileProvider = new PhysicalFileProvider(path);
   //    var changeToken = phyFileProvider.Watch("*.*");//监听，返回的就是IChangeToken
   //    changeToken.RegisterChangeCallback(
   //        o => Console.WriteLine($"&&&&&&&&&&&&&&{path}文件有更新&&&&&&&&&&&&&&"),
   //        new object());
   //    Console.Read();
   //});
   
   //持续生效
   Task.Run(() =>
            {
                string path = "D:\\zhaoxi\\Architect03\\20230117Architect03Course006.NET7Advanced-6";
   
                PhysicalFileProvider phyFileProvider = new PhysicalFileProvider(path);
                ChangeToken.OnChange(
                    changeTokenProducer: () => phyFileProvider.Watch("*.*"),
                    changeTokenConsumer: () => Console.WriteLine($"&&&&&&&&&&&&&&{path}文件有更新&&&&&&&&&&&&&&")
                );//Producer发生了，Consumer就会消费---消费完会再注册生产消费关系
                Console.Read();
            });
   ```

   

######  PhysicalFileProvider研究 

 配置文件用的就是PhysicalFileProvider----然后看PhysicalFilesWatcher---这里是基 于System.IO下的基础类库的事件来完成的 

 结果： 

 通过基础类库完成对文件的监听----任何动作，都转成IChangeToken---经过 ChangeToken.Onchange绑定到Configuration的更新动作，然后一切就都ok了 

 其实IChangeToken--ChangeToken.Onchange都是业务无关，是抽象出来的 

######  配置文件更新源码 

 有了前面的基础，这会儿再看看配置文件是如何支持reloadOnChange的 

1.  从AddJson开始，返回个委托，包含一些配置，reloadOnChange-provider 

2.  然后就是JsonConfigurationSource，里面的Build就2个：1 EnsureDefaults—指定 FilProvider为PhysicalFileProvider----2 new一个JsonConfigurationProvider()— 核心在父类FileConfigurationProvider构造函数里 

3.  FileConfigurationProvider构造函数里面，通过ChangeToken.OnChange来注册，生产者 是文件监听---消费者是Load加载配置文件数据方法 

   ```c#
   public FileConfigurationProvider(FileConfigurationSource source)
   {
       ThrowHelper.ThrowIfNull(source);
   
       Source = source;
   
       if (Source.ReloadOnChange && Source.FileProvider != null)
       {
           _changeTokenRegistration = ChangeToken.OnChange(
               () => Source.FileProvider.Watch(Source.Path!),
               () =>
               {
                   Thread.Sleep(Source.ReloadDelay);
                   Load(reload: true);
               });
       }
   }
   ```

#####  总结Configuration

1.  支持各种数据源共同存在，后注册覆盖前面---程序启动时加载，保存在内存中，有 数据变化支持更新---支持Provider式扩展 
2.  ChangeToken只是个生产者消费者绑定的封装，是业务无关—将来在路由环节还会有 ----这里提个小练习：去看看之前OptionsMonitor里面是如何用的 

#### 5. Use中间件

管道模型配置

#####  理解Http请求流程 

Http请求响应流程，也就是浏览器输入个地址，发生了什么事儿： 

1.  浏览器输入地址，www.zhaoxiedu.net 
2.  DNS解析，找到IP+Port，然后浏览器向该地址发Http报文---纯文本 
3.  Nginx/IIS/Kestrel监听端口，收到报文解析得到HttpContext 
4.  将请求转发到业务代码处理---怎么进入到controller+action？ 
5.  处理结果由服务器回发到客户端，浏览器解析报文完成渲染 

 所谓HttpPipeline就是程序如何处理请求的全过程--- 

 理解Http管道所处的位置：Web服务器解析报文之后，在Web服务器回发报文之前 

 Http管道和controller-action的关系：其实是包含关系，任何处理动作都是管道的一部 分  

#####  理解Http管道 

HttpPipeline本质是个啥？ 

接受HttpContext，然后做一系列的处理(Cookie Session 鉴权授权 缓存 路由 MV-C)---最终将结果保存在HttpContext的Response里面 然后在ASP.NET Core里面，就被抽象成为一个委托—RequestDelegate---接受一个 HttpContext，然后执行一系列操作 

 但是开发中，管道模型是很复杂的呀？ 因为Http请求的处理并不简单，包含了很多环节---Cookie/Session/鉴权授权/缓存 /https/静态文件-------还有就是各种不同的业务处理需求(M-V-C)-----还有开发者的 扩展需求(限流—黑名单白名单)-----所以需要一套扩展性的框架 

 这套实现，就是ASP.NET Core的HttpPipeline  

#####  Middleware原生演示 

管道模型是委托RequestDelegate---它的构建是需要很多东西，所以在框架里面又来了 一个ApplicationBuilder 然后各种Use来配置，最后来个Build得到管道模型。 

 然后中间件Middleware就是其中Use的参数，用来配置管道模型---是个 Func 

1.  定义个基础的中间件 
2.  Use一下 
3.  多来几个嵌套 
4.  演示下 
5.  结果分析 

![](.\img\企业微信截图_16762718174920.png)

#####  管道模型上帝视角 

1.  先创建个WebApplicationBuilder()---然后各种配置IOC/Logging/Configuration-- -然后Build()完成了各种初始化---得到WebApplication 
2.  WebApplication去Use添加添加中间件(委托)---实际上是ApplicationBuilder在 Use------就是把中间件(Func委托)保存到一个集合 
3.  最后框架在Run的时候，会执行ApplicationBuilder的Build方法，将委托集合遍历 调用，组装成最终的RequestDelegate，也就是Http管道  

#####  源码阅读

######  梳理执行时间点 

 其他各种初始化，IOC注册等，在WebApplicationBuilder.Build()得到 WebApplication(这时候中间件还没出场)----然后是Run之前会BuildApplication-- -IApplicationFactory—得到ApplicationBuilder 

1.  IOC获取全部的StartupFilter---然后把中间件配置动作包装成一个委托，然 后倒序遍历StartupFilter，分别执行Configure方法，得到一个委托 
2.  最后执行这个委托，等同于先执行Filter里面的配置，再执行默认Use中间件 配置 
3.  然后就Build 

 结论：这里就调用了各种Use，且在Use配置前面，又留下了一个扩展点 

```c#
var builderFactory = _applicationServices.GetRequiredService<IApplicationBuilderFactory>();
var builder = builderFactory.CreateBuilder(Server.Features);
builder.ApplicationServices = _applicationServices;

var startupFilters = _applicationServices.GetService<IEnumerable<IStartupFilter>>();
Action<IApplicationBuilder> configure = _startup!.Configure;
if (startupFilters != null)
{
    foreach (var filter in startupFilters.Reverse())
    {
        configure = filter.Configure(configure);
    }
}

configure(builder);
```

 ApplicationBuilder的Use方法，就是把**中间件保存到_components对象**  ApplicationBuilder的Build方法，**就是倒序遍历components**，执行外层Func委托，最终 得到其实就是RequestDelegate---这就是管道模型 

这个RequestDelegate里面是嵌套了一系列的委托，请求来了会执行一堆动作 

这真的是一个很cool的设计， 只是利用了委托的嵌套 双层委托 

Use时是2层委托---Build就执行外层 所以流程就可以随意组装-随意扩展  

```c#
public IApplicationBuilder Use(Func<RequestDelegate, RequestDelegate> middleware)
{
    _components.Add(middleware);
    return this;
}
```

```c#
public RequestDelegate Build()
{
    RequestDelegate app = context =>
    {
        // If we reach the end of the pipeline, but we have an endpoint, then something unexpected has happened.
        // This could happen if user code sets an endpoint, but they forgot to add the UseEndpoint middleware.
        var endpoint = context.GetEndpoint();
        var endpointRequestDelegate = endpoint?.RequestDelegate;
        if (endpointRequestDelegate != null)
        {
            var message =
                $"The request reached the end of the pipeline without executing the endpoint: '{endpoint!.DisplayName}'. " +
                $"Please register the EndpointMiddleware using '{nameof(IApplicationBuilder)}.UseEndpoints(...)' if using " +
                $"routing.";
            throw new InvalidOperationException(message);
        }

        context.Response.StatusCode = StatusCodes.Status404NotFound;
        return Task.CompletedTask;
    };

    for (var c = _components.Count - 1; c >= 0; c--)
    {
        app = _components[c](app);
    }

    return app;
}
```

#####  HttpPipeline设计 

 后面还有各种封装，各种中间件的用法，但其实都是在基础Use上封装的， 理解下ASP.NET Core HttpPipeline的设计？！ 

1.  多层委托嵌套组装，最终呈现为一个委托 
2.  达到俄罗斯套娃效果，方便扩展 

 设计真的挺coool---无限制扩展----实现也特简单----再追溯下历史  

![](.\img\企业微信截图_16764285712894.png)

#####  ASP.NET/MVC HttpPipeline 

 ASP.NET/MVC时代，Http管道是如何设计的，如何满足扩展性？ 

 那里面也有一个Application对象---请求的全部环节都是它处理---考虑到开发者的扩展， 提供了19个事件，外部可以通过+=来扩展动作-----请求处理时，按顺序去触发事件 这样框架就具备了扩展性--- 

 要扩展动作，就是通过HttpModule(跟中间件很像)  

![](.\img\企业微信截图_1676428661300.png)

![](.\img\20200215124328132.png)

#####  全家桶 VS 自选 

 ASP.NET和ASP.NET MVC是一套管道，属于全家桶----管道的固定动作都写死了，比如一 定检测/生成Session----要也得要，不要也得要 

 ASP.NET Core管道---完全自选式，没有任何的事先写死(只有那个404)---需要任何东西， 都是自行配置—要什么组装什么---Pay for what you use 

 所以： ASP.NET Core的性能会更高 对使用者的要求也更高  

#####  IStartupFilter扩展 

1.  执行在Use之前，可以直接组装中间件 
2.  扩展CustomStartupFilter 
3.  注册IOC容器 
4.  可自行尝试下，放在前后的不同 

 StartupFilter：管道外面包一层(可前可后)---黑白名单/反爬虫/缓存/限流  

 HostingStartup：更早执行，支持更多配置----无侵入式扩展 

 不仅中间件前后可以随意扩展---这还增加了不同的扩展中间件的方式  

#####  Middleware扩展演示 

 除了最原生app.Use，还有多种扩展使用方式： 

1.  app.Run 
2.  app.Use---无Next 
3.  app.Use其他 
4.  app.UseWhen 
5.  app.Map 
6.  app.MapWhe 

 总结下： 无非就是在委托加一层包装，再就是加一层判断，最终还是靠ApplIcationBuider的Use 那套东西  

#####  UseMiddleware 

  app.UseMiddleware<TMiddleWare>() 

 这个跟前面的稍微有点区别，分4种情况： 

 FirstMiddleware：构造函数带个委托+Invoke/invokeAsync(只能有一个)方法即可，然 后直app.UseMiddleware()： 

 SecondMiddleware：实现IMiddleware接口，需要IOC注册 

 SecondMiddleware+ SecondMiddlewareFactory：是在2的情况下，再做工厂替换 builder.Services.Replace(ServiceDescriptor.Singleton()); ThirdMiddleware：中间件想做注入  

###### Middleware类源码 

 UseMiddleware会检测类型是否实现接口，然后走不通的分支 普通Middleware类，没有实现IMiddleware接口，是怎么调用的？ 

```c#
public static IApplicationBuilder UseMiddleware(
        this IApplicationBuilder app,
        [DynamicallyAccessedMembers(MiddlewareAccessibility)] Type middleware,
        params object?[] args)
{
    if (typeof(IMiddleware).IsAssignableFrom(middleware))
    {
        // IMiddleware doesn't support passing args directly since it's
        // activated from the container
        if (args.Length > 0)
        {
            throw new NotSupportedException(Resources.FormatException_UseMiddlewareExplicitArgumentsNotSupported(typeof(IMiddleware)));
        }

        return UseMiddlewareInterface(app, middleware);
    }
```

 其实就是各种检测---然后各种硬编码赋值构造函数，最终还是调用的app.Use 非常多的小细节，可以自己去细看 

1.  没有接口约束，所以各种检查校验 
2.  对象初始化靠反射，但是构造参数支持依赖注入 
3.  构造函数支持传入参数---第一个参数是Next 
4.  方法名字Invoke---InvokeAsync  

###### IMiddleware源码 

 实现IMiddleware接口，有啥区别？再看源码细节 

1.  实现IMiddleware接口---需要注册IOC---每次请求来了都要重新生成，而且可以支 持资源释放，可以写一个Dispose方法 
2.  依赖IMiddlewareFactory，默认是MiddlewareFactory，可以扩展，注册---然后会 自动释放资源  

```c#
private static IApplicationBuilder UseMiddlewareInterface(
        IApplicationBuilder app,
        Type middlewareType)
{
    return app.Use(next =>
                   {
                       return async context =>
                       {
                           var middlewareFactory = (IMiddlewareFactory?)context.RequestServices.GetService(typeof(IMiddlewareFactory));
                           if (middlewareFactory == null)
                           {
                               // No middleware factory
                               throw new InvalidOperationException(Resources.FormatException_UseMiddlewareNoMiddlewareFactory(typeof(IMiddlewareFactory)));
                           }

                           var middleware = middlewareFactory.Create(middlewareType);
                           if (middleware == null)
                           {
                               // The factory returned null, it's a broken implementation
                               throw new InvalidOperationException(Resources.FormatException_UseMiddlewareUnableToCreateMiddleware(middlewareFactory.GetType(), middlewareType));
                           }

                           try
                           {
                               await middleware.InvokeAsync(context, next);
                           }
                           finally
                           {
                               middlewareFactory.Release(middleware);
                           }
                       };
                   });
}
```

######  对比总结 

 用接口实现时，源码都简单多了---为啥要那个不实现接口的呢？---猜测是历史包袱 

1.  普通Middleware类是启动时，在组装管道模型时实例化，嵌套委托常驻内存-------- -初始化支持传参数 
2.  接口实现Middleware类是响应请求的时候才实例化，每次请求来了才实例化，支持主动释放---------默认实例化不能传参数 

 更多时候，用的是第1种----只要中间件自身不考虑资源释放  

 ###### 标准中间件封装 

 中间件的源码层面，学得很透彻了----下面是应用封装层面 

 框架有各式各样的中间件，当下基本原理已经清楚了，下面也按照标准模式来一套封装： 

1.  基本结构认知：UseMiddleware + AddIOC 
2.  UseMiddleware封装一套 
3.  AddIOC封装一套 
4.  Options 

 需求：写一个浏览器过滤中间件----如果是Chrome就正常响应，否则返回Refuse  

######  Middleware封装 

 Middleware是注册到Http请求流程： 

1.  中间件 
2.  Extensions封装中间件注册 
3.  Options  

 ###### 组件依赖封装 

 Middleware是注册流程，组件依赖靠的IOC封装 

1.  抽象+具体实现来一套 
2.  Options传值安排 
3.  Extensions封装IOC注册 

######  常见Middleware扩展 

 结合下大家的网站开发经历，Middleware适合干啥？ 

1.  全部请求都要执行的-----日志/性能监控/跨域/压缩/前端缓存---为所有的请求添 加通用操作 
2.  包裹在外层做请求拦截---黑白名单/反爬虫/限流/链路追踪---可装配可拆卸的这种 操作 
3.  应对特殊条件的请求-----UseWhen/MapWhen---Robot/RSS/防盗链/静态文件中间件- ---只针对某种请求 

######  Stream读取问题

 Http的响应应该是先Header然后再Body---写完之后不能随意改动---是框架限制的---- 因为请求响应已完成，里面的Content-Length已固定，所以不能改-----所以，中间件的 扩展其实是有限制的，不能随意读取和修改请求内容，也不能随意读取和修改响应内容 ---那怎么改？---OnStarting可以修改Header--- 但是我想读写Request和Response—就需要特殊的处理办法， StreamReadMiddleware  

1.  请求体读取问题 
2.  请求体读取问题 

![](.\img\企业微信截图_16764317242710.png)

######  Stream写入问题 

 StreamWriteMiddleware 

1.  请求体写入问题 
2.  响应体写入问题 

 请求的信息和响应的信息， 绕弯一下，是可以读取---也可以修改的 但是，请慎重  

######  Header读写 

 HeaderReadWriteMiddleware 中间件扩展时操作  

```c#
context.Response.OnStarting(async state =>
            {
                var httpContext = (HttpContext)state;
                httpContext.Response.Headers.Add("middlewareStarting", "HeaderReadWriteMiddleware12345");
                //await httpContext.Response.WriteAsync("This is Eleven"); //写不进去，带上中间件，写入时才发生的
                await Task.CompletedTask;
            }, context);

            context.Response.OnCompleted(async state =>
            {
                var httpContext = (HttpContext)state;
                Console.WriteLine($"请求结果StatusCode={httpContext.Response.StatusCode}");
                httpContext.Response.Headers.Add("middlewareComplated", "HeaderReadWriteMiddleware12345222");//写不进去了，不生效
                //主要是用来做资源释放，或者通知
                await Task.CompletedTask;
            }, context);
```

######  页面静态化架构 

 StaticPageMiddleware---中间件扩展时操作 

1.  可以在请求处理过程中，通过中间件扩展，能读取到请求的响应 
2.  那就可以把响应的html文件保存在本地硬盘---就等同于缓存 
3.  下次请求时，可以直接读取这个文件，不用去访问程序+数据库 
4.  这个被称之为页面静态化架构 

 这算是个Stream读取写入的典型应用  

#### 6. Run()

启动监听—Kestrel  

#####  理解Web服务器 

Http请求响应流程： 

1.  浏览器输入地址---DNS解析---IP:Port---发送请求报文，其实就是固定格式的字符串 
2.  Nginx/IIS/Kestrel监听端口---解析Http请求报文，得到HttpContext 
3.  将HttpContext交给HttpPipeline执行，把各种信息写入response对象---服务器再把内 容转成响应报文，回发浏览器 
4.  浏览器接受响应报文，解析，渲染  

![](.\img\企业微信截图_16774669551925.png)

![](.\img\企业微信截图_16774669718835.png)

#####  理解Kestrel服务器 

1.  Kestrel是开源的事件驱动的异步I / O服务器，.NET Core程序，开源地址： https://github.com/aspnet/KestrelHttpServer 

2.  IIS不能跨平台的，而Kestrel是跨平台---.NET Core能跨平台，一是运行时环境能 支持，再就是Web服务器能跨平台，以前是Mono+Jexus 

3.  Web服务器跨平台最核心就是异步IO实现，这里用的是libuv 

    libuv is a multi-platform support library with a focus on asynchronous I/O.  It was primarily developed for use by Node.js---Kestrel  

![](.\img\企业微信截图_16774669551925.png)

#####  Kestrel使用建议 

 Comparing Kestrel Web Server vs IIS----MATT WATSON IIS does almost everything. Kestrel does as little as possible. Because of  this, Kestrel is much faster but also lacks a lot of functionality. I would  think of Kestrel as really more of an application server. It is recommended  to use another web server in front of it for public web applications. Kestrel  is designed to run ASP.NET as fast as possible. It relies on a full fledged  web server to do deal with things like security, management, etc. 

 https://stackify.com/kestrel-web-server-asp-net-core-kestrel-vs-iis/  

![](.\img\企业微信截图_16774671587059.png)

#####  ASP.NET Core部署 

1.  ASP.NET Core脚本启动----裸Kestrel 
2.  IIS+ASP.NET Core部署----编译目录不行，需要发布一下，且IIS要安装ASP.NET  Core Module v2------编译和发布就差个web.config 
3.  Nginx部署 

 推荐： 如果不对外，直接Kestrel，足够快 如果对外，Windows就IIS,一套技术栈 是Linux自然是Nginx 需要负载均衡，推荐Nginx  

#####  Kestrel配置 

 WebApplication.CreateBuilder(args);默认启用Kestrel 

1.  可以通过builder.WebHost.ConfigureKestrel配置KestrelServerOptions 
2.  通过配置文件完成配置 

 通常就是用的默认配置 https://learn.microsoft.com/zhcn/aspnet/core/fundamentals/servers/kestrel/options?view=aspnetcore-7.0 

#####  Kestrel源码-上帝视角 

 Kestrel源码，这里分为2层来解析，含Kestrel启动过程和Kestrel工作过程 

 启动过程(asp.net core如何初始化+启动的)上帝视角： 

1.  builder.Build()时，完成Kestrel服务注入IServer, KestrelServerImpl 
2.  然后组装Http管道，然后app.Run()时，将Http管道(委托)交付给Kestrel 
3.  Ketrel根据配置的监听信息，启动线程死循环监听，有请求再交给Http管道处理 

```c#
public static IWebHostBuilder UseKestrel(this IWebHostBuilder hostBuilder)
{
    hostBuilder.UseQuic(options =>
                        {
                            // Configure server defaults to match client defaults.
                            // https://github.com/dotnet/runtime/blob/a5f3676cc71e176084f0f7f1f6beeecd86fbeafc/src/libraries/System.Net.Http/src/System/Net/Http/SocketsHttpHandler/ConnectHelper.cs#L118-L119
                            options.DefaultStreamErrorCode = (long)Http3ErrorCode.RequestCancelled;
                            options.DefaultCloseErrorCode = (long)Http3ErrorCode.NoError;
                        });

    return hostBuilder.ConfigureServices(services =>
                                         {
                                             // Don't override an already-configured transport
                                             services.TryAddSingleton<IConnectionListenerFactory, SocketTransportFactory>();

                                             services.AddTransient<IConfigureOptions<KestrelServerOptions>, KestrelServerOptionsSetup>();
                                             services.AddSingleton<IServer, KestrelServerImpl>();
                                         });
}
```



 工作过程(Kestrel自身的实现源码)上帝视角： 

1.  ASP.NET Core程序将日志/IOC/Http管道/HttpContextFactory等打包给Kestrel， Kestrel检查参数然后完成StartAsync 
2.  Kestrel循环监听端口，绑定处理动作：根据不同的Http协议，创建不同的处理管道 KestrelConnection，然后Http管道是其中的一环 
3.  开启线程监听请求，每个请求分配id，然后交给KestrelConnection管道处理，包括 自身动作，以及Http报文解析，Http管道处理等  

#####  Kestrel启动过程源码 

 只关注asp.net core如何初始化+启动的关键点 

1.  WebApplication.CreateBuilder(args)用委托注册，builder.Build()完成IOC注册， services.AddSingleton(); 

2.  app.Run()最终是到了WebHost.StartAsync()，里面组装好了Http管道，还有各种信 息，包括HttpContextFactory等，打包成一个HostingApplication，交给 KestrelServer 

   ```c#
   public async Task StartAsync(CancellationToken cancellationToken = default)
   {
       Debug.Assert(_applicationServices != null, "Initialize must be called first.");
   
       HostingEventSource.Log.HostStart();
       _logger = _applicationServices.GetRequiredService<ILoggerFactory>().CreateLogger("Microsoft.AspNetCore.Hosting.Diagnostics");
       Log.Starting(_logger);
   
       var application = BuildApplication();
   
       _applicationLifetime = _applicationServices.GetRequiredService<ApplicationLifetime>();
       _hostedServiceExecutor = _applicationServices.GetRequiredService<HostedServiceExecutor>();
   
       // Fire IHostedService.Start
       await _hostedServiceExecutor.StartAsync(cancellationToken).ConfigureAwait(false);
   
       var diagnosticSource = _applicationServices.GetRequiredService<DiagnosticListener>();
       var activitySource = _applicationServices.GetRequiredService<ActivitySource>();
       var propagator = _applicationServices.GetRequiredService<DistributedContextPropagator>();
       var httpContextFactory = _applicationServices.GetRequiredService<IHttpContextFactory>();
       var hostingApp = new HostingApplication(application, _logger, diagnosticSource, activitySource, propagator, httpContextFactory);
       await Server.StartAsync(hostingApp, cancellationToken).ConfigureAwait(false);
       _startedServer = true;
   
       // Fire IApplicationLifetime.Started
       _applicationLifetime?.NotifyStarted();
   
       Log.Started(_logger);
   
       // Log the fact that we did load hosting startup assemblies.
       if (_logger.IsEnabled(LogLevel.Debug))
       {
           foreach (var assembly in _options.GetFinalHostingStartupAssemblies())
           {
               Log.StartupAssemblyLoaded(_logger, assembly);
           }
       }
   
       if (_hostingStartupErrors != null)
       {
           foreach (var exception in _hostingStartupErrors.InnerExceptions)
           {
               _logger.HostingStartupAssemblyError(exception);
           }
       }
   }
   ```

   

3.  KestrelServer根据监听信息，通过线程池启动死循环监听， (ConnectionDispatcher. StartAcceptingConnections) 

   ```c#
   public Task StartAcceptingConnections(IConnectionListener<T> listener)
   {
       ThreadPool.UnsafeQueueUserWorkItem(StartAcceptingConnectionsCore, listener, preferLocal: false);
       return _acceptLoopTcs.Task;
   }
   
   private void StartAcceptingConnectionsCore(IConnectionListener<T> listener)
   {
       // REVIEW: Multiple accept loops in parallel?
       _ = AcceptConnectionsAsync();
   
       async Task AcceptConnectionsAsync()
       {
           try
           {
               while (true)
               {
                   var connection = await listener.AcceptAsync();
   
                   if (connection == null)
                   {
                       // We're done listening
                       break;
                   }
   
                   // Add the connection to the connection manager before we queue it for execution
                   var id = _transportConnectionManager.GetNewConnectionId();
                   var kestrelConnection = new KestrelConnection<T>(
                       id, _serviceContext, _transportConnectionManager, _connectionDelegate, connection, Log);
   
                   _transportConnectionManager.AddConnection(id, kestrelConnection);
   
                   Log.ConnectionAccepted(connection.ConnectionId);
                   KestrelEventSource.Log.ConnectionQueuedStart(connection);
   
                   ThreadPool.UnsafeQueueUserWorkItem(kestrelConnection, preferLocal: false);
               }
           }
           catch (Exception ex)
           {
               // REVIEW: If the accept loop ends should this trigger a server shutdown? It will manifest as a hang
               Log.LogCritical(0, ex, "The connection listener failed to accept any new connections.");
           }
           finally
           {
               _acceptLoopTcs.TrySetResult();
           }
       }
   }
   ```

   

4.  监听到请求数据，启动线程再丢给Kestrel管道模型处理(KestrelConnection),里面 其实包含了Http管道  

#####  Kestrel工作过程源码 

 Kestrel自身的实现源码，这个要复杂很多，简约点解析 

1.  从KestrelServerImpl的StartAsync开始，参数检查-做个开关-心跳检测，然后把各 种信息打包丢进去 

2.  各种Bind调用，包括获取配置监听信息，循环Address，进行绑定---其实是绕回到 KestrelServerImpl的StartAsync方法中的OnBind方法：会组装 ConnectionMiddleware(连接中间件)+ connectionDelegate连接管道—管道里面包 含了Http管道 

   ```c#
   public async Task StartAsync<TContext>(IHttpApplication<TContext> application, CancellationToken cancellationToken) where TContext : notnull
   {
       try
       {
           ValidateOptions();
   
           if (_hasStarted)
           {
               // The server has already started and/or has not been cleaned up yet
               throw new InvalidOperationException(CoreStrings.ServerAlreadyStarted);
           }
           _hasStarted = true;
   
           ServiceContext.Heartbeat?.Start();
   
           async Task OnBind(ListenOptions options, CancellationToken onBindCancellationToken)
           {
               var hasHttp1 = options.Protocols.HasFlag(HttpProtocols.Http1);
               var hasHttp2 = options.Protocols.HasFlag(HttpProtocols.Http2);
               var hasHttp3 = options.Protocols.HasFlag(HttpProtocols.Http3);
               var hasTls = options.IsTls;
   
               // Filter out invalid combinations.
   
               if (!hasTls)
               {
                   // Http/1 without TLS, no-op HTTP/2 and 3.
                   if (hasHttp1)
                   {
                       hasHttp2 = false;
                       hasHttp3 = false;
                   }
                   // Http/3 requires TLS. Note we only let it fall back to HTTP/1, not HTTP/2
                   else if (hasHttp3)
                   {
                       throw new InvalidOperationException("HTTP/3 requires HTTPS.");
                   }
               }
   
               // Quic isn't registered if it's not supported, throw if we can't fall back to 1 or 2
               if (hasHttp3 && _multiplexedTransportFactory is null && !(hasHttp1 || hasHttp2))
               {
                   throw new InvalidOperationException("This platform doesn't support QUIC or HTTP/3.");
               }
   
               // Disable adding alt-svc header if endpoint has configured not to or there is no
               // multiplexed transport factory, which happens if QUIC isn't supported.
               var addAltSvcHeader = !options.DisableAltSvcHeader && _multiplexedTransportFactory != null;
   
               var configuredEndpoint = options.EndPoint;
   
               // Add the HTTP middleware as the terminal connection middleware
               if (hasHttp1 || hasHttp2
                   || options.Protocols == HttpProtocols.None) // TODO a test fails because it doesn't throw an exception in the right place
                   // when there is no HttpProtocols in KestrelServer, can we remove/change the test?
               {
                   if (_transportFactory is null)
                   {
                       throw new InvalidOperationException($"Cannot start HTTP/1.x or HTTP/2 server if no {nameof(IConnectionListenerFactory)} is registered.");
                   }
   
                   options.UseHttpServer(ServiceContext, application, options.Protocols, addAltSvcHeader);
                   var connectionDelegate = options.Build();
   
                   // Add the connection limit middleware
                   connectionDelegate = EnforceConnectionLimit(connectionDelegate, Options.Limits.MaxConcurrentConnections, Trace);
   
                   options.EndPoint = await _transportManager.BindAsync(configuredEndpoint, connectionDelegate, options.EndpointConfig, onBindCancellationToken).ConfigureAwait(false);
               }
   
               if (hasHttp3 && _multiplexedTransportFactory is not null)
               {
                   // Check if a previous transport has changed the endpoint. If it has then the endpoint is dynamic and we can't guarantee it will work for other transports.
                   // For more details, see https://github.com/dotnet/aspnetcore/issues/42982
                   if (!configuredEndpoint.Equals(options.EndPoint))
                   {
                       Trace.LogError(CoreStrings.DynamicPortOnMultipleTransportsNotSupported);
                   }
                   else
                   {
                       options.UseHttp3Server(ServiceContext, application, options.Protocols, addAltSvcHeader);
                       var multiplexedConnectionDelegate = ((IMultiplexedConnectionBuilder)options).Build();
   
                       // Add the connection limit middleware
                       multiplexedConnectionDelegate = EnforceConnectionLimit(multiplexedConnectionDelegate, Options.Limits.MaxConcurrentConnections, Trace);
   
                       options.EndPoint = await _transportManager.BindAsync(configuredEndpoint, multiplexedConnectionDelegate, options, onBindCancellationToken).ConfigureAwait(false);
                   }
               }
           }
   
           AddressBindContext = new AddressBindContext(_serverAddresses, Options, Trace, OnBind);
   
           await BindAsync(cancellationToken).ConfigureAwait(false);
       }
       catch
       {
           // Don't log the error https://github.com/dotnet/aspnetcore/issues/29801
           Dispose();
           throw;
       }
   
       // Register the options with the event source so it can be logged (if necessary)
       KestrelEventSource.Log.AddServerOptions(Options);
   }
   ```

   

3.  去开启监听： TransportManager--BindAsync---StartAcceptLoop--StartAcceptingConnections-开启线程来监听请求 

   a.BindAsync

   ```c#
   public async Task<EndPoint> BindAsync(EndPoint endPoint, ConnectionDelegate connectionDelegate, EndpointConfig? endpointConfig, CancellationToken cancellationToken)
       {
           if (_transportFactory is null)
           {
               throw new InvalidOperationException($"Cannot bind with {nameof(ConnectionDelegate)} no {nameof(IConnectionListenerFactory)} is registered.");
           }
   
           var transport = await _transportFactory.BindAsync(endPoint, cancellationToken).ConfigureAwait(false);
           StartAcceptLoop(new GenericConnectionListener(transport), c => connectionDelegate(c), endpointConfig);
           return transport.EndPoint;
       }
   ```

   b.StartAcceptLoop

   ```c#
   private void StartAcceptLoop<T>(IConnectionListener<T> connectionListener, Func<T, Task> connectionDelegate, EndpointConfig? endpointConfig) where T : BaseConnectionContext
   {
       var transportConnectionManager = new TransportConnectionManager(_serviceContext.ConnectionManager);
       var connectionDispatcher = new ConnectionDispatcher<T>(_serviceContext, connectionDelegate, transportConnectionManager);
       var acceptLoopTask = connectionDispatcher.StartAcceptingConnections(connectionListener);
   
       _transports.Add(new ActiveTransport(connectionListener, acceptLoopTask, transportConnectionManager, endpointConfig));
   }
   ```

   c.StartAcceptingConnections

   ```c#
   public Task StartAcceptingConnections(IConnectionListener<T> listener)
   {
       ThreadPool.UnsafeQueueUserWorkItem(StartAcceptingConnectionsCore, listener, preferLocal: false);
       return _acceptLoopTcs.Task;
   }
   
   private void StartAcceptingConnectionsCore(IConnectionListener<T> listener)
   {
       // REVIEW: Multiple accept loops in parallel?
       _ = AcceptConnectionsAsync();
   
       async Task AcceptConnectionsAsync()
       {
           try
           {
               while (true)
               {
                   var connection = await listener.AcceptAsync();
   
                   if (connection == null)
                   {
                       // We're done listening
                       break;
                   }
   
                   // Add the connection to the connection manager before we queue it for execution
                   var id = _transportConnectionManager.GetNewConnectionId();
                   var kestrelConnection = new KestrelConnection<T>(
                       id, _serviceContext, _transportConnectionManager, _connectionDelegate, connection, Log);
   
                   _transportConnectionManager.AddConnection(id, kestrelConnection);
   
                   Log.ConnectionAccepted(connection.ConnectionId);
                   KestrelEventSource.Log.ConnectionQueuedStart(connection);
   
                   ThreadPool.UnsafeQueueUserWorkItem(kestrelConnection, preferLocal: false);
               }
           }
           catch (Exception ex)
           {
               // REVIEW: If the accept loop ends should this trigger a server shutdown? It will manifest as a hang
               Log.LogCritical(0, ex, "The connection listener failed to accept any new connections.");
           }
           finally
           {
               _acceptLoopTcs.TrySetResult();
           }
       }
   }
   ```

   

4.  监听到请求： KestrelConnection的ExecuteAsync，里面后就是把信息交给 _connectionDelegate连接管道---这里面就包括httpcontext解析，http管道----最 后还有finally做全部资源释放  

   StartAcceptingConnections--StartAcceptingConnectionsCore--KestrelConnection--ExecuteAsync

   ```c#
   internal async Task ExecuteAsync()
   {
       var connectionContext = _transportConnection;
   
       try
       {
           KestrelEventSource.Log.ConnectionQueuedStop(connectionContext);
   
           Logger.ConnectionStart(connectionContext.ConnectionId);
           KestrelEventSource.Log.ConnectionStart(connectionContext);
   
           using (BeginConnectionScope(connectionContext))
           {
               try
               {
                   await _connectionDelegate(connectionContext);
               }
               catch (Exception ex)
               {
                   Logger.LogError(0, ex, "Unhandled exception while processing {ConnectionId}.", connectionContext.ConnectionId);
               }
           }
       }
       finally
       {
           await FireOnCompletedAsync();
   
           Logger.ConnectionStop(connectionContext.ConnectionId);
           KestrelEventSource.Log.ConnectionStop(connectionContext);
   
           // Dispose the transport connection, this needs to happen before removing it from the
           // connection manager so that we only signal completion of this connection after the transport
           // is properly torn down.
           await connectionContext.DisposeAsync();
   
           _transportConnectionManager.RemoveConnection(_id);
       }
   }
   ```



#####  总结Kestrel服务器 

 Kestrel服务器做了啥？ 

1.  ASP.NET Core中的Kestrel服务器只是对Socket的简单封装， 
2.  直接用socket通过while(true)的方式来循环接收socket请求， 
3.  然后直接放入clr线程池中来等待线程池调度处理。 

 ASP.NET Core做了啥？ 

1.  准备了Kestrel监听Socket请求 
2.  准备了Http管道处理请求 

 Kestrel基本不用扩展—只需要配置一下  

![](.\img\企业微信截图_16774669551925.png)

#####  委托传递 

 ASP.NET Core应用程序 和 Kestrel之间的纠葛描述： 

 最初是ASP.NET Core去调用Kestrel 然后是Kestrel里面要调用ASP.NET Core 如果各自一个类库，其实就是个循环引用 怎么解决？ 

 用的是委托传递逻辑，解决循环调用： A要调用B，B里面又要调用A的X逻辑，就可以把A的X逻辑封装成委托，传递给B---然后B 在需要时，直接调用委托 

 类似的其实特别多，组件封装+Options 

###  响应流程 

请求来了，执行的一系列操作---Kestrel监听到HttpRequest，解析成 HttpContext---交给HttpPipeline来处理---路由匹配、鉴权授权、控制器-Action—View—Filter---响应回发和资源释放  

#### 框架默认中间件

 框架默认组装的Http管道里面，一系列中间件就是做了什么? 

1.  含注册 和 响应2个阶段，都有对应的逻辑 
2.  中间件分为2大类，通用处理 + MVC密切关联 

 理解这些，也是为了理解MVC是处理请求的完整生命周期，不仅仅是M-V-C 也在期间会拓展丰富的知识点，建立立体的知识体系  

 中间件其实有3件事儿： 

1.  AddXXX----builder.Build()期间，去完成的IOC注册 
2.  UseXXXX---app.Run()期间，去完成中间件注册，组装Http管道 
3.  Middleware的InvokeAsync()—请求响应时，才执行  

#####  ExceptionHandler全局异常处理 

 ASP.NET Core管道最外面，就是 app.UseExceptionHandler("/Home/Error"); 这个是全局异常处理，这个/Home/Error在哪里？ 

 直接看源码 

1.  UseExceptionHandler----ExceptionHandlerExtensions，开启新的Http管道，里面 就是个ExceptionHandlerMiddlewareImpl------注册中间件 
2.  异常真的发生了，中间件的Invoke方法，看catch里面，最终是到 ErrorPage.design.cs文件---请求响应时 
3.  没有Add，因为太简单了---没啥东西 

#####  ExceptionHandler换个方式 

1. 直接UseDeveloperExceptionPage--里面是DeveloperExceptionPageMiddlewareImpl
2.  这种方式比较粗暴，如果希望自定义不同错误呢？ExceptionHandlerMiddleware 

 关于全局异常处理中间件和Filter的区别 

1.  ExceptionHandlerMiddleware是全局性质，是兜底的----而ExceptionFilter其实只 管控制器-Action的异常，是管不了View-ResultFilter的异常 
2.  精细度不同，中间件只知道模糊信息，不适合做业务处理----ExceptionFilter贴近 业务的，可以返回业务信息 
3.  其实完整的异常方案，就应该是二者叠加 

#####  HstsMiddleware 

 HSTS 是 HTTP 严格传输安全(HTTP Strict Transport Security) 的缩写 效果是：优先使用https---如果有https，就使用这个 

 源码解读：HstsMiddleware-----就是设置个header--- Strict-Transport-Security= max-age=  

#####  HttpsRedirection 

UseHttpsRedirection 就是优先访问https，访问http会自动跳转（后台有配置https） 

效果：直接VS的https启动，访问http，开关中间件尝试 https://localhost:7066 http://localhost:5154 

 源码： 

1.  HttpsPolicyBuilderExtensions去注册下HttpsRedirectionMiddleware 
2.  没有Add 
3.  请求来了，会调用HttpsRedirectionMiddleware的Invoke方法 

 逻辑： 

 如果是https---就继续----不是呢，就看看有没有配置https，然后返回个307+https地 址，浏览器再次访问 

#####  StaticFiles 

 UseStaticFiles静态文件处理中间件---StaticFileHandler 

1.  静态文件是怎么响应的？自动读取的？任何文件的响应，都是由代码完成的---- StaticFile中间件得在Routing之前 
2.  直接读源码，从StaticFileExtensions到StaticFileMiddleware，其核心逻辑： 
   1.  各种检测---Get/Head—是否存在文件---类型是否支持 
   2.  都通过就TryServeStaticFile----常规的就是200+SendFileAsync 
   3.  不常规包括Header请求直接200-----缓存lastmodify请求，304和412---等专题最后 一次课，讲缓存再细说  

######  StaticFiles管理 

1.  路径问题，默认是wwwroot，也可以修改 
2.  静态文件响应时，Header缓存处理---缓存；no-cache；no-store； 
3.  静态文件浏览DirectoryBrowser---学了这个+Startup,就是个大后门  

######  防盗链 

 概念：就是防止外站盗用连接，www.a.com直接用 image.b.com的图，B不乐意 原理：图片是互联网公开的---处理图片请求时，做下urlreferer校验 实现：写个RefuseStealingMiddleWare，注册上去----请求来了，就是检测referer，合 法，就继续默认流程；不合法，直接返回个指定图片 验证：done 

1.  可以通过ip去校验或者其他，随便写 
2.  浏览器是不会伪造referer，可以程序伪造referer，能避开，只能下载不能直接盗 链 
3.  也可以用Nginx等服务器去配置完成  

#####  Session中间件 

1.  Session是做啥的？用户持久化，保存在服务端的一段用户信息---以前是用的最多 的，最常用的，直接内置在ASP.NET---ASP.NET Core没有内置，需要自己添加 
2.  全家桶设计与pay for what you use----其实现在用的更多的是Token 
3.  Session的基础使用，Use+Add+读取使用  

######  Session中间件源码 

 从SessionMiddlewareExtensions到SessionMiddleware，直接看源码逻辑： 

1.  请求来了，走Invoke，基于Cookie检查，有没有SessionId 
2.  没有SessionId,则创建Guid的Sessionid，然后SetCookie写入响应 
3.  保存为context.Features.ISessionFeature，后面可以用 
4.  在finally里面把session重新存储下，就滑动过期了  

######  Redis-Session 

 Session是存在服务器进程内存---1 重启丢失 2 不能共享 

 集群下Session如何共享？ 

SQLServer、第三方组件、Redis 

1.  配置分布式存储，用Redis保存 
2.  启动Redis---启动多节点验证 

####  MVC流程 

#####  核心流程概述 

1.  执行AddMVC() 将MVC的核心服务注册到容器，且通过反射扫描把相关dll收集起来 
2.  执行app.UseRouting() 将EndpointRoutingMiddleware中间件注册到http管道 
3.  执行app. MapControllerRoute() 将本程序集定义的所有Controller-Action-ParameterConversion转换为一个个的ControllerActionDescriptor放到路由中间件的配置对象 RouteOptions中,注册并传入EndpointMiddleware中间件注册到http管道中 
4.  请求来了，管道模型Build---没啥动作，倒序执行中间件的实例化 
5.  收到一条http请求，先进入EndpointRoutingMiddleware，首次请求时会完成 ControllerActionDescriptor到EndPoint的转化，然后通过DFA算法匹配一个Endpoint，并 放到HttpContext中去 
6.  鉴权/授权/其他中间件可以根据根据Endpoint的信息对这个请求进行鉴权授权或其他操作。 
7.  EndpointMiddleware中间件执行Endpoint中的RequestDelegate逻辑，即执行FilterController-Action-Result等系列操作(MVC)  

##### AddMVC（AddControllersWithViews）

######  AddMVC上帝视角 

 AddMVC（AddControllersWithViews）是最初发生的，是IOC注册环节的事儿，在管道注 册之前，其职责有2个： 

1.  添加MVC各种IOC注册，超多。。 
2.  反射遍历相关程序集，封装成ApplicationPart，供后续使用  

######  AddMVC源码解读 

1.  从MvcServiceCollectionExtensions开始，绕一圈，最终是AddMvcCore()方法 
2.  先实例化ApplicationPartManager-----拿着项目名字，通过反射加载Dll，将信息 封装到ApplciationPartManager的ApplicationParts属性中---扩展点 
3.  ConfigureDefaultFeatureProviders(partManager);的调用，这行代码是创建了一 个新的ControllerFeatureProvider实例放进了partManager的FeatureProviders属 性中，注意这个ControllerFeatureProvider对象在后面遍历ApplicationPart的时 候负责找出里面的Controller 
4.  ConfigureDefaultServices其实是AddRouting，通过 RoutingServiceCollectionExtensions放了一堆Routing的IOC注册 
5.  AddMvcCoreServices其实就是添加MVC框架需要的各种服务，东西很多---之前扩展 的

######  设计理解 

   AddMVC其实就是添加MVC各种IOC注册，然后反射遍历相关程序集，封装成 ApplicationPart，供后续使用 

1.  小扩展点，就是controller-action可以写在独立类库，完全一样的效果—ToDo 
2.  动态热插拔，运行时dll能被路由识别？其实要扩展这个ApplicationPart —ToDo 

#####  EndPoint 

 AddMVC为啥要先dll反射加载全部dll？这里有个关于路由的设计需要理解！ MVC用的是路由匹配，是请求来了，再一一去查找合适ControllerAction？ 还是提前把全部的ControllerAction都收集好，去里面做筛选？---加载后是dll--- controller-action-parameter-route，得有个对象打包一下这些 

 EndPoint：终结点，一个抽象的概念，两个核心属性： 

1.  RequestDelegate---请求处理动作 
2.  Metadata----是各种元数据 

 EndPoint是路由匹配的结果，当http请求到来时，路由模块就会将请求匹配到某个 EndPoint。然后EndPoint这个类封装了action的信息，比如：Controller类型、Action 方法、[Attribute]情况等。然后就能做到路由匹配和具体执行分开，中间可以放入其他 动作，如鉴权授权等--晚点扩展 

 终极理解---在程序启动完成后，请求来临时，MVC已经把所有的action转化成了 Endpoint并交给“路由”模块去管理了！ 

 如何从ApplicationPart变成EndPoint？----UseRouting和MapControllerRoute！  

##### UseRouting--MapControllerRoute 

######  中间件顺序问题 

 UseRouting----MapControllerRoute都是在注册中间件， 但因为二者的时空交叉在一起，不能单独看一个中间件了，得按时间顺序 先理解3个时间点： 

1.  Use：Use中间件—顺序 
2.  Build：组装管道模型---倒序的 
3.  请求来了：--顺序的 

######  Use环节 

 中间件第一波是Use，通常是完成基础注册，其实也可以加入很多逻辑 

1.  UseRouting()---实例化了一个DefaultEndpointRouteBuilder传到中间件，注册了 EndpointRoutingMiddleware中间件----AddRouting前面已完成 

2.  MapControllerRoute()核心是GetOrCreateDataSource，里面发生了大事儿： 

   -  里面会构造ControllerActionEndpointDataSourceFactory，这里面最终注册 了ChangeToken到UpdateCollection的订阅---该方法完成ApplicationPart转成 ActionDescriptorCollection—细节见后面 
   -  里面有个ControllerActionEndpointDataSource对象的构造函数里面，注意 那个Subscribe(),是注册了一个UpdateEndPoint()方法到ChangeToken，这里会触发一次 UpdateCollection()，完成转变ActionDescriptorCollection 

    然后UpdateEndPoint()是完成ActionDescriptorCollection到EndPoint的转换的 ---这里将二者订阅，等于是ActionDescriptorCollection有啥变化，EndPoint就得更新 下--没毛病！ 

######  Build环节 

 Build管道模型时，会把所有注册中间件倒序执行一遍，这里用的是Use 其实就是简单构造下2个中间件，注意是倒着来的， 先EndpointMiddleware构造，再EndpointRoutingMiddleware构造 这里没啥东西，但理解下转变时间点： 

1.  AddMVC只是扫描Dll---没有其他操作，因为还不知道怎么应用 
2.  中间件组装时是转成ActionDescriptorCollection---只是各种元数据，还不是 EndPoint-----还没有跟路由关联起来，因为还没请求来，免得浪费资源 

 啥时候转变成EndPoint? 请求第一次来的时候！  

######  请求来了-Route 

 先看EndpointRoutingMiddleware的Invoke方法发生了什么： 

1.  第一次来请求时，会CreateMatcher，期间new DataSourceDependentMatcher()的构 造函数里面，通过 _cache.EnsureInitialized()调用了 dataSource.GetChangeToken()触发了前面的UpdateEndPoint(),完成EndPoint生成， 并缓存起来---UpdateEndPoint细节后面再看 
2.  后续再来请求，就不需要CreateMatcher，直接去匹配路由---使用DFAMatcher，找 到合适的EndPoint，保存到Context，DFAMatcher后面再看 
3.  然后SetRoutingAndContinue去下一环节 ---等待后续环节处理，可以鉴权/授权/其 他，当然也可以直接去EndpointMiddleware了  

######  请求来了-Endpoint 

 EndpointMiddleware的Invoke方法，其实就是具体处理请求了 

1.  Invoke方法，做几个检测，然后调用endpoint.RequestDelegate(httpContext)--- 会进入Filter-Controller-Action-View 
2.  看看endpoint. Metadata,其实是各种元数据信息 
3.  看看endpoint. RequestDelegate，其实是个委托 
4.  再来一次请求，这次就直接路由匹配—到EndPointMiddleware,看看 RequestDelegate究竟是啥？其实是ControllerRequestDelegateFactory的一个委托！ ---肯定是个委托呀，因为endpoint里面不可能实例化控制器  

######  再说EndPoint 

 EndPoint是.NET Core2.2推出的 到这里，AddMVC---UseRouting---MapRoute就都串联在一起了，请求也进入MVC了，然后 是更进一步的细节，EndPoint究竟是啥？ 

1.  诞生是为了满足路由匹配的机制，应该先扫描，存储，再进行匹配---基本职责 
2.  RoutingMiddleware是匹配得到EndPoint，保存到Context，然后 EndPointMiddleware又从context里去获取EndPoint，再去执行，为啥呀？ -----将路由匹配和动作执行分开，期间可以植入逻辑，比如鉴权授权，比如其他 的--案例 
3.  执行时间点真的很细腻---真的有请求来了，才转成EndPoint委托---直到需要的时 候，才去处理，按需处理  

#####  中间件顺序问题 

 这还是个经典的考题---考验大家对中间件的理解： 

1.  UseExceptionHandler：必须在最外层，全局异常处理，兜底的 
2.  UseStaticFiles：先处理静态，再处理动态，所以第2 
3.  UseSession：动态数据才需要session 
4.  UseRouting：得先路由匹配(endpoint初始化) 
5.  UseAuthentication：路由匹配后，才知道需要鉴权 
6.  UseAuthorization：路由匹配后，才知道需要授权，必须鉴权后才能授权 
7.  MapControllerRoute：然后才能完成MVC处理---鉴权授权都通过了，才有意义  

#####  阶段总结 

 UseRouting----MapControllerRoute---AddMVC 是MVC密切相关的流程： AddMVC本职工作是各种MVC相关的IOC注册，且把进程相关dll反射加载为PartManager UseRouting-Use时本职工作是路由匹配 MapControllerRoute-Use时本质工作注册EndPoint中间件，且完成2个ChangeToken注册， 触发了PartManager到ControllerActionDescriptor的转化 

 UseRouting-Build时本职工作是实例化EndPointRoutingMiddleware MapControllerRoute-Build时本职工作是实例化EndPointMiddleware 

 EndPointRoutingMiddleware—请求来了时是头次请求时触发ChangeToken，将 ControllerActionDescriptor转换成EndPoint，且组装DFA路由匹配----本职工作是匹配 路由，得到EndPoint，保存到context EndPointMiddleware---请求来了时，本职工作就是调用EndPoint的RequestDelegate， 也就是后续的MVC流程  

##### 核心三组件-总结 

 UseRouting----MapControllerRoute---AddMVC核心三组件，是MVC密切相关的流程： 

1.  AddMVC本职工作是各种MVC相关的IOC注册，且把进程相关dll反射加载为ApplicationPart 
2.  UseRouting-Use时本职工作是路由中间件注册； 
3.  MapControllerRoute-Use时本职工作注册EndPoint中间件，且完成2个ChangeToken注册， 触发了ApplicationPart到ActionDescriptorCollection的转化 
4.  UseRouting-Build时本职工作是实例化EndPointRoutingMiddleware MapControllerRoute-Build时本职工作是实例化EndPointMiddleware 
5.  EndPointRoutingMiddleware—请求来了时是头次请求时触发ChangeToken，将 ActionDescriptorCollection转换成EndPoint，且组装DFA路由匹配----本职工作是匹配路由， 得到EndPoint，保存到context 
6.  EndPointMiddleware---请求来了时，本职工作就是调用EndPoint的RequestDelegate，也 就是后续的MVC流程  

#####  从ApplicationPart到ActionDescriptorCollection 

 因为路由匹配，所以得先反射加载dll，然后转成ActionDescriptorCollection(就是各种控制 器 action filter等元数据，旧版本路由匹配) 

1.  从MapControllerRoute进去，核心就在GetOrCreateDataSource(endpoints)方法，构造 ControllerActionEndpointDataSourceFactory时，注册了UpdateCollection的订阅 
2.  factory.Create(orderProvider.GetOrCreateOrderedEndpointsSequenceProvider(endpoi nts));时，构造ControllerActionEndpointDataSource的构造函数里面的 Subscribe()方 法----这里会直接调用一次collectionProviderWithChangeToken. GetChangeToken()的， 然后触发了前面注册的UpdateCollection()，完成ActionDescriptorCollection转换 
3.  然后是UpdateCollection()---- GetDescriptors()---找控制器(遍历ApplicationPart， 遍历里面的类，按规则筛选，满足的存起来)---找Filter(全局Filter)--- ControllerModel(类型+Filter特性)—找Action(遍历方法，把Action+特性封装了一个 ActionModel)---就是把Controller—Property—Action---Parameter+特性都找出来并且 封装 
4.  然后还有个ApplyConventions---这又是一个扩展点！---细节很多  

#####  细节1-Controller规则 

 ControllerFeatureProvider： 

1.  必须是类 
2.  不能是抽象的 
3.  必须是public的 
4.  不能包含泛型参数 
5.  不能具有[NonController]特性 
6.  名字以“Controller”结尾或者具有[Controller]特性  

 ##### 细节2-Action规则 

 DefaultApplicationModelProvider 

1.  不能是特殊的方法(运算符重载，属性的set/get方法) 
2.  不能是[NonAction] 
3.  不能是object继承下来的 
4.  不能是dispose方法 
5.  不能是静态方法 
6.  不能是抽象方法 
7.  不能是构造函数 
8.  不能是泛型方法 
9.  必须是public方 

 ##### 关于Controller规则 

 来系列案例，扩展or验证， PartDemo 

1.  PartController：反射加载dll时，是可以分类库的---控制器可以在别的类库 
2.  NoInheritController：没有继承controller，也就是名字带个controller，然后就 变成控制器---又是一个后门
3.  NoInheritWithViewController：没有继承Controller，但想用上View+传递数据， 也可以办到 
4.  WithAttribute：靠特性标记，想使用HttpContext，注入IHttpContextAccessor 
5.  WithAttributeNotController：不想当控制器，就靠特性标记 

#####  细节3-ModelConventions 

 在ApplicationPart经过严格的controller-action等规则筛选后，还支持了一个 ApplyConventions，用的是Options.Conversions的，支持对各个元素进行 ModelConventions模型约定，这其实是个扩展点 

1.  从ApplicationModel开始，里面包括了Controller、Action等多种要素，然后有支 持一套规则Convention来处理这些模型，支持自定义约束来扩展 
2.  要操作ModelConvention，有以下几个接口,都有个Apply方法，接受的参数就是对应 的Model，而model里面的东西就特别多了，啥都有 

IApplicationModelConvention

IControllerModelConvention 

IActionModelConvention 

IParameterModelConvention 

IPageRouteModelConvention  

#####  ModelConventions扩展 

1.  实现IXXXModelConvention 
2.  全局注册到options.Conventions，即可查看效果 
3.  特性标记方式，即可查看效果 

#####  从ActionDescriptorCollection到EndPoint 

 上面完成Controller和Action的找出来,最终通过Build生成的是 ControllerActionDescriptor---然后也绑定了ControllerActionDescriptor到EndPoint的 订阅，啥时候触发呢？ 

 是第一个请求来了，在RoutingMiddleware里面触发，再执行UpdateEndpoints 

 从var matcherTask = InitializeAsync();开始，到_matcherFactory.CreateMatcher，里 面new DataSourceDependentMatcher时，_调用cache.EnsureInitialized(); 里面回去调 用_dataSource.GetChangeToken(); 终于触发之前的再执行UpdateEndpoints()方法  

#####  UpdateEndPoints 

 UpdateEndpoints()---ControllerActionEndpointDataSource 里面完成把Action转成了 EndPoint 

 循环每个Action 其实这里只是把各种信息都交给了ActionEndPointFactory,各种检测， 然后遍历路由—匹配Action，组成RoutePattern,期间创建RequestDelegate（是个委托） 

1 得到RoutePattern结合—Action+Route 

 2 得到一系列的EndPoint 里面其实也就是各种元数据，然后RequestDelegate委托，里面并未发生调用 

 理解一下RequestDelegate ：这里还不是具体请求响应，只是路由匹配，所以肯定没有 实例化Controller，而只是用委托保存了一下 

 描述：UpdateEndPoints把controller-action-property-parameter跟路由进行交叉遍历， 组成一系列匹配源(规则-EndPoint)---- EndPoint就是保存了元数据metadate，第二就 是个委托(毫无技术含量，就是个普通的委托声明)  

#####  Why EndPoint? 

1.  由于路由匹配机制，所以得先反射加载dll，然后转成ActionDescriptorCollection， 已经能满足路由机制需求了，为啥最终要转成EndPoint？ 为了匹配和运行的分离！ 为了方便中间的扩展---案例----之前就是匹配完就立即处理没法扩展 
2.  比如鉴权-授权，就应该在匹配完控制器-Action-Filter等之后立即执行，才是最高 效的---如果匹配和执行连在一起，那就只能执行的过程中再鉴权授权 
3.  自己写个小案例  

#####  Why 三层 

1.  为啥ApplicationPart（AddMVC）---ActionDescriptorCollection(Use)--- EndPoint（请求来了）这么多步骤？为了兼容旧版本，.NET Core2.2才出的 EndPoint 
2.  三层如何保障同步更新？多层ChangeToken.Onchange绑定---尤其是 ActionDescriptorCollection---EndPoint是后期添加的，来个OnChange---分拆到 不同的时间点去执行  

#####  阶段总结 

 AddMVC---UseRounting---MapControllerRoute 三个MVC关联的核心方法 其职责---其交互---源码---程序设计---扩展小案例 期间还有一个漏洞，Route是怎么匹配，后面再来！ 再往后，是鉴权授权 再往后，就是MVC流程  

#### 路由

#####  理解Route 

 **官方说明**： 

 路由系统提供的请求URL模式与对应终结点（Endpoint）之间的映射关系，我们可以将具 有相同URL模式的请求分发给应用的终结点进行处理。 

 **接地气儿**： 

 Http请求到aspx/ashx、或者是匹配到静态文件、或者是到某个Action，都不是无缘无故 的，一定得有一个解析-匹配的过程--- 也就是根据请求的URL(也可能包含其他信息：参数、HttpMethod等)，解析为调用某个控 制器某个方法，包括解析出URL中的参数(部分参数)，这个就是路由了 

 **代码案例**： 

 请求http://localhost:5726/home/index，就转到Home控制器，Index的Action了  

#####  路由规则基础 

![](.\img\企业微信截图_16787755263932.png)

默认路由，能满足80%以上的请求 

路由模板和对应的行为 http://localhost:5726/route/index  

#####  基础路由 

```c#
app.MapControllerRoute(
              name: "about-route",
              pattern: "about",
              defaults: new { controller = "Route", action = "About" }
              );

//伪静态路由
app.MapControllerRoute(
    name: "static",
    pattern: "static/{id:int}.html",
    defaults: new { controller = "Route", action = "StaticPage" });
```

指向路由： http://localhost:5726/about 

默认理由： http://localhost:5726/route/about 

伪静态路由： http://localhost:5726/static/133.html 

默认理由： http://localhost:5726/Route/StaticPage/133  

#####  特性路由 

```c#
[Route("/Item1/{id:int}.html")]
[Route("/Item2/{id:int}.html")]
public IActionResult PageInfo(int id)
{
    this._logger.LogWarning("This is RouteController-PageInfo LogWarning");
    string des = $"controller={this.HttpContext.Request.RouteValues["controller"]}&action={this.HttpContext.Request.RouteValues["action"]}&Id={this.HttpContext.Request.RouteValues["id"]}";
    base.ViewBag.Des = des;
    return View();
}
```

 特性路由： http://localhost:5726/Item1/133.html 

 特性路由不需要注册了(相对ASP.NET时代)，靠的是metadata收集 

1.  支持多个特性路由 
2.  http://localhost:5726/Route/PageInfo/133 是404，说明有特性路由等同于是 个约束了，默认路由不再匹配了，不再匹配全局路由 

#####  路由约束 

![](.\img\企业微信截图_16787760056057.png)

  路由约束，限定变量的范围类型等，框架默 认提供了一系列---也能自定义 

http://localhost:5726/Item1/133.html 

http://localhost:5726/hello/11e  

http://localhost:5726/hello/11 

http://localhost:5726/hello/eleven 

1.  约束是有效的 
2.  No constraint居然不会拦截请求—新版路由 的特点—靠的是优先级(精准度) 
3.  Required咋报错了—说明这个优先级也 很高---这事儿记不住，只能靠测试  

#####  正则表达式+约束 

```c#
app.MapControllerRoute(
    name: "range",
    pattern: "{controller=Home}/{action=Index}/{year:range(2019,2021)}-{month:range(1,12)}");

app.MapControllerRoute(
    name: "regular",
    pattern: "{controller}/{action}/{year}-{month}",
    constraints: new { year = "^\\d{4}$", month = "^\\d{2}$" },
    defaults: new { controller = "Home", action = "Index", });
```

 http://localhost:5726/Route/Data/2019-11 

 http://localhost:5726/Route/Data/2019-13 

 http://localhost:5726/Route/Data/2018-09 

 http://localhost:5726/Route/Data/2018-9

 http://localhost:5726/Route/Data?year=2019&month=11 

分别是谁响应的 

#####  路由约束扩展 

 自定义约束---支持 

1.  CustomGenderRouteConstraint自定义约束规则，实现IRouteConstraint 
2.  IOC注册options.ConstraintMap.Add("GenderConstraint",  typeof(CustomGenderRouteConstraint)); 
3.  约束时使用GenderConstraint 

 不太推荐，除非有特殊需求 

1.  不好理解 
2.  还不如匹配成功了去Action里面检测参数再报错，优势是不进MVC流程，但是信息有 点缺失  

#####  Host约束 

 Host约束 将约束应用于需要指定主机的路由  

```c#
/// <summary>
/// http://localhost:5726/Route/HostInfo/123
/// http://localhost:5727/Route/HostInfo/123  不行
/// </summary>
/// <param name="id"></param>
/// <returns></returns>
[Host("*:5726")]
public IActionResult HostInfo(int id)
{
    this._logger.LogWarning("This is RouteController-HostInfo LogWarning");
    string des = $"controller={this.HttpContext.Request.RouteValues["controller"]}&action={this.HttpContext.Request.RouteValues["action"]}&Id={this.HttpContext.Request.RouteValues["id"]}";
    base.ViewBag.Des = des;
    return View();
}
```

#####  MapGet 

 直接指定的规则和RequetDelegate---类似的还有MapPost 

1. MapGet在默认路由后，但是会被匹配上---优先级问题 
2. 匹配路由后，直接指定RequetDelegate，不走Filter流程  

```c#
//约束---会被拦截的刷1  
//http://localhost:5726/hello/11e  
//http://localhost:5726/hello/11
//http://localhost:5726/hello/eleven
app.MapGet("/hello/{name}", async context =>
           {
               var name = context.Request.RouteValues["name"];
               await context.Response.WriteAsync($"Hello {name} no constraint !");
           }); //在前面，但是不能拦截

app.MapGet("/hello/{name:alpha}", async context =>
           {
               var name = context.Request.RouteValues["name"];
               await context.Response.WriteAsync($"Hello {name} alpha!");
           }); //处理动作   http://localhost:5726/hello/eleven  

app.MapGet("/hello/{name:int}", async context =>
           {
               var name = context.Request.RouteValues["name"];
               await context.Response.WriteAsync($"Hello {name} int!");
           }); //处理动作   http://localhost:5726/hello/11
```

#####  动态路由 

 截止目前，路由都是静态写死的---能否去数据库/Redis去校验？--同样请求，能随着数 据库信息而变化，动态路由------ 

1.  app.UseDynamicRouteDefault(); 
2.  services.AddDynamicRoute(); 
3.  全套TranslationTransformer实现，能做到自定义动态规则匹配路由，写在 CustomTranslationSource 

 http://localhost:5726/en/route/info 

http://localhost:5726/ch/route1/info1

 http://localhost:5726/hk/route2/info2 

能支持动态数据库配置请求响应---只能说很炫，实际效果一般，不能每次都去数据库 使用场景： 控制器版本/Action版本---V1—全版本升级到V2 

#####  路由使用总结 

1.  路由就是路由，用来从URL到具体处理的转换 
2.  依赖URL、HttpMethod、约束、自定义规则 
3.  已扩展的：现有URL规则配置、特性路由、约束&正则、自定义约束、动态路由 

 理解下执行时机，分3层： 

1.  常规路由、正则约束等等，皆是通过路由匹配，走Controller-Action 
2.  路由匹配后，能指定不同的handler(requestdelegate)---其实是MapGet 
3.  不走路由匹配，命中后走的是新的中间件流程-----mapwhen useWhen等 

 然后再去读一下源码，大致理解一下  

#####  Route生命周期 

 Route源码分了2个：一是生命周期，二是实现机制 

 无需上帝视角了，之前都讲过的，生命周期内容 

1.  UseRouting时，注册了RoutingMiddleware----MapControllerRoute时，订阅了 UpdateEndPoint 
2.  请求第一次来时，RoutingMiddleware会初始化DFAMatcher，期间触发UpdateEndPoint 并将路由和EndPoint映射起来，组成RouteEndpoint 
3.  全部请求都是通过DFAMatcher来Match，同时检测多个路由状态机，找出最合适的 RouteEndpoint，保存到IEndpointFeature，后续从中获取EndPoint 

 实现机制又非常复杂---关系到DFA算法  

#####  Route核心对象 

1.  RouteEndPoint：继承自EndPoint，多了2个属性，也就是路由匹配结果实际是这个- ----传统的路由以IRouter--RouteHandler对象为核心，对请求匹配后得到的是一个 具体handler(aspx--mvchandler)----然后从ASP.NET Core 2.2中的新路由系统，开 始使用EndPoint 
2.  Order：就是优先级---越小 优先级越高 
3.  RoutePattern里面包含了全部的规则和约束，由RoutePatternFactory生成的,其中 RawText属性就是匹配的具体路由模板 

 路由匹配后，得到RouteEndPoint---一是终结点，二是匹配的信息---这是个匹配结果， 其实是在之前生成的，请求匹配就是通过RoutePattern在匹配  

#####  Route规则和顺序 

 路由匹配的规则和顺序，都放在RoutePattern里面 

1.  规则：地址路径吻合---HttpMethod吻合---路径约束满足 

2.  多个吻合的路由间如何筛选？按注册顺序循环，找到第一个满足条件的结束，要求路由注册 顺序？------以前是这样的，现在用的是DFA算法，同时校验各种路由，注册顺序没用了， 是根据优先级返回 

    	访问http://localhost:5726/ElevenTest 匹配的是具体路由而不是默认路由 /ElevenTest 更具体，因此优先级更高。 

   ​	 访问http://localhost:5726/hello/11和 http://localhost:5726/hello/eleven 也没有匹配通用路由，因为有约束的优先级更高 

 UseEndpoints 内的操作顺序并不会影响路由行为，但有一个例外MapControllerRoute 和 MapAreaRoute 会根据调用顺序，自动将顺序值分配给其终结点。 

#####  RoutePattern 

 RoutePattern+ DFA 

1.  断点看看RoutePattern对象：有多个属性，去完整描述全部的规则 
2.  也去找找源码RoutePatternFactory：一个个属性去拼装---会把地址用斜线拆成 segment数组---为了后续DFA匹配 
3.  DfaGraphWriter扩展，呈现矢量图：就是课树，需要实现初始化，然后请求来按 segment匹配 

 RoutePattern：是个规则结合体，一方面在匹配前，必须将全部Action都转成 RouteEndpoint---二方面是请求来了，按照RoutePattern进行匹配筛选---按路径分 segment来做匹配(所以跟路由注册顺序没关系)  

#####  RoutePattern矢量图 

1.  实现GraphEndpointMiddleware，注入了EndpointDataSource：就是程序启动时，将 Action-Route映射转换的全部数据结合----DfaGraphWriter：DFA图形展示 
2.  做个中间件注册， app.Map("/graph", branch =>  branch.UseMiddleware()); 
3.  dotnet run --urls=http://*:5728 然后访问：http://localhost:5728/graph 
4.  去站点https://graphviz.christine.website/ 

 完成了对DFA数据源头的可视化，能看到这个数据源  

![](.\img\graphviz.png)

#####  DFA匹配算法 

 刚才都是在拼凑匹配的源头---EndpointDataSource---那是如何匹配出来的呢？ 请求来了Home/index，通过RouteParse得到 Home,Index---然后去树里面匹配 

 DFA，全称 Deterministic Finite Automaton 即确定有穷自动机：从一个状态通过一系 列的事件转换到另一个状态，即 state -> event -> state。 确定：状态以及引起状态转换的事件都是可确定的，不存在“意外” 。 有穷：状态以及事件的数量都是可穷举的。 

 简而言之：就是一堆字符串里面匹配时，双层for循环---关键字触发事件，只检测固定 几个值 https://blog.csdn.net/xushiyu1996818/article/details/89308561  

#####  路由Route总结 

 关于Route匹配机制，当下只能到这个程度，欢迎深究和分享 

 关于Route匹配规则，也是记不住的，只能靠测试 

1.  理解路由：就是个请求地址(其他信息)到具体处理动作的转换 
2.  多种案例演示：茫茫多。。。以及多层扩展 
3.  生命周期理解：这个是前面讲的 
4.  匹配机制理解： 
   -  第一次请求匹配前，会将全部的Action和Route映射成一个RouteEndpoint，存 入集合EndpointDataSource 
   -  关乎匹配规则都放入在RoutePattern，按照斜线拆成segment保存 
   -  匹配时用DFAMatcher，使用DFA算法，将请求地址按照斜线拆成segment,然后 一层层比较，找到结果，完成匹配---得到EndPoint-------RoutingMiddleware  

#### 鉴权授权

##### 什么是鉴权授权？

从Http无状态协议---用户持久化的需求---最初是Cookie+Session---Token 

但是不管是哪种方式，有一段内容是不变的： 

1.  请求服务端，获取凭证 
2.  客户端再次请求，带上凭证 
3.  服务端识别凭证，判断是否允许访问 

鉴权授权的本质是做啥的？其实就是第3步 

解决用户识别和判断授权的问题 

被拆分成了2个动作： 

1.  鉴权Authentication：凭证识别解析，有没有登陆，凭证有没有过期，是张三还是 李四 
2.  授权Authorization：基于解析来权限检测，判断下张三/李四有没有权限访问这个 资源 

##### 快捷实操-1 

 鉴权授权流程实操 

1.  添加相关控制器，Cookie式登陆方法 
2.  UseAuthentication + AddAuthentication + Cookie 
3.  UseAuthorization + AddAuthorization(可不写) 
4.  [Authorize] 
5.  [AllowAnonymous] 

 校验流程： 

1.  未登陆，直接访问，跳转登录页 
2.  AllowAnonymous可直接访问 
3.  登陆后跳转，正常访问 
4.  退出后访问，跳转登陆页  

##### 快捷实操-2 

 前面是没有任何的权限要求，只是鉴权通过就行 

1.  Roles授权：[Authorize(Roles = "Admin")] 
2.  Policy授权：[Authorize(Policy = "AdminPolicy")] 
3.  复杂Policy授权：[Authorize(Policy = "MutiPolicy")] 

 授权的设计，也是可以满足开发者各种这样的需求—目前还没演示完 

 如果是没有登陆，是跳转LoginPath---401 

 如果是有凭证，但没有权限，是跳转AccessDeniedPath--403  

##### 快捷实操-3 

1. 主动鉴权

   ```c#
   public async Task<IActionResult> Authentication()
   {
       //Console.WriteLine($"base.HttpContext.User?.Claims?.First()?.Value={base.HttpContext.User?.Claims?.First()?.Value}");
   	//鉴权动作，用什么方式鉴权
       var result = await base.HttpContext.AuthenticateAsync(CookieAuthenticationDefaults.AuthenticationScheme);
       if (result?.Principal != null)
       {
           base.HttpContext.User = result.Principal;//鉴权成功通过user传梯用户信息
           return new JsonResult(new
                                 {
                                     Result = true,
                                     Message = $"认证成功，包含用户{base.HttpContext.User.Identity.Name}"
                                 });
       }
       else
       {
           return new JsonResult(new
                                 {
                                     Result = false,
                                     Message = $"认证失败，用户未登录"
                                 });
       }
   }
   ```

   

2. 主动授权 

   ```c#
   public async Task<IActionResult> Authorization()
   {
       var result = await base.HttpContext.AuthenticateAsync(CookieAuthenticationDefaults.AuthenticationScheme);
       if (result?.Principal == null)
       {
           return new JsonResult(new
                                 {
                                     Result = true,
                                     Message = $"认证失败，用户未登录"
                                 });
       }
       else
       {
           base.HttpContext.User = result.Principal;
       }
   
       //授权
       var user = base.HttpContext.User;
       if (user?.Identity?.IsAuthenticated ?? false)//是否鉴权
       {
           //是否有权限访问
           if (!user.Identity.Name.Equals("Eleven", StringComparison.OrdinalIgnoreCase))
           {
               await base.HttpContext.ForbidAsync(CookieAuthenticationDefaults.AuthenticationScheme);
               return new JsonResult(new
                                     {
                                         Result = false,
                                         Message = $"授权失败，用户{base.HttpContext.User.Identity.Name}没有权限"
                                     });
           }
           else
           {
               return new JsonResult(new
                                     {
                                         Result = false,
                                         Message = $"授权成功，用户{base.HttpContext.User.Identity.Name}具备权限"
                                     });
           }
       }
       else
       {
           await base.HttpContext.ChallengeAsync(CookieAuthenticationDefaults.AuthenticationScheme);
           return new JsonResult(new
                                 {
                                     Result = false,
                                     Message = $"授权失败，没有登录"
                                 });
       }
   }
   ```

   

##### 理解总结 

鉴权授权是ASP.NET Core框架封装的2个中间件， 目的在于请求进入具体的Filter-M-V-C之前，通过中间件完成用户权限检测 

包含2个步骤  ： 

1.  鉴权Authentication：鉴别有没有登陆，解析是张三还是李四(且将信息传递下去)- --UseAuthentication配置Http管道，保证请求来了，都要做一次凭证的解析 AddAuthentication配置IOC，告诉如何鉴权(凭证在那儿，啥格式的) 
2.  授权Authorization：判断下张三/李四有没有权限访问这个资源--- UseAuthorization告诉框架要做授权检测，AddAuthorization告诉框架如何授权检 测---[Authorize]标记显示声明该方法需要授权+需要什么样的授权 

这里是基本过程，全是配置下就生效了，封装的挺厉害的。。 

然而框架是如何实现的？ 各种封装又是怎么回事儿？ 开始深入 

##### 理解鉴权的设计 

细化鉴权---深入鉴权--- 

鉴权就是检测凭证，鉴别是张三，是如何解析？ 

1.  凭证的位置：凭证是怎么传递的，Cookie/JWT就不一样 
2.  凭证的格式：凭证是什么格式的，加密/序列化 
3.  信息有效性：过期时间啥的，token 
4.  鉴权信息传递：解析得到的信息保存起来，后面使用--context.User 
5.  特殊情况处理：没登陆/没权限怎么跳转？ 

 以上都是鉴权该完成的动作  

```c#
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)//凭证的位置
    .AddCookie(CookieAuthenticationDefaults.AuthenticationScheme, options =>
               {
                   //options.ExpireTimeSpan//过期时间
                   options.LoginPath = "/Home/Index";
                   options.AccessDeniedPath = "/Home/Privacy";//有登陆，鉴权成功--没有授权
               });//使用Cookie的方式
```

##### 自定义鉴权 

Cookie鉴权 JWT鉴权，抑或其他鉴权，本质应该都是一样的

这里先来个自定义的鉴权方式，一是帮助理解鉴权，二是方便后续调试学源码 

1.  实现个UrlTokenAuthenticationHandler---主要是实现IAuthenticationHandler 或者继承父类AuthenticationHandler<UrlTokenAuthenticationOptions>
2.  注册鉴权---AddScheme---告诉框架如何解析 
3.  标记+验证---带Token就ok 不带就跳转---自定义Scheme已完成！ 

##### 鉴权Authentication上帝视角 

鉴权授权是紧密关联的，通过context.User通信的(授权期间其实也会调用鉴权) 

先执行AddAuthentication： 

1.  设置默认Scheme+4个核心对象的IOC注册 
2.  支持配置AuthenticationOptions，里面支持了AddScheme， AuthenticationSchemeBuilder去组装Scheme(未Build) 
3.  AddCookie完成校验规则配置---暂时不看Cookie细节，UrlToken是一样的 

 后执行UseAuthentication： 

1.  注册AuthenticationMiddleware中间件，Build-AuthenticationScheme 
2.  请求来了，执行Invoke方法，然后里面又是一套Provider--Handler--Service多对 象绕一下，完成鉴权 
3.  只做基于默认Scheme解析，信息经过一波转换，保存到context.User  

##### AddAuthentication 

 通过扩展后的源码直接看核心点 

1.  核心是AddAuthenticationCore，里面完成了4核心对象的注册---默认名字就是指定 下DefaultScheme 

   ```c#
   services.TryAddScoped<IAuthenticationService, AuthenticationService>();
   services.TryAddSingleton<IClaimsTransformation, NoopClaimsTransformation>(); // Can be replaced with scoped ones that use DbContext
   services.TryAddScoped<IAuthenticationHandlerProvider, AuthenticationHandlerProvider>();
   services.TryAddSingleton<IAuthenticationSchemeProvider, AuthenticationSchemeProvider>();
   ```

2.  Options支持了AddScheme，其实是new一个AuthenticationSchemeBuilder保存起来 (里面就是存了个那个handler的Type)，且用Name作为key，存入字典----(没有 Build,注册中间件的时候注入Provider，在构造函数里面才Build成 AuthenticationScheme) 

3.  AddCookie---AddJWT---AddUrlToken，都是一样的 

最终是做了几个IOC注册，支持了不同的 字符串对应一个AuthenticationScheme (其实这里HandlerType就是将来的处理方式)  

##### UseAuthentication 

通过扩展后的源码直接看核心点 

1.  Use中间件AuthenticationMiddleware时，构造函数注入 

   AuthenticationSchemeProvider，其构造函数中先完成Options的初始化(AddSchme)，  然后把AuthenticationOptions的全部SchemeBuilder.Build(里面就namedisplayname-HandleType)，得到全部的AuthenticationScheme，字典缓存起来---- 

    (至此，启动环节完成—Add时把SchemeName跟Handler类型保存到字典，给到中间件） 

2.  请求来了---有个基于AuthenticationHandlerProvider的扩展点，直接自行处理掉请 求（非常规，不管了）-----用AuthenticationSchemeProvider获取默认的 AuthenticationScheme，然后准备鉴权 

3.  context.AuthenticateAsync来鉴权，里面会调用AuthenticationService的 AuthenticateAsync()方法----这里调用注入的AuthenticationHandlerProvider，找 的是AuthenticationSchemeProvider---里面查找的AuthenticationScheme---然后是 里面的HandlerType(之前存的)，且完成实例化+初始化----然后调用handler的 AuthenticateAsync()方法做鉴权---然后执行一次数据转换---转换后保存到 context.User  

##### Authentication核心对象 

1.  AuthenticationSchemeProvider：包含了全部的Scheme，字典保存 
2.  AuthenticationScheme：2个名字+具体handler的类型 
3.  AuthenticationService：鉴权服务，context.鉴权其实就是调用它的 
4.  AuthenticationHandlerProvider：鉴权handler的提供程序，它会通过Scheme的 Provider来获取Scheme，且完成Scheme的handler的实例化，且初始化，然后缓存 
5.  NoopClaimsTransformation可以完成数据的转换 

 这么设计，就是为了单一职责，为了扩展  

##### 总结鉴权流程 

1.  AddAuthentication注册最主要的三个对象注册 
2.  Options把Scheme和Handler注册进去 
3.  UseAuthentication注册认证中间件，完成AuthenticationScheme的初始化 
4.  请求来了，就是用默认scheme的名字去绕一圈，完成鉴权，保存到context.User  

##### 3个鉴权扩展 

1.  自定义实现IAuthenticationHandler，也就是UrlTokenAuthenticationHandler完成 鉴权来源、方式等全部扩展 
2.  简单扩展下IClaimsTransformation---CustomClaimsTransformation，然后IOC注册， 可以将信息做一次转换 
3.  多Scheme问题---1 程序是可以支持多个Scheme并存 2 鉴权中间件是默认只走默 认Scheme，其他Scheme信息怎么获取？其实靠授权标记的 3 多Scheme，那授 权怎么算？，其实还很复杂！ 这些，下次课见分晓 

##### 多Scheme鉴权演练 

一个站点支持多个渠道的鉴权，比如APP、H5、微信端、PC----Cookie+JWT+Ids4+UrlToken 多Scheme鉴权配置和测试流程，8次访问完成验证过程： 

1. 访问地址 http://localhost:5726/Authentication/UrlCookieByDefault 
2. 访问地址 http://localhost:5726/Authentication/UrlCookieByDefault?UrlToken=eleven-123456 
3. 访问地址http://localhost:5726/Authentication/UrlCookieByUrlToken?UrlToken=eleven-123456
4. 访问地址http://localhost:5726/Authentication/UrlCookieByCookie
5. 访问地址http://localhost:5726/Authentication/Login?name=Eleven&password=123456
6. 访问地址http://localhost:5726/Authentication/UrlCookieByCookie
7. 访问地址http://localhost:5726/Authentication/UrlCookieByCookie?UrlToken=eleven-123456
8. 访问地址http://localhost:5726/Authentication/UrlCookieByDouble?UrlToken=eleven-12345 

##### 多Scheme鉴权结论 

1. 程序支持多套Scheme解析，是可以共存的 
2. 不标记Scheme等同于是标记默认Scheme
3. 无论是否标记，或标记啥Scheme，默认鉴权都会走一遍，在鉴权中间件(它不管 Filter特性)
4. 可以通过标记来指定Scheme去鉴权
5. 声明多个Scheme信息都保存context.User---标记的Scheme，解析后的信息是存入到 context.User---默认、其他的、多个都一样，多个会合并--
6. 如果标记的是其他Scheme---然后默认Scheme也传值了也解析了，但不会保存到 context.User 

 标记相同的Scheme，会distinct—授权源码里面 

 鉴权还好，难在授权  

##### 授权3个属性 

鉴权Authentication是解析出当下是张三，信息保存在context.User 然后该授权Authorization了，判断下张三有没有权限访问这个资源 (其实因为其他scheme的原因，授权环节会根据Scheme触发鉴权) 

1.  标记[Authorize] 走的是默认Policy，只要有合法凭证就行(有源码) 
2.  Roles：要求用户角色信息满足(具体校验有源码) 
3.  Policy：支持自定义各种规则(多API show 
4.  AuthenticationSchemes：表示用户信息的来源(已展示)  

##### PolicyBuilder多API使用 

RequireRole

RequireUserName

RequireClaim

RequireAssertion

AddRequirements

Requirements.Add

Combine 

AddAuthenticationSchemes---其实就是指定Scheme 

```c#
            builder.Services.AddAuthorization(options =>
            {
                options.AddPolicy("AdminPolicy", policyBuilder =>
                {
                    policyBuilder.RequireRole("Admin");
                });//等价于  Roles=Admin
                options.AddPolicy("MutiPolicy", policyBuilder =>
                {
                    policyBuilder.RequireRole("Admin")//都属于框架封装好的
                    .RequireUserName("Eleven")//Role  UserName都是最常用的
                    .RequireClaim(ClaimTypes.Country)//只要求有Country属性
                    .RequireAssertion(context =>//可以灵活的扩展规则--cliam之外其他信息也可以IP等
                    {
                        return context.User.HasClaim(c => c.Type == ClaimTypes.Email)
                        && context.User.Claims.First(c => c.Type == ClaimTypes.Email).Value.Equals("57265177@qq.com");
                    })
                    .RequireAssertion(context =>//ClaimTypes和字符串的区别
                    {
                        return context.User.HasClaim(c => c.Type == "Email")
                        && context.User.Claims.First(c => c.Type == "Email").Value.Equals("12345678@163.com");
                    })
                    .AddRequirements(new SingleEmailRequirement("@qq.com"))
                    ;
                    policyBuilder.Requirements.Add(new SingleEmailRequirement("@qq.com"));

                    policyBuilder.Combine(new AuthorizationPolicyBuilder().AddRequirements(new SingleEmailRequirement("@qq.com")).Build());
                });
            });
```

##### Authorization源码-上帝视角 

应用了这么多，看源码之前，可以合理推测个上帝视角了 

AddAuthorization： 

1.  一系列IOC的注册，分在AddAuthorizationCore()和 AddAuthorizationPolicyEvaluator()-评估，执行授权 
2.  支持Options里面组装各种Policy规则，Name-PolicyBuilder的缓存组合 
3.  Role是Policy的特例，Policy的各种Require，都是转换成了Requirement，将来授权就是遍历Requirement做检验 

UseAuthorization： 

1. 先注册中间件(实例化middleware时通过注入初始化了Policy)—然后请求来了 
2. 获取全部授权特性Authorize，拼装全部Requirement(Roles、Policy)和Scheme，放 入到AuthorizationPolicy 
3. 将全部的Scheme都鉴权一遍，信息保存到context.User 
4. 校验全部的Requirement(Roles、Policy) 
5. 成功则继续 未登录就Challenge ，未授权就Forbid  

##### AddAuthorization源码 

1. 各种IOC注册，主要是AddAuthorizationCore()和AddAuthorizationPolicyEvaluator()， 里面就是一系列的IOC注册和一个services.Configure(configure) 

   ```c#
   public static IServiceCollection AddAuthorization(this IServiceCollection services, Action<AuthorizationOptions> configure)
       {
           if (services == null)
           {
               throw new ArgumentNullException(nameof(services));
           }
   
           services.AddAuthorizationCore(configure);
           services.AddAuthorizationPolicyEvaluator();
           services.AddSingleton<AuthorizationPolicyCache>();
           return services;
       }
   ```

2.  启动过程中，某类(也许我们写的测试)初始化依赖于 IOptions，然后就执行了Configure委托---(之前 AddAuthorization只是做了Configure未执行) 

跟AddScheme一个套路---所以先理解为 是UseAuthorization()使用中间件AuthorizationMiddleware，会注入 IAuthorizationPolicyProvider，其依赖于IOptions就是会 完成Configure委托---(之前AddAuthorization只是做了Configure未执行 

##### 授权策略的设计与实现 

AuthorizationPolicy授权策略是如何设计的？ 

各种Roles、Policy里面N多方法，都是要放入到AuthorizationPolicy，最终形态是怎样的？ AuthorizationPolicyBuilder是如何Build的？  

1.  AddPolicy就是实例化AuthorizationPolicyBuilder(这里都没有Scheme的，后面授权检测时会有) 
2.  然后ConfigureBuilder，各种API了--实际特别简单，都转成AuthorizationRequirement 存起来 
3.  Build()就是把各种Requirement和Scheme传递过去，变成AuthorizationPolicy，而且保 存到map缓存 
4.  校验时，就是遍历Requirement的HandleRequirementAsync 

 支持AuthorizationOptions配置，然后里面是AddPolicy---写入 AuthorizationPolicyBuilder----保存在PolicyMap---当下并未初始化------等注入这个 Options时，各种规则转成一个个AuthorizationPolicy，等着校验时使用

##### 关于Requirement 

Policy最终其实都是转成了Requirement---执行校验(猜)，就是看Requirement- RolesAuthorizationRequirement源码 AssertionRequirement源码 

Requirement 就是实现了IAuthorizationHandler和IAuthorizationRequirement(空接 口)，核心内容就是构造函数初始化+验证处理HandleAsync----成功就 context.Succeed(this); 失败了不管------调用的地方，会检测全部Requirement是否 设置成功 

通过AuthorizationHandler会找出全部满足这个Trequirement然后遍历执行---其实支持 多个(DoubleEmailRequirement)----这种就可以遍历执行2个Requirement---支持或规则 ----成功设置succeed，失败不管---  

```c#
/// <summary>
/// Makes a decision if authorization is allowed.
/// </summary>
/// <param name="context">The authorization context.</param>
public async Task HandleAsync(AuthorizationHandlerContext context)
{
    foreach (var handler in context.Requirements.OfType<IAuthorizationHandler>())
    {
        await handler.HandleAsync(context);
    }
}
```

##### Requirement扩展 

扩展一下Requirement： 

1.  SingleEmailRequirement是实现接口+继承父类，然后直接Add 

   ```c#
   public class SingleEmailRequirement : AuthorizationHandler<SingleEmailRequirement>, IAuthorizationRequirement
   {
       public SingleEmailRequirement(string requiredName)
       {
           if (requiredName == null)
           {
               throw new ArgumentNullException(nameof(requiredName));
           }
   
           RequiredName = requiredName ?? "@qq.com";
       }
       /// <summary>
       /// 邮件域名，默认@qq.com
       /// </summary>
       public string RequiredName { get; }
   
       protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, SingleEmailRequirement requirement)
       {
           if (context.User != null && context.User.HasClaim(c => c.Type == ClaimTypes.Email))
           {
               var emailCliamList = context.User.FindAll(c => c.Type == ClaimTypes.Email);//支持多Scheme
               if (emailCliamList.Any(c => c.Value.EndsWith(RequiredName, StringComparison.OrdinalIgnoreCase)))//数据库校验--Redis校验
               {
                   context.Succeed(requirement);
               }
               else
               {
                   //context.Fail();//失败不管
               }
           }
           return Task.CompletedTask;
       }
   }
   ```

2.  CountryRequirement、 DateOfBirthRequirement-----拆开的需要做IOC注册才能校 验---如果不注册，不报错，但不能校验通过 

   ```c#
   // 需要IOC注册
   builder.Services.AddSingleton<IAuthorizationHandler, CountryRequirementHandler>();
   
   public class CountryRequirement : IAuthorizationRequirement
   {
       public string Country = null;
       public CountryRequirement(string country)
       {
           this.Country = country;
       }
   }
   
   public class CountryRequirementHandler : AuthorizationHandler<CountryRequirement>
   {
       protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, CountryRequirement requirement)
       {
           if (context.User != null && context.User.HasClaim(c => c.Type == ClaimTypes.Country))
           {
               var emailCliamList = context.User.FindAll(c => c.Type == ClaimTypes.Country);//支持多Scheme
               if (emailCliamList.Any(c => c.Value.Contains(requirement.Country, StringComparison.OrdinalIgnoreCase)))
               {
                   context.Succeed(requirement);
               }
               else
               {
                   //context.Fail();//没成功就留给别人处理
               }
           }
           return Task.CompletedTask;
       }
   }
   ```

   

3.  或条件：DoubleEmailRequirement，必须继承AuthorizationHandler，拆开接口--- -然后分别IOC注册----然后就是遍历多实现，支持或关系 

   ```c#
   // 需要IOC注册
   builder.Services.AddSingleton<IAuthorizationHandler, ZhaoxiMailHandler>();
   builder.Services.AddSingleton<IAuthorizationHandler, QQMailHandler>();
   
   
   /// <summary>
   /// 两种邮箱都能支,二选一
   /// </summary>
   public class DoubleEmailRequirement : IAuthorizationRequirement
   {
   }
   
   public class QQMailHandler : AuthorizationHandler<DoubleEmailRequirement>
   {
       protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, DoubleEmailRequirement requirement)
       {
           if (context.User != null && context.User.HasClaim(c => c.Type == ClaimTypes.Email))
           {
               var emailCliamList = context.User.FindAll(c => c.Type == ClaimTypes.Email);//支持多Scheme
               if (emailCliamList.Any(c => c.Value.EndsWith("@qq.com", StringComparison.OrdinalIgnoreCase)))
               {
                   context.Succeed(requirement);
               }
               else
               {
                   //context.Fail();//不设置失败 交给其他处理
               }
           }
           return Task.CompletedTask;
       }
   }
   
   public class ZhaoxiMailHandler : AuthorizationHandler<DoubleEmailRequirement>
   {
       protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, DoubleEmailRequirement requirement)
       {
           if (context.User != null && context.User.HasClaim(c => c.Type == ClaimTypes.Email))
           {
               var emailCliamList = context.User.FindAll(c => c.Type == ClaimTypes.Email);//支持多Scheme
               if (emailCliamList.Any(c => c.Value.EndsWith("@ZhaoxiEdu.Net", StringComparison.OrdinalIgnoreCase)))
               {
                   context.Succeed(requirement);
               }
               else
               {
                   //context.Fail();//不设置失败 交给其他处理
               }
           }
           return Task.CompletedTask;
       }
   }
   ```

还有一个核心价值，就是能做到前面做不到的，就是去数据库动态校验！---- 到这里，其实完整的授权方案是出来了： 

1.  最基础的Roles角色授权 
2.  复杂的Policy自定义授权 
3.  支持动态检测授权  

##### 总结下AddAuthorization 

1.  AddAuthorization各种IOC的注册 
2.  把各种规则Policy组装好，都是变成requirement，都放入Map缓存了下，里面是 PolicyName+各种AuthorizationPolicy 
3.  关于Policy的设计：一个builder，支持7种方式去Add，最终都requirement，就是 规则 
4.  Requirement的3层验证：Role/Policy/动态校验 

##### UseAuthorization上帝视角 

AddAuthorization已经将Role、Policy、 Requirement等规则，都转成了Requirement集 合，然后把Scheme一起存起来，放入缓存Map了 而UseAuthorization，自然是注册中间件+请求来了开始处理！ 

1.  先注册中间件(实例化middleware时通过注入初始化了Policy，应该是这样设计) 
2.  然后是请求来了 
3.  获取全部授权特性Authorize，拼装全部Requirement(Roles、Policy)和Scheme，放 入到AuthorizationPolicy 
4.  将全部的Scheme都鉴权一遍，信息保存到context.User 
5.  校验全部的Requirement(Roles、Policy) 
6.  由AuthorizationMiddlewareResultHandler中间件处理结果：成功则继续 未登录 就Challenge ，未授权就Forbid  

##### UseAuthorization源码 -1 

这里的对象有点多，步骤也有点多，得分步骤讲： 

1.  常规的UseMiddleware，Build时实例化中间件----注入 IAuthorizationPolicyProvider---里面注入IOptions  options---就是上端配置的各种Policy规则初始化放入缓存(特殊说明：Scheme是这样设计，但授权不知道前面是哪里注入了Options)---保存到Options的 PolicyMap(又是缓存了) 
2.  请求来了-----检查endpoint---给context的Item增加个值，然后是从Metadata里面 找下IAuthorizeData(其实就是AuthorizeAttribute)—包括Aciton、Controller、 全局、ModelConversion 

##### UseAuthorization源码-2 

第3步：AuthorizationPolicy.CombineAsync(_policyProvider, authorizeData)把Policy 和授权特性的要求都组装起来----先看有没有authorizeData(授权要求，没有就啥也不 干)---有授权特性，会把全部规则都组装起来： 

1.  遍历authorizeData(代表可以声明多个，效果叠加)，实例化一个 AuthorizationPolicyBuilder(负责包含全部规则) 
2.  先看Policy---标记特性AuthorizeData有Policy-----用Name去PolicyProvider去map 缓存中获取AuthorizationPolicy----然后Combine就是将AuthenticationSchemes和 Requirements保存在PolicyProvider(依旧是policy的scheme和requirement)--没有的 话采用默认Policy，就是不允许匿名，也就是得有个合法凭证 
3.  再看Roles---标记特性AuthorizeData有Roles----先Split拆分，再每个Role都是 RolesAuthorizationRequirement----再添加到PolicyBuilder的Requirements（其实 也是转成了Requirement) 
4.  最后看Schemes---标记特性AuthorizeData有Schemes,就直接添加到PolicyBuilder的 AuthenticationSchemes 
5.  最后再Builder下，把全部Requirements和去重后的AuthenticationSchemes，交给 AuthorizationPolicy实例化  

组装完了，就是一个AuthorizationPolicy，里面是全部的Schemes和Requirement，后面 就是校验了，能得到的信息： 

1.  Options还能设置DefaultPolicy，默认就是个 DenyAnonymousAuthorizationRequirement，不允许匿名使用，也就是日常只标记个 [Authorize]，只要鉴权成功就能通过，其实是通过这个requirement 
2.  Authorize特性可以标记多个，controller+action+全局，都是叠加规则 
3.  Authorize特性的Policy只能一个名字，Role和Scheme可以逗号分隔的多个值组装全部规则  

##### UseAuthorization源码-3 

第4步：继续授权检测，这次是全面鉴权---先找IPolicyEvaluator(PolicyEvaluator)， 一堆注入不看，然后执行AuthenticateAsync()----执行鉴权 

1.  为啥又鉴权？ 因为前面鉴权中间件只管默认Scheme，这里可能多Scheme 
2.  如何鉴权？如果没有AuthenticationSchemes，则查看 context.User?.Identity?.IsAuthenticated？，也就是之前的默认Scheme鉴权结果 
3.  如果有指定Scheme，则去遍历执行全部Scheme，分别鉴权，并MergeUserPrincipal合 并到ClaimsPrincipal属性(List) 
4.  都为空就NoResult了，失败了 

 收获： 

1.  支持多个Scheme鉴权，同名的去重了，各渠道都解析一遍，然后信息都保存到 context.User了 
2.  不添加Authorization的Scheme，则是用默认的Scheme,就不用鉴权 添加了Scheme，这里会根据Scheme去鉴权一次的--即使就是默认Scheme  

##### UseAuthorization源码-4 

第5步，检查IAllowAnonymous特性，有就直接过去了---(为啥不早点检查，甚至在鉴权就检 查？我没有答案) 

第6步，继续授权检测----用policyEvaluator来做AuthorizeAsync根据信息检查授权，其实 靠的DefaultAuthorizationService做授权检测 

具体动作： 

1.  IAuthorizationHandlerContextFactory准备上下文，就是各种信息拼起来 
2.  IAuthorizationHandlerProvider获取授权AuthorizationHandler，这个是IOC注册进来的 各种IAuthorizationHandler合集，默认是CustomPassThroughAuthorizationHandler，可以支持多个注册 
3.  遍历执行AuthorizationHandler中的HandleAsync----其实就是遍历全部Requiredment去 执行的HandleAsync()，在父类里面，遍历全部该Requiredment类型，执行 HandleRequirementAsync(到了Requiredment里面的校验方法了)-----把角色都拿出来， 任意一个只要在contextUser里面的某个Identity满足就行(确实支持多个scheme获取的多 个用户信息)---任意一个校验通过就行 
4.  规则是全部成功就过去，一个Failed就失败了 
5.  最后转化个Succeed结果，成功---Forbid---Challenge，保存起来了  

##### UseAuthorization源码-5 

第7步，根据授权结果处理----找IAuthorizationMiddlewareResultHandler去 HandleAsync(),根据不同的authorizeResult分别执行不同的逻辑 

1.  Challenged：就先看Scheme，没指定就看默认Scheme 有指定，就遍历Schme，去AuthenticationHttpContextExtensions执行Scheme的 Challenged ，实际上又跑到AuthenticationService去Challenged ，又找到那个 AuthenticateHandler(自定义各种处理方式的)去Challenged--回到鉴权的结果--return 
2.  Forbidden 同Challenged  
3.  成功就继续咯 然后这里执行的next()下个中间件  

##### 总结UseAuthorization

也就是几个步骤，比较好理解，只是其中很多细节： 

1.  根据特性标记，找到全部的授权检测要求Requirement和Scheme 
2.  用全部的Scheme全部鉴权一下，都存起来 
3.  匿名特性支持 
4.  用全部的requirement分别检测下 
5.  成功--未登录--未授权分别执行对应的动作 

##### 扩展定制 

1.  自定义IAuthorizeData和AllowAnonymous，代替默认的 
2.  单Requirement扩展+多Requirement扩展 
3.  多Scheme，多Policy，多演练,会有蛮多细节  

##### 多Scheme授权演练 

1.  启动：dotnet run --urls="http://*:5726" ip="127.0.0.1" /port=5726  ConnectionStrings:Write=CommandLineArgumen 
2.  配置：鉴权是UrlToken（默认）+Cookie 
3.  访问： http://localhost:5726/Authorization/Login?name=Eleven&password=123456&rol e=Admin 

Cookie方式--邮箱是57265177@qq.com Role是Admin Country是Chinese  DateOfBirth是1986  自定义UrlToken邮箱是xuyang@ZhaoxiEdu.Net Role是User Country是China  DateOfBirth是1986  

```c#
//默认scheme是UrlTokenScheme
builder.Services.AddAuthentication(options =>
                                   {
                                       options.AddScheme<UrlTokenAuthenticationHandler>(UrlTokenAuthenticationDefaults.AuthenticationScheme, "UrlTokenScheme-Demo");
                                       //其实会保存成key-value     也就是name不能重复  value就是UrlTokenAuthenticationHandler
                                       options.DefaultAuthenticateScheme = UrlTokenAuthenticationDefaults.AuthenticationScheme;
                                       options.DefaultChallengeScheme = UrlTokenAuthenticationDefaults.AuthenticationScheme;
                                       options.DefaultSignInScheme = UrlTokenAuthenticationDefaults.AuthenticationScheme;
                                       options.DefaultForbidScheme = UrlTokenAuthenticationDefaults.AuthenticationScheme;
                                       options.DefaultSignOutScheme = UrlTokenAuthenticationDefaults.AuthenticationScheme;
                                   })
    .AddCookie(CookieAuthenticationDefaults.AuthenticationScheme, options =>
               {
                   //options.ExpireTimeSpan//过期时间
                   options.LoginPath = "/Home/Index";
                   options.AccessDeniedPath = "/Home/Privacy";//有登陆，鉴权成功--没有授权
               })//使用Cookie的方式
    ;
```

##### 多Scheme授权验证总结 

1.  多个Role是并列关系，多个Policy会报错 
2.  单个的Authorize声明，Policy和Role需要同时满足 
3.  多Scheme时，信息合集满足约束就行 
4.  多Policy同时满足---标记多个Authorize特性 
5.  多Policy的或关系？---只能多Requirement 

##### 鉴权授权流程总结 

1.  鉴权授权的时机，在Routing后，在EndPoint前 
2.  二者的交互，通过一个context.User来完成---其实在授权环节，又调用了鉴权 
3.  鉴权的Scheme设计和授权的Policy设计，几乎一样-----通过Options去Add、里面都 是用的Builder、通过注入Options完成初始化、提供key-value式缓存、中间件里面 再通过Provider来获取、找XXService、提供的各种Handler，非常典型的设计 
4.  多Scheme多Policy的设计的有点巧妙有点复杂 
5.  细节太多了，不可能都记住，但是理解很重要，很有底！  

##### JWT 

前面讲清楚了ASP.NET Core鉴权授权 这个是后台框架的一段代码，如何去识别凭证、读取凭证、解析凭证、验证凭证、对凭证授 权等操作 

然后Cookie、UrlToken、JWT都是一种凭证的传递方式，写在不同的地方---数据格式不一样 ---有无加密等等 

然后当下最流行的是JWT，Json Web Token  

######  4W学习法 

学习JWT，还是What Why When How 要讲Why，就得从起源说起： 

1.  Http无状态&轻量级，请求--响应式--传输是文本 
2.  Cookie/Session阶段 
3.  Session共享阶段 
4.  分布式的去中心化需求---Token 
5.  JWT：Json Web Token，是token的一种，json通用格式，语言无关的  

######  JWT信任问题 

A验证账号密码，颁发个Token给U 

U带着Token去访问W---W都不跟A通信，直接认可Token？

U带着Token去访问T-----T都不跟A通信，直接认可Token？

T带着Token去访问W----W都不跟A通信，直接认可Token？ 

 为啥可以这样做？信任基础在哪里？ 加密算法！ 

 解决了信任问题，那去中心化就实现了 而且特别棒，没有瓶颈  

######  加密算法学习 

1.  HS256对称可逆加密算法：双方之间仅共享一个 密钥。由于使用相同的密钥生成签名和验证签名, 因此必须注意确保密钥不被泄密---**加密速度快，字符串短点，性能高**
2.  RS256非对称可逆加密算法：使用公共/私钥对: 标识提供方采用私钥生成签名, 使 用方获取公钥以验证签名。公钥通常通过一个元数据URL提供----**安全性好，但性能低**
3.  HMACSHA256不可逆加密算法：将不同长度字符串转成固定长度且唯一的字符串，不可逆向解密---通常用来做签名防篡改

######  JWT令牌结构 

1.  Header头--{ “alg”: “HS256”, “typ”: “JWT”}—描述下加密算法 
2.  Payload有效载荷---没加密，只是序列化，任何人都可以轻松读取(前端轻松可用) 
3.  Signature签名---防止抵赖，防止篡改 

 签名原理： 

1.  颁发时：账号密码验证通过后，获取用户信息---然后先base64UrlEncode(header)  +“.”+ base64UrlEncode(payload), 得到一个字符串xx.yyyyy 
2.  用HMACSHA256不可逆算法对xx.yyyyy加密，(还有个key参数，就用是公开的解密钥)， 得到固定长度字符串b---(目的是把内容变短) 
3.  然后用加密钥把字符串b加密，得到字符串zzzz，然后JWT就是xx.yyyyy.zzzz 
4.  客户端验证时：先拿到公钥，对zzzz进行解密得到字符串c---能解开证明秘钥是对 的，zzzz来自服务器 
5.  然后对xx.yyyyy进行HMACSHA256加密，得到字符串d，比对字符串c和d---相同则证 明Token信息没有被篡改 
6.  至于有效期啥的，解析后再根据业务要求进阶检测：有效期、其他信息等等 

######  实操JWT-服务端 （HS256）

 建立独立鉴权授权中心，Core WebAPI，完成JWT生成： 

1.  Zhaoxi.NET7.AuthenticationCenter鉴权授权服务器(验证用户，颁发token) 
2.  配置基础信息(对称+不对称) 
3.  Swagger发起请求，完成验证，创建Token 

 dotnet run --urls=http://localhost:7200 http://localhost:7200/swagger/index.html  

######  实操JWT-客户端 

 来个客户端集成JWT鉴权，就用AuthenticationCenter，增加个AuthController： 

1.  添加鉴权授权中间件 
2.  配置鉴权方式 
3.  新增AuthController，标记授权 
4.  配置下Swagger配置接受BearerToken  

######  完整流程验证 

 http://localhost:7200/swagger/index.html 

1.  直接访问无权限要求地址---200 
2.  访问有权限要求地址---401 
3.  登录后获取token 
4.  拿着token登录需要权限认证地址---200 

######  Web使用JWT 

 完整的Web页面登陆+验证使用： 

1.  浏览器请求服务端，服务端调用鉴权中心获取Token 
2.  前端用localStorage存储，ajax请求时放到Header 
3.  Get/Post要求基础鉴权授权 

 也可以前端直接请求鉴权授权中心，获取Token  

```javascript
$("#btnGet").on("click", function () {
    $.ajax({
        url: "/Token/InfoGet",
        type: "get",
        data: {},
        beforeSend: function (XHR) {
            //发送ajax请求之前向http的head里面加入验证信息
            XHR.setRequestHeader('Authorization', 'Bearer ' +  localStorage["token"]);
        },
        success: function (data) {
            alert(data.result);
            alert(data.tValue);
        }
        , datatype: "json"
    })
});
```

######  RSA算法(RS256)

HS256是对称可逆加密，是同一个秘钥，只能用在公司内部

RSA256是非对称可逆加密，是一组密钥对，支持外部第三方使用 

1.  鉴权授权中心升级，使用RSA算法 
2.  启动时生成密钥对，提供公钥获取方法(没用上) 
3.  客户端URL获取，或手动拷贝到本地指定目录 
4.  客户端升级验证方式 
5.  流程验证是一样的---完全不需要改动  

######  JWT与鉴权授权 

 JWT的使用过程，基本上也是一入一出： 

 一入：鉴权授权中心写入Token 

 一出：客户端使用Token信息 

 然后，JWT+鉴权+授权是怎么协同的？ 

1.  JWT只是个数据传输格式，基于加密算法完成信任---然后其信息是公开的，配置一 下后自动解析，其实也能自定义解析，前端js也能解析----JWTTokenDeserialize.  AnalysisToken(token) 
2.  鉴权，就是在通过指定位置读取Token信息，然后解析，然后写入到context.User 
3.  授权，这是根据这个context.User完成权限检测  

######  JWT+鉴权 

 鉴权需要的几个信息： 

1. 数据来源：基于Scheme知悉
2. 数据格式：基于Scheme知悉
3. 检测要求：TokenValidationParameters，是否检测、值、检测方式
4. 处理事件：JwtBearerEvents—也是扩展处理动作---观察者模式扩展
5. 数据传递：base.HttpContext.User.Identities---数据最好是CliamType类型的  

######  JWT+授权 

 JWT+授权，跟Cookie UrlToken没啥区别，因为授权跟Scheme没啥关联： 

1.  JWT+授权，跟Cookie UrlToken没啥区别，因为授权跟Scheme没啥关联： 
2.  策略授权 
3.  各种自定义授权 

 授权跟鉴权的方式是没有任何关系的 鉴权是获取用户信息 授权是检测用户信息是否合格  

######  AddCookie源码 

 AddCookie的源码看看，其实跟AddJWT差别不大，在ASP.NET Core源码里： 

1.  通过AuthenticationBuilder来AddScheme，还是Add到Option—就是AddScheme---  CookieAuthenticationHandler 

2.  CookieAuthenticationHandler多层父类，其实落脚点在AuthenticationHandler 

3.  几个核心方法在父类，子类override一下 

   InitializeAsync：完成初始化 

   ForbidAsync：搞个虚方法，403--给子类覆写 

   ChallengeAsync：搞个虚方法，401--给子类覆写 

   AuthenticateAsync：去子类的看HandleAuthenticateAsync，EnsureCookieTicket就是 去读取cookie值---CheckForRefreshAsync 做滑动过期---然后做检验---然后发现了 Event的执行 

   Event在AuthenticationHandler就初始化对象时完成初始化，用Options的Event---然后 在整个生命周期中，就直接调用该事件  

######  AddJWT源码 

 AddJWT跟AddCookie是几乎一样的 

1.  通过AuthenticationBuilder来AddScheme，还是Add到Option 

2.  靠的是JwtBearerHandler来解析 

3.  几个核心方法在父类，子类override一下 

4.  植入多个Event动作，观察者，用来扩展执行流程中的各种动作 

   InitializeAsync：完成初始化 

   ForbidAsync：搞个虚方法，403--给子类覆写 

   ChallengeAsync：搞个虚方法，401--给子类覆写 

   AuthenticateAsync：去子类的看HandleAuthenticateAsync--Events.MessageReceived- --然后按规则读取Token---然后根据TokenValidationParameters遍历校验---一系列的 Event扩展支持  

######  JWT鉴权授权扩展 

 试试扩展： 

1.  多属性验证以及验证逻辑扩展TokenValidationParameters 
2.  JWT鉴权多事件扩展，Event 
3.  JWT灵活授权扩展 
4.  多Scheme多Policy，跟之前一样  

###### 阶段总结 

 JWT是什么，以及JWT跟ASP.NET Core对接使用，应该很清晰了 各种鉴权 各种授权—动态授权 

 但JWT还存在一些局限性：  Token泄露、滑动过期、改密码等 

######  JWT局限性 

​	Token机制是通过加密算法来保证Token的来源，以及准确性 因此可以做到去中心化验证 

​	既然去中心化了，那就有几个问题无法解决： 

- Token泄露，安全问题 

- 修改密码/删除用户，Token失效问题
- Token能做到滑动过期吗？ 

   其实都是去中心化带来的问题， 或者说是JWT机制带来的问题  

1.  Token泄露 

    关乎Token泄露问题： 

   -  SSL通信，传输中途不会窃取；但是到了浏览器确实没办法，别人能拿到浏览器，那 真的没办法---(Cookie或者其他方案都没区别) 
   -  额外验证点东西，IP地址，浏览器类型(写在Validator---Event，或者自己额外写 在授权环节)---做的少，非常规 

   重放攻击：无止境的重复请求----请求表单带个随机数---随机数搞个Redis---执行前先 Redis一下(随机数不是在token)  

2.  Token过期问题 

    修改密码、删除用户，希望让Token立即失效---目前是做不到，有以下思路： 

   -  修改密码/删除用户，就更新秘钥，所有的token都失效—而且不断变证书—不可取 

   -   要想及时过期，客户端和鉴权授权中心必须有通信！ 

      生成token时—除了生成token(含guid)—还生成个guid+用户id—写入redis 

      验证token时---拿guid去redis校验 

      改密码---redis那一项数据—之前的给删掉/过期/无效

      验证旧token—发现过期

      验证新token就没事儿 有企业在用，一般是局域网式—但是比较有限制，不太JWT 

   -  减少有效期---降低伤害，这也算是一个思路  

3.  滑动过期 

    不能用着用着，突然说我过期了，要重新登陆---就是要修改token的有效期？--不行！ Token是不会变的，而且只能鉴权授权中心发的 

    不能默认检测有效期，可以扩展自定义校验(或者写在授权环节)---第一次校验成功后把 token写一下Redis(有效期30分钟)—再往后的检测就是：要么Redis有，可以直接用-用 了修改有效期；没有的话就检测token有效期----能做到滑动过期，也是依赖于Redis 

    其实，更推荐用的叫刷新Token  

######  刷新Token设计 

 更推荐的是刷新Token设计，业界主流方式： 

1.  也叫双Token，一个AccessToken，一个RefreshToken，有效期不一样，AccessToken 有效期60分钟，RefreshToken有效期7x24小时 
2.  常规请求是AccessToken，超时后程序自动用用RefreshToken去授权中心获取新的 AccessToken（不是账号密码---是为了自动请求，不需要用户操作，无感） 
3.  颁发token时，把RefreshToken缓存一下，再次请求直接看缓存，颁发新的Token， 一直到RefreshToken过期，就结束了，才重新登陆 

 既能长时间有效，又能定期去验证一下，而且用户无感  

![](.\img\企业微信截图_16822996415426.png)

######  刷新Token全流程 

 鉴权授权中心升级： 

1.  支持双Token获取，和缓存 
2.  支持Token刷新获取 
3.  refreshToken管理：A 有效期校验 B 清除 

 基于swagger完成测试： 

1.  调用LoginWithRefresh获取双Token 
2.  基于refreshToken去调用RefreshToken 

 客户端支持刷新Token，包括前端和后端： 

1.  后端得支持双Token，支持刷新Token 
2.  前端支持，N多前端代码细节 
3.  后端JWT校验事件支持 

 每隔1分钟，其实accesstoken都要过期---就会跟授权服务器通信一次---全过程是无感 的 

 流程校验： 

1.  先登陆---然后点击，是正常的 
2.  过期后，再点击---过期+刷新Token+自动点击，通了  

######  总结刷新Token 

1.  其实就是JWT，只是加了端编程逻辑 
2.  客户端定期跟服务端通信一次，频率可控---防止数据长期滞后 
3.  做好封装，全程自动化的 

JWT的局限性

1.  泄露安全----------没用 
2. Token失效---非实时，但可控 
3. 滑动过期-----直接长有效期  

#####  RateLimiter限流 

 ASP.NET Core7.0新增的中间件，限流中间件，提供访问速率限制 

1.  理解限流？其实是为了程序的高可用---一个景区只能容纳5000人，真的放进来1w人 那就毫无体验可言，上海迪士尼就有票售罄-----程序只能承载100QPS，如果来1000， 大概率会挂----为了可用，限制请求 
2.  如何使用？Use + Add+ 标记 
3.  多策略： 
   - 固定窗口
   - 滑动窗口
   - 令牌存储桶
   - 并发
4.  应用场景：WebAPI可能需要限制下—来不及调优，就简单粗暴 

官方文档： [ASP.NET Core 中的速率限制中间件 | Microsoft Learn](https://learn.microsoft.com/zh-cn/aspnet/core/performance/rate-limit?preserve-view=true&view=aspnetcore-7.0#fixed) 

######  固定窗口策略 

 固定窗口策略，最简单也是最容易实现的一种算法，但最有问题 

 从第一个请求开始计时，在一个时间段内(窗口)请求数不能超过100 

 00:00:01~00:01:00---最多只能100请求，更多就排队，甚至放弃 

 00:01:01~00:02:00---排队的执行，接收新请求，加起来不超过100 

 缺陷：可能在临界点，集中遭遇请求，出现风险  

![](.\img\企业微信截图_16823002848707.png)

######  滑动窗口策略 

 滑动窗口策略，是改进了固定窗口策略： 

 加权计数添加到当前窗口中的计数来计算估计数，如果估计数超过计数限制，则请求将 被阻止。 

 估计数 = 前一窗口计数 * (1 - 当前窗口经过时间 / 单位时间) + 当前窗口计数 避免请求集中添加，会分散一点  

![](.\img\企业微信截图_16823004056665.png)

######  令牌桶策略 

 令牌桶算法是比较常见的限流算法之一 

1. 所有的请求在处理之前都需要拿到一个可用的令牌才会被处理 
2. 根据限流大小，设置按照一定的速率往桶里添加令牌； 
3. 桶设置最大的放置令牌限制，当桶满时、新添加的令牌就被丢弃或者拒绝； 
4. 请求达到后首先要获取令牌桶中的令牌，拿着令牌才可以进行其他的业务逻辑，处 理完业务逻辑之后，将令牌直接删除； 
5. 令牌桶有最低限额，当桶中的令牌达到最低限额的时候，请求处理完之后将不会删 除令牌，以此保证足够的限流；  

![](.\img\企业微信截图_16823004176294.png)

######  并发限制器 

并发限制程序限制并发请求数。 每个请求都会将并发限制减少 1。请求完成后，限制将 增加 1。 

与限制指定时间段内请求总数的其他请求限制器不同，并发限制程序仅限制并发请求数， 不限制一段时间内的请求数。 

 (等同于一个初始化固定数量的令牌桶) 

##### FilterPipeline 

######  流程衔接点 

MVC(Filter+Controller+Action+Result)流程和前面的各种中间件Middleware处理，是如何衔接起来的？  

1.  Http请求穿过一系列中间件，最终由路由RoutingMiddleware匹配得到 RouteEndpoint 
2.  EndpointMiddleware里面会执行其RequestDelegate，委托执行时就是Filter-Controller-Action-Result 
3.  该委托在ControllerRequestDelegateFactory里面来构建的，里面通过 ControllerActionInvoker来调用 

这里有个新名字叫 FilterPipeline 

图1：Middleware的管道模型设计图 

![](.\img\75040b0f33f447d192afbd2947dad13c.png)

图2：ASP.NET Core MVC的Filter流程图  

![](.\img\e18b27857c5d4f22c266d8c05e121fcd.png)

######  快速认识Filter 

Filter是MVC里面核心组成部分，有着丰富的种类，交叉的顺序，复杂的嵌套，很有挑战！ 

1.  声明特性，实现Filter接口 

   CustomSimpleShowActionFilterAttribute

   CustomSimpleShowAsyncActionFilterAttribute

   CustomSimpleShowDoubleActionFilterAttribute:

    一个Filter同时实现同步和异步接口，只执行异步实现！ 能改参数，能改结果，非常强大

   注意异步版本的结果修改坑 

2.  控制器类实现Filter接口 

    包括Action同步异步-Result同步异步---控制器生效，全部Action都有效  

######  AOP面向切面编程 

要说Filter，得理解下AOP面向切面编程 Aspect Oriented Programming 