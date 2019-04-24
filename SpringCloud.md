# 1.分布式系统的问题(事务)

**1.单体应用拆分为多服务后，进程间的通讯机制，和故障处理措施变得更加复杂。**

随着RPC的发展，这个问题已差不多解决。SpringCloud支持Restful调用。

**2.微服务数量众多，测试，部署，监控变得更加困难。**

随着docker、devops技术的发展以及各公有云paas平台自动化运维工具的推出，微服务的测试、部署与运维会变得越来越容易。

**3.系统微服务化后，一个看似简单的功能需要调用多个服务，并操纵多个数据库，微服务的事务问题就非常突出。**

## 1.基于XA协议的两阶段提交方案

使用两个阶段，第一个阶段是所有参与者将事务能否成功的信息，返回给协调者，如果成功进入第二阶段，执行阶段。由于锁定时间较长，并不怎么适合微服务架构。

## 2.TCC方案

蚂蚁金服使用的TCC

TCC方案其实是两阶段方案的一种改进，其将本地资源管理器的功能融入到了业务实现中。其将整个业务逻辑显示的分成了Try、Confirm、Cancel三部分。

1.Try 预留资源校验

2.confirm 确认执行业务操作

3.cancel 取消预留资源

## ![img](https://static.oschina.net/uploads/space/2018/0605/185458_edch_3552485.png)

1.TCC的优点:

应用自己定义数据库操作的粒度，降低锁冲突，提高吞吐量。

2.TCC的缺点：

​            1.开发工总量大，服务的每个接口都要实现try confirm cancel接口

​            2.实现难度大：系统应该记录每个服务调用链路。为了满足一致性的要求，二阶段不管调用confirm还是cancle都必须调用成功，如果一次调用不成功，事务协调器必须尝试重试。这就要求confirm和cancle接口必须实现幂等。

3.TCC允许空回滚:当try时，出现丢包情况。触发cancel.

4.防悬挂控制：就是空回滚之后，拒绝空回滚之后的try请求。

5.幂等控制：即执行一次try,confirm,cancel和执行多次的结果是一样的。

6.可见性控制：当try时，对预留资源进行控制，应尽量减少别的服务访问这个资源。

## 3.消息事务一致性方案 mq

接口幂等性：当相同接口，使用相同的参数，被调用的多次，其返回的结果是一样的。

消息一致性方案是通过消息中间件保证上、下游应用数据操作的一致性。基本思路是将本地操作和发送消息放在一个事务中，保证本地操作和消息发送要么两者都成功或者都失败。下游应用向消息系统订阅该消息，收到消息后执行相应操作。

以下单业务为例进行说明，下单基本流程是先存储订单信息，然后扣相应商品的库存，两个操作必须在一个事务中。如下图，业务应用首先调用订单服务，订单存储成功后，订单服务会通过消息处理服务投递订单消息到MQ。库存服务从MQ收到消息后进行扣库存操作，如果执行成功会向消息处理服务发送通知。消息处理服务会实时监测订单消息是否超时，如果超时会重新投递到MQ中，以驱动库存服务进行扣库存操作。如果扣库存操作执行失败后，库存服务后续还会从MQ接收到相同的订单消息，需要多次重复执行，直到成功或者进行人工干预。库存服务需要实现幂等。 

![img](https://static.oschina.net/uploads/space/2018/0605/185539_ACzy_3552485.png)

消息方案从本质上讲是将分布式事务转换为两个本地事务，然后依靠下游业务的重试机制达到最终一致性。相对TCC方案来讲，消息方案技术难度相对低，落地较容易,如果对一致性不敏感的应用也是一个不错的选择。美国著名电商e-bay以及国内的蘑菇街都做过尝试。消息一致性方案的不足之处是其对应用侵入性较高，应用需要基于消息接口进行改造，而且需要建设专门的消息系统，成本较高。

## 4.GTS方案

GTS分布式事务解决方案 阿里巴巴开发的

### 1.优势

1.性能强：解决了事务ACID特性与高性能、高可用、低侵入不可兼得的问题。单事务分支的平均响应时间在2ms左右，3台服务器组成的集群可以支撑3万TPS以上的分布式事务请求。

2.应用侵入性极低：

业务代码中只需要添加@TxcTransaction即可，业务与实务分离，微服务专注于业务本身，不考虑反向接口，幂等，回滚策略等复杂问题。

3.容错能力强：

GTS解决了XA事务协调器单点问题，实现真正的高可用，保证严格的数据一致。

### 2.GTS与微服务的集成

GTS包括客户端(GTS-Client)，资源管理器(GTS-RM)，事务协调器(GTS-Server)

客户端主要负责界定实务边界，完成事务的发起和结束。

资源管理器主要负责**事务分支**的创建，提交，回滚等操作。

事务协调器：负责**整体事务**的推进，生命周期的管理。

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20180307135937146?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamlhbmd5dV9ndHM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 3.GTS的输出形式

1.公有云输出

这种输出形式面向阿里云用户。如果用户的业务系统已经部署到阿里云上，可以申请开通公有云GTS。开通后业务应用即可通过GTS保证服务调用的一致性。这种使用场景下，业务系统和GTS间的网络环境比较理想，达到很好性能。 

2.公网输出

这种输出形式面向于非阿里云的用户，使用更加方便、灵活，业务系统只要能连接互联网即可享受GTS提供的云服务（与公有云输出的差别在于客户端部署于用户本地，而不在云上）。

在正常网络环境下，以包含两个本地事务的全局事务为例，事务完成时间在20ms左右，50个并发就可以轻松实现1000TPS以上分布式事务，对绝大多数业务来说性能是足够的。在公网环境，网络闪断很难完全避免，这种情况下GTS仍能保证服务调用的数据一致性。 

3.专有云输出

这种形式主要面向于已建设了自己专有云平台的大用户，GTS可以直接部署到用户的专有云上，为专有云提供分布式事务服务。目前已经有10多个特大型企业的专有云使用GTS解决分布式事务难题，性能与稳定性经过了用户的严格检测。

### 4.GTS的使用

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20180307140212381?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamlhbmd5dV9ndHM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

上图是GTS与SpringCloud集成，应用于某共享出行系统。业务共享出行场景下，通过GTS支撑物联网系统、订单系统、支付系统、运维系统、分析系统等系各统应用的数据一致性，保证海量订单和数千万流水的交易。

### 5,样例使用

A和B分别在两个不同的数据库，A向B赚钱，保证数据的一致性

```
public class Simple {

	private static JdbcTemplate jdbcTemplate1 = null;
	private static JdbcTemplate jdbcTemplate2 = null;

	@TxcTransaction(timeout = 60000 * 3)
	public void transfer(int money) throws InterruptedException {
		//A账户从db1减钱
		jdbcTemplate1.update("update user_money_a set money = money - ?", money);
		if (new Random().nextInt(100) < 2) {
			throw new RuntimeException("error");
		}

		//B账户从db2加钱
		jdbcTemplate2.update("update user_money_b  set money = money + ?", money);
		if (new Random().nextInt(100) < 2) {
			throw new RuntimeException("error");
		}

		//余额不足时抛异常，GTS会回滚前两次操作
		int money_a = getMoney(jdbcTemplate1, "user_money_a");
		if (money_a < 0) {
			Thread.sleep(20);
			throw new RuntimeException("Balance is not enough.");
		}
	}

	public static void main(String[] args) throws SQLException {
		ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("txc-client-context.xml");
		Simple simple = (Simple) context.getBean("clientTest");

		jdbcTemplate1 = (JdbcTemplate)context.getBean("jdbcTemplate1");
		jdbcTemplate2 = (JdbcTemplate)context.getBean("jdbcTemplate2");
		//清空账户
		jdbcTemplate1.update("truncate user_money_a");
		jdbcTemplate2.update("truncate user_money_b");
		//每个账户初始值为10000元
		jdbcTemplate1.update("insert into user_money_a(money) values(10000)");
		jdbcTemplate2.update("insert into user_money_b(money) values(10000)");

		for (int i = 0; i < 100; ++i) {
			System.out.println("transfer" + i);
			try {
				simple.transfer( 2);
			} catch (Exception e) {
				System.out.println("Tranfer failed，transaction is rollbacked.");
				e.printStackTrace();
			}
			int money1 = getMoney(jdbcTemplate1, "user_money_a");
			int money2 = getMoney(jdbcTemplate2, "user_money_b");
			System.out.println("A account：" + money1 + "    B account：" + money2 + "    total：" + (money1 + money2));
		}
		System.exit(0);
	}

	private static int getMoney(JdbcTemplate jdbcTemplate, String tableName) {
		String sql = "select money from " + tableName;
		List<Integer> moneyList = jdbcTemplate.queryForList(sql, java.lang.Integer.class);
		if (moneyList.size() > 0) {
			return moneyList.get(0);
		}
		return 0;
	}
}

```

### 6.浅-》深

# 2.cap理论

Consistency（一致性）、 Availability（可用性）、Partition tolerance（分区容错性），三者不可得兼

分布式系统的CAP理论：理论首先把分布式系统中的三个特性进行了如下归纳：

- 一致性（C）：在分布式系统中的所有数据备份，在同一时刻是否同样的值。（等同于所有节点访问同一份最新的数据副本）
- 可用性（A）：在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。（对数据更新具备高可用性）
- 分区容错性（P）：以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择。

CAP理论就是说在分布式存储系统中，最多只能实现上面的两点。而由于当前的网络硬件肯定会出现延迟丢包等问题，所以分区容忍性是我们必须需要实现的。所以我们只能在一致性和可用性之间进行权衡。

所以系统在一定的时间内表现为数据不一致，但是通过自动或者手动补偿之后，达到最终的一致性。

# 3.Base理论

解决关系型数据库的强一致性而引起的可用性降低的方案

BASE是下面三个术语的缩写：

- 基本可用（Basically Available）
- 软状态（Soft state）
- 最终一致（Eventually consistent）
   BASE 理论和ACID原则不同，满足CAP理论，牺牲强一致性来保证系统的高可用性。由于放弃了强一致性，在请求系统的时候，数据在短时间内可以不一致，但是一定要最终一致。

系统在处理业务的时候，保存每一步的的临时状态，当出现异常时，根据临时状态判断是否继续还是回滚到原始的状态。从而达到数据的一致。



# 4.zuul

![img](https://images2017.cnblogs.com/blog/280044/201707/280044-20170731002541646-1718153933.png)

## 1.简介:

外界访问内部的锁，需要匹配的钥匙才能访问。

spring cloud zuul是提供负载均衡，反向代理，权限认证的一个API gateway.

## 2.网关的设计要素：

系统级别。高可用性。均衡负载：容错，防止血崩。并发控制：错峰流控。动态路由制定和修改。应用级别。监控统计。版本控制。认证，鉴权。协议转换(http->rpc)。数据安全：防篡改，参数脱敏。

限流：实现微服务访问流量计算，基于流量计算分析进行限流，可以定义多种限流规则。
缓存：数据缓存。
日志：日志记录。
监控：记录请求响应数据，api耗时分析，性能监控。
鉴权：权限身份认证。
灰度：线上灰度部署，可以减小风险。
路由：路由是API网关很核心的模块功能，此模块实现根据请求，锁定目标微服务并将请求进行转发。

## 3.网关超时解决方案

ribbon.ReadTimeout， ribbon.SocketTimeout这两个就是ribbon超时时间设置，当在yml写时，应该是没有提示的，给人的感觉好像是不是这么配的一样，其实不用管它，直接配上就生效了。 
还有zuul.host.connect-timeout-millis， zuul.host.socket-timeout-millis这两个配置，这两个和上面的ribbon都是配超时的。区别在于，如果路由方式是serviceId的方式，那么ribbon的生效，如果是url的方式，则zuul.host开头的生效。（此处重要！使用serviceId路由和url路由是不一样的超时策略） 
如果你在zuul配置了熔断fallback的话，熔断超时也要配置，不然如果你配置的ribbon超时时间大于熔断的超时，那么会先走熔断，相当于你配的ribbon超时就不生效了。 
熔断超时是这样的： 
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds: 60000 

default代表默认，如果你想为某个特定的service配熔断超时策略，可以用这种方式： 

总结起来就是三种超时配置： 
网关的超时层级 

```
zuul

zuul: 
max: 
host: 
connections: 500 
host: 
socket-timeout-millis: 60000 
connect-timeout-millis: 60000

ribbon

ribbon: 
ReadTimeout: 10000 
ConnectTimeout: 10000 
MaxAutoRetries: 0 
MaxAutoRetriesNextServer: 1 
eureka: 
enabled: true

hystrix

hystrix: 
command: 
default: 
execution: 
timeout: 
enabled: true 
isolation: 
thread: 

timeoutInMilliseconds: 60000 


```

## 4.微服务下的鉴权网关服务

单服务下用session,但在微服务架构下，每个服务都需要单独鉴权，不能继续用session.

![single](http://blueskykong.com/pic/%E5%8D%95%E4%BD%93%E5%BA%94%E7%94%A8.png)

​                                                                    单系统

![distrubted](http://blueskykong.com/pic/%E5%BE%AE%E6%9C%8D%E5%8A%A1%E6%9E%B6%E6%9E%84%E6%9D%83%E9%99%90.png)

​                                                                               分布式系统

### 4.1分布式系统鉴权的两个方案：

####  1.分布式session方案：

单体应用架构中，默认 Session 会存储在应用服务器中，并且将 Session ID 返回到客户端，存储在浏览器的 Cookie 中。

分布式会话方案原理主要是将关于用户认证的信息存储在共享存储中，且通常由用户会话作为 key 来实现的简单分布式哈希映射。当用户访问微服务时，用户数据可以从共享存储中获取。在某些场景下，这种方案很不错，用户登录状态是不透明的。同时也是一个高可用且可扩展的解决方案。这种方案的缺点在于共享存储需要一定保护机制，因此需要通过安全链接来访问，这时解决方案的实现就通常具有相当高的复杂性了。

#### 2.基于 token方案

随着 Restful API、微服务的兴起，基于`Token`的认证现在已经越来越普遍。Token和Session ID 不同，并非只是一个 key。Token 一般会包含用户的相关信息，通过验证 Token 就可以完成身份校验。用户输入登录信息，发送到身份认证服务进行认证。AuthorizationServer验证登录信息是否正确，返回用户基础信息、权限范围、有效时间等信息，客户端存储接口。用户将 Token 放在 HTTP 请求头中，发起相关 API 调用。被调用的微服务，验证`Token`。ResourceServer返回相关资源和数据。

![img](https://img-blog.csdn.net/20170905093518444?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2RteGR6Yg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

#### 3.JWT 

JSON Web Token  和OAutu2 token差不多，但JWT是json形式的

JSON Web Token 是一种紧凑的，URL 安全的方式，表示要在双方之间传输的声明。JWT 一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该 Token 也可直接被用于认证，也可被加密。

JWT 是由三段信息构成的，第一段为头部（Header），第二段为载荷（Payload)，第三段为签名（Signature）。每一段内容都是一个 JSON 对象，将每一段 JSON 对象采用 BASE64 编码，将编码后的内容用. 链接一起就构成了 JWT 字符串。

1. 头部（Header）

头部用于描述关于该 JWT 的最基本的信息，例如其类型以及签名所用的算法等。这也可以被表示成一个 JSON 对象。

![img](https://img-blog.csdn.net/20170905093604892?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2RteGR6Yg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

在头部指明了签名算法是 HS256 算法。

\2. 载荷（payload）

载荷就是存放有效信息的地方。有效信息包含三个部分：

- 标准中注册的声明
- 公共的声明
- 私有的声明

标准中注册的声明（建议但不强制使用）：

iss：JWT 签发者

sub：JWT 所面向的用户

aud：接收 JWT 的一方

exp：JWT 的过期时间，这个过期时间必须要大于签发时间

nbf：定义在什么时间之前，该 JWT 都是不可用的

iat：JWT 的签发时间

jti：JWT 的唯一身份标识，主要用来作为一次性 token, 从而回避重放攻击。

公共的声明 ：

公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息. 但不建议添加敏感信息，因为该部分在客户端可解密。

私有的声明 ：

私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为 base64 是对称解密的，意味着该部分信息可以归类为明文信息。

示例如下：

![img](https://img-blog.csdn.net/20170905093637707?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2RteGR6Yg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

\3. 签名（signature)

创建签名需要使用 Base64 编码后的 header 和 payload 以及一个秘钥。将 base64 加密后的 header 和 base64 加密后的 payload 使用. 连接组成的字符串，通过 header 中声明的加密方式进行加盐 secret 组合加密，然后就构成了 jwt 的第三部分。

比如：HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)

JWT 的优点：

跨语言，JSON 的格式保证了跨语言的支撑

基于 Token，无状态

占用字节小，便于传输

#### 4.OAuth2

##### 1.简介：

OAuth2是一个授权框架，可以使第三方应用或者客户端获得对HTTP服务上用户账户信息的有限访问权限。

```
开发者A注册某IT论坛后，发现可以在信息栏中填写自己的 Github 个人信息和仓库项目，但是他又觉得手工填写十分麻烦，直接提供 Github 账户和密码给论坛管理员帮忙处理更是十分智障。
不过该论坛似乎和 Github 有不可告人的秘密，开发者A可以点击“导入”按钮，授权该论坛访问自己的 Github 账户并限制其只具备读权限。这样一来， Github 中的所有仓库和相关信息就可以很方便地被导入到信息栏中，账户隐私信息也不会泄露。
这背后，便是 OAuth 2 在大显神威。
```

##### 2.OAuth2角色

1.资源所有者：就是上面的开发者A

2.资源服务器/授权服务器：上面github既是资源服务器也是授权服务器，里面的所有仓库和相关信息既是资源。

3.客户端：第三方应用，即这个IT论坛。

##### 3.授权流程

![img](https://images2018.cnblogs.com/blog/647983/201805/647983-20180504165446261-1334969394.png)

1. Authrization Request
   客户端向用户请求对资源服务器的`authorization grant`。
2. Authorization Grant（Get）
   如果用户授权该次请求，客户端将收到一个`authorization grant`。
3. Authorization Grant（Post）
   客户端向授权服务器发送它自己的客户端身份标识和上一步中的`authorization grant`，请求访问令牌。
4. Access Token（Get）
   如果客户端身份被认证，并且`authorization grant`也被验证通过，授权服务器将为客户端派发`access token`。授权阶段至此全部结束。
5. Access Token（Post && Validate）
   客户端向资源服务器发送`access token`用于验证并请求资源信息。
6. Protected Resource（Get）
   如果`access token`验证通过，资源服务器将向客户端返回资源信息。

##### 4.客户端应用注册

在应用 OAuth 2 之前，你必须在授权方服务中注册你的应用。如 [Google Identity Platform](https://developers.google.com/identity/) 或者 [Github OAuth Setting](https://developer.github.com/apps/building-oauth-apps/creating-an-oauth-app/)，诸如此类 OAuth 实现平台中一般都要求开发者提供如下所示的授权设置项。

- 应用名称
- 应用网站
- 重定向URI或回调URL

重定向URI是授权方服务在用户授权（或拒绝）应用程序之后重定向供用户访问的地址，因此也是用于处理授权码或访问令牌的应用程序的一部分。

4.1 Client ID 和 Client Secret

一旦你的应用注册成功，授权方服务将以`client id`和`client secret`的形式为应用发布`client credentials`（客户端凭证）。`client id`是公开透明的字符串，授权方服务使用该字符串来标识应用程序，并且还用于构建呈现给用户的授权 url 。当应用请求访问用户的帐户时，`client secret`用于验证应用身份，并且必须在客户端和服务之间保持私有性。

##### 5.授权许可(Authorization Grant)

授权许可类型取决于应用请求授权的方式，和授权方支持的Crant type.

1.Authorization code:结合普通服务器端使用

2.Implicit:结合移动应用和webapp

3.Resource Owner Password Credentials
适用于受信任客户端应用，例如同个组织的内部或外部应用。

4.Client Credentials
适用于客户端调用主服务API型应用（比如百度API Store）

5.1Authorization code

`Authorization Code` 是最常使用的一种授权许可类型，它适用于第三方应用类型为`server-side`型应用的场景。`Authorization Code`授权流程基于重定向跳转，客户端必须能够与`User-agent`（即用户的 Web 浏览器）交互并接收通过`User-agent`路由发送的实际`authorization code`值。

![img](https://images2018.cnblogs.com/blog/647983/201805/647983-20180504165524831-873187279.png)

#### 1. User Authorization Request

首先，客户端构造了一个用于请求`authorization code`的URL并引导`User-agent`跳转访问。

```
https://authorization-server.com/auth
 ?response_type=code
 &client_id=29352915982374239857
 &redirect_uri=https%3A%2F%2Fexample-client.com%2Fcallback
 &scope=create+delete
 &state=xcoiv98y2kd22vusuye3kch
```

- response_type=code
  此参数和参数值用于提示授权服务器当前客户端正在进行`Authorization Code`授权流程。
- client_id
  客户端身份标识。
- redirect_uri
  标识授权服务器接收客户端请求后返回给`User-agent`的跳转访问地址。
- scope
  指定客户端请求的访问级别。
- state
  由客户端生成的随机字符串，步骤2中用户进行授权客户端的请求时也会携带此字符串用于比较，这是为了防止`CSRF`攻击。

5.2

### 4.2操作权限控制

1. `Shiro`
   Shiro是一个强大而灵活的开源安全框架，能够非常清晰的处理认证、授权、管理会话以及密码加密。Shiro很容易入手，上手快控制粒度可糙可细。自由度高，Shiro既能配合Spring使用也可以单独使用。
2. `Spring Security`
   Spring社区生态很强大。除了不能脱离Spring，Spring Security具有Shiro所有的功能。而且Spring Security对Oauth、OpenID也有支持,Shiro则需要自己手动实现。Spring Security的权限细粒度更高。但是Spring Security太过复杂。

## 5.zuul的基础使用

请求转发是zuul的基础功能。

还有鉴权，流量转发，请求统计等功能。下面是基础功能，请求转发。

1.依赖

```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-zuul</artifactId>
</dependency>
```

2.配置

```
spring.application.name=gateway-service-zuul
server.port=8888

#这里的配置表示，访问/it/** 直接重定向到http://www.ityouknow.com/**
zuul.routes.baidu.path=/it/**
zuul.routes.baidu.url=http://www.ityouknow.com/
#这里的配置表示，访问/hello/**,自动调转到9000服务，意思就是访问的前缀是http://localhost:9000/
zuul.routes.hello.path=/hello/**
zuul.routes.hello.url=http://localhost:9000/
```

```
http://localhost:8888/hello/hello?name=%E5%B0%8F%E6%98%8E
直接跳转
http://localhost:9000/hello/hello?name=%E5%B0%8F%E6%98%8E
```

3.注解

```
@EnableZuulProxy 
```

### 5.1将zuul服务化

通过url映射的方式来实现zull的转发有局限性，比如每增加一个服务就需要配置一条内容，另外后端的服务如果是动态来提供，就不能采用这种方案来配置了。实际上在实现微服务架构时，服务名与服务实例地址的关系在eureka server中已经存在了，所以只需要将Zuul注册到eureka server上去发现其他服务，就可以实现对serviceId的映射。

依赖：

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
就是添加到注册中心
```

配置修改成：

```
spring.application.name=gateway-service-zuul
server.port=8888

zuul.routes.api-a.path=/producer/**
zuul.routes.api-a.serviceId=spring-cloud-producer

eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/
```

跟之前的差不多，可以直接通过服务名重定向。

zuul也实现了负载均衡，当两个服务都是spring-cloud-producer时，轮询访问。

访问http://localhost:8888/producer/hello?name=%E5%B0%8F%E6%98%8E可以直接访问spring-cloud-producer

或者直接访问http://localhost:8888/spring-cloud-producer/hello?name=%E5%B0%8F%E6%98%8E



## 6.filter

filter是zuul的核心。

### 1.filter的生命周期

PRE:这种过滤器在路由请求之前被调用。 用于身份验证，在集群中选择调试的微服务，记录调试信息等。

counting:这种过滤器将请求路由到微服务。用于构建发送给微服务的请求。

post:路由到微服务之后。相应添加标准的Http header,收集统计信息和指标，将相应从微服务发送给客户端。

Error:发生错误时，执行过滤器。除了默认的过滤器类型，也可以自定义过滤器。

### 2.Zuul中默认实现的Filter

| 类型  | 顺序 | 过滤器                  | 功能                       |
| ----- | ---- | ----------------------- | -------------------------- |
| pre   | -3   | ServletDetectionFilter  | 标记处理Servlet的类型      |
| pre   | -2   | Servlet30WrapperFilter  | 包装HttpServletRequest请求 |
| pre   | -1   | FormBodyWrapperFilter   | 包装请求体                 |
| route | 1    | DebugFilter             | 标记调试标志               |
| route | 5    | PreDecorationFilter     | 处理请求上下文供后续使用   |
| route | 10   | RibbonRoutingFilter     | serviceId请求转发          |
| route | 100  | SimpleHostRoutingFilter | url请求转发                |
| route | 500  | SendForwardFilter       | forward请求转发            |
| post  | 0    | SendErrorFilter         | 处理有错误的请求响应       |
| post  | 1000 | SendResponseFilter      | 处理正常的请求响应         |

可以取消默认的filter

```
zuul:
	FormBodyWrapperFilter:
		pre:
			disable: true

```

### 3.自定义filter

继承ZuulFilter类

```
public class TokenFilter extends ZuulFilter {

    private final Logger logger = LoggerFactory.getLogger(TokenFilter.class);

    @Override
    public String filterType() {
        return "pre"; // 可以在请求被路由之前调用
    }

    @Override
    public int filterOrder() {
        return 0; // filter执行顺序，通过数字指定 ,优先级为0，数字越大，优先级越低
    }

    @Override
    public boolean shouldFilter() {
        return true;// 是否执行该过滤器，此处为true，说明需要过滤
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();

        logger.info("--->>> TokenFilter {},{}", request.getMethod(), request.getRequestURL().toString());

        String token = request.getParameter("token");// 获取请求的参数

        if (StringUtils.isNotBlank(token)) {
            ctx.setSendZuulResponse(true); //对请求进行路由
            ctx.setResponseStatusCode(200);
            ctx.set("isSuccess", true);
            return null;
        } else {
            ctx.setSendZuulResponse(false); //不对其进行路由
            ctx.setResponseStatusCode(400);
            ctx.setResponseBody("token is empty");
            ctx.set("isSuccess", false);
            return null;
        }
    }

}

```

将TokenFilter加入到请求拦截队列，在启动类中添加以下代码：

```
@Bean
public TokenFilter tokenFilter() {
	return new TokenFilter();
}
```

```
http://localhost:8888/spring-cloud-producer/hello?name=neo&token=xx
访问时，路径带token的才能进
```

## 7.路由熔断

当后端服务出现异常时，会在网页上出现类似500的那种错误。可通过路由熔断，返回想要的信息

实现一个接口，ZuulFallbackProvider默认有两个方法，一个用来指明熔断拦截哪个服务，一个定制返回内容。

```
@Component
public class ProducerFallback implements FallbackProvider {
    private final Logger logger = LoggerFactory.getLogger(FallbackProvider.class);

    //指定要处理的 service。
    @Override
    public String getRoute() {
        return "spring-cloud-producer";
    }

    public ClientHttpResponse fallbackResponse() {
        return new ClientHttpResponse() {
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.OK;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return 200;
            }

            @Override
            public String getStatusText() throws IOException {
                return "OK";
            }

            @Override
            public void close() {

            }

            @Override
            public InputStream getBody() throws IOException {
                return new ByteArrayInputStream("The service is unavailable.".getBytes());
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.APPLICATION_JSON);
                return headers;
            }
        };
    }

    @Override
    public ClientHttpResponse fallbackResponse(Throwable cause) {
        if (cause != null && cause.getCause() != null) {
            String reason = cause.getCause().getMessage();
            logger.info("Excption {}",reason);
        }
        return fallbackResponse();
    }
}

```

当服务出现异常时，打印相关异常信息，并返回”The service is unavailable.”。

发生情况例子：

启动项目spring-cloud-producer-2，这时候服务中心会有两个spring-cloud-producer项目，我们重启Zuul项目。再手动关闭spring-cloud-producer-2项目，多次访问地址：`http://localhost:8888/spring-cloud-producer/hello?name=neo&token=xx`，会交替返回：

```
hello neo，this is first messge
The service is unavailable.
...
```

## 8.路由重试

跟路由重试差不多，就是不返回想要的错误提示信息，而是进行重试。

依赖：

```
<dependency>
	<groupId>org.springframework.retry</groupId>
	<artifactId>spring-retry</artifactId>
</dependency>

```

配置：

```
#是否开启重试功能
zuul.retryable=true
#对当前服务的重试次数
ribbon.MaxAutoRetries=2
#切换相同Server的次数
ribbon.MaxAutoRetriesNextServer=0
```

```

将producer2修改，模拟producer2服务坏了。
@RequestMapping("/hello")
public String index(@RequestParam String name) {
    logger.info("request two name is "+name);
    try{
        Thread.sleep(1000000);
    }catch ( Exception e){
        logger.error(" hello two error",e);
    }
    return "hello "+name+"，this is two messge";
}

```

当访问http://localhost:8888/spring-cloud-producer/hello?name=neo&token=xx时，如果producer2坏了访问不上，则重试两次。

## 9.zuul的高可用

我们实际使用Zuul的方式如上图，不同的客户端使用不同的负载将请求分发到后端的Zuul，Zuul在通过Eureka调用后端服务，最后对外输出。因此为了保证Zuul的高可用性，前端可以同时启动多个Zuul实例进行负载，在Zuul的前端使用Nginx或者F5进行负载转发以达到高可用性。

# 5.SpringCloudGateWay

**1.简介：**

Spring Cloud GateWay是spring Cloud自己的东西。相比较zuul,支持长连接，如;WebSockets,支持限流等新特性。

**2.Spring Cloud Gateway 的特征：**

- 基于 Spring Framework 5，Project Reactor 和 Spring Boot 2.0
- 动态路由
- Predicates 和 Filters 作用于特定路由
- 集成 Hystrix 断路器
- 集成 Spring Cloud DiscoveryClient
- 易于编写的 Predicates 和 Filters
- 限流
- 路径重写

依赖：

```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>

```

配置：

```
server:
  port: 8080
spring:
  cloud:
    gateway:
      routes:
      - id: neo_route                                 id为neo_route的路由规则
        uri: http://www.ityouknow.com                  目标地址
        predicates:
        - Path=/spring-cloud

```

- predicates:路由条件，Predicate 接受一个输入参数，返回一个布尔值结果。该接口包含多种默认方法来将 Predicate 组合成其他复杂的逻辑（比如：与，或，非）。
- filters：过滤规则，本示例暂时没用。

## 1.路由规则

### 1.通过时间匹配

```
server:
  port: 8080
spring:
  cloud:
    gateway:
      routes:
      - id: neo_route
        uri: http://www.ityouknow.com
        predicates:
        - After=2018-01-20T06:06:06+08:00[Asia/Shanghai]
注意空格，有时候空格差一点都不好使。
```

请求时间在 2018年1月20日6点6分6秒之后的所有请求都转发到地址`http://ityouknow.com`。`+08:00`是指时间和UTC时间相差八个小时，时间地区为`Asia/Shanghai`。

### 2.通过Cookie匹配

### 3.通过Header属性匹配

4.通过Host匹配

5.通过请求方式匹配

6.通过请求路径匹配。

7.通过请求参数匹配

8.通过请求ip地址进行匹配

### 9.综合使用

```
server:
  port: 8080
spring:
  cloud:
    gateway:
      routes:
       - id: host_foo_path_headers_to_httpbin
        uri: http://ityouknow.com
        predicates:
        - Host=**.foo.org
        - Path=/headers
        - Method=GET
        - Header=X-Request-Id, \d+
        - Query=foo, ba.
        - Query=baz
        - Cookie=chocolate, ch.p
        - After=2018-01-20T06:06:06+08:00[Asia/Shanghai]

```

## 2.Gateway服务化

将使用的服务都调整到springboot2.0以上。

可参考http://www.ityouknow.com/springcloud/2019/01/19/spring-cloud-gateway-service.html

依赖：

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

```

配置：

```
server:
  port: 8888
spring:
  application:
    name: cloud-gateway-eureka
  cloud:
    gateway:
     discovery:
        locator:
         enabled: true             
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8000/eureka/
logging:
  level:
    org.springframework.cloud.gateway: debug 调整相 gateway 包的 log 级别，以便排查问题

```

- `spring.cloud.gateway.discovery.locator.enabled`：与注册中心自动结合。直接通过服务名进行访问

```
http://localhost:8888/SPRING-CLOUD-PRODUCER/hello？name=xxx
 http://localhost:9000/hello?name=xxx
```

# 6.Docker

一个开源的应用容器引擎 基于 [Go 语言](http://www.runoob.com/go/go-tutorial.html) 并遵从Apache2.0协议开源。

Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。



解决"在我的电脑上运行好使啊，在你的电脑上运行咋就不好使"的情况。

# 7.Eureka

依赖:

```
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka-server</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>
```

```
@EnableEurekaServer                      注解
```

```
配置文件：
spring.application.name=spring-cloud-eureka

server.port=8000
eureka.client.register-with-eureka=false //表示是否将自己注册到Eureka Server上，这里选择否
eureka.client.fetch-registry=false//表示是否从Eureka获取注册信息，否

eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/    
//设置与Eureka Server交互地址,默认是http://localhost:8761/eureka 
```

访问：http://localhost:8000/

这是单点服务的 正常项目必须搭建集群

http://www.ityouknow.com/spring-cloud.html

# 8.生产和消费

## 1.生产

依赖：

```
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>
```

配置：

```
spring.application.name=spring-cloud-producer
server.port=9000
eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/
```

```
注解：
@EnableDiscoveryClient    //能被发现的客户端
```

设置接口路径：

```
@RestController
public class HelloController {
	
    @RequestMapping("/hello")
    public String index(@RequestParam String name) {
        return "hello "+name+"，this is first messge";
    }
}

```

## 2.消费

依赖一样

配置：只需要更改端口号和服务名

```
spring.application.name=spring-cloud-consumer
server.port=9001
eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/
```

注解：

```
 @EnableDiscoveryClient //能被发现的客户端
 @EnableFeignClients //能远程调用的客户端
```

### 2.1Feign

依赖

```
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```

实现一个接口上面加上注解

```
@FeignClient(name= "spring-cloud-producer")     //远程调用
```

```
@FeignClient(name= "spring-cloud-producer")
public interface HelloRemote {
    @RequestMapping(value = "/hello")
    public String hello(@RequestParam(value = "name") String name);
}
```

这里的spring-cloud-producer是生产者的名字，生产者和消费者的方法名和参数名应一致。

一个消费者调用多个生产者时，会负载均衡，轮询访问。

Feign可以与Eureka和Ribbon组合使用以支持负载均衡。

# 9.Ribbon

是一个基于Http和TCP的负载均衡工具，通过Spring Cloud的封装，可以讲面向服务的Rest模板转换成客户端的负载均衡调用。

5.1Ribbon和Feign的区别

Ribbon和Feign都是用于调用其他服务的，不过方式不同。

1.启动类使用的注解不同，Ribbon用的是@RibbonClient，Feign用的是@EnableFeignClients。

2.服务的指定位置不同，Ribbon是在@RibbonClient注解上声明，Feign则是在定义抽象方法的接口中使用@FeignClient声明。

3.调用方式不同，Ribbon需要自己构建http请求，模拟http请求然后使用RestTemplate发送给其他服务，步骤相当繁琐。

  Feign则是在Ribbon的基础上进行了一次改进，采用接口的方式，将需要调用的其他服务的方法定义成抽象方法即可，

  不需要自己构建http请求。不过要注意的是抽象方法的注解、方法签名要和提供服务的方法完全一致。

# 10.Springcloud Springboot2.0

3.springCloud springboot2.0的整体依赖。

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.6.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>spring-cloud-gateway</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>spring-cloud-gateway</name>
	<description>springcloudgateway for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Finchley.SR2</spring-cloud.version>
	</properties>

	<dependencies>
		<!-- spring cloud 自己的网关 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-gateway</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<repositories>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
		</repository>
	</repositories>

</project>

```

