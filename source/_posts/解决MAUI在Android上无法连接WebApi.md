---
title: 解决MAUI在Android上无法连接Web Api
date: 2024-6-11 23:53:02
tags: 
- MAUI
categories:
- [.NET,MAUI]
---
最近在给自动异议做联网检查更新的功能，做完联网就拿去备案。但出乎意料的，HttpClient出现问题了。
![](/uploads/225931.jpg)
<!--more-->
## 问题描述
在Windows上调试时，使用HttpClient来发送网络请求能够正常获取返回的数据，但是部署到Android上运行时会遇到网络错误。
{% codeblock HttpClientService.cs lang:csharp %}
public class HttpClientService
{
    private static HttpClient sharedClient = new()
    {
        BaseAddress = new Uri("http://example.com"),
        //这只是一个示例地址
    };
    public static async Task<Model> GetVersionAsync()
    {
        try
        {
            using HttpResponseMessage response = await sharedClient.GetAsync("version");
            response.EnsureSuccessStatusCode();
            var jsonResponse = await response.Content.ReadAsStringAsync();
            var versionInfo = JsonSerializer.Deserialize<Model>(jsonResponse);
            return versionInfo;
        }
        catch (HttpRequestException e)
        {
            return null;
        }
    }
}
{% endcodeblock %}
很正常的代码，看不出什么问题。但在Android上运行时必定会进入异常处理，然后返回一个null。
## 故障排查
检查应用网络权限，已经允许访问网络。
使用Android的浏览器访问Web Api，可以正常访问。
断点调试，没有在Android的特定平台代码发现问题。
换了一个公共api，可以正常访问了，仔细观察发现公共api配置了https，而我的api是http。
## 解决方案
使用https即可正常访问...
MAUI似乎在Android上禁用了明文流量，需要使用https。
如果非要用http的话，必须手动配置启用明文流量。
修改Android平台文件下的MainApplication.cs：
{% codeblock MainApplication.cs lang:csharp %}
#if DEBUG
[Application(UsesCleartextTraffic = true)]
#else
[Application]
#endif
public class MainApplication : MauiApplication
{
    public MainApplication(IntPtr handle, JniHandleOwnership ownership)
        : base(handle, ownership)
    {
    }

    protected override MauiApp CreateMauiApp() => MauiProgram.CreateMauiApp();
}
{% endcodeblock %}
我选择使用https，签个ssl证书也不是什么麻烦事。

## 参考
[从 Android 仿真器和 iOS 模拟器连接到本地 Web 服务](https://learn.microsoft.com/dotnet/maui/data-cloud/local-web-services?wt.mc_id=studentamb_255264&view=net-maui-8.0#enable-clear-text-network-traffic-for-all-domains)