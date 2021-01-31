# SpringSecurity：配置与前后端场景应用

[TOC]



## 1 安全简介

安全应该再什么时候考虑？设计之初

* 漏洞，隐私泄露
* 应用的基本架构已经确定，要修复安全漏洞，可能需要对系统的架构做出比较重大的调整

安全常用框架：

* Shiro
* Spring Security

> 官网描述：
>
> Spring Security is a powerful and highly customizable authentication and access-control framework. It is the de-facto standard for securing Spring-based applications.
>
> Spring Security is a framework that focuses on providing both authentication and authorization to Java applications. Like all Spring projects, the real power of Spring Security is found in how easily it can be extended to meet custom requirements

**身份验证**、**访问控制**、**权限控制**

* 功能权限
* 访问权限
* 菜单权限

以前需要用过滤器、拦截器等，冗余

## 2 认识 Spring Security

Spring Security 是针对Spring项目的安全框架，也是Spring Boot底层安全模块默认的技术选型，他可以实现强大的Web安全控制，对于安全控制，我们仅需要引入 spring-boot-starter-security 模块，进行少量的配置，即可实现强大的安全管理！

记住几个类：

- `WebSecurityConfigurerAdapter`：自定义Security策略
- `AuthenticationManagerBuilder`：自定义认证策略
- `@EnableWebSecurity`：开启WebSecurity模式

Spring Security的两个主要目标是 “认证” 和 “授权”（访问控制）。

**“认证”（Authentication）**

身份验证是关于验证您的凭据，如用户名/用户ID和密码，以验证您的身份。

身份验证通常通过用户名和密码完成，有时与身份验证因素结合使用。

 **“授权” （Authorization）**

授权发生在系统成功验证您的身份后，最终会授予您访问资源（如信息，文件，数据库，资金，位置，几乎任何内容）的完全权限。

这个概念是通用的，而不是只在Spring Security 中存在。

## 3 认证和授权

### 3.1 Demo

使用Postman来模拟请求

1. 引入maven

    ```xml
    <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    ```

2. 书写 demo controller

	随便写一个controller，然后启动

    ```java
    @RestController
    public class TestController {
        @GetMapping("/hello")
        public String hello(){
            return "hello";
        }
    }
    ```

3. 获取默认密码，体验

    项目启动后，默认用户名是user，可以通过控制台输出获取到默认密码

    ![image-20210131150940261](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210131150940261.png)

4. 未认证，返回错误

    ![image-20210131151105811](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210131151105811.png)

5. 模拟认证后，返回正确结果

    ![image-20210131151142315](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210131151142315.png)

### 3.2 配置类说明

就是认证是否为合法用户，简单的说是登录。一般为匹对用户名和密码，即认证成功。

在spring security认证中，我们需要注意的是：哪个类表示用户？哪个属性表示用户名？哪个属性表示密码？怎么通过用户名取到对应的用户？密码的验证方式是什么？

只要告诉spring security这几个东西，基本上就可以了。

使用配置类：

```java
@Configuration
@EnableWebSecurity // 开启WebSecurity模式
public class SecurityConfig extends WebSecurityConfigurerAdapter {

}
```

事实上只要继承WebSecurityConfigurerAdapter ，spring security就已经启用了，当你访问资源时，它就会跳转到它自己默认的登录页。但是这还不行，

当用户点击登录时，

1. 它会拿到用户输入的用户名密码；

2. 根据用户名通过`UserDetailsService` 的 loadUserByUsername(username)方法获得一个用户对象；

3. 获得一个`UserDetails` 对象，获得内部的成员属性password；

4. 通过`PasswordEncoder` 的 matchs(s1, s2) 方法对比用户的输入的密码和第3步的密码；

5. 匹配成功；

所以我们要实现这三个接口的三个方法：

1. 实现`UserDetailsService` ，可以选择同时实现用户的正常业务方法和UserDetailsService ；

	例如：UserServiceImpl implement IUserService，UserDetailsService {}

2. 实现`UserDetails` ，一般使用用户的实体类实现此接口。

	其中有getUsername(), getPassword(), getAuthorities()为获取用户名，密码，权限。可根据个人情况实现。

3. 实现`PasswordEncoder` ，spring security 提供了多个该接口的实现类，可百度和查看源码理解，也可以自己写。

### 3.3 配置用户

在配置类中添加以下方法

```java
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        //在内存中定义，也可以在jdbc中去拿....
        auth.inMemoryAuthentication()
                .withUser("admin").password("123456").roles("ADMIN","USER")
                .and()
                .withUser("user").password("123456").roles("USER")
                .and()
                .withUser("db").password("123456").roles("DB");
    }
    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
```

这里我们在 configure 方法中配置了两个用户，用户的密码都是加密之后的字符串(明文是 123)，从 Spring5  开始，强制要求密码要加密，如果非不想加密，可以使用一个过期的 PasswordEncoder 的实例  NoOpPasswordEncoder，但是不建议这么做，毕竟不安全。

