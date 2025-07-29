---
title: Grpc记录
description: 
published: true
date: 2025-07-29T05:55:09.052Z
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
```
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

