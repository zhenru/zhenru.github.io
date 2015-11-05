---
layout: post
title: 如何从HttpServletRequest中获取到IP地址
---
###如何从HttpServletRequest中获取到IP地址

获取ip地址的时候可以直接通过`getRemoteAddr()`获取到用户的ip地址，事情并不这样简单。
很多的时候用户的HttpServletRequest并不能直接传递到最终执行业务的服务器，而`getRemoteAddr()`方法只能获取到到达当前的机器的请求的IP地址。
如果服务器是单台机器，用户的请求直接发送到服务器上的时候，此时使用`getRemoteAddr()`获取到的IP地址就是用户的IP地址，很多时候服务器并不是一台机器，往往是一个集群。前端由一台机器负责接受各种用户传递过来的请求，然后将当前的请求转发到相对应的机器上。如果服务器上使用`getRemoteAddr()`获取到的IP地址是前端负责分发的机器的IP地址，这时候使用`getRemoteAddr()`方法就没有办法获取用户的地址。使用了一种解决方法：在HTTP的请求报文头中添加一个字段，将用户的请求的IP地址记录在里面，然后转发到后端的服务器上，后端的服务器上只要先跟据请求的报文头中的值获取到IP地址的值。这些报文头的字段对于不同的服务器的架构会使用不同的字段。可能包括的字段有：
```
X-FORWARDED-FOR
Proxy-Client-IP
WL-Proxy-Client-IP
HTTP_CLIENT_IP

```
存在多种的网络架构，如Nginx+Resin , Apache+WebLogic，Squid+Nginx等，在这些不同的网络架构中，作为前端的服务器会使用不同的报文头字段来传递这个信息。以下就简单描述一下：
```
其中使用nginx作为前端节点（也是最常用的）配置如下：
location/{ 
      proxy_pass  http://yourdomain.com; 
      proxy_set_header  Host  $host; 
      proxy_set_header   X-Real-IP     $remote_addr; 
      proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for; 
       }
```

 - X-Forwarded-For
背景：
这个字段是Squid开发的字段，不是标准。这个简称XXF。当通过使用了http代理或者是负载均衡代理服务器的时候就会添加这个字段。squid的文档中描述是`X-Forwarded-For:client1,proxy1,proxy2`.其中第一是真实的用户的请求的IP地，后面的IP地址就是中间所经过的各个代理服务器和负载均衡的的IP地址。
使用场景：
客户端--CDN--Nginx
用户的请求首先经过了CDN，到达nginx的时候，XFF应该是`客户端的IP地址,CDN的IP`。很多时候，CDN会隐藏地址。那么当请求到达Nginx的时候：
一般情况下并不会对用户的地址进行处理，Nginx后面通过`getHeader("X-FORWORDED-FOR")`获取到的IP地址就是原始的IP地地址。然后将自己的地址转添加到XFF头的后面转发到具体执行业务的机器上去。
伪造：
由于XFF是http报文的头字段，用户可以伪造一个XFF头字段转发到后端的服务器，后面的服务器会首先获取到XFF字段然后将当前服务器的IP地址添加到XFF字段后面。然后传递到后面，对于用户传递过来的XFF任何字段服务器都往往没有办法进行识别，到底是用户传递的IP地址还是用户伪装的IP地址。


- Proxy-Client-IP/WL-Proxy-Client-IP:
背景：
这种字段主要是在Apache+Weblogic中使用这种架构，其中WL就是WebLogic的缩写，其中访问的路径是`Client + Apache WebServer + Web Logic http Plugin + Weblogic Instance `.如果当前的网络系统的架构是Apache+WebLogic会使用这个报文头关键字。

- HTTP-Client-IP
这个一般是代理服务器转发的HTTP头，很多时候Nginx配置中都没有这个选项所以就忽略这种。

- getRemoteAddr()
背景：从HttpServletRequest.getRemoteAddress()获取到IP地址，方法的描述是`Returns the Internet Protocol(IP) address of the client or last proxy that sent the request.`实际上，REMOTE_ADDR是用户和服务器端握手时的地址，如果Client使用了“匿名代理”的时候，服务器端就获取到匿名代理服务器的地址。



code范例：
```
 public final static String getIpAddress(HttpServletRequest request)  {
        // 获取请求主机IP地址,如果通过代理进来，则透过防火墙获取真实IP地址

        String ip = request.getHeader("X-Forwarded-For");

        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
                ip = request.getHeader("Proxy-Client-IP");
            }
            if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
                ip = request.getHeader("WL-Proxy-Client-IP");
            }
            if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
                ip = request.getHeader("HTTP_CLIENT_IP");
            }
            if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
                ip = request.getHeader("HTTP_X_FORWARDED_FOR");
            }
            if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
                ip = request.getRemoteAddr();

            }
        } else if (ip.length() > 15) {
            String[] ips = ip.split(",");
            for (int index = 0; index < ips.length; index++) {
                String strIp = (String) ips[index];
                if (!("unknown".equalsIgnoreCase(strIp))) {
                    ip = strIp;
                    break;
                }
            }
        }
        return ip;
    }
```


参考资料：[http://www.udpwork.com/item/8135.html](http://www.udpwork.com/item/8135.html)