Spring Security 中提供了 BCryptPasswordEncoder 密码编码工具，可以非常方便的实现密码的加密加盐，相同明文加密出来的结果总是不同，这样就不需要用户去额外保存`盐`的字段了，这一点比 Shiro 要方便很多。

### 3.4 配置权限

在配置类中加入以下方法

```java
	 @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable(); // 这个要先写！
        // 定制请求的授权规则
        // 首页所有人可以访问
        http.authorizeRequests().antMatchers("/").permitAll()
                .antMatchers("/hello").hasRole("USER")
                .antMatchers("/db/**").hasRole("DB")
                .antMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest()
                .authenticated(); //其它请求需要认证
        //以下登录配置开始
		  
        // 如果没有登录，会跳转到登录页面
        http.formLogin()
                .loginProcessingUrl("/login")//登录处理地址
                .usernameParameter("u_id")//定义登录时，用户名的参数名，默认为 username
                .passwordParameter("u_pwd")//定义登录时，密码的参数名，默认为 password
                .successHandler((req, resp, authentication) -> {
                    resp.setContentType("application/json;charset=utf-8");
                    PrintWriter out = resp.getWriter();
                    out.write(JSON.toJSONString(Result.success("登录成功")));
                    out.flush();
                })//登录成功的处理器
                .failureHandler((req, resp, exception) -> {
                    resp.setContentType("application/json;charset=utf-8");
                    PrintWriter out = resp.getWriter();
                    out.write(JSON.toJSONString(Result.fail("登录失败！")));
                    out.flush();
                })//登录失败的处理器
                .permitAll();//登录相关访问地址一律放行
        //以上登录配置
        http
                //以下退出配置
                .logout()
                .logoutUrl("/logout")//退出地址
                .logoutSuccessHandler((req, resp, authentication) -> {
                    resp.setContentType("application/json;charset=utf-8");
                    PrintWriter out = resp.getWriter();
                    out.write(JSON.toJSONString(Result.success("您已成功退出系统！")));
                    out.flush();
                })//退出成功的处理器
                .permitAll(); //退出地址一律放行
        //认证授权配置-结束
    }
```

### 3.5 体验

1. 未登录，访问时候会跳到登录？

	![image-20210131164247760](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210131164247760.png)

2. 登录user

	![image-20210131164306160](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210131164306160.png)

3. 此时请求USER权限的，正常；

	![image-20210131164338379](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210131164338379.png)

	USER无权限的，返回403

	![image-20210131164417773](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210131164417773.png)

4. 退出后登录DB

	![image-20210131164452990](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210131164452990.png)

	![image-20210131164506304](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210131164506304.png)

	再次请求/db/hello，正常

	![image-20210131164515831](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210131164515831.png)

## 4 连接数据库

在前文中，我们提到：

事实上只要继承WebSecurityConfigurerAdapter ，spring security就已经启用了，当你访问资源时，它就会跳转到它自己默认的登录页。但是这还不行，

当用户点击登录时，

1. 它会拿到用户输入的用户名密码；

2. 根据用户名通过`UserDetailsService` 的 loadUserByUsername(username)方法获得一个用户对象；

3. 获得一个`UserDetails` 对象，获得内部的成员属性password；

4. 通过`PasswordEncoder` 的 matchs(s1, s2) 方法对比用户的输入的密码和第3步的密码；

5. 匹配成功；

所以我们要实现这三个接口的三个方法：

1. 实现`UserDetailsService` ，可以选择同时实现用户的正常业务方法和UserDetailsService ；

	例如：UserServiceImpl implement IUserService，UserDetailsService {}

2. 实现`UserDetails` ，一般使用用户的实体类实现此接口。

	其中有getUsername(), getPassword(), getAuthorities()为获取用户名，密码，权限。可根据个人情况实现。

3. 实现`PasswordEncoder` ，spring security 提供了多个该接口的实现类，可百度和查看源码理解，也可以自己写。

### 4.1 实现UserDetails

```java
public class User implements UserDetails {
    private Long id;
    private String username;
    private String password;
    private String nickname;
    private boolean enabled;
    private List<String> roles;
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        List<GrantedAuthority> authorities = new ArrayList<>();
        for (String role : roles) {
            authorities.add(new SimpleGrantedAuthority("ROLE_" + role));
        }
        return authorities;
    }

    @Override
    public String getPassword() {
        return password;
    }

    @Override
    public String getUsername() {
        return username;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }

   // getter and setter
}
```

* 四个返回Boolean的方法都是见名知意，为了简单期间都直接返回true

* getAuthorities方法返回当前用户的角色信息，用户的角色其实就是roles中的数据，将roles中的数据转换为List之后返回即可，**这里有一个要注意的地方，由于我在数据库中存储的角色名都是诸如‘超级管理员’、‘普通用户’之类的，并不是以`ROLE_`这样的字符开始的，因此需要在这里手动加上`ROLE_`,切记**

### 4.2 实现UserDetailsService

```java
@Service
public class UserService implements UserDetailsService {
    // 这里应该有个@Autowired xxxMapper 简单起见直接省略
    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        User user = new User();
        switch (s) {
            case "user":
                user.setUsername("user");
                user.setPassword(new BCryptPasswordEncoder().encode("123456"));
                user.setRoles(Arrays.asList("USER"));
                break;
            case "admin":
                user.setUsername("user");
                user.setPassword(new BCryptPasswordEncoder().encode("123456"));
                user.setRoles(Arrays.asList("USER","ADMIN"));
                break;
            case "db":
                user.setUsername("user");
                user.setPassword(new BCryptPasswordEncoder().encode("123456"));
                user.setRoles(Arrays.asList("USER","DB"));
                break;
            default:
                // nothing
        }
        return user;
    }
}
```

实现了UserDetailsService接口之后，我们需要实现该接口中的loadUserByUsername方法，即根据用户名查询用户。

> 这里应该注入了MyBatis中的Mapper，用于查询User，这个查询结果将在user对象的getAuthorities方法中用上。
>
> 因为只是演示，直接硬编码模拟查数据库了

### 4.3 修改Config

```java
@Configuration
@EnableWebSecurity // 开启WebSecurity模式
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    UserService userService;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userService);
    }
    
    @Bean
    PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
	// ...其他配置不变
}
```

重新启动项目即可

![image-20210131180324467](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210131180324467.png)

## 5 前后端分离

前面介绍了，如果未登录，会跳转到登陆页面，这样一来其实前后端的耦合是比较强的，我们在实际的前后端分离开发中不太可能这样实现

同样，权限不足的返回也可以定制，可以参考这个博客：https://blog.csdn.net/u012702547/article/details/79019510，这里不再赘述

### 5.1 问题

* 需求是：前后端分离，需要自己的登录页面，使用ajax请求。

* 出现问题：自己的登录页面请求登录后，后端返回302跳转主页，ajax无法处理；未认证请求资源时，后端返回302跳转登录页，也无法处理。

* 解决思想：修改302状态码，修改为401，403或者200和json数据。

### 5.2 解决思路

这里就需要实现一个特殊的方法：AuthenticationEntryPoint 接口的 commence()方法。

这个方法主要是，用户未认证访问资源时，所做的处理。

spring security给我们提供了很多现成的AuthenticationEntryPoint 实现类，

比如默认的302跳转登录页，比如返回403状态码，还比如返回json数据等等。当然也可以自己写。和上面的登录处理一样，实现接口方法，将实现类实例传到配置方法（推荐spring注入）。

我们只需要加上`http.exceptionHandling().authenticationEntryPoint(new Http403ForbiddenEntryPoint());`即可

当然，其中的`AuthenticationEntryPoint`也可以自己定义一个状态码，这里随便拿个588举例

```java
public class NoLoginEntryPoint implements AuthenticationEntryPoint {

    private static final Log logger = LogFactory.getLog(Http403ForbiddenEntryPoint.class);
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException arg2) throws IOException {
        logger.debug("Pre-authenticated entry point called. Rejecting access");
        response.sendError(588, "No Login");
    }
}
```

修改后的代码：

```java
	 @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();
        // 定制请求的授权规则
        // 首页所有人可以访问
        http.exceptionHandling().authenticationEntryPoint(new NoLoginEntryPoint());
        http.authorizeRequests().antMatchers("/").permitAll()
                .antMatchers("/hello").hasRole("USER")
                .antMatchers("/db/**").hasRole("DB")
                .antMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest()
                .authenticated(); //其它请求需要认证
        //以下登录配置开始

        // 如果没有登录，会跳转到登录页面
        http.formLogin()
                .loginProcessingUrl("/login")//登录处理地址
                .usernameParameter("u_id")//定义登录时，用户名的参数名，默认为 username
                .passwordParameter("u_pwd")//定义登录时，密码的参数名，默认为 password
                .successHandler((req, resp, authentication) -> {
                    resp.setContentType("application/json;charset=utf-8");
                    PrintWriter out = resp.getWriter();
                    out.write(JSON.toJSONString(Result.success("登录成功")));
                    out.flush();
                })//登录成功的处理器
                .failureHandler((req, resp, exception) -> {
                    resp.setContentType("application/json;charset=utf-8");
                    PrintWriter out = resp.getWriter();
                    out.write(JSON.toJSONString(Result.fail("登录失败！")));
                    out.flush();
                })//登录失败的处理器
                .permitAll();//登录相关访问地址一律放行
        //以上登录配置
        http
                //以下退出配置
                .logout()
                .logoutUrl("/logout")//退出地址
                .logoutSuccessHandler((req, resp, authentication) -> {
                    resp.setContentType("application/json;charset=utf-8");
                    PrintWriter out = resp.getWriter();
                    out.write(JSON.toJSONString(Result.success("您已成功退出系统！")));
                    out.flush();
                })//退出成功的处理器
                .permitAll(); //退出地址一律放行
        //认证授权配置-结束
    }
```

发个请求试一下，成功，这样一来前端就可以根据定义的规则来处理了

![image-20210131174202540](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210131174202540.png)

## 参考

https://blog.csdn.net/serdonty/article/details/105981798

https://blog.csdn.net/u012702547/article/details/79019510

https://blog.csdn.net/u012702547/article/details/78928307