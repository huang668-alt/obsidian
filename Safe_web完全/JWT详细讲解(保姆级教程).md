版权声明：

本文内容由阿里云实名注册用户自发贡献，版权归原作者所有，阿里云开发者社区不拥有其著作权，亦不承担相应法律责任。具体规则请查看《 [<font style="color:#1366ec;">阿里云开发者社区用户服务协议</font>](https://developer.aliyun.com/article/768092)》和 《[<font style="color:#1366ec;">阿里云开发者社区知识产权保护指引</font>](https://developer.aliyun.com/article/768093)》。如果您发现本社区中有涉嫌抄袭的内容，填写 [<font style="color:#1366ec;">侵权投诉表单</font>](https://yida.alibaba-inc.com/o/right)进行举报，一经查实，本社区将立刻删除涉嫌侵权内容。 

**个人推荐:**

>  📢📢📢 前些天发现了一个蛮有意思的人工智能学习网站,8个字形容一下 `**"通俗易懂，风趣幽默"**`
>
> ，感觉非常有意思,忍不住分享一下给大家。 [点击跳转到教程](https://www.captainai.net/itguy)
>
> 。 
>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314005-86b74393-dd05-49fb-a19d-89fa4a822f22.webp)

## 一.JWT
### 1.什么是JWT
>  JSON Web Token (JWT)是一个开放标准(RFC 7519) ，它定义了一种紧凑和自包含的方式， `**用于作为 JSON 对象在各方之间安全地传输信息**`
>
> 。此信息可以进行验证和信任，因为它是经过数字签名的。JWT 可以使用机密(使用 HMAC 算法)或使用 RSA 或 ECDSA 的公钥/私钥对进行签名。   
虽然可以对 JWT 进行加密，以便在各方之间提供保密性，但是我们将关注已签名的Token。签名Token可以验证其中包含的声明的完整性，而加密Token可以向其他方隐藏这些声明。当使用公钥/私钥对对令牌进行签名时，该签名还证明只有持有私钥的一方才是对其进行签名的一方( `**签名技术是保证传输的信息不被篡改,并不能保证信息传输的安全**`
>
>  )。 
>

官网地址:[https://jwt.io/introduction](https://jwt.io/introduction)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741313794-f4f17511-881a-4176-bce6-65afad127f0f.webp)

### 2.JWT的结构
在其紧凑的形式中，JWT由以点(.)分隔的三个部分组成，它们是:

+ Header
+ Payload
+ Signature

类似于xxxx.xxxx.xxxx格式,真实情况如下:

```plain
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

并且你可以通过官网[https://jwt.io/#debugger-io](https://jwt.io/#debugger-io)解析出三部分表示的信息( `**可使用 JWT.io Debugger 来解码、验证和生成 JWT**` ):  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741313801-6773c649-3aaf-4bb1-98ed-8a25852725d1.webp)

#### (1) Header
>  报头通常由两部分组成: Token的类型(即 JWT)和所使用的签名算法(如 HMAC SHA256或 RSA)。 
>

例如:

```plain
{
  "alg": "HS256",
  "typ": "JWT"
}
```

最终这个 JSON 将由base64进行加密（该加密是可以对称解密的)，用于构成 JWT 的第一部分,eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9就是base64加密后的结果。

#### (2) Payload
>  Token的第二部分是有效负载，其中包含声明。声明是关于实体(通常是用户)和其他数据的语句。有三种类型的声明: registered claims, public claims, and private claims。 
>

例如:

```plain
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true 
}
```

这部分的声明也会通过base64进行加密,最终形成JWT的第二部分eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ  
**registered claims(注册声明)**

>  这些是一组预定义的声明，它们 `**不是强制性的，而是推荐的**`
>
>  ，以 `**提供一组有用的、可互操作的声明**`
>
>  。 
>

例如:

+ iss: jwt签发者
+ sub: jwt所面向的用户
+ aud: 接收jwt的一方
+ exp: jwt的过期时间，这个过期时间必须要大于签发时间
+ nbf: 定义在什么时间之前，该jwt都是不可用的.
+ iat: jwt的签发时间
+ jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击

`**注意:声明名称只有三个字符，因为 JWT 意味着是紧凑的。**`

**Public claims(公共的声明)**

>  使用 JWT 的人可以随意定义这些声明( `**可以自己声明一些有效信息如用户的id,name等,但是不要设置一些敏感信息,如密码**`
>
>  )。但是为了避免冲突，应该在 JWT注册表中定义它们，或者将它们定义为包含抗冲突名称空间的 URI。 
>

**Private claims(私人声明)**

>  这些是创建用于在同意使用它们的各方之间共享信息的习惯声明，既不是注册声明，也不是公开声明( `**私人声明是提供者和消费者所共同定义的声明**`
>
>  )。 
>

`**注意:对于已签名的Token，这些信息虽然受到保护，不会被篡改，但任何人都可以阅读。除非加密，否则不要将机密信息放在 JWT 的有效负载或头元素中。**`

#### (3) Signature
>  要创建Signature，您必须获取编码的标头（header）、编码的有效载荷(payload)、secret、标头中指定的算法，并对其进行签名。 
>

例如，如果您想使用 HMAC SHA256算法，签名将按以下方式创建:

```plain
HMACSHA256(
  base64UrlEncode(header) + "." +base64UrlEncode(payload),
  secret
  )
```

上面的JSON将会通过HMACSHA256算法结合secret进行加盐签名(私钥加密)，其中header和payload将通过base64UrlEncode()方法进行base64加密然后通过字符串拼接 `**"."**` 生成新字符串,最终生成JWT的第三部分SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

`**注意:secret是保存在服务器端的，jwt的签发生成也是在服务器端的，secret就是用来进行jwt的签发和验证，所以，它就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端得知这个secret, 那就意味着客户端是可以自我签发jwt了**`

#### (4) JWT的生成与解析
>  JWT输出是三个由点分隔的 Base64-URL 字符串，这些字符串可以在 HTML 和 HTTP 环境中轻松传递，同时与基于 XML 的标准(如 SAML)相比更加紧凑。 
>

下面显示了一个 JWT，该 JWT 对前一个头和有效负载进行了编码，并使用一个 secret 进行签名。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741313822-1479fdee-0765-4089-9be6-b0bbbf171fb7.webp)

真实情况,一般是在请求头里加入Authorization，并加上Bearer标注最后是JWT(格式:Authorization: Bearer `**<token>**`)：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741313812-418bc14f-ab1e-45dc-9299-2c40b7047545.webp)

+ 通过Java代码实现JWT的生成( `**使用的是JJWT框架**` )

先导入JJWT的依赖(JJWT是JWT的框架)

```plain
<dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.1</version>
        </dependency>
```

测试代码如下:

```plain
public class JjwtTest {
    @Test
    public void generateToken() {
        
        Map<String, Object> header = new HashMap<>();
        header.put("alg", "HS256");
        header.put("typ", "JWT");

        
        Map<String, Object> payload = new HashMap<>();
        payload.put("sub", "1234567890");
        payload.put("name","John Doe");
        payload.put("admin",true);

        
        Calendar instance = Calendar.getInstance();
        instance.add(Calendar.SECOND,300);

        
        String token = Jwts.builder()
                .setHeader(header)
                .setClaims(payload) 
                .setExpiration(instance.getTime())
                .signWith(SignatureAlgorithm.HS256,"secret") 
                .compact(); 

        System.out.println(token);
    }
}
```

运行结果:

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314057-1c418c48-0780-42e1-9c00-7a86a524a0f1.webp)

通过官网进行解码:  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314131-ebdecae3-2e54-4446-b8bd-d9ead99ba35d.webp)

+ 通过Java代码实现JWT的解码( `**使用的是JJWT框架**` )

测试代码如下:

```plain
@Test
    public void getInfoByJwt() {
        
        String token = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImV4cCI6MTY2MzI5NzQzMX0.Ju5EzKBpUnuIRhDG1SU0NwMGsd9Jl_8YBcMM6PB2C20";
        
        JwsHeader jwsHeader = Jwts
                .parser()
                .setSigningKey("secret")
                .parseClaimsJws(token)
                .getHeader();

        System.out.println(jwsHeader); 
        System.out.println("typ:"+jwsHeader.get("typ"));

        
     Claims claims =    Jwts
                .parser()
                .setSigningKey("secret")
                .parseClaimsJws(token)
                .getBody();
        System.out.println(claims);
        System.out.println("admin:"+claims.get("admin"));

        
     String signature =    Jwts
                .parser()
                .setSigningKey("secret")
                .parseClaimsJws(token)
                .getSignature();
        System.out.println(signature); 
    }
```

运行结果:  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314072-d3655a1c-4e23-431f-88b7-5453f8dc381c.webp)

+ JWT工具类(`**使用的是JJWT框架**` )

当然在实际项目中一般会将上面的操作封装成工具类来使用如下( `**该项目是一个Spring Security+JWT的项目**` ):

应用配置文件application.yaml中加入如下配置:

```plain
jwt:
  tokenHeader: Authorization 
  secret: mall-admin-secret 
  expiration: 604800 
  tokenHead: 'Bearer '
```

工具类代码如下:

```plain
package com.dudu.mall.utils;

import cn.hutool.core.date.DateUtil;
import cn.hutool.core.util.StrUtil;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;


public class JwtTokenUtil {
    private static final Logger LOGGER = LoggerFactory.getLogger(JwtTokenUtil.class);
    private static final String CLAIM_KEY_USERNAME = "sub";
    private static final String CLAIM_KEY_CREATED = "created";
    @Value("${jwt.secret}")
    private String secret;
    @Value("${jwt.expiration}")
    private Long expiration;
    @Value("${jwt.tokenHead}")
    private String tokenHead;

    
    private String generateToken(Map<String, Object> claims) {
        return Jwts.builder()
                .setClaims(claims)
                .setExpiration(generateExpirationDate())
                .signWith(SignatureAlgorithm.HS512, secret)
                .compact();
    }

    
    private Claims getClaimsFromToken(String token) {
        Claims claims = null;
        try {
            claims = Jwts.parser()
                    .setSigningKey(secret)
                    .parseClaimsJws(token)
                    .getBody();
        } catch (Exception e) {
            LOGGER.info("JWT格式验证失败:{}", token);
        }
        return claims;
    }

    
    private Date generateExpirationDate() {
        return new Date(System.currentTimeMillis() + expiration * 1000);
    }

    
    public String getUserNameFromToken(String token) {
        String username;
        try {
            Claims claims = getClaimsFromToken(token);
            username = claims.getSubject();
        } catch (Exception e) {
            username = null;
        }
        return username;
    }

    
    public boolean validateToken(String token, UserDetails userDetails) {
        String username = getUserNameFromToken(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }

    
    private boolean isTokenExpired(String token) {
        Date expiredDate = getExpiredDateFromToken(token);
        return expiredDate.before(new Date());
    }

    
    private Date getExpiredDateFromToken(String token) {
        Claims claims = getClaimsFromToken(token);
        return claims.getExpiration();
    }

    
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put(CLAIM_KEY_USERNAME, userDetails.getUsername());
        claims.put(CLAIM_KEY_CREATED, new Date());
        return generateToken(claims);
    }

    
    public String refreshHeadToken(String oldToken) {
        if(StrUtil.isEmpty(oldToken)){
            return null;
        }
        String token = oldToken.substring(tokenHead.length());
        if(StrUtil.isEmpty(token)){
            return null;
        }
        
        Claims claims = getClaimsFromToken(token);
        if(claims==null){
            return null;
        }
        
        if(isTokenExpired(token)){
            return null;
        }
        
        if(tokenRefreshJustBefore(token,30*60)){
            return token;
        }else{
            claims.put(CLAIM_KEY_CREATED, new Date());
            return generateToken(claims);
        }
    }

    
    private boolean tokenRefreshJustBefore(String token, int time) {
        Claims claims = getClaimsFromToken(token);
        Date created = claims.get(CLAIM_KEY_CREATED, Date.class);
        Date refreshDate = new Date();
        
        if(refreshDate.after(created)&&refreshDate.before(DateUtil.offsetSecond(created,time))){
            return true;
        }
        return false;
    }
}
```

### 3.JWT是如何工作的
#### (1) JWT工作流程
下图显示了 JWT 是如何获得并用于访问 API 或资源的:  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314140-167392c8-56fe-457e-a321-dd7e796a087d.webp)

1. 应用程序或客户端向授权服务器请求授权。
2. 授予授权后，授权服务器将向应用程序返回访问令牌。
3. 应用程序使用访问令牌访问受保护的资源(如 API)。

#### (2) SpringBoot整合JWT案例
>  本项目JWT使用的是 `**JJWT框架**`
>
>  来实现,下图流程图来源于 [【编程不良人】JWT认证原理、流程整合springboot实战应用,前后端分离认证的解决方案!](https://www.bilibili.com/video/BV1i54y1m7cP?p=6&vd_source=5d242c523ffcdeae19faea79735ffef6)
>
> ,项目采用SpringBoot+mybatis实现,业务逻辑非常简单,下面截取关键代码来了解整个流程。 
>

业务流程图如下:  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314194-62064b93-446f-4bff-990c-192a6b259a12.webp)  
登入功能通过业务层提供的login(user)方法进行登入,登入成功返回token信息,如果token信息不为null就将token信息返回给前端系统  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314343-c7609a3e-f8a9-4e20-8230-d03f5d5688bd.webp)  
业务层login(User user)方法实现逻辑非常简单,就是通过查询数据库中是否存在该用户,存在表示登入成功,并通过Token工具类生成token  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314294-86efe56a-1647-427e-9483-d745a4717854.webp)  
JWT工具类中generateToken(User user)虽然接受的是user对象,但是实际只使用了user的name作为载荷( `**不要将敏感信息写入到Token中**` )  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314458-39cb42cf-8d4f-432a-af36-d8b0049733ae.webp)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314396-99f49c9f-fc06-49e0-a83d-b6d2816943b9.webp)

Postman中模拟前端访问[http://localhost](http://localhost/):8086/user/login?name=test&password=123456:  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314398-0e5dc032-6da2-4e2e-845d-9f512fa1565e.webp)

执行成功后前端会接受到服务器端传递过来的Token信息  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314531-fdbe6960-2ca1-43ef-888d-f6feb01ddea8.webp)  
访问Api信息/资源,前端只需要将token信息保存到请求头中,发送请求到对应Api/资源即可:  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314562-3f6e334e-3730-41fd-8d0c-cff4b451f9d9.webp)  
Postman模拟前端访问:[http://localhost](http://localhost/):8086/user/info?id=1  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314608-c4929405-4c26-42ec-8252-dc8a98c3ca4f.webp)  
至于Token的验证交给拦截器进行处理:

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314763-8275a057-3299-4888-b4f0-2e84b9d41920.webp)  
Postman访问/info:

+ 访问成功

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314680-cd44fa04-2fae-4b4b-bbad-a5939ebcd8cb.webp)

+ 访问失败

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314749-3a8ad40e-cf4b-4e6f-844b-94a07e9c82a0.webp)

### 4.JWT的使用
>  📢下面的内容借鉴博文: [https://zhuanlan.zhihu.com/p/66037342](https://zhuanlan.zhihu.com/p/66037342)
>
> ,该博文详细的介绍了单点登入相关知识,借鉴博文 [什么是 JWT -- JSON WEB TOKEN](https://www.jianshu.com/p/576dbf44b2ae?u_atoken=91172cd8-ee38-4f00-bd87-5dab015b84f6&u_asession=01Q62IwvW6pph_Ncb2bPTfJTtcPIIDd_JBDPweKSj23X4YV8vLFItAzGVhyzguKl8_X0KNBwm7Lovlpxjd_P_q4JsKWYrT3W_NKPr8w6oU7K88eJ9fBntafi4EfbpIWArOS_nntv0fnCqKMMYW1x5qJGBkFo3NEHBv0PZUm6pbxQU&u_asig=05VwcfgbreX6hu5gaiqbdbQPvTtZl3IjQykoynrX1-4ExH5Ei0IDhjyv82ZyZE9wTBqdHkfwlUM1djsumiL9sKCbyFJjrbA7vd09MjD-P1ZOAx6cYJ27w4jv5eIwPQDzPNg6zhtt9JS9Y2jr8XricVeh4fFGLUdkWD1st6FbkSzcX9JS7q8ZD7Xtz2Ly-b0kmuyAKRFSVJkkdwVUnyHAIJzdXCcbaxhcYEdIoWbAvQAIUFfrsZmVP-RYvrDdSkFZgr4Nx7nJyT20ni5onZOBbfUe3h9VXwMyh6PgyDIVSG1W9HpIJPqVKGPRDUkkOkQzbkjwQ0iZx7g0iaRIPGdTGFH-NI70t7RyxF--pNyQek64UfCt-d4M_yJ8xADtgch1kemWspDxyAEEo4kbsryBKb9Q&u_aref=W%2BMkoYG9XQQ5tQ52WnFs4GpiT%2BY=)
>
> ,该篇博文详细的介绍了JWT相关知识。 
>

#### (1) Authorization(授权)
>  这是使用 JWT 最常见的场景。 `**一旦用户登录，每个后续请求都将包含 JWT，允许用户访问该Token所允许的路由、服务和资源**`
>
> 。 `**单点登录**`
>
>  是目前广泛使用 JWT的一个特性，因为它的开销很小，而且 `**能够很容易地跨不同域使用**`
>
>  。 
>

+ **单系统**

>  单系统就是所有功能都在同一个系统上 
>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314771-5223d86e-0c7a-4a28-b524-c0278958f11f.webp)

+ **多系统**

>  为了合理利用资源和降低耦合性，于是把单系统拆分成多个子系统（ `**传统基于session认证的登入设计是无法实现多系统登入的,因为session无法跨域**`
>
>  ） 
>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314841-9d52951f-8c13-4806-be7b-c56b2105b73e.webp)

+ **单点登入**

>  关于什么是单点登入,简单概述一下就是,在多个系统中，用户只需一次登录，各个系统即可感知该用户已经登录,例如淘宝和天猫这二个不同的子系统,当登入淘宝的时候,天猫自然而然就登入了。 
>

+ 单系统登入操作(基于session认证)

>  Http是一种无状态的协议,意味着服务器无法确认用户信息,所以W3C推出Cookie来确认用户的信息,Cookie就相当于一个用户凭证,服务器可以通过这个凭证(Cookie)知道是哪一个用户发起了请求，虽然我们可以通过Cookie知道发起请求是谁,但是我们怎么知道这个用户是否已经发送过Cookie了呢,意思就是我们怎么知道某个用户是否已经登入了,这个时候就需要通过一张表来记录用户的信息,这就是Session，相当于在服务器中建立了一份“客户明细表”,Session无法知道发起的用户是谁,所以服务器为了确定用户的信息,会向服务器发送一个名为JESSIONID的Cookie，它的值是Session的id值,来确认用户的身份。 
>

单系统登入常见的操作:

+ 登入:将用户信息保存到Seesion对象中(Seesion中有用户信息表示用户已经登入,没有表示用户没有登入)
+ 注销(退出登入): 将服务器中的Seesion对象移除
+ 记住我:通过Cookie保存用户相关信息(这里的信息只能是字符串不能是对象,所以对象信息常常保存到数据库中),游览器关闭后任然还可以保持登入

业务代码如下:

```plain
@PostMapping(value = "/user/session", produces = {"application/json;charset=UTF-8"})
public Result login(String mobileNo, String password, String inputCaptcha, HttpSession session, HttpServletResponse response) {

    
    if (WebUtils.validateCaptcha(inputCaptcha, "captcha", session)) {

        
        User user = userService.userLogin(mobileNo, password);
        if (user != null) {
            
            String loginToken = WebUtils.md5(new Date().toString() + session.getId());
            user.setLoginToken(loginToken);
            User user1 = userService.userUpload(user);

            session.setAttribute("user", user1);

            CookieUtil.addCookie(response,"loginToken",loginToken,604800);

            return ResultUtil.success(user1);

        } else {
            return ResultUtil.error(ResultEnum.LOGIN_ERROR);
        }
    } else {
        return ResultUtil.error(ResultEnum.CAPTCHA_ERROR);
    }

}


@DeleteMapping(value = "/session", produces = {"application/json;charset=UTF-8"})
public Result logout(HttpSession session,HttpServletRequest request,HttpServletResponse response ) {

    
    session.removeAttribute("user");

    CookieUtil.clearCookie(request, response, "loginToken");

    return ResultUtil.success();
}

public class UserInterceptor implements HandlerInterceptor {


@Autowired
private UserService userService;

public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object o) throws Exception {
    User sessionUser = (User) request.getSession().getAttribute("user");

    
    if (sessionUser != null) {
        return true;
    } else {
        
        String loginToken = CookieUtil.findCookieByName(request, "loginToken");
        if (StringUtils.isNotBlank(loginToken)) {
            
            User user = userService.findUserByLoginToken(loginToken);
            if (user != null) {
                request.getSession().setAttribute("user", user);
                return true;
            } else {
                
                CookieUtil.clearCookie(request, response, "loginToken");
                return false;
            }
        } else {

            
            if (request.getRequestURI().contains("session")) {
                return true;
            }

            
            response.sendRedirect("/login.html");
            return false;
        }
    }
}
}
```

上面的代码可以结合下图进行理解:  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314969-a272ca2f-0357-4e2f-a94d-556b9711eac9.webp)

基于session认证的登入系统存在的问题:

+ session

>  每个用户经过我们的应用认证之后，我们的应用都要在服务端做一次记录，以方便用户下次请求的鉴别，通常而言session都是保存在内存中，而随着认证用户的增多，服务端的开销会明显增大。 
>

+ 扩展性

>  用户认证之后，服务端做认证记录，如果认证的记录被保存在内存中的话，这意味着用户下次请求还必须要请求在这台服务器上,这样才能拿到授权的资源，这样在分布式的应用上，相应的限制了负载均衡器的能力。这也意味着限制了应用的扩展能力。 
>

+ CSRF

>  因为是基于cookie来进行用户识别的, cookie如果被截获，用户就会很容易受到跨站请求伪造的攻击。 
>

基于token的鉴权机制,就没有这些问题:

+ 基于token的鉴权机制

>  基于token的鉴权机制类似于http协议也是无状态的，它不需要在服务端去保留用户的认证信息或者会话信息。这就意味着基于token认证机制的应用不需要去考虑用户在哪一台服务器登录了，这就为应用的扩展提供了便利。 
>

实际流程如下（ `**对应项目demo前面已经简述了,这里不在叙述**` ）:  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741315021-bff04f2a-ab9a-42ca-898b-9d530d06edd3.webp)

#### (2) Information Exchange(信息交换)
>  JWT是在各方之间安全传输信息的好方法。因为可以对 JWT 进行签名(例如，使用公钥/私钥对) ，所以可以确保发件人就是他们所说的那个人。此外，由于签名是使用头和有效负载计算的，因此还可以验证内容是否被篡改。 
>

+ 数字签名

>  数字签名就是发送方使用自己的私钥进行加密,接收方收到该签名后就可以使用发送方的公钥(公开的密钥,所有人都可以获取)进行解密,解密成功表示该消息没有被假冒且发送方也不能否认自己发送过。 
>

### 5.为什么使用JWT
+ 由于 JSON 没有 XML 那么冗长，所以当对它进行编码时，它的大小也更小，这使得 JWT 比 SAML 更加紧凑。这使得 JWT 成为在 HTML 和 HTTP 环境中传递的一个很好的选择。
+ 在安全性方面，SWT 只能由使用 HMAC 算法的共享秘密对称签名。但是，JWT 和 SAML Token可以使用 X.509证书形式的公钥/私钥对进行签名。与签名 JSON 的简单性相比，使用 XML 数字签名,签名 XML 而不引入模糊的安全漏洞是非常困难的。
+ JSON 解析器在大多数编程语言中都很常见，因为它们直接映射到对象。相反，XML 没有自然的文档到对象映射。这使得使用 JWT 比使用 SAML 断言更容易。
+ 关于使用，JWT 是在 Internet 规模上使用的。这突出了在多个平台(尤其是移动平台)上对 JSON Web 令牌进行客户端处理的便利性。

下图就是JWT和SAML长度的比较:  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741315177-f82291d5-1ad7-4150-aafb-a8056f48d5cb.webp)

### 6.资源下载
微信公众号搜索程序员孤夜(或扫描下方二维码),后台回复 `**JWT**` ,即可获取SpringBoot整合JWT案例的源码。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/58546395/1763741314999-53da63b3-c0be-4034-8c6b-6ce0a4b6347b.webp)



> 来自: [JWT详细讲解(保姆级教程)-阿里云开发者社区](https://developer.aliyun.com/article/995894)
>





