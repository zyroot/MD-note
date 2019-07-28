# spring-cloud-security

# 是什么：

>  Spring Cloud Security提供了一组原语，用于构建安全的应用程序和服务，而且操作简便。可以在外部（或集中）进行大量配置的声明性模型有助于实现大型协作的远程组件系统，通常具有中央身份管理服务。它也非常易于在Cloud Foundry等服务平台中使用。==在Spring Boot和Spring Security OAuth2的基础上==，可以快速创建实现常见模式的系统，如单点登录，令牌中继和令牌交换。
>
>  它是基于spring-security之上的产物，对应的它的功能更强大

**功能：**

* 安全的认证、授权

- 从Zuul代理中的前端到后端服务中继SSO令牌
- 资源服务器之间的中继令牌
- 使Feign客户端表现得像`OAuth2RestTemplate`（获取令牌等）的拦截器
- 在Zuul代理中配置下游身份验证



# cloud版本控制流程图：

![](spring-cloud-security-resouces/cloud版本控制流程图.png)

==注：== spring-cloud-starter-oauth 也是由security-dependencies管理

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>
```



# 怎么做？

一：引入依赖

```xml
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Greenwich.SR2</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>
	</dependencies>
```

引入依赖后启动项目，springsecurity会自动配置安全，任何请求都会要求登录，且默认登录为表单登录方式（spirngboot2.0开始）。

第一个类：webSecurity

```java
/**
 * <p>@Description:web安全配置</p>
 * <p>@Author: zhengyong</p>
 * <p>@Date: 2019/7/20 11:10</p>
 * <p>@Version: 1.0.0</p>
 **/
@Component
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
          		//表单登录
                .formLogin()
          		//basic登录
//                .httpBasic()
          		//认证请求+所有请求+鉴定---》认证鉴定所有请求
                .and().authorizeRequests().anyRequest().authenticated()
                ;
    }
}
```

# spring-security基本原理：

![](spring-cloud-security-resouces/security基本原理.png)

```java
UsernamePasswordAuthenticationFilter
//表单登录过滤器
```

```java
BasicAuthenticationFilter
//httpbacis登录过滤器
```

```java
FilterSecurityInterceptor
//过滤器安全拦截器---》守门员
```

>只有绿色的过滤器可以控制，其他的是无法进行控制的。

# 自定义用户认证逻辑

## 一：处理用户信息获取逻辑

封装在一个UserDetailsService接口来获取用户信息的：

```java
public interface UserDetailsService {
	// ~ Methods
	// ========================================================================================================

	/**
	 * Locates the user based on the username. In the actual implementation, the search
	 * may possibly be case sensitive, or case insensitive depending on how the
	 * implementation instance is configured. In this case, the <code>UserDetails</code>
	 * object that comes back may have a username that is of a different case than what
	 * was actually requested..
	 *
	 * @param username the username identifying the user whose data is required.
	 *
	 * @return a fully populated user record (never <code>null</code>)
	 *
	 * @throws UsernameNotFoundException if the user could not be found or the user has no
	 * GrantedAuthority
	 */
 	 //获取用户名字、返回一个用户信息（封装在了UserDetails中）--》最终存入session中
	UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

返回的UserDetails是一个接口，User（security.core.userdetails包下）的类实现了这个接口

```java
public class User implements UserDetails, CredentialsContainer {

	private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;

	private static final Log logger = LogFactory.getLog(User.class);

	// ~ Instance fields
	// ================================================================================================
	private String password;//密码
	private final String username;//用户名
	private final Set<GrantedAuthority> authorities;//授权的权限集合
	private final boolean accountNonExpired;//账号没过期
	private final boolean accountNonLocked;//账号没被锁
	private final boolean credentialsNonExpired;//认证没有过去
	private final boolean enabled;//是可用
  /**
   *四个Boolean值是默认为true的，有一个返回为false则视为校验失败
   *
   /
}  
```

eg:

```java
@Component
public class UserServiceImpl implements UserDetailsService {
	//加密类，新版本必须制定一个
    @Bean
    public PasswordEncoder passwordEncoder(){
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

	//重写的方法，返回UserDetails的实现类（这么默认使用security自带的user类，可以自定义）
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return new User(username,passwordEncoder().encode("123"), AuthorityUtils.createAuthorityList("admin"));
    }
}
```



## 二：处理用户校验逻辑

用户的校验逻辑封装在UserDetails这个接口中，可以选择自己实现这个接口，也可以使用自带的User类

```java
@Component
public class UserServiceImpl implements UserDetailsService {

    @Resource
    private ClientUserDAO clientUserDAO;

    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }


    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        QueryWrapper<ClientUser> wrapper = new QueryWrapper<>();
        wrapper.eq("username",username);
        ClientUser clientUser = clientUserDAO.selectOne(wrapper);
        if(clientUser == null){
            throw new UsernameNotFoundException("用户不存在");
        }
        return new User(username,clientUser.getPassword(),
                //设置用户的其他信息
                true,true,true,true
                ,AuthorityUtils.createAuthorityList("admin"));
    }
}
```



## 三：处理密码加密解密

```java
PasswordEncoder
//处理密码的加密和解密的接口,现有版本是必须制定一个加密的方式，不然会报错
```



# 个性化用户认证流程

## 一：自定义登录页面

1）、新建一个html登录页面

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>自定义登录页面</title>
</head>
<body>
<h2>标准登录页面</h2>
<form action="/form" method="post">
    <input type="text" name="username">
    <input type="password" name="password">
    <button type="submit">登录</button>
</form>
</body>
</html>
```

2）、配置自定义登录页面

```java
/**
 * <p>@Description:web安全配置</p>
 * <p>@Author: zhengyong</p>
 * <p>@Date: 2019/7/20 11:10</p>
 * <p>@Version: 1.0.0</p>
 **/
@Component
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
             //springsecurity配置了csrf()，csrf会拦截我所有的post请求，这涉及到csrf攻击.。暂时关闭
                .csrf().disable()
                .formLogin()
                //自定义登录页面
                .loginPage("/eim-login.html")
                //自定义表单登录请求地址（让security知道何时使用UserNamePasswordFilter）
                .loginProcessingUrl("/form")
                .and()
                .authorizeRequests()
                //放行自定义登录页
                .antMatchers("/eim-login.html").permitAll()
                .anyRequest()
                .authenticated()
                ;
    }
}
```



**根据不同的请求方式，做不同的响应：** 

![](spring-cloud-security-resouces/处理不同类型的请求.png)



* 是html请求则跳转到登录页面
* 是请求则返回401和状态码

①、修改http自定义登录页面的url，.loginPage

```java
/**
 * <p>@Description:web安全配置</p>
 * <p>@Author: zhengyong</p>
 * <p>@Date: 2019/7/20 11:10</p>
 * <p>@Version: 1.0.0</p>
 **/
@Component
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                //springsecurity配置了csrf()，csrf会拦截我所有的post请求，这涉及到csrf攻击.。暂时关闭
                .csrf().disable()
                .formLogin()
                //自定义登录页面
                .loginPage("/authentication/require")
                //自定义表单登录请求地址（让security知道何时使用UserNamePasswordFilter）
                .loginProcessingUrl("/form")
                .and()
                .authorizeRequests()
                //放行自定义登录页
                .antMatchers("/authentication/require").permitAll()
                .anyRequest()
                .authenticated()
                ;
    }
}
```

②、自定义返回内容

```java
/**
 * <p>@Description:浏览安全跳转控制器</p>
 * <p>@Author: zhengyong</p>
 * <p>@Date: 2019/7/22 18:18</p>
 * <p>@Version: 1.0.0</p>
 **/
@RestController
public class BrowserSecurityController {

    //可以拿到上一次请求中的数据
    private RequestCache requestCache = new HttpSessionRequestCache();

    //重定向的工具类
    private RedirectStrategy redirectStrategy = new DefaultRedirectStrategy();

    /**
     * 当需要身份认证时，跳转到这里
     *
     * @param request  请求
     * @param response 响应
     * @return         状态码或者登录页面
     */
    @ResponseStatus(code = HttpStatus.UNAUTHORIZED)
    @RequestMapping("/authentication/require")
    public ResponseEntity<String> requireAuthentication(HttpServletRequest request, HttpServletResponse response) throws IOException {
        //获取上一次请求
        SavedRequest savedRequest = requestCache.getRequest(request,response);
        if(savedRequest != null){
            //获取到引发跳转的URL
            String targetUrl = savedRequest.getRedirectUrl();
            if(StringUtils.endsWithIgnoreCase(targetUrl,".html")){
                //是html跳转到登录页
                redirectStrategy.sendRedirect(request,response,"可以自己配置");
            }
        }
        return ResponseEntity.ok("请引导用户到登录页面");
    }
}
```



## 二：自定义登录成功处理

## 三：自定义登录失败处理