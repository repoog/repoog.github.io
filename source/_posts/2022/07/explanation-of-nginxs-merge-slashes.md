---
title: Nginx的merge_slashes解析
date: 2022-07-13 19:26:08
author: repoog
excerpt: 一家企业的安全人员发现产品在测试环境和生产环境中有一个完全不同的路径穿越漏洞，该漏洞在测试环境中没有发现，却神奇的会在生产环境中出现。经过分析发现是由于Amazon的负载均衡内部的Nginx配置导致。本文从源码层面分析Nginx中merge_slashes的配置，并介绍该配置是如何阻止路径穿越漏洞的。
comments: true
tags:
  - merge_slashes
  - Nginx
  - 路径穿越
  - 漏洞分析
categories:
  - 系统安全
---

Nginx有一个配置项是merge\_slashes，官方的说明如下：

> Syntax: merge\_slashes on  off;
> 
> Default: merge\_slashes on;
> 
> Context: http, server
> 
> Enables or disables compression of two or more adjacent slashes in a URI into a single slash.

顾名思义，就是会在处理URI时将相邻的单斜杠合并。这个配置项在Nginx中对应的源码是在[https://github.com/nginx/nginx/blob/master/src/http/ngx\_http\_parse.c#L1248](https://github.com/nginx/nginx/blob/master/src/http/ngx_http_parse.c#L1248)，该函数的声明是：

> ngx\_int\_t ngx\_http\_parse\_complex\_uri(ngx\_http\_request\_t \*r, ngx\_uint\_t merge\_slashes)

第一个参数是URI的请求结构，第二个int类型参数merge\_slashes就是配置项的开（on）或关（off）。

函数中，首先有一个枚举类型的变量state用于函数执行中标识每个字符的状态，或者是字符的分类，初始情况下是sw\_usual正常状态。

``` C
enum {
        sw_usual = 0,
        sw_slash,
        sw_dot,
        sw_dot_dot,
        sw_quoted,
        sw_quoted_second
} state, quoted_state;
```

字符型指针\*u在函数执行过程中用于记录处理的中间URI字符串，字符型指针\*p是URI的字符串，字符ch用于记录当前在处理中的URI的字符。

接着是一个大的while循环，用于逐个处理URI中的字符：

``` C
ch = *p++;
while (p <= r->uri_end) {
        switch (state) {
        case sw_usual:
        ...
                switch (ch) {
                        ...
                case '/':
                        ...
                        state = sw_slash;
                        *u++ = ch;
                        break;
                        ...
                }
        ch = *p++;
        break;

        case sw_slash:
        ...
                switch (ch) {
                case '/':
                        if (!merge_slashes) {
                                *u++ = ch;
                        }
                break;
                ...
                }
        ch = *p++;
        break;
        ...
        case sw_dot_dot:
        ...
                switch (ch) {
                case '/':
                        u -= 4;
                        for ( ;; ) {
                                if (u < r->uri.data) {
                                        return NGX_HTTP_PARSE_INVALID_REQUEST;
                                }
                                if (*u == '/') {
                                        u++;
                                        break;
                                }
                        u--;
                        }
                state = sw_slash;
                break;
                }
        }
}
```

函数根据每个处理的字符类型标识当前字符的状态或类型，该状态决定在之后的条件判断语句中该如何处理后续字符。通过以上函数的逻辑可知，如果merge\_slashes为1，相邻出现的多个单斜杠会在sw\_slash状态的条件处理中被忽略，\*u不记录单斜杠，如果为0，则会记录单斜杠字符。

因此，多个斜杠相邻的URI会被处理为单个斜杠，如果需要在URI中传输BASE64字符，则需要关闭merge\_slashes，避免其中的斜杠被处理。

另外，如果URI中存在路径穿越的利用，比如“/../../../”，会条件处理到sw\_dot\_dot中，并返回NGX\_HTTP\_PARSE\_INVALID\_REQUEST，即无效的请求。所以Nginx默认情况下可以防范路径穿越攻击，除非将merge\_slashes参数设置为关闭。