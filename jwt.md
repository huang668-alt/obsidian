
**JWT（JSON Web Token）** 的结构非常标准且紧凑。在物理形式上，它是由点（`.`）分隔的三部分构成的英文字符串。

一个典型的 JWT 看起来是这样的：

`xxxxx.yyyyy.zzzzz`

这三部分从左到右依次是：**Header（头部）**、**Payload（负载）** 和 **Signature（签名）**。

以下是这三个部分的底层结构拆解：

### 1. Header（头部）

Header 通常由一个 JSON 对象组成，用来描述这个 Token 的**元数据**（基本信息）。它通常包含两个部分：

- `alg`：声明用来生成签名的加密算法（例如 `HS256` 即 HMAC SHA256，或 `RS256`）。
    
- `typ`：声明这个 Token 的类型，固定为 `JWT`。
    

**JSON 明文示例：**

``` json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**处理方式：** JVM 会将这个 JSON 经过 **Base64URL 编码**，变成 JWT 的第一部分（`xxxxx`）。

### 2. Payload（负载）

Payload 也是一个 JSON 对象，用来存放**实际传输的业务数据（Claims/声明）**。它就像是信件的正文。G1、Spring Security 等架构中经常用它来传递用户信息。

Payload 里面的键值对通常分为三类：

- **标准注册声明（Registered Claims，建议使用但非强制）**：
    
    - `iss` (Issuer)：签发人
        
    - `exp` (Expiration time)：过期时间（绝对重要的物理物理防过期机制）
        
    - `sub` (Subject)：主题（通常放用户 ID）
        
    - `aud` (Audience)：接收方
        
- **公共声明（Public Claims）**：可以由使用者随意定义，但要防冲突。
    
- **私有声明（Private Claims）**：业务自定义的字段，比如用户的角色、姓名等。
    

**JSON 明文示例：**

``` json
{
  "sub": "1234567890",
  "name": "黄景涵",
  "roles": ["ROLE_USER", "ROLE_ADMIN"],
  "exp": 1780000000
}
```

**处理方式：** 同样经过 **Base64URL 编码**，变成 JWT 的第二部分（`yyyyy`）。

> ⚠️ **致命物理安全防范**：Base64URL 编码**不是加密**，它是可逆的！任何人拿到这段字符串都可以轻松解码看出明文。因此，**绝对不能在 Payload 中存放密码、银行卡号等敏感隐私数据**。

### 3. Signature（签名）

这是整个 JWT 的**安全核心**，用来防止 Token 被篡改（防伪标记）。

**生成物理公式：**

如果要使用 HMAC SHA256 算法生成签名，服务器底层的执行逻辑是：

```Plaintext
Signature = HMACSHA256(
    base64UrlEncode(Header) + "." + base64UrlEncode(Payload),
    secret_key (服务器独享的私钥)
)
```

**它的防篡改原理：**

1. 签名是把**编码后的 Header** 和**编码后的 Payload** 用点拼接，再配合服务器才知道的密钥（Secret Key）进行单向哈希加密生成的。
    
2. 最终生成的签名字符串作为第三部分（`zzzzz`）。
    
3. 如果黑客尝试在客户端篡改 Payload（比如把自己的角色改成 `ROLE_ADMIN`），由于他没有服务器的 `secret_key`，他无法重新算出一份合法的签名。
    
4. 当服务器收到这个 Token 时，会用自己的密钥对前两部分重新算一次签名，发现和第三部分对不上，就会立即判定 Token 非法，无情拒绝！
    

### 💡 总结 JWT 的流转本质

``` Plaintext
明文 JSON  ──► Base64URL 编码 ──► [Header].
明文 JSON  ──► Base64URL 编码 ──► [Header].[Payload].
前两部分 + 密钥 ──► 算法加密 ──► [Header].[Payload].[Signature] (最终生成的JWT)
```

在 Spring Security 或分布式微服务架构中，JWT 作为**无状态认证**的完美载体，由于它自身就包含了用户信息和过期时间（Payload），且自带防伪证明（Signature），因此服务器不需要再查 Session 数据库，直接在内存中解码验签即可通行，极大地提升了系统的并发吞吐量。