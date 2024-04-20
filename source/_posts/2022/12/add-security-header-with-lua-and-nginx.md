---
title: 用Lua增加Nginx安全响应头
date: 2022-12-23 17:22:27
author: repoog
excerpt: Lua语言可以作为Nginx插件的开发语言使用，本文介绍的是使用Lua语言开发Nginx的插件来实现对于恶意访问IP的阻拦，并自动添加和设置HTTP的安全响应头，以节省Nginx的配置操作。
comments: true 
tags:
  - Lua
  - Nginx
  - 插件开发
  - 安全配置
categories:
  - 系统安全
---

Lua是一款轻量级的开发语言，主要用于增强应用能力的脚本编写，其中也包括Nginx这样的Web服务器。

LuaJIT是非常快速的Lua解释器，能够快速解释Lua代码，非常适合Nginx这样同样高性能的应用，通常Nginx下的Lua开发会使用OpenResty（其适用LuaJIT的分支项目LuaJIT2）。

在Nginx下也可以直接进行Lua脚本开发，以Ubuntu为例，在已经安装Nginx之后执行以下命令安装Lua以及LuaJIT：

``` Shell
sudo apt install lua5.2 liblua5.2-dev

sudo apt install luajit
```

安装完毕之后，需要检查是否在Nginx中加载了ndk\_http\_module和ngx\_http\_lua\_module模块，如果是手动加载，需要在nginx.conf中的event段前添加：

```
include modules-enabled/*.conf;
```

或者

```
load_module modules/ndk_http_module.so;

load_module modules/ngx_http_lua_module.so;
```

Nginx下开发Lua主要用到了Nginx的Lua指令以及Nginx API，具体可参考[https://www.nginx.com/resources/wiki/modules/lua](https://www.nginx.com/resources/wiki/modules/lua)

对于简短的Lua代码可以使用content\_by\_lua或content\_by\_lua\_block指令，在Nginx配置文件server段中直接编写，较长的Lua代码可以使用content\_by\_lua\_file指定Lua文件，该文件的相对路径是基于Nginx的prefix路径，可以使用Nginx的API函数ngx.config.prefix()打印该路径。

比如通过Lua自动添加安全响应头，在Nginx的prefix目录下创建lua/sec\_header.lua文件：

```
ngx.header["X-Content-Type-Options"] = "nosniff";

ngx.header["Content-Security-Policy"] = "upgrade-insecure-requests; script-src https: 'self' 'unsafe-eval' 'unsafe-inline' https://cdn.staticfile.org;";

ngx.header["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains";

ngx.header["X-Frame-Options"] = "SAMEORIGIN";

ngx.header["X-XSS-Protection"] = "1; mode=block";
```

然后在Nginx的server段中配置：

```
access_by_lua_file lua/sec_header.lua;
```

之后访问所有页面都会默认添加安全响应头，修改响应头信息只需要修改sec-header.lua文件即可。

又比如通过Lua和Redis来阻止特定的IP地址访问，将黑名单IP地址存储在Redis中，从Lua中读取黑名单IP进行源IP判断，如果源IP在名单中则阻止访问。

首先需要下载[https://github.com/openresty/lua-resty-redis/blob/master/lib/resty/redis.lua](https://github.com/openresty/lua-resty-redis/blob/master/lib/resty/redis.lua)文件到/usr/local/lib/lua-redis/lib目录下。

而后，在Nginx的配置文件nginx.conf中的HTTP段增加以下内容，用于引入上面下载的lua包文件：

```
lua_package_path "/usr/local/lib/lua-redis/lib/?.lua;;";
```

接着，在上文的Lua脚本所在目录下创建block\_ips.lua文件，代码如下：

``` Lua
local redis = require "resty.redis"
local red = redis:new()

-- Connect to Redis server
red:set_timeout(1000)

local ok, err = red:connect("127.0.0.1", 6379)
if not ok then
  ngx.log(ngx.ERR, "Redis connection error")
  return
end

local count, err = red:get_reused_times()
if 0 == count then
  ok, err = red:auth("password")
  if not ok then
    ngx.log(ngx.ERR, "Auth failed with: " .. err)
    return
  end
elseif err then
  ngx.log(ngx.ERR, "Failed to get reused times: " .. err)
  return
end

-- Get current remote address
local ip = ngx.var.remote_addr

-- Use Redis command to check if current IP is in blocked ip list
local res, err = red:sismember("blocked_ips", ip)

if res == 1 then
  -- if current IP is in blocked ip list, return 403 error
  ngx.exit(ngx.HTTP_FORBIDDEN)
end

red:close()
```

当前版本的Redis默认启用身份认证，在配置文件Redis.conf中可以更改配置，认证口令也在该文件中记录。上述代码中使用red:auth对Redis服务做身份验证，避免后续的集合查询命令无法执行。

通过在Redis的集合blocked\_ips中增、删想要阻止访问的IP地址即可实现阻止多IP地址访问。

最后，在Nginx的站点配置中将该Lua脚本加载，上文中的access\_lua\_by\_file不允许多次使用，但可以使用loadfile指令起到多个文件加载的效果：

```
access_by_lua_block {
loadfile('/usr/share/nginx/lua/sec_header.lua')();
loadfile('/usr/share/nginx/lua/block_ips.lua')();
}
```

通过以上指令可以让站点加载上述两个Lua脚本，达到启用安全响应头和阻止恶意IP的效果。