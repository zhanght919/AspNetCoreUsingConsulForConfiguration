# AspNetCoreUsingConsulForConfiguration
## 使用Consul作为配置中心，配置Asp.Net Core应用程序
**背景**：通常，.Net 应用程序中的配置存储在配置文件中，例如 App.config、Web.config 或 appsettings.json。从 ASP.Net Core 开始，出现了一个新的可扩展配置框架，它允许将配置存储在配置文件之外，并从命令行、环境变量等等中检索它们。
配置文件的问题是它们很难管理。实际上，我们通常最终做法是使用配置文件和对应的转换文件，来覆盖每个环境。它们需要与 dll 一起部署，因此，更改配置意味着重新部署配置文件和 dll 。不太方便。

因此我们通过Consul在线实时配置，则达到了只更改配置不重启服务即可实时响应的目的。

下面来介绍一下使用 **Winton.Extensions.Configuration.Consul** 来做配置中心。

首先，**安装Consul**，具体安装步骤本文就不详细介绍了，当打开 http://127.0.0.1:8500 可以看到Consul的UI界面代表安装成功。

然后，**安装NuGet包**：Winton.Extensions.Configuration.Consul

然后，**更改Program.cs ConfigureAppConfiguration配置**
```
public class Program
    {
        public static void Main(string[] args)
        {
            CreateWebHostBuilder(args).Build().Run();
        }

        public static IWebHostBuilder CreateWebHostBuilder(string[] args)
        {
            var cancellationTokenSource = new CancellationTokenSource();
            return WebHost.CreateDefaultBuilder(args)
                .ConfigureAppConfiguration((hostingContext, config) =>
                {
                    var env = hostingContext.HostingEnvironment;
                    hostingContext.Configuration = config.Build();
                    string consul_url = hostingContext.Configuration["Consul_Url"];
                    config.AddConsul(
                                $"{env.ApplicationName}/appsettings.{env.EnvironmentName}.json",
                                cancellationTokenSource.Token,
                                options =>
                                {
                                    options.Optional = true;
                                    options.ReloadOnChange = true;
                                    options.OnLoadException = exceptionContext => { exceptionContext.Ignore = true; };
                                    options.ConsulConfigurationOptions = cco => { cco.Address = new Uri(consul_url); };
                                }
                                );

                    hostingContext.Configuration = config.Build();
                })
                 .UseStartup<Startup>();
        }
    }
```
从以上代码 $"{env.ApplicationName}/appsettings.{env.EnvironmentName}.json"  
可以看出，我们**首先需要在Consul界面中以项目名称创建文件夹**即
![avatar](https://img-blog.csdnimg.cn/20190711111155899.png)
然后**再以环境变量文件名创建Key**,Value则为项目中appsettings.{env.EnvironmentName}.json的文件内容，全部拷贝即可。
![avatar](https://img-blog.csdnimg.cn/20190711112358353.png)

这时打开项目，就可以通过配置Consul中的value值，来配置Asp.Net Core项目了。 

**原文地址**：https://tech.winton.com/2016/12/configuring-net-core-applications-using-consul/

**官方Demo GitHub地址**：https://github.com/wintoncode/Winton.Extensions.Configuration.Consul/tree/master/test/Website
