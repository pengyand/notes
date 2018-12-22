# 概念

应用程序的两个主要区域是“认证”和“授权”（或者访问控制）。这两个主要区域是Spring Security的两个目标

- “认证”（Authentication），是建立一个他声明的主体的过程（“主体”一般是指用户、设备或一些可以在你的应用程序中执行动作的其他系统）
- “授权“（Authorization）指确定一个主体是否允许在你的应用程序执行一个动作的过程。



**步骤**

1、引入maven

~~~xml
<dependencies>
    <!-- ... other dependency elements ... -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
</dependencies>

<properties>
    <!-- ... -->
    <spring-security.version>5.1.2.RELEASE</spring-security.version>
</properties>
~~~

 参考向导：https://spring.io/guides/gs/securing-web/

2、编写SpringSecurity的配置类 继承WebSecurityConfigurerAdapter

~~~java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            //认证请求
            .authorizeRequests()
            	//这些路径所有人都可以访问
                .antMatchers("/", "/home").permitAll()
            	//其他路径都需要认证
            	.anyRequest().authenticated()
                .and()
            //开启自动配置的登录功能：如果没有登录，就会来到登录页面
            .formLogin()
            // /login来到登录页面
            // 重定向到/login?error表示登录失败
            // /login  get请求是跳转到登录页面的
            // /login post请求是进行处理登录的
         	//如果 loginPage修改了，比如login修改成userlogin ，那么 userlogin的post请求才是登录处理uri，原来的失效，也可以通过.loginProcessingUrl("/userlogin")进行指定
                .loginPage("/login")
            //可以指定登录的用户和密码参数 默认是username和pasword，可以修改成下面的
            .usernameParameter("user").passwordParameter("pwd")
          
            
                .permitAll()
                .and()
            //开启自动配置的注销功能，访问/logout表示用户注销，清空session 
            //注销成功，会返回到login?logout页面(默认，可以修改)
            .logout()
            //指定注销成功之后跳转的页面
            .logoutSuccessUrl("/home");
                .permitAll();

        //开启记住我功能 登录成功以后，将cookie发给浏览器保存，以后登录带上这个cookie，只要通过检查，就可以免登录。如果点击注销，会删除cookie 
        http.rememberMe();
        //可以指定记住的参数
        http.rememberMeParamter("myrememberme");
    }

    
    //认证规则1
    @Bean
    @Override
    public UserDetailsService userDetailsService() {
        UserDetails user =
             User.withDefaultPasswordEncoder()
                .username("user")
                .password("password")
                .roles("USER")
                .build();

        return new InMemoryUserDetailsManager(user);
    }
    
    //认证规则2 和上面一样
    protected void configure(AuthenticationManagerBuilder auth)throws Exception{
        auth.inMemoryAuthentication()
            .withUser("zhangsan").password("123456").roles("role1","role2")
            .and()
            .withUser("lisi").password("123456").roles("role3");
    }
}
~~~

**记住我**

