> 版本5.2

# 安全指南

CAS为Web应用程序提供了安全的单点登录。单点登录在安全和便利方面提供了一个双赢的方案:
由于无需重复登录便可以访问多个服务,其减少了将密码暴露给单一受信证书中间人的次数。
通常来说使用CAS会改善安全环境,但应该考虑某些CAS的配置,规则,部署关注点获取合适的安全保证。

<div class="alert alert-info"><strong>报告问题</strong>
<p>安全团队要求你创建讨论安全脆弱性问题的问题或者报告<strong>不要</strong>公共可见。
关于如何适当地报告和回复问题,请<a href="https://apereo.github.io/cas/developer/Sec-Vuln-Response.html">看这个指南</a>。
</p></div>

## 最新公告

- [2017年3月6日漏洞报告](https://apereo.github.io/2017/03/06/moncfgsecvulndisc/)
- [2016年10月24日漏洞报告](https://apereo.github.io/2016/10/24/servlvulndisc/)
- [2016年4月8日漏洞报告](https://apereo.github.io/2016/04/08/commonsvulndisc/)

## 系统安全注意事项

需要考虑的基础设施安全问题可能包括以下方面。

### 安全传输 (https)

所有和CAS服务器的通信必须建立在安全信道上(比如 TLSv1)。 对于这条要求有两个主要理由:

1. 认证过程需要传输安全凭据。
2. CAS 票据授予票据(ticket-granting ticket, TGT) 持票人的令牌。

由于泄露的数据可能导致模拟攻击,保护CAS客户端和服务器直接的通信信道的安全及其重要。

实际上,这意味着所有CAS url必须使用HTTPS, 但这**也** 意味着所有从CAS服务器到应用
的连接必须使用HTTPS:

- 将服务票据通过"服务"url发送回应用时的url。
- 当一个代理回调(proxy callback)被调用时的url。

要了解相关的CAS功能清单并开启这一功能,请 [看这个文档](../installation/Configuration-Properties.html#http-client).


### 与依赖系统的连接

CAS通常需要连接到其他系统,比如LDAP目录,数据库,缓存服务等。我们通常推荐可能的话和
那些系统使用安全传输(SSL/TLS, IPSec),但是也许有其它控制方法的话就会使安全传输没有必要。
私网或者带有严格访问控制的公司网络通常是个例外,尽管如此仍然推荐使用安全传输。为了足够安全,
LDAP客户端证书校验可能是另一个好的解决方案。

如前所述, 和其它系统的连接必须是安全的。但是如果CAS服务器被部署在多个节点上,
安全传输同样适用于CAS服务器本身。如果说一个基于缓存的票据注册服务在单个CAS服务器上没有
运行任何安全问题,那么当使用多节点时,如果网络不被保护,节点同步将成为安全问题。

如果没有安全保护,任何硬盘存储同样脆弱。可以关掉EhCache向硬盘写入来提高保护,然而
高级加密数据机制应该被应用到数据库硬盘上。

## 部署驱动(Deployment-Driven)的安全特性

CAS提供了许多特性来实现不同的安全规则。下面这些规则通过CAS配置和CAS客户端集成来提供。
需要注意的是游戏特性是开箱即用的,而有些则需要明确的设置。

### 强制认证

许多CAS客户端和支持的协议支持强制认证的概念, 就是用户为了访问一个特定的服务必须经过重新认证。
CAS协议通过 _renew_ 参数支持强制认证。
由于用户必须在访问前认证其证书,强制认证为一个SSO会话的主要身份提供额外保证。
强制认证适用于安全要求更高的服务。
虽然每个服务的基础上都配置了强制认证,但是[服务管理](#service-management)工具对
实现强制认证的中央安全策略提供了支持。
强制认证可以结合[多因子认证](#multifactor-authentication)特性来实现任意服务特性的访问控制策略。


### 被动认证

当需要匿名访问CAS保护的服务时,有些CAS协议支持被动认证。CASv2和CASv3协议通过 _网关_
功能支持这个特性。被动认证对强制认证起到补充作用,强制认证需要通过认证才能访问一个服务,
被动认证允许匿名的服务访问。


### 代理认证

代理认证,或者叫委托认证为CAS提供了一个强大重要,或许增强了安全性的功能。
CASv2和CASv3协议支持代理认证,通过代表用户的服务所请求的代理票据实现,因此服务为用户代理了认证。
代理认证一般被用于服务不能直接和用户交互的情况,以及作为替代方案将终端用户证书重放给服务。

然而,代理票据存在一个风险,即接收代理证书的服务负责验证代理链
(终端用户的认证服务列表被委托给到达的票据验证服务)。
服务完全可以简单地通过靠/serviceValidate检验入口检验票据而选择不接受代理票据
(从而避免检验代理链的责任),
但是经验显示很容易把这个和无意中配置使用不检查任何出现票据认证响应的代理链的/proxyValidate入口点
搞混。因此代理认证需要仔细配置合适的安全控制。
建议如果不需要,请关闭CAS服务器的代理认证组件

历史上任何服务都能够获得代理授权票据(proxy-granting ticket),从这个票据代理票据访问其它服务。
换句话说,安全模型被分散了,而不是被集中了。服务管理工具通过在每个服务基础上禁用或启用一个代理认证标识
提供代理认证的集中管理。默认情况下不给予注册服务代理认证的功能。

### 凭据缓存和回放

_清除密码_ 扩展 提供了一个获取主认证凭据(primary authentication credential),将
其缓存(被加密),根据需求将其回放来访问遗留服务的机制。
虽然建议使用[代理认证](#proxy-authentication)在代替密码回放,
但也许需要它来整合遗留服务和CAS。
查看[清除密码](../integration/ClearPass.html)中的详细信息。


### 服务管理

服务管理工具提供了许多影响安全性策略,对集中化安全策略提供支持的服务特性的配置管理。
(注意,CAS历来支持非集中策略模型)服务管理控制的一些亮点:

* 已授权的服务
* 强制认证
* 属性释放
* 代理认证控制
* 模式控制
* 服务认证控制
* 多因子服务访问策略

服务管理工具包括一个包含一个或多个已注册服务的服务注册中心,每一个都指定了上面的管理控制。
可以通过静态配置文件,Web用户接口来控制服务注册中心。查看[服务管理](../installation/Service-Management.html)获取更多信息。

<div class="alert alert-warning"><strong>已授权的服务</strong><p>
作为一个安全最佳实践,<strong>强烈</strong>推荐限制服务管理工具,只包括
已被CAS授权的已知应用的列表。对所有应用开发管理接口会留下安全隐患。
</p></div>

### SSO Cookie加密

票据授予cookie是CAS根据单点登录会话设置的HTTP cookie。cookie值默认被CAS属性中定义的设置加密和签名。
由于初始部署提供了样本数据,这些键 **必须** 在你的特定环境中重新生成。
请 [看这个指南](../installation/Configuring-SSO-Session-Cookie.html)来获取更多信息。

### 密码管理安全链接

通过发送到用户注册邮箱中的安全链接处理帐户密码重置请求。
链接只有在定义的时间窗口内才有效,请求会被CAS适当加密和签名。
由于初始部署提供了样本数据,这些键 **必须** 在你的特定环境中重新生成。
请[看这个指南](../installation/Password-Policy-Enforcement.html)来获取更多信息。

### 协议票据(Protocol Ticket)加密

被CAS发布,与其它应用共享的协议票据,例如服务票据(service ticket),
可以可选地通过一个签名/加密过程。
即使CAS服务器会永远交叉检验票据有效策略,为了确保票据在传输过程中没有被篡改,仍然是真实的,
这可以被作为一个强制的额外检查。
由于初始部署提供了样本数据,这些键 **必须** 在你的特定环境中重新生成。

<div class="alert alert-warn"><strong>注意</strong><p>
根据使用对加密方法和算法,对一个生成对票据做加密和签名会增加票据长度。
不是所有CAS客户端准备好处理过长对票据字符串,然后可能会对你不满。
在使用这个之前评估现有的情况,并且思考你的部署环境是否真的需要这个特性。
</p></div>

相关属性请[查看这个文档](../installation/Configuration-Properties.html#protocol-ticket-security)。


### 票据注册加密

由于会被视为CAS集群部署,为了确保CAS生成的票据在传输过程中没有被篡改,要求备份安全票据。
(Secure ticket replication as it regards clustered CAS deployments may be required to ensure generated tickets by CAS are not tampered with in transit. )
CAS通过允许票据原生加密和签名解决了这个问题。
由于初始部署提供了样本数据,这些键 **必须** 在你的特定环境中重新生成。
请[查看这个文档](../installation/Ticket-Registry-Replication-Encryption.html)来获取更多信息。

### 管理网页安全

CAS为系统管理员和开发者提供了许多web接口。
这些页面与许多REST入口点一起允许CAS开发者不需要求助命令行接口就能管理和配置CAS行为。
不用多说,这些入口点和页面肯定是被加密的,且只允许访问被授权的部分。
请 [查看这个文档](../installation/Monitoring-Statistics.html)来获取更多信息。

### 票据过期策略

票据过期策略是实现安全策略的主要机制。
票据过期策略允许控制CAS SSO会话的一些重要方面:

* SSO 会话持续时间 (滑动期限 绝对期限)
* 票据复用

查看 [配置票据组件](../installation/Configuring-Ticketing-Components.html)
一节获取不同过期策略和配置指令的详细讨论.

### 单点登出

单点登出 (SLO)是CAS服务的特性,该服务被告知结束一个CAS SSO会话,期望该服务终止SSO会话拥有者的访问。
虽然单点登出可以提高安全性,但它本质上是一个最优的工具,不能在SSO会话期内结束对所有服务的访问消费。
下列补充控制可以用来改善与单点登出缺点相关的风险:

* 敏感服务需要强制认证
* 减少应用会话超时
* 减少SSO会话有效期

在两种情况下可能发生SLO: 从CAS服务器(后台信道登出)和/或从浏览器(前台信道登出)。
对于后台信道登出, SLO过程依赖 `SimpleHttpClient` 类,该类有一个线程池:
必须定义该线程池大小以正确处理登出请求。
额外的没准备好处理的登出请求在被发送前临时保存在一个队列中:
该队列大小被定义为全局线程池大小的20%,并且可调。
这两个大小是CAS系统重要的设置,它们的值永远不应该超过CAS服务器的真实容量。


### 登录限流

CAS支持一个规则驱动的特性,用来限制连续认证失败,希望帮助阻止暴力破解和拒绝服务攻击。
这个特性在后端认证缺少等效特性的环境中很有用。
在底层系统支持这个特性的情况下,我们鼓励使用它来替代CAS的特性。
理由是底层系统的支持是对包括CAS的所有依赖的系统提供的。
查看[登录限流配置](../installation/Configuring-Authentication-Components.html#login-throttling)来获取更多信息。

### 凭据加密

要了解怎样通过加密来保护CAS的敏感设置, [请看这个指南](Configuration-Properties-Security.html)。

### CAS安全过滤器

CAS项目为基于CAS协议的恶意输入攻击(bad-CAS-protocol-input attacks)的
明确请求参数提供了许多适合Java CAS服务器和客户端的
[通用安全过滤器][https://github.com/apereo/cas-server-security-filter]补丁。
这些过滤器被配置为清理认证请求参数,并且拒绝不遵从CAS协议的请求,例如一个参数重复多次,
包括多个值,包含无法接受的值等等。

**强烈** 建议所有CAS部署必须被评估,如有必要,包含这个配置来阻止在CAS容器和环境不能
阻塞恶意构造的请求情况下的协议攻击。

#### CORS

CAS对HTTP访问控制功能(CORS)提供了最高支持。
CORS的一个应用是当一个资源产生了一个HTTP跨域请求,跨域请求就是它请求了和它自己不同域名
的资源。这对于通过XHR/Ajax请求访问的有CAS能力的应用帮助更大。

关于相关CAS属性列表和开启这个行为,请[看这个文档](../installation/Configuration-Properties.html#http-web-requests).

#### 安全响应头

作为CAS安全过滤器的一部分,CAS项目为阻止HSTS,XSS,X-FRAME和其它攻击
而在web响应增加HTTP安全头自动提供了必要的配置。目前这些设置是默认关闭的。
关于相关CAS属性列表和开启这个行为,请[看这个文档](../installation/Configuration-Properties.html#http-web-requests)。

要了解这些选项,请看[这个文档][cas-sec-filter]。

### Spring Webflow 会话

CAS项目使用Spring Webflow来管理和编排认证过程。
客户端管理CAS使用的Webflow的会话状态,然后被整个认证过程的不同状态传递和跟踪。
这个状态必须是安全和被加密的,以防止会话劫持。
虽然CAS提供了开箱即用的加密设置,
**强烈** 推荐在生产部署和重新生成这个配置之前评估[CAS全部部署](../installation/Webflow-Customization.html),
以防止攻击。

### 长期认证

长期认证特性,一般被称作"记住我",在CAS登录表单中被选中以避免一段时间后重新认证。
长期认证允许用户以减少安全性为代价选择额外的便利性。
安全性减少的程度是用于建立一个CAS SSO会话的设备的特点的函数(The extent of reduced security is a function of the characteristics of the device used to establish a CAS SSO session.)。
从一个单独用户拥有或操作的设备建立的长期SSO会话比标准CAS SSO会话减少了一些安全性。
唯一真正需要关心的是生命周期的增加及因此增加的CAS票据授予票据的泄露 。
另一方面,从一个共享的设备建立一个长期CAS SSO会话会明显减少安全性。
从之前的SSO会话影响后来的其它用户的SSO会话的建立可能性,即使面对单点登出,也可能增加模拟的可能性。
(The likelihood of artifacts from previous SSO sessions affecting subsequent SSO sessions established by other users, even in the face of single sign-out, may increase the likelihood of impersonation.)
虽然增加共享设备上SSO长期会话安全性没有可行的缓解措施,教育用户这些固有风险可能提高整体的安全性。

注意,强制认证会取代长期认证。因此如果一个服务以前被配置为强制认证,
那么即使在长期会话的环境中访问服务也需要认证。

长期认证必须通过
[配置和UI自定义](../installation/Configuring-Authentication-Components.html#long-term-authentication)
在安装过程中明确启动。因此开发者选择提供长期认证支持,然后在可用时用户可以通过在CAS登录
表单中选中来使用它。

### 警告

CAS支持在建立SSO会话时可选的访问服务通知。默认情况下CAS透明的请求访问服务需要的票据,
然后将它们展示给目标服务进行检验,检验成功后访问服务才被允许。
大多数情况下这个过程几乎立即发生,而用户不知道访问CAS功能的服务所需的CAS认证过程。
这个过程中也许有一些安全性利益需要意识到,在登录页面上CAS支持一个用户可选的 _警告_ 标识,
来使在访问服务前提供一个间隙的通知页面展示。默认时这个通知页面给用户提供一个选项
来开始CAS认证,或者靠被从目标服务导航开而终止。

[cas-sec-filter]: https://github.com/apereo/cas-server-security-filter

[返回目录](../../README.md)