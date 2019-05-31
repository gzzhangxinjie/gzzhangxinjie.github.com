---
layout: post
title: "dnspython 包更新域名时，如何发现域名更新不成功"
description: ""
category: dns
tags: ['dns', 'dnspython']
---
{% include JB/setup %}


#### 没有正确地捕获域名更新的异常

我们使用 dnspython 更新域名时，相关的代码部分如下:

```python
    try:
        dns.query.tcp(update, server)
    except (dns.tsig.PeerBadKey, dns.tsig.PeerBadSignature, BaseException), error:
        logger.error("[update record] error: %s" % error)
```



这段代码能够检测出 tsig 出错时的报错。例如

```
>>> update.add("zhangxinjie3", 1800, 'A', '1.2.3.4')
>>> try:
...     dns.query.tcp(update, server)
... except (dns.tsig.PeerBadKey, dns.tsig.PeerBadSignature, BaseException), error:
...     print error
...
The peer didn't like the signature we sent
```

然而当服务器执行 `rndc freeze`(即不允许域名动态更新)时，并没有这个报错。



#### dnspython 域名更新消息

实际上，每次域名更新时，都是一个 dynamic update 消息，具体消息的格式可参照 [rfc2136](<https://tools.ietf.org/html/rfc2136>)。

我们可以根据 `rcode` 的值判断域名更新是否成功。

```
              错误类型   值     描述
              ------------------------------------------------------------
              NOERROR     0       No error condition.
              FORMERR     1       The name server was unable to interpret
                                  the request due to a format error.
              SERVFAIL    2       The name server encountered an internal
                                  failure while processing this request,
                                  for example an operating system error
                                  or a forwarding timeout.
              NXDOMAIN    3       Some name that ought to exist,
                                  does not exist.
              NOTIMP      4       The name server does not support
                                  the specified Opcode.
              REFUSED     5       The name server refuses to perform the
                                  specified operation for policy or
                                  security reasons.
              YXDOMAIN    6       Some name that ought not to exist,
                                  does exist.
              YXRRSET     7       Some RRset that ought not to exist,
                                  does exist.
              NXRRSET     8       Some RRset that ought to exist,
                                  does not exist.
              NOTAUTH     9       The server is not authoritative for
                                  the zone named in the Zone Section.
              NOTZONE     10      A name used in the Prerequisite or
                                  Update Section is not within the
                                  zone denoted by the Zone Section.
```

从上面可以可以看到，只要 `rcode` 不是为 0， 那么都可以认为是域名更新有异常。而使用 dnspython 进行域名更新时，都会返回一个 message，我们可以根据 message 里面的 rcode 判断域名更新是否成功。

#### 测试

1. 当域名更新正常时, `rcode` 返回值是 0 (NOERROR)。

   ```python
   >>> import dns.tsig
   >>> import dns.update
   >>> import dns.query
   >>> import dns.tsigkeyring
   >>> keyring = dns.tsigkeyring.from_text({"xxxxx": "yyyyyy"})
   >>> update = dns.update.Update('example.com', keyring=keyring, keyalgorithm="hmac-sha256")
   >>> update.add("zhangxinjie3", 1800, 'A', '1.2.3.4')
   >>> server = '127.0.0.1'
   >>> message = dns.query.tcp(update, server)
   >>> print message
   id 16
   opcode UPDATE
   rcode NOERROR
   flags QR
   ;ZONE
   netease.com. IN SOA
   ;PREREQ
   ;UPDATE
   ;ADDITIONAL
   >>> print message.rcode()
   0
   ```

   

2. 当执行 `rndc freeze` 之后，域名无法更新，`rcode` 返回值是 5 ( REFUSED)。 

   ```python
   >>> import dns.tsig
   >>> import dns.update
   >>> import dns.query
   >>> import dns.tsigkeyring
   >>> keyring = dns.tsigkeyring.from_text({"xxxxx": "yyyy"})
   >>> update = dns.update.Update('example.com', keyring=keyring, keyalgorithm="hmac-sha256")
   >>> update.add("zhangxinjie3", 1800, 'A', '1.2.3.4')
   >>> server = '127.0.0.1'
   >>> message = dns.query.tcp(update, server)
   >>> print message
   id 16
   opcode UPDATE
   rcode REFUSED
   flags QR
   ;ZONE
   netease.com. IN SOA
   ;PREREQ
   ;UPDATE
   ;ADDITIONAL
   >>> print message.rcode()
   5
   ```

3. `NOTAUTH` 报错

   ```
   >>> update = dns.update.Update("example.com", keyring=keyring, keyalgorithm="hmac-sha256")
   >>> update.add("zhangxinjie3", 1800, "A", "2.3.3.4")
   >>> message = dns.query.tcp(update, server)
   >>> print message
   id 25588
   opcode UPDATE
   rcode NOTAUTH
   flags QR
   ;ZONE
   example.com. IN SOA
   ;PREREQ
   ;UPDATE
   ;ADDITIONAL
   >>> print message.rcode()
   9
   ```


