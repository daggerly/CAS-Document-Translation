> 版本5.2

# 架构
![CAS架构](../../pic/cas-documenttation/planning/architecture/cas_architecture.png)

## 系统组件

CAS系统由服务器和客户端组成,它们之间以多种协议通信。

### CAS服务器
CAS服务器是构建在Spring框架上的Java servlet, 其主要职责是认证用户以及通过下发和
验证票据(ticket)进行有CAS功能服务的授权访问,这些服务通常称为CAS客户端。
当服务器下发一个票据获取票据(ticket-granting ticket, TGT)给成功登录的用户时就
产生了一个单点登录会话(SSO session)。当浏览器带着作为token的TGT重定向用户的请求时,
一个服务票据(service ticket, ST)被分发给一个服务。这个ST随后被CAS服务器通过
后台通信验证。


### CAS客户端
术语"CAS客户端"有两种共同使用的完全不同的意思。CAS客户端是能和服务器通过支持的
协议进行通信的任何有CAS功能的应用。同时也是一个为了和CAS服务器通过一些
认证协议(例如CAS, SAML, OAuth)通信的,能被集成进不同软件平台和应用的软件包。
CAS客户端已被开发可以支持许多软件平台和产品。

平台:

* Apache httpd Server ([mod_auth_cas module](https://github.com/Jasig/mod_auth_cas))
* Java ([Java CAS Client](https://github.com/apereo/java-cas-client))
* .NET ([.NET CAS Client](https://github.com/apereo/dotnet-cas-client))
* PHP ([phpCAS](https://github.com/Jasig/phpCAS))
* Perl (PerlCAS)
* Python (pycas)
* Ruby (rubycas-client)

应用:

* Canvas
* Atlassian Confluence
* Atlassian JIRA
* Drupal
* Liferay
* uPortal
* ...

当术语"CAS客户端"在这个文档中出现而没有附带更多的条件时,它是指例如Java CAS客户端的
集成组件,而不是依赖CAS服务器的应用。


## 支持的协议

客户端可以和服务器通过任何支持的协议通信。所有协议在概念上是类似的,但是有些协议对于
具体的应用或者使用场景有更令人满意的功能或特性。例如, CAS协议支持代理认证, SAML协议
支持属性释放(attribute release)和单点登出(single sign-out)。

支持的协议:

* [CAS (versions 1, 2, and 3)](../protocol/CAS-Protocol.html)
* [SAML 1.1 and 2](../protocol/SAML-Protocol.html)
* [OpenID Connect](../protocol/OIDC-Protocol.html)
* [OpenID](../protocol/OpenID-Protocol.html)
* [OAuth 2.0](../protocol/OAuth-Protocol.html)
* [WS Federation](../protocol/WS-Federation-Protocol.html)


## 软件组成
依次描述CAS服务器的三个子系统:

* Web (Spring MVC/Spring Webflow)
* [Ticketing](../installation/Configuring-Ticketing-Components.html)
* [Authentication](../installation/Configuring-Authentication-Components.html)

这三个子系统几乎涉及了所有的部署注意事项和组件配置。Web层是和包括CAS客户端的所有外
部系统通信的入口点。Web层委托票据层生成供CAS客户端访问的票据。SSO会话是由成功
认证后的TGT分发开始的, 因此票据子系统会频繁的访问认证子系统。

认证系统是值处理SSO会话开始的请求, 尽管有其他场景会被调用(例如强制认证)。

### Spring框架

CAS在许多方面使用了Sping框架,最明显的是
[Spring MVC](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/mvc.html)
和[Spring Webflow](http://www.springsource.org/spring-web-flow)。
Spring为CAS核心代码库和开发者提供了完整和可扩展的框架。很容易通过hookCAS和Spring API
扩展点来自定义和扩展CAS行为。通用的Spring知识对理解一些框架组件的相互影响很有帮助,
但这并不强制要求。

### Spring Boot

CAS也严重依赖[Spring Boot](http://projects.spring.io/spring-boot/),
Spring Boot允许Spring平台和第三方库尽可能不使用XML配置来创建独立的web应用。
Spring Boot允许CAS隐藏许多内部组件的复杂性和它们的配置,取而代之的是提供了能简单和自动配置运行中
的应用环境,而不需要过多人工干涉的自动配置模块。

[返回目录](../../README.md)