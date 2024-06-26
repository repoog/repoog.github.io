---
title: 企业数据安全实践总结
date: 2022-12-03 17:52:41
author: repoog
excerpt: 数据安全是当下很多企业所关注的核心部分，同时也是所有企业安全建设的最终目标，但数据安全又牵涉到业务和开发深处，以至于从数据安全构建上存在极大的复杂性和艰难性。本文通过笔者的实践经验粗浅的介绍企业安全建设过程中数据安全相关的实践经验和总结。
comments: true
tags:
  - KMS
  - VPN
  - 企业安全
  - 加密解密
  - 数据安全
categories:
  - 数据安全
---

数据是企业业务的灵魂，如果企业能够不断发展和壮大，其业务或是能够在技术领域遥遥领先，形成技术壁垒，利用技术优势赚取利润，比如特斯拉、Google，或是业务发展过程中在数据方面获得独一无二的优势，形成网络效应，即便其他企业获得同类数据也依然不具有同等优势，比如微信、微博。因此，当和企业高层谈论起企业安全建设和投入的主要目标时候，几乎所有人会给出相同的答案：数据不要（能）丢了。

即便是在不同的外部审计中，无论是投资方指定的审计公司，还是处于认证考虑寻找的审计公司，不同的审计方式都离不开数据的安全，甚至其他的审计范围和内容其最终目标都是为了确保数据的安全，避免人为的、技术的、意外的原因导致数据出现无法预知、无法跟踪、无法追溯的偏差、错误、缺失，即便偏差只有0.01元人民币。

数据之所以重要，是它和企业的财务表现息息相关，因此企业中的数据不仅仅包括业务数据，还包括财务数据，现今的企业财务报表统计和分析，均源于业务数据的计算和分析，同时需要和企业银行账户的现金关联比对，如果企业当年的实际收入是1000万元，而银行账户流水的进账却没有这么多，那么收入统计可能是有问题的。所以，保障数据安全，不仅仅能够让企业业务正常、健康的发展，而且能够让企业的管理层真实地了解企业的健康状况（财务状况），让投资人（私募机构或上市后的股民）真实地了解企业过去的盈利。审计机构的作用便在于如实地呈现企业财务状况，而财务状况又源于数据真实性、完整性、机密性、可用性、可追溯性，如果企业数据造假，或连同审计机构作假，或审计结构未发现企业数据造假，便会无法取得公众和投资人的信赖，导致企业自身的发展危机甚至审计机构的信任危机，比如安然公司在2001年宣告破产，连同彼时第五大会计师事务所安达信解体，又比如2020年瑞信咖啡因交易额造假导致股票崩盘，同时也让其外部审计公司安永走到了舆论的风口浪尖（实际上安永公司对此事无责）。

数据是企业的灵魂，也是企业与公众、与投资人之间的信任媒介，而安全的本质也是信任。在企业安全体系建设中，几乎所有的安全控制措施都可以认为是为了数据安全。但常常会被认为只需要进行数据的加密解密处理。

现如今的企业数据不仅包括常见的结构化数据，也包括近年来随着技术发展而来的非结构化数据，以至于企业面临的数据安全实质上是大数据安全（另一层含义是利用大数据来辅助信息安全），大数据的特点在于其多样性、海量化、快速化、价值密度低和复杂性。数据的生命周期包括产生、传输、存储和使用，这个周期中涉及到不同的设备、系统、人员、组织，任何一个环节存在风险，都有可能破坏数据的完整性、保密性和可用性，因此确保数据自身安全需要考虑到设备层、系统层、数据层和隐私层四个方面。

如今公有云已经是很多企业的首选，在企业安全建设和数据安全建设过程中可以不用过多考虑设备层面的问题（该问题由云服务商解决和负责），如果选择的是私有云或混合云，则需要考虑相应的设备安全问题，一块硬盘损坏，一台交换机故障，都会造成相应的数据风险或业务连续性问题。云计算的能力也使得大数据能够更容易的管理和建设，其资源共享、按需分配、弹性调度、可扩展、地理分布的特点使得企业减少了诸多管理成本、维护成本。即云计算是大数据的基础，大数据是云计算的价值延伸。

### **数据产生及传输**

数据在采集或产生过程中，可能存在数据损坏、数据丢失、数据泄露、数据窃取等安全风险，需要使用相应的控制措施来缓解风险，如身份认证、通讯加密、数据加密、完整性校验等机制。

以某个Web业务应用为例，一条数据从产生到存储，会历经用户身份验证、数据创建、数据传输、数据处理。

用户身份管理包括外部用户身份和内部用户身份，最佳的实践是通过SSO（单点登录）系统来管理外部用户和内部用户，分散的或各自实施的用户身份管理会让安全管理和控制成本上升，且其中如果涉及到多个部门，更会大大增加安全的不可控和安全风险。如果有可能，用户身份管理最好是由安全部门来负责或监督，笔者曾经在某公司的内部用户（员工/外部人员）的身份管理便是由安全部门负责和维护，再将身份管理日志接入SOC（安全运营中心）系统，可以大大降低身份安全管理和维护成本，同时极大提高身份安全应急的响应速度。身份认证和权限管理内容过多，可以另起一文，本文不做过多描述。

数据创建需要在产品设计层面做防呆设计和逻辑校验，这可以降低脏数据产生的可能性，比如用户完善个人信息时填写身份证号码和性别，可以通过身份证号倒数第二位自动判断性别（奇数为男，偶数为女），如若是由用户自行选择，会出现性别数据与真实数据不匹配的情况。笔者曾经所在公司有BI部门，从各个业务线获取数据进行业务分析，在统计分析用户分布的时候，采用的是身份证信息，显然这是有问题的，因为身份证归属地不一定是用户所在地，如果产品设计时没有获取用户的定位信息，做用户分布分析便是不准确，甚至无意义的，就像微信通过用户设置的所属国家分析用户所在地，发现许多用户都在安道尔。

数据创建同时需要考虑账户自身的可靠性，比如黑卡账户（即黑产或灰产人员创建的账户），其账户下的所有行为都可认为是不正当或恶意的，因此识别、清理黑卡账户可以大大降低数据风险，笔者曾经所在公司的电商业务，通过黑卡识别的账户所占比例不多，但对于电商业务营销活动的资金支出可以占到正常用户的很大比例（即薅羊毛）。而业务人员可能会认为黑卡账户其实也是增加了用户量。

数据传输过程最佳实践是全网HTTPS，如果产品部署结构中业务流量经过CDN（内容分布网络）、LD（负载均衡），可以直接在CDN、LD上传公司的SSL证书实现，同时在反向代理节点或Web节点配置HSTS（HTTP Strict Transport Security），强制访问者需要经过SSL证书加密的流量进行访问，在移动端可以从编码环节设置证书强校验。具体需要根据业务部署结构和产品结构配置，但也并非如此简单，这其中需要分析、评估访问流量每个环节的可用性，避免产品部分功能或能力的问题。

数据处理过程中需要根据业务的特点以及数据处理过程中涉及到的系统、应用、人员分析，常见的情况是开发人员为了方便调试或bug跟踪，将敏感数据通过日志方式打印，从而让开发人员可以读取到不必要的业务数据或个人隐私数据。因此需要专门的日志中心以及设置统一的日志格式，确保日志内容中没有不必要获取的信息，同时日志系统的访问也需要身份验证和操作日志管理。

### **数据存储**

数据在存储中涉及到三个部分，第一部分是数据加密，包括静态数据加密和动态数据加密，静态数据指的是存储在硬盘中不参与计算的数据，比如文档、报表、资料，动态数据加密指的是在内存中参与运算或检索的数据；第二部分是隐私保护，隐私是个人或机构不愿意被外部知晓的信息，有分为个人隐私和共同隐私，典型的是PII（个人身份信息）数据；第三部分是数据备份和恢复，通过业务连续性或灾难恢复计划来保障数据的可用性和完整性，如果数据丢失或被损坏，可以通过备份机制来恢复数据，保证故障发生后数据尽可能少的丢失或不丢失。

加密算法分对称加密和非对称加密，常见的对称加密包括DES、AES、IDEA、RC4等，非对称加密算法有RSA、EIGamal等。对称加密运算速度快，但需要提前交换密钥，非对称加密无需提前交换密钥，但运算速度较慢。实际工程中采用的解决办法是用非对称密钥系统进行密钥分发，利用对称密钥加密算法进行数据加密。

密钥管理方案包括密钥粒度（粒度越小，管理难度越大，访问控制越细）的选择、密钥管理体系以及密钥分发机制。

适合大数据存储的密钥管理办法主要是分层密钥管理，即“金字塔”式密钥管理体系。这种密钥管理体系是将密钥以金字塔的方式存放，上层密钥用来加/解密下层密钥，只需将顶层密钥分发给数据节点，其他层密钥均可直接存放于系统中。也可以使用基于PKI体系的密钥分发方式对顶层密钥进行分发，用每个数据节点的公钥加密对称密钥，发送给相应的数据节点，数据节点接收到密文的密钥后，使用私钥解密获得密钥明文。

几乎所有的云服务商都会提供KMS（密钥管理服务），使用该服务可以很方便的对静态数据进行加密和解密管理。以某云服务商为例，常使用的加密方式有信封加密和直接加密，前者适用于大数据量、高QPS的数据加密，如PII数据，后者适用于小数据量（<6k）、低QPS的数据加密，如应用配置信息。

信封加密过程中，开发人员使用用户主密钥（CMK）生成数据密钥，再用离线的数据密钥在本地加密数据，即加密数据使用的是CMK生成的密钥（明文密钥和密文密钥），加密在内存中完成后销毁明文密钥，存储加密后的密文数据和密文密钥。解密时再解密密文密钥获得明文密钥对密文数据进行解密。该过程的思路即是金字塔式密钥管理。

直接加密使用用户主密钥（CMK）对数据直接进行加解密操作。

在隐私数据保护方面，常见的方法有3类：

1、基于数据变换的隐私保护技术

对敏感属性进行转换，使原始数据部分失真，但同时保持数据或数据属性不变。数据失真技术通过扰动原始数据来实现隐私保护，扰动后的数据需要满足：攻击者不能发现真实的原始数据；失真后的数据仍然保持某些性质不变。相关的技术包括随机化、数据交换和添加噪声。

2、基于数据加密的隐私保护技术

采用对称或非对称加密技术在数据挖掘过程中隐藏敏感数据，多用于分布式应用环境。分布式应用一般采用两种模式存储数据：垂直划分（每个站点只存储部分属性数据，站点之间数据不重复）、水平划分（将数据记录存储到多个站点，每个站点数据不重复）。

3、基于匿名化的隐私保护技术

根据具体情况有条件发布数据，即限制发布，比如有选择的发布原始数据、不发布或发布精度较低的敏感数据。数据匿名化一般采用抑制（不发布某项数据）、泛化（对数据进行概括、抽象）两种方法。

实践过程中，数据加密和隐私数据保护需要根据数据安全保护策略进行数据保护等级划分，将不同等级的数据进行分开存储和处理，如果一个数据表中不同字段的等级不同，在加解密处理中会增加技术处理的复杂性。

数据备份与恢复方面，公有云服务商会提供诸多功能帮助解决该问题，包括快照功能、可用区功能。

快照可以迅速恢复遭破坏的数据，减少宕机损失，能够进行在线数据备份与恢复。当存储设备发生应用故障或者文件损坏时可以进行快速的数据恢复，将数据恢复某个可用时间点的状态。实践中可以对云平台中ECS（Elastic Compute Service）和RDS（Relational Database Service）都采用定时快照功能，OSS（Object Storage Service）也有定时备份功能，定期对云计算实例的硬盘和云数据库进行备份。结合多地区、多可用区的功能，通过良好的部署构建，可以实现多地区、多可用区数据镜像，将数据在多个可用区或地区之间进行同步。

如果是本地存储管理，可以采用异地备份和RAID（独立磁盘冗余阵列）技术。异地备份需要足够的带宽连接支持和数据备份技术或软件支持。RAID适用于单个存储设备，可以减少磁盘部件的损坏，RAID系统使用许多小容量磁盘驱动器来存储大量数据，并且使可靠性和冗余度得到增强，RAID 系统的特点是“热交换”能力，即用户可以取出一个存在缺陷的驱动器，并插入一个新的予以更换，根据备份数据的性能需要，常使用的是RAID-5，即至少需要3块硬盘做（2块硬盘）数据镜像以及（1块硬盘）奇偶校验。

### **数据使用**

数据使用过程主要会涉及到企业内部的系统和人员，比如数据分析需要、业务分析需要、业务数据统计需要等等。

针对单个业务的数据使用安全，可以根据业务数据流图以及业务系统结构图来获得数据接触和使用相关的系统、部门、人员。最常见的是业务人员、产品经理、产品运营人员对于业务数据的使用和分析需要，最简单也最不安全的做法是提供生产线数据库的只读权限账户，每个人直接通过该账号连接数据库进行数据查询和分析。

因此需要有数据库访问平台或系统，对所有人的数据访问权限进行收敛、管理和审计，比如可以在DB Proxy基础上进行二次开发。该系统至少需要具备的能力包括：

* 账户管理，或接入内部员工账户管理系统；
* 权限申请，粒度为数据库表中的字段；
* 权限审批，需要申请人上级同意，如是跨部门申请权限，即A部门人员申请B部门业务数据权限，则需要B部门数据所有者同意；
* 查询和写入分开，非技术人员只有数据读取权限，且无法进行全量查询（SELECT \*），每次查询结果有数据量限制，导出数据需要做二次身份验证，技术人员写入操作需要单独申请，需部门负责人同意，且有时间限制；
* 日志审计，审计员账户可以查看所有账户的操作记录、申请记录和审批记录，操作记录包括执行的SQL语句内容；

另外，在数据使用中需要内部人员远程访问系统或应用，需要采用VPN技术。常用的技术有路由过滤技术、通用路由封装协议（GRE，Generic Routing Encapsulation）、第二层转发协议（L2F，Layer 2 Tunneling Protocol）、第二层隧道协议（L2TP，Layer 2 Tunneling Protocol）、OpenVPN、IP 安全协议（IPSec，IP Security）、SSL协议等。从易用性和安全性考虑，最佳选择是SSL VPN，无论PPTP、L2TP、OpenVPN都会面临诸多网络问题或操作问题带来的维护成本，比如不同运营商网络下的VPN连接效果会有不同，移动端和PC端配置的复杂性造成的IT问题飙升（技术人员认为的简单不等于员工眼中的简单）。相比之下，SSL VPN可以较好地解决该问题。

SSL VPN 采用标准的安全套接层协议，基于X.509 证书，支持多种加密算法。可以提供基于应用层的访问控制，具有数据加密、完整性检测和认证机制，而且客户端无需特定软件的安装，更加容易配置和管理等特点，从而降低用户的总成本并增加远程用户的工作效率。

SSL VPN系统的组成按功能可分为 SSL VPN服务器和 SSL VPN客户端。SSL VPN服务器是公共网络访问私有局域网的桥梁，它保护了局域网内的拓扑结构信息。SSL VPN 客户端是运行在远程计算机上的程序，它为远程计算机通过公共网络访问私有局域网提供一个安全通道，使得远程计算机可以安全地访问私有局域网内的资源。

SSL VPN常见的工作模式有三种：Web浏览器模式、SSL VPN客户端模式和LAN到LAN模式。如果被访问系统均是基于Web的系统，可以使用Web浏览器模式，使用者不需要安装客户端，可以降低用户成本，否则可以选择SSL VPN客户端模式，也仅需要安装一个客户端，输入服务器地址、账户名和密码即可连接，相对的维护成本也较低。

同时在实践中选择和部署VPN系统时，需要考虑2FA（双因子认证）的问题，如短信验证码，避免VPN账户信息记住之后由于客户端风险（如恶意软件）造成的VPN账户信息泄露或恶意远程访问。     数据安全在企业安全建设过程中需要考虑的地方远不止以上内容，对于安全人员而言，重要的是将安全技术与业务建设相结合，从业务建设倒推安全技术的选择，从而才能在获得助力业务发展的同时，获得同事、上级及管理层对于安全建设和安全部门的重视。