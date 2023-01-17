---
title: JWT(json web token)学习
date: 2023-1-17 20:10:00
tags:
---

## JWT

### 1.流程图

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/38d15042-93df-4e77-8ded-fce73cac4aa1/Untitled.png)

### 2.过程

**2.1**

1.前端通过Web表单将自己的用户名和密码发送到后端的接口。该过程一般是HTTP的POST请求。建议的方式是通过SSL加密的传输(https协议)，从而避免敏感信息被嗅探。
2.后端核对用户名和密码成功后，将用户的id等其他信息作为JWT Payload，将其与头部分别进行Base64编码拼接后签名，形成一个JWT(Token)。
3.后端将JWT字符串作为登录成功的返回结果返回给前端。前端可以将返回的结果保存在localStorage（浏览器本地缓存）或sessionStorage（session缓存）上，退出登录时前端删除保存的JWT即可。
4.前端在每次请求时将JWT放入HTTP的Header中的Authorization位。(解决XSS和XSRF问题）HEADER
5.后端检查是否存在，如存在验证JWT的有效性。例如，检查签名是否正确﹔检查Token是否过期;检查Token的接收方是否是自己(可选）验证通过后后端使用JWT中包含的用户信息进行其他逻辑操作，返回相应结果。

**2.2**

JWT的优点在于：

1.轻量：数据量小，传输速度快

2.包含用户信息：避免多次查询数据库

3.跨语言

4.适合分布式服务

**2.3 结构**

JWT由下面三部分组成：

1.header

2.Payload

3.Signature

token格式：head.payload.singurater 如：xxxxx.yyyy.zzzz

• Header：有令牌的类型和所使用的签名算法，如HMAC、SHA256、RSA；使用Base64编码组成；（Base64是一种编码，不是一种加密过程，可以被翻译成原来的样子

• Payload ：有效负载，包含声明；声明是有关实体（通常是用户）和其他数据的声明，不放用户敏感的信息，如密码。同样使用Base64编码

• Signature ：前面两部分都使用Base64进行编码，前端可以解开知道里面的信息。Signature需要使用编码后的header和payload

加上我们提供的一个密钥，使用header中指定的签名算法(HS256)进行签名。签名的作用是保证JWT没有被篡改过

**tips：**

Base64是一种编码，是可逆的，适合传递一些非敏感信息；JWT中不应该在负载中加入敏感的据。比如不能存放用户的密码