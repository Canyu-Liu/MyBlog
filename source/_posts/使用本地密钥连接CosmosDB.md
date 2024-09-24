---
title: 在.NET中使用本地密钥连接到 Azure Cosmos DB
date: 2023-06-12 20:19:10
tags: 
- Azure
- Azure Cosmos DB
- 数据库
categories:
- [Azure,Azure Cosmos DB]
---
使用本地密钥保存连接字符串以提高安全性。
<!--more-->
在校那会的课堂作业和项目实践一直都是把连接字符串写在代码里，到自己部署项目了才发觉这样做有多么不安全。（只在自己电脑折腾的话还是怎么舒服怎么写）
这里用的是Azure Cosmos DB，配置好连接之后会生成一个本地用户机密文件。
![本地密钥](/uploads/213341.jpg)
虽然本地密钥也是明文保存连接字符串，但至少它没有直接暴露在代码里，尤其是在使用公共存储库的时候，不用担心一个不留神把连接字符串也给别人看见了。
配置方法也非常简单，给CosmosClient写个依赖注入，然后从配置里读取连接字符串就可以了。
{% codeblock Program.cs lang:csharp %}
builder.Services.AddSingleton<CosmosClient>(serviceProvider =>
{
    var configuration = serviceProvider.GetRequiredService<IConfiguration>();
    // 这里的ConnectionStrings是本地密钥文件的键。
    var connectionString = configuration["ConnectionStrings"];
    return new CosmosClient(connectionString);
});
{% endcodeblock %}
其他数据库同理。上云的话还是买云服务器厂商提供的密钥保管服务更安全，也更方便，就是得花点小钱。