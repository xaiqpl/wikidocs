---
title: Grpc记录
description: 
published: true
date: 2025-08-04T07:23:22.471Z
tags: grpc
editor: markdown
dateCreated: 2025-07-29T05:30:31.572Z
---

# gRPC 入门
## gRPC 简介
> gRPC是一个开源的高性能RPC框架，利用HTTP/2协议和Protocol Buffers进行通信。
> 通过定义.proto文件来创建服务接口和消息类型，并在代码（C#）中实现这些接口。
> 一般项目内容包括protobuf文件的定义、服务接口的实现、gRPC服务器的启动以及客户端调用方法，还包括了错误处理和流式传输的概念。
{.is-info}


## 1. gRPC和Protocol Buffers简介
> 在现代的微服务架构中，gRPC已经成为一种高效、跨语言的通信机制，它基于HTTP/2协议，提供强大的RPC（远程过程调用）支持。gRPC使用Protocol Buffers作为接口定义语言和消息序列化格式，旨在简化分布式系统中的服务间通信。
> 
> gRPC的多语言支持能力让它成为了构建微服务应用的理想选择。通过定义服务接口，gRPC能够自动生成客户端和服务端的代码，极大地加快开发周期。此外，它还支持多种通信类型，包括简单RPC、服务器端流式RPC、客户端流式RPC以及双向流式RPC，提供了灵活的交互模式。
{.is-info}


## 2. Protobuf文件定义与数据序列化
### 2.1 Protobuf语法基础
> Protocol Buffers（protobuf）是一种语言无关、平台无关的可扩展机制，用于序列化结构化数据，类似于XML或JSON，但更小、更快、更简单。protobuf在二进制格式中定义数据结构，它允许您定义数据结构后，可以使用特殊生成的代码轻松地在各种数据流中读写结构化数据。在protobuf中定义消息时，需要遵循特定的语法规则。这些规则允许您定义数据类型、字段以及如何序列化这些字段。
{.is-info}


下面是一个protobuf基本消息定义的示例：

```proto
syntax = "proto3";//proto3 是 Protocol Buffers 的第三个版本
 
// 指定了生成的 C# 代码的命名空间为 LinkService。当使用 protobuf 编译器 (protoc) 将这个 .proto 文件转换为 C# 代码时，生成的类将位于 LinkService 命名空间中
option csharp_namespace = "LinkService";
 package TClient.UIToServer;
 import "Protos/Message.proto";
//定义了一个名为 Link 的 gRPC 服务。在 gRPC 中，服务是由一个或多个 RPC 方法组成的
service Link
{
    //定义了一个 RPC 方法。这个方法名为 GetMessage，它接受一个 Mes 类型的消息作为参数，
    //并返回一个 Mes 类型的消息。在 gRPC 中，客户端可以调用这个方法，并发送一个 Mes 消息给服务端，然后服务端会处理这个消息并返回一个 Mes 消息给客户端。
    rpc GetMessage(Mes) returns (Mes);
}
 
//定义了一个名为 Mes 的消息类型。在 protobuf 中，消息是由一系列字段组成的，每个字段都有一个名称、一个类型和一个标识符。
message Mes
{
    // 客户端发送
    // 定义了一个名为 StrRequest 的字段，类型为 string，标识符为 1。这个标识符在消息内部是唯一的，并且一旦分配就不能更改，因为它被用于序列化和反序列化过程中的字段识别。
    string StrRequest = 1;
    // 同样定义了一个名为 StrReply 的字段，类型为 string，标识符为 2。(服务端回复)
    string StrReply = 2;
}
message MesList{
    repeated Mes mesList = 1; // repeated 表示这个字段可以包含多个 Mes 类型的消息
}
```
 在这个消息定义中：

> syntax = "proto3"; 这行指定了语法版本是proto3。必须在第一条语句{.is-info}

>  option csharp_namespace = "LinkService"; 
> 指定了生成的 C# 代码的命名空间为 LinkService。当使用 protobuf 编译器 (protoc) 将这个 .proto 文件转换为 C# 代码时，生成的类将位于 LinkService 命名空间中如果不写，生成的 C# 命名空间会根据 package 名称自动推断，但通常会带上下划线或点号，和项目结构不够贴合。{.is-info}

>  package TClient.UIToServer; 
> 定义了一个包名，它有助于防止不同项目之间的命名冲突，C#中可被csharp_namespace覆盖{.is-info}

>  message Mes { ... } 
> 定义了一个名为 Mes 的消息类型。在 protobuf 中，消息是由一系列字段组成的，每个字段都有一个名称、一个类型和一个标识符。
> 每个字段都有如下格式：
>  **字段类型（如 string , int32 ） 	字段名称 	字段编号（例如 1 , 2 , 3 ）**
> 字段编号是在二进制消息中识别字段的唯一标识符，并且一旦使用，就不应更改，以防止数据破坏。在protobuf中，字段编号从1开始，且通常使用连续的数字为新字段分配编号。
{.is-info}

>  repeated Mes mesList = 1; // repeated 表示这个字段可以包含多个 Mes 类型的消息
>{.is-info}

> import "Protos/Message.proto";
> 把同目录（或 protoc 指定的 –proto_path 下）名为 Messages.proto 的文件内容“包含进来”。
使本文件能够引用 Messages.proto 中定义的 message、enum、service 等类型。
{.is-info}

### 2.2 protobuf数据类型和结构
基本数据类
protobuf支持多种基本数据类型，包括布尔值、数值类型（整数、浮点数等）、字符串以及字节类型。每种类型都对应特定的编码方式，以确保在二进制格式下以最小的数据量表示。

例如，定义消息时可以使用如下基本数据类型：
```
message BasicTypes {
  bool flag = 1;    //bool flag 表示布尔值，占用一个字节。
  int32 number = 2; //int32 number 是一个32位的有符号整数，用于较小的整数。
  float fnum = 3;   //float fnum 是32位的IEEE 754浮点数。
  double dnum = 4;  //double dnum 是64位的IEEE 754浮点数。
  string text = 5;  //string text 表示字符串，用UTF-8编码，并且会加上长度前缀。
  bytes bin_data = 6; //bytes bin_data 表示字节类型，用于存储任意的字节数据。
}
```
 #### 枚举和嵌套消息
 ```
 message EnumMessage {
  enum Color {
    RED = 0;
    GREEN = 1;
    BLUE = 2;
  }
  Color favoriteColor = 1;
  message Contact {
    string name = 1;
    int32 phone_number = 2;
  }
  Contact contact_info = 2;
}
 ```
 
### 2.3 服务定义与RPC方法
> 在定义了数据结构之后，protobuf还允许您定义服务接口，这是gRPC的基础。
gRPC是一个高性能、开源和通用的RPC框架，使用protobuf作为其接口定义语言(IDL)来描述服务接口以及方法和服务之间的消息结构。使用protobuf定义服务后，就可以使用gRPC工具自动生成服务器和客户端存根代码，这简化了远程过程调用(RPC)的实现。
{.is-info}

下面是如何定义一个简单的protobuf服务：
``` java
syntax = "proto3";
package example;
//定义了一个名为 Link 的 gRPC 服务。在 gRPC 中，服务是由一个或多个 RPC 方法组成的
service Link
{
    //定义了一个 RPC 方法。这个方法名为 GetMessage，它接受一个 Mes 类型的消息作为参数，
    //并返回一个 Mes 类型的消息。在 gRPC 中，客户端可以调用这个方法，并发送一个 Mes 消息给服务端，然后服务端会处理这个消息并返回一个 Mes 消息给客户端。
    rpc GetMessage(Mes) returns (Mes);
}
 
//定义了一个名为 Mes 的消息类型。在 protobuf 中，消息是由一系列字段组成的，每个字段都有一个名称、一个类型和一个标识符。
message Mes
{
    // 客户端发送
    // 定义了一个名为 StrRequest 的字段，类型为 string，标识符为 1。这个标识符在消息内部是唯一的，并且一旦分配就不能更改，因为它被用于序列化和反序列化过程中的字段识别。
    string StrRequest = 1;
    // 同样定义了一个名为 StrReply 的字段，类型为 string，标识符为 2。(服务端回复)
    string StrReply = 2;
}
```
在这个服务定义中：
> service Link { ... } 定义了一个服务接口。
>   rpc GetMessage(Mes) returns (Mes); 定义了一个名为 GetMessage 的RPC方法，它接收一个 Mes 消息作为请求，并返回一个 Mes 消息作为响应。
> 
> 这种服务定义后，gRPC可以生成客户端和服务端代码，这样开发者就可以专注于实现服务逻辑，而不必担心通信协议细节。
{.is-info}

## 3 C#中服务接口的定义与实现
### 3.1 定义服务接口和方法
#### 添加必要的引用
> **Google.Protobuf** ：
实现 Protocol Buffers（protobuf）的序列化与反序列化。
任何时候你需要对 protobuf 消息进行编码/解码（无论是通过 gRPC 还是直接使用 protobuf 存储/传输）都要引用它。
> **Grpc.Core**：
提供 gRPC 客户端与服务器的核心运行时。
> **Grpc.Tools** ：
在编译阶段提供对 .proto 文件的处理支持。
{.is-success}

> 在C#中定义服务接口涉及到从protobuf定义生成C#代码，并在此基础上定义服务接口和方法。为了在C#中使用protobuf定义的消息和服务接口，首先需要使用protobuf编译器 protoc 生成C#类。
{.is-info}

在项目文件中添加以下配置后，可自动触发源代码生成器生成代码：
``` html
<ItemGroup>
  <!-- 只生成服务端基类,*.proto匹配目录下所有proto文件  -->
  <Protobuf Include="Protos\*.proto" GrpcServices="Server" />
</ItemGroup>
<ItemGroup>
  <!-- 只生成客户端存根  *.proto匹配目录下所有proto文件-->
  <Protobuf Include="Protos\*.proto" GrpcServices="Client" />
</ItemGroup>
GrpcServices="Both"：同时生成 FooBase + FooClient，常用于 Shared 库。
GrpcServices="None"（或省略此属性）：只生成消息类型（message、enum），但不生成任何 RPC 存根。
```


#### 实现服务接口
> 借助 Grpc.Tools NuGet包，可以将protobuf文件中的服务定义转换为C#中的接口。
> 代码中定义了 LinkServerFunc 类，该类实现了 LinkBase ，并重写了 GetMessage、GetMessageList方法，该方法处理消息并返回相应的回复。
{.is-info}

示例如下：
``` csharp
using Grpc.Core;
using GrpcDemo;
using LinkService;
using static LinkService.Link;

namespace TGrpcService
{
    public class LinkServerFunc:LinkBase
    {
        public override Task<Mes> GetMessage(Mes request, ServerCallContext context)
        {
            // 处理接收到的消息
            Mes mes = new Mes();
            mes.StrReply=LinkFunc.ReplyMes(request.StrRequest);
            return Task.FromResult(mes);
        }
        public override Task<MesList> GetMessageList(Mes request, ServerCallContext context)
        {
            // 处理接收到的消息列表
            MesList mesList = new MesList();            		mesList.MesList_.AddRange(LinkFunc.ReplyMesList(request.StrRequest));
            return Task.FromResult(mesList);
        }
    }
```
#### 启用服务监听
开发者需要创建服务端应用程序来承载 LinkServerFunc 服务。
``` csharp
 public class LinkFunc
 {
 			//创建委托，在外部进行实现
     public static Func<string, string> ReplyMes;
     public static Func<string,IEnumerable<Mes>> ReplyMesList;
     //定义服务 
     public static Server LinkServer;
     //启动服务监听
     public static void LinkServerStart(string host, int port)
     {
         LinkServer = new Server
         {
            // Services 处可同时添加多个服务监听
            //当你有多个 .proto 文件，每个生成一组服务存根（stubs）时，只需要在创建 Server 对象时，把每个服务的 BindService 调用都加到 Services 集合里。
             Services =
                 {
                   Link.BindService(new LinkServerFunc()),
                   GrpcDemo.Greeter.BindService(new GreeterService())

                 },
             Ports = { new ServerPort(host, port, ServerCredentials.Insecure) }
         };
         LinkServer.Start();
     }
     /// <summary>
     ///  服务端关闭
     /// </summary>
     public static void LinkServerClose()
     {
         LinkServer?.ShutdownAsync().Wait();
     }
 }
```
可以通过创建S额viceIntercept来拦截服务消息的接受和发送情况。
``` java
public class ServiceInterceptor : Interceptor
{
    private readonly string _serviceName;
    public ServiceInterceptor(string serviceName)
    {
        _serviceName = serviceName;
    }
    public override async Task<TResponse> UnaryServerHandler<TRequest, TResponse>(TRequest request, ServerCallContext context, UnaryServerMethod<TRequest, TResponse> continuation)
    {
        // 在这里可以添加日志、验证等逻辑
        Metadata requestHeader;
        if (context.RequestHeaders == null)
        {
            requestHeader = new Metadata();
        }
        else
        {
            requestHeader = context.RequestHeaders;
        }
        Console.WriteLine($"***Service: {_serviceName}, Begin, Method: {context.Method}, Request: {request}");
        var response = await base.UnaryServerHandler(request, context, continuation);
        Console.WriteLine($"***Service: {_serviceName}, End, Method: {context.Method}, response: {response}");
        return response;
    }
}
```
 在创建给Servers添加Serve时，添加通过服务对象调用Intercept返回的ServiceDefinition可以实现消息拦截
> LinkServer = new Server
> {
>     Services =
>         {
>           Link.BindService(new LinkServerFunc()).Intercept(new ServiceInterceptor("LinK:")),
>           GrpcDemo.Greeter.BindService(new GreeterService()).Intercept(new ServiceInterceptor("Greeter:"))
> 
>         },
>     Ports = { new ServerPort(host, port, ServerCredentials.Insecure) }
> }; 
{.is-info}

在前面LinkFunc中我们创建了相应的服务接口委托，在使用的时候我们需要给注册上相应的函数。
``` csharp
    static void Main(string[] args)
    {
        Console.WriteLine("Hello, World!");
        LinkFunc.LinkServerStart("127.0.0.1", 9008);
        Thread.Sleep(500);
        LinkFunc.ReplyMes = ReplyMes;//注册函数
        LinkFunc.ReplyMesList = ReplyList;//注册函数
        Console.ReadKey();
    }

    private static IEnumerable<Mes> ReplyList(string strRequest)
    {
        Console.WriteLine("接收到:" + strRequest);
        switch (strRequest)
        {
            case "1":
                return new List<Mes> { new Mes { StrReply = "Server识别到1" }, new Mes { StrReply = "Server识别到11" }, new Mes { StrReply = "Server识别到111" } };
            case "2":
                return new List<Mes> { new Mes { StrReply = "Server识别到2" }, new Mes { StrReply = "Server识别到22" }, new Mes { StrReply = "Server识别到222" } };
            case "测试":
                return new List<Mes> { new Mes { StrReply = "开始测试" }, new Mes { StrReply = "开始测试2" } };
            case "连接服务端":
                return new List<Mes> { new Mes { StrReply = "true" } , new Mes { StrReply = "true2" } };
        }
        return new List<Mes> { new Mes { StrReply = "Server未识别到指定参数" } };
    }

    /// <summary>
    /// 接收到客户端信息后回复
    /// </summary>
    /// <param name="strRequest">客户端发送过来的内容</param>
    /// <returns></returns>
    public static string ReplyMes(string strRequest)
    {
        Console.WriteLine("接收到:" + strRequest);
        switch (strRequest)
        {
            case "1":
                return "Server识别到1";
            case "2":
                return "Server识别到2";
            case "测试":
                return "开始测试";
            case "连接服务端":
                return "true";
        }
        return "Server未识别到指定参数";
    }
}
```
## 4 C#中客户端接口的定义与调用
### 3.1 定义服务接口
#### 添加必要的引用
> **Google.Protobuf** ：
实现 Protocol Buffers（protobuf）的序列化与反序列化。
> **Grpc.Core**：
提供 gRPC 客户端与服务器的核心运行时。
> **Grpc.Tools** ：
在编译阶段提供对 .proto 文件的处理支持。
{.is-success}