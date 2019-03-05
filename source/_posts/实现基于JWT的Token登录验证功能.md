---
title: 实现基于JWT的Token登录验证功能
tags: Java Web
abbrlink: 58158
date: 2019-01-20 01:07:35
---

## 前言

放假之前做了几个小项目+课设，都用到了 token 实现登录验证和权限判断，然鹅当时和同组的小伙伴也都是第一次接触到了 token，于是乎都是一脸懵逼（xjbx）的写完了登录验证的前后端逻辑（我写前端，同组的小伙伴写后端）。今天有空仔细学习了一下 SpringBoot 实现 token 认证以及和前端的交互，踩了不少坑，在这里记录一下（现在已经 12:30 了…好困=·=）

<!-- more -->

## 后端实现

- 首先需要导入 jwt 的包，相关的 pom.xml 文件如下：

```
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.5.0</version>
</dependency>
```

- 然后开始编写 TokenUtil 类，首先定义 token 的过期时间和私钥

```
private static final long EXPIRE_TIME = 15 * 60 * 1000;
private static final String TOKEN_SECRET = "thefirsttoken123";
```

- 实现签名方法：
  这里不应该使用密码进行加密，不安全，但是是自己的小 demo 就这样写了。

```
/**
 * 生成签名，15分钟过期
 * @param **username**
* @param **password**
* @return
 */
public static String sign(String username, String password) {
    try {
        // 设置过期时间
        Date date = new Date(System.currentTimeMillis() + EXPIRE_TIME);
        // 私钥和加密算法
        Algorithm algorithm = Algorithm.HMAC256(TOKEN_SECRET);
        // 设置头部信息
        Map<String, Object> header = new HashMap<>(2);
        header.put("Type", "Jwt");
        header.put("alg", "HS256");
        // 返回token字符串
        return JWT.create()
                .withHeader(header)
                .withClaim("loginName", username)
                .withClaim("pwd", password)
                .withExpiresAt(date)
                .sign(algorithm);
    } catch (Exception e) {
        e.printStackTrace();
        return null;
    }
}
```

- 实现 token 的检验方法：

```
/**
 * 检验token是否正确
 * @param **token**
* @return
 */
public static boolean verify(String token){
    try {
        Algorithm algorithm = Algorithm.HMAC256(TOKEN_SECRET);
        JWTVerifier verifier = JWT.require(algorithm).build();
        DecodedJWT jwt = verifier.verify(token);
        return true;
    } catch (Exception e){
        return false;
    }
}
```

至此，工具类就编写完成啦！

- 登录的 controller 层方法
  这里获取到前端发送过来的请求体，取出其中的用户名和密码，和数据库比对如果无误的话，签发 token，并返回给前端。
  （API 响应结果还没有封装，看着有点乱，嘿嘿）

```
@PostMapping(value = "/login")
public Map<String, Object> login(@RequestBody SysUser sysUser){
    Map<String, Object> map = new HashMap<>();
    String username = sysUser.getUsername();
    String password = sysUser.getPassword();
    if (sysUserService.login(username, password)){
        String token = TokenUtil.sign(username,password);
        if (token != null){
            map.put("code", "10000");
            map.put("message","认证成功");
            map.put("token", token);
            return map;
        }
    }
    map.put("code", "00000");
    map.put("message","认证失败");
    return map;
}
```

现在服务端给客户端签发 token 的功能已经差不多实现了。
那么客户端如何将 token 应用到以后的请求中，服务端又如何识别 token 呢？

- 实现服务端自定义拦截器

```
/**
 * 自定义token拦截器
 */
@Component
public class TokenInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        if (request.getMethod().equals("OPTIONS")){
            response.setStatus(HttpServletResponse.SC_OK);
            return true;
        }
        response.setCharacterEncoding("utf-8");
        String token = request.getHeader("admin-token");
        if (token != null){
            boolean result = TokenUtil.verify(token);
            if(result){
                System.out.println("通过拦截器");
                return true;
            }
        }
        System.out.println("认证失败");
        response.getWriter().write("50000");
        return false;
    }
}
```

TokenInterceptor 实现了 HandlerInterceptor 接口，重写了 preHandle 方法，该方法是在每个请求之前触发执行，从 request 的头里面取出 token，这里我们统一了存放 token 的键为 admin-token，验证通过，放行，验证不通过，返回认证失败信息。
**这里有一个坑**，由于每次前端发送请求，都会先发一次预请求，也就是 RequestMethod 为**OPTIONS**不是我们常见的 get、post 等（关于 OPTIONS 的解释，可以谷歌一下）。所有在这里我们需要做一次判断，如果请求方法为 OPTIONS，那么就直接 return 通过。

- 配置拦截器
  对登录界面的请求不拦截

```
@Configuration
public class InterceptorConfig extends WebMvcConfigurerAdapter {
    private TokenInterceptor tokenInterceptor;

    public InterceptorConfig(TokenInterceptor tokenInterceptor) {
        this.tokenInterceptor = tokenInterceptor;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        List<String> excludePath = new ArrayList<>();
        String sysUserLogin = "/api/sysUser/login";
        excludePath.add(sysUserLogin);
        registry.addInterceptor(tokenInterceptor).excludePathPatterns(excludePath);
    }
}
```

- 服务端解析 token
  现在为了之后根据 token 去做相关的查询，我们需要对 token 进行解密，取出之前加密的 loginName。然后就可以愉快的增删查改啦～

```
/**
 * 从token中获取username信息
 * @param **token**
* @return
 */
public static String getUserName(String token){
    try {
        DecodedJWT jwt = JWT.decode(token);
        return jwt.getClaim("loginName").asString();
    } catch (JWTDecodeException e){
        e.printStackTrace();
        return null;
    }
}
```

## 前端实现

前端使用 vue+axios，主要是实现对 axios 的再封装。
相关代码如下
大致逻辑就是，如果 vuex 中已经存在了 token，那么就把它放到请求头中发往服务端。

```
// 创建axios实例
const service = axios.create({
  baseURL: process.env.BASE_API // api 的 base_url
})

// request拦截器
service.interceptors.request.use(
  config => {
    if (store.getters.token) {
      config.headers['admin-token'] = getToken() // 让每个请求携带自定义token
    }
    return config
  },
  error => {
    // 出错
    console.log(error)
    Promise.reject(error)
  }
)
```

## 结语

嗯…..现在已经 1 点多了，不由得又想起那副对联。

> 昨日今日明日日日码不停蹄，  
> 去年今年明年年年都是单身。  
> 横批：这就是命

晚安～
