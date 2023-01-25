[原文链接](https://juejin.cn/post/6962142423879270437#heading-8)

### JWT

**JWT**(JSON Web Token)是为了在网络应用环境中传递声明而执行的一种基于JSON的开放标准

### 举例登录过程

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92c804f51d1b433f8c88bf16f825ded8~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2fbf09f15bfc4915999a98d0ac0759a9~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a487c1269314e81ba95f869ae46eaff~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 组成

JWT具体长什么样呢？ JWT是由三段信息构成的，将这三段信息文本用.链接一起就构成了JWT字符串。就像这样:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ

```

### 元素

#### header

**JWT的头部承载两部分信息：**

- 声明类型，这里是JWT；
- 声明加密的算法，通常直接使用 HMAC SHA256；
- 完整的头部就像下面这样的JSON：

```json
{
  'typ': 'JWT',
  'alg': 'HS256'
}

```

使用base64加密，构成了第一部分。
>eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9

#### playload（重点）

**载荷就是存放有效信息的地方，这些有效信息包含三个部分:**

- 标准中注册的声明；
- 公共的声明；
- 私有的声明； 其中，标准中注册的声明 (建议但不强制使用)包括如下几个部分 ：
	- iss: jwt签发者；
	- sub: jwt所面向的用户；
	- aud: 接收jwt的一方；
	- exp: jwt的过期时间，这个过期时间必须要大于签发时间；
	- nbf: 定义在什么时间之前，该jwt都是不可用的；
	- iat: jwt的签发时间；
	- jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击；

>**公共的声明部分：** 公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息，但不建议添加敏感信息，因为该部分在客户端可解密。
>**私有的声明部分：** 私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为base64是对称解密的，意味着该部分信息可以归类为明文信息。

**定义一个payload：**
```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}

```

**然后将其进行base64加密，得到Jwt的第二部分：**
>eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9


#### signature

**jwt的第三部分是一个签证信息，这个签证信息由三部分组成：**
```JavaScript
var encodedString = base64UrlEncode(header) + '.' + base64UrlEncode(payload);
 
 
var signature = HMACSHA256(encodedString, '密钥');
加密之后，得到signature签名信息。
```

**第三部分：**
>TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ

**jwt最终格式**
>eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ

**secret用来进行jwt的签发和jwt的验证，所以，在任何场景都不应该流露出去。**

### 元素

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/15422ee2ae3349a6b668c7084d986354~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/285a60c80b5242a9b608da990410f0f5~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### SpringBoot整合JWT【正片】

#### 引入依赖
```xml
<!--token-->
    <!-- jwt支持 -->
    <dependency>
        <groupId>com.auth0</groupId>
        <artifactId>java-jwt</artifactId>
    </dependency>

```

#### 创建JWT工具类

**注意静态属性的配置文件注入方式：**
```java
package com.neuq.common.util;

import com.auth0.jwt.JWT;
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.exceptions.TokenExpiredException;
import com.neuq.common.exception.ApiException;
import com.neuq.common.response.ResultInfo;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.Date;

/**
 * @Author: xiang
 * @Date: 2021/5/11 21:11
 * <p>
 * JwtToken生成的工具类
 * JWT token的格式：header.payload.signature
 * header的格式（算法、token的类型）,默认：{"alg": "HS512","typ": "JWT"}
 * payload的格式 设置：（用户信息、创建时间、生成时间）
 * signature的生成算法：
 * HMACSHA512(base64UrlEncode(header) + "." +base64UrlEncode(payload),secret)
 */

@Component
@ConfigurationProperties(prefix = "jwt")
public class JWTUtils {

    //定义token返回头部
    public static String header;

    //token前缀
    public static String tokenPrefix;

    //签名密钥
    public static String secret;

    //有效期
    public static long expireTime;

    //存进客户端的token的key名
    public static final String USER_LOGIN_TOKEN = "USER_LOGIN_TOKEN";

    public void setHeader(String header) {
        JWTUtils.header = header;
    }

    public void setTokenPrefix(String tokenPrefix) {
        JWTUtils.tokenPrefix = tokenPrefix;
    }

    public void setSecret(String secret) {
        JWTUtils.secret = secret;
    }

    public void setExpireTime(int expireTimeInt) {
        JWTUtils.expireTime = expireTimeInt*1000L*60;
    }

    /**
     * 创建TOKEN
     * @param sub
     * @return
     */
    public static String createToken(String sub){
        return tokenPrefix + JWT.create()
                .withSubject(sub)
                .withExpiresAt(new Date(System.currentTimeMillis() + expireTime))
                .sign(Algorithm.HMAC512(secret));
    }


    /**
     * 验证token
     * @param token
     */
    public static String validateToken(String token){
        try {
            return JWT.require(Algorithm.HMAC512(secret))
                    .build()
                    .verify(token.replace(tokenPrefix, ""))
                    .getSubject();
        } catch (TokenExpiredException e){
            throw new ApiException(ResultInfo.unauthorized("token已经过期"));
        } catch (Exception e){
            throw new ApiException(ResultInfo.unauthorized("token验证失败"));
        }
    }

    /**
     * 检查token是否需要更新
     * @param token
     * @return
     */
    public static boolean isNeedUpdate(String token){
        //获取token过期时间
        Date expiresAt = null;
        try {
            expiresAt = JWT.require(Algorithm.HMAC512(secret))
                    .build()
                    .verify(token.replace(tokenPrefix, ""))
                    .getExpiresAt();
        } catch (TokenExpiredException e){
            return true;
        } catch (Exception e){
            throw new ApiException(ResultInfo.unauthorized("token验证失败"));
        }
        //如果剩余过期时间少于过期时常的一半时 需要更新
        return (expiresAt.getTime()-System.currentTimeMillis()) < (expireTime>>1);
    }
}


```

#### yaml属性配置

```yaml
jwt:
  header: "Authorization" #token返回头部
  tokenPrefix: "Bearer " #token前缀
  secret: "qwertyuiop7418520" #密钥
  expireTime: 1 #token有效时间 (分钟) 建议一小时以上
```

#### 登录方法将用户信息存入token中返回

```java
  @Override
  public Map<String,Object> login(User user) {
      //phone是除id外的唯一标志 需要进行检查
      if (user.getPhone() == null || user.getPhone().equals(""))
          throw new ApiException("手机号不合法");
      User selectUser = userDao.selectUserByPhone(user.getPhone());
      if (selectUser == null) {
          //注册用户
          int count = userDao.insertUser(user);
          if (count < 1) throw new ApiException(ResultInfo.serviceUnavailable("注册异常"));
      }
      //将userId存入token中
      String token = JWTUtils.createToken(selectUser.getUserId().toString());
      Map<String,Object> map = new HashMap<>();
      map.put("user",selectUser);
      map.put("token",token);
      return map;
  }

```

**注意将token保存到Http 的 header**
```java
@GetMapping("/login")
public ResultInfo login(User user, HttpServletResponse response{
	Map<String, Object> map = userService.login(user);
	//将token存入Http的header中
	response.setHeader(JWTUtils.USER_LOGIN_TOKEN, (String) map.get("token"));
	return ResultInfo.success((User)map.get("user"));
}

```


##### 拦截器验证每次请求的token

```java
/**
 * @Author: xiang
 * @Date: 2021/5/7 20:56
 * <p>
 * 拦截器：验证用户是否登录
 */
public class UserLoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //http的header中获得token
        String token = request.getHeader(JWTUtils.USER_LOGIN_TOKEN);
        //token不存在
        if (token == null || token.equals("")) throw new ApiException("请先登录");
        //验证token
        String sub = JWTUtils.validateToken(token);
        if (sub == null || sub.equals(""))
            throw new ApiException(ResultInfo.unauthorized("token验证失败"));
        //更新token有效时间 (如果需要更新其实就是产生一个新的token)
        if (JWTUtils.isNeedUpdate(token)){
            String newToken = JWTUtils.createToken(sub);
            response.setHeader(JWTUtils.USER_LOGIN_TOKEN,newToken);
        }
        return true;
    }
}

```

**注册自定义拦截器**
```java
@Configuration
@ComponentScan(basePackages = "com.neuq.common") //全局异常处理类需要被扫描才能
public class WebMvcConfig implements WebMvcConfigurer {
    /**
     * 注册自定义拦截器
     *
     * @param registry
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new UserLoginInterceptor())
                .addPathPatterns("/user/**")
                .addPathPatterns("/userInfo/**")
                .excludePathPatterns("/user/login");//开放登录路径
    }

}

```

#### 单点登录

>将token或者一个唯一标识UUID=UUID.randomUUID().toString()存进Cookie中（别存在Http的header中了），设置路径为整个项目根路径/*； 往往以这个唯一标识为key，用户信息为value缓存在服务器中！！！

