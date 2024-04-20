---
title: Nmap识别MongoDB 6.0指纹
date: 2023-06-30 22:55:27
author: repoog
excerpt: Nmap工具常常被用来识别端口以及端口对应的系统服务或系统应用，在安全测试过程中能够识别更为详细的服务版本也是相当重要的。但Nmap不一定能够对所有的服务版本都能够识别，本文解决的是Nmap识别MongoDB 6.0的问题，通过手动修改Nmap的指纹文件识别该服务版本。
tags:
  - MongoDB
  - Nmap
  - 指纹识别
  - 网络扫描
categories:
  - 系统安全
---

朋友反馈一个问题，说使用Nmap扫描MongoDB服务时对于6.0以上的版本默认无法识别到服务版本信息。

![Nmap无法识别MongoDB版本](images/2023/06/probe-problem.png 'Nmap无法识别MongoDB版本')

如上图所示，对应的VERSION信息是空，在提示信息中可以看到，官方推荐将指纹信息上传以帮助更新服务指纹，又或者可以通过Nmap的默认脚本mongodb-info来做版本识别。前一种方式需要等待官方做指纹文件更新或产品版本更新，后者使用脚本又会影响到扫描速度。如果想在当前的Nmap版本下不使用脚本来识别MongoDB 6.0以上版本，唯一的办法是使用Nmap内置的服务指纹将MongoDB 6.0服务版本识别出来。

### **Nmap服务指纹解析**

Nmap默认的服务指纹文件位于/usr/share/nmap目录下，文件名是nmap-service-probes，根据默认端口27017可以找到Nmap识别MongoDB服务的内容：

![](images/2023/06/service-probe.png)

上图中的匹配规则用到了四个指令：

* Probe：为识别服务发送的字符串信息；
* rarity：1-9范围的数字，是服务识别和可靠性的概率，数字越大，扫描概率越小，返回信息价值越低；
* ports：服务识别常用的端口，效果与nmap的-p参数相同；
* match：根据Probe指令发送字符串返回的信息，匹配服务指纹规则；

上图中用于识别MongoDB服务，通过TCP协议发送到服务端的字符串是：

```
\x41\0\0\0\x3a\x30\0\0\xff\xff\xff\xff\xd4\x07\0\0\0\0\0\0test.$cmd\0\0\0\0\0\xff\xff\xff\xff\x1b\0\0\0\x01serverStatus\0\0\0\0\0\0\0\xf0\x3f\0
```

match指令的格式包括：

* 服务名称（service）
* 匹配规则（pattern）
* 版本信息（versioninfo）

以第一行的match指令为例：

```
match mongodb m^.*version.....([\.\d]+)s p/MongoDB/ v/$1/ cpe:/a:mongodb:mongodb:$1/
```

* 服务名称：mongodb
* 匹配规则：m^.\*version.....(\[\\.\\d\]+)s，采用Perl格式的正则表达式，s表示包含换行符；
* 版本信息：p/MongoDB/ v/$1/ cpe:/a:mongodb:mongodb:$1/，其中p指的是厂商信息，v指的是版本信息，这里引用自匹配规则里匹配的参数，cpe指的CPE（Common Platform Enumeration）格式，用于识别服务、操作系统和硬件；

CPE的格式如下：

```
cpe:/<part>:<vendor>:<product>:<version>:<update>:<edition>:<language>
```

除了v指令之外，CPE的第4个值也是产品版本信息，因此上图匹配规则中，只有第一行的规则可以用于识别MongoDB服务版本，通过匹配返回信息中的version信息来显示MongoDB版本。

### **MongoDB 5.0.1指纹解析**

以MongoDB 5.0.1版本为例，当使用nmap对目标的27017端口进行扫描时：

![](images/2023/06/5.0-example.png)

nmap会发送上文中的Probe指令：

![](images/2023/06/5.0-probe.png)

MongoDB响应数据包中可以匹配上文中第一条match指令：

![](images/2023/06/5.0-match.png)

因为在Nmap的扫描结果中，可以看到version（版本号）是p指令+指令，即“MongoDB v5.0.1”。

### **MongoDB有线协议**

MongoDB有线协议（Wire Protocol）是请求/响应形式的TCP/IP的Socket，客户端和服务端通过该协议进行通讯，服务端实例（mongod和mongos）的默认端口是27017。

以MongoDB 6.0为例，客户端和服务端在通讯时的消息格式采用的是OP\_MSG操作符，即采用以下格式对消息进行编码：

``` C
OP_MSG {
    MsgHeader header;           // standard message header
    uint32 flagBits;            // message flags
    Sections[] sections;        // data sections
    optional<uint32> checksum;  // optional CRC-32C checksum
}
```

其中，MsgHeader是标准信息头，客户端与服务端来往的消息都包含标准信息头（在SSL/TLS连接时没有checksum字段），其后才是消息内容（如OP\_MSG结构中sections数组，数组每一项由Kind值开头，后接载体数据），标准信息头的结构如下：

``` C
struct MsgHeader {
    int32   messageLength; // total message size, including this
    int32   requestID;     // identifier for this message
    int32   responseTo;    // requestID from the original request
                           //   (used in responses from the database)
    int32   opCode;        // message type
}
```

MsgHeader中opCode消息类型会决定OP\_MSG的结构，以OP\_QUERY为例：

``` C
struct OP_QUERY {
    MsgHeader header;                 // standard message header
    int32     flags;                  // bit values of query options.  See below for details.
    cstring   fullCollectionName ;    // "dbname.collectionname"
    int32     numberToSkip;           // number of documents to skip
    int32     numberToReturn;         // number of documents to return
                                      //  in the first OP_REPLY batch
    document  query;                  // query object.  See below for details.
  [ document  returnFieldsSelector; ] // Optional. Selector indicating the fields
                                      //  to return.  See below for details.
}
```

OP\_QUERY类型用于查询数据库中的集合，也是Nmap进行指纹识别时发送的请求消息类型，即上文图中的Probe指令。

但在MongoDB 5.1版本之后，OP\_QUERY等消息类型已被淘汰，而相应的Nmap的服务指纹文件和NSE脚本中的服务识别脚本都未更新MongoDB的OP\_MSG消息格式，因此无法正确识别出MongoDB的版本信息。

### **MongoDB 6.0指纹识别**

知道MongoDB 5.0.1版本的指纹识别方式，以及MongoDB的OP\_MSG消息格式，便可以通过抓取MongoDB 6.0的版本信息请求的流量，修改Nmap的服务探针文件nmap-service-probe来识别服务版本。

首先写一段Python代码作为客户端向MongoDB服务端发起版本信息的请求

``` Shell
#!/bin/env python

import pymongo

client = pymongo.MongoClient("mongodb://192.168.168.133:27017")
server_status = client.admin.command("serverStatus")
print(server_status["version"])
```

同时，在脚本执行的同一环境下使用TCPDump抓取流量包

``` Shell
sudo tcpdump -i eth0 host 192.168.168.133 -w mongo.pcap
```

基于抓取的流量包进行分析，可以看到获得最终版本信息的请求头：

![](images/2023/06/6.0-stream.png)

从/0x5f/0x00/0x00/0x00开始的便是Python脚本发送给MongoDB服务端的OP\_MSG消息头，使用该十六进制字符串替换/usr/share/nmap/nmap-service-probes文件中的Probe指令，由于响应消息中的version匹配格式与原match指令中相同，所以不用变更match指令，这样即可实现对于MongoDB 6.0版本的识别。

![](images/2023/06/modified-service-probe.png)

<div align='center'>nmap-service-probe修改内容</div>

![](images/2023/06/6.0-example.png)

<div align='center'>修改指纹文件后的扫描结果</div>

但现在的识别仅是相当于流量重放的结果，不一定能够识别所有MongoDB 6.0的指纹，因此需要解析上面流量的OP\_MSG消息，剥离出通用版本的OP\_MSG请求。

根据6.0版本的OP\_MSG格式，可以得知上图中的十六进制字符串的构成：

```
\x5f\x00\x00\x00：OP_MSG.MsgHeader.messageLength

\x51\xdc\xb0\x74：OP_MSG.MsgHeader.requestID

\x00\x00\x00\x00：OP_MSG.MsgHeader.responseTo

\xdd\x07\x00\x00：OP_MSG.MsgHeader.opCode

\x00\x00\x00\x00：OP_MSG.flagBits

\x00：OP_MSG.sections[0].kind

\x4a\x00\x00\x00\x10\x73\x65\x72\x76\x65\x72\x53\x74\x61\x74\x75\x73\x00\x01\x00\x00\x00\x03\x6c\x73\x69\x64\x00\x1e\x00\x00\x00\x05\x69\x64\x00\x10\x00\x00\x00\x04\x7c\xc1\xc2\x6c\xd7\x11\x40\x21\xb6\xf7\x5a\x52\x08\xaf\xca\x5e\x00\x02\x24\x64\x62\x00\x06\x00\x00\x00\x61\x64\x6d\x69\x6e\x00\x00：OP_MSG.sections[0].payload
```

核心部分是OP\_MSG中sections的payload，使用Python中的bson模块对payload解析：

![](images/2023/06/bson-payload.png)

<div align='center'>OP\_MSG.sections\[0\].payload解码结果</div>

其中的lsid是客户端请求服务端的会话id，是bson类型的UUID：**7cc1c26cd7114021b6f75a5208afca5e**。

因此可以可以使用自定义生成uuid来创建能够查询MongoDB版本信息的OP\_MSG消息头：

``` Python
import socket
import bson
import binascii
import uuid

def send_op_msg_request(request):
    host = '192.168.168.133'
    port = 27017

    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client.connect((host, port))

    client.sendall(request)
    response = client.recv(4096)

    client.close()

    return response

def build_op_msg_request():
    msg_header = b'\x5f\x00\x00\x00'  # total message size
    msg_header += b'\x00\x00\x00\x00'  # identifier for message
    msg_header += b'\x00\x00\x00\x00'  # request id
    msg_header += b'\xdd\x07\x00\x00'  # message type

    op_msg = msg_header + b'\x00\x00\x00\x00'  # message flags

    section = {
        'serverStatus': 1,
        'lsid': {'id': bson.binary.Binary(uuid.uuid4().bytes, bson.binary.UUID_SUBTYPE)},
        '$db': 'admin'
    }

    request = op_msg + b'\x00' + bson.encode(section)

    return request

op_msg_request = build_op_msg_request()
response = send_op_msg_request(op_msg_request)

print(response)
```

上面代码中的request值就是发送至服务端的OP\_MSG消息，可以用于替换nmap-service-probe文件的指纹信息。

以下是更换指纹前后的对比效果：

![](images/2023/06/before-modified.png)

<div align='center'>更改指纹文件前</div>

![](images/2023/06/after-modified.png)

<div align='center'>更改指纹文件后</div>

### **参考材料**

* [https://www.mongodb.com/docs/v6.0/reference/mongodb-wire-protocol/](https://www.mongodb.com/docs/v6.0/reference/mongodb-wire-protocol/)
* [https://www.mongodb.com/docs/v6.0/reference/bson-types/#std-label-bson-types](https://www.mongodb.com/docs/v6.0/reference/bson-types/#std-label-bson-types)
* [https://nmap.org/book/vscan-fileformat.html](https://nmap.org/book/vscan-fileformat.html)