---
title: spring security 学习
---

> 1.  拦截器链
> 2.  主要类/组件
> 3.  核心服务
> 4.  工作原理
> 5.  总结
>
> [中文机翻文档](https://www.springcloud.cc/spring-security-zhcn.html#overall-architecture)

## 一. 拦截器链

- WebAsyncManagerIntegrationFilter 
-  SecurityContextPersistenceFilter 
-  HeaderWriterFilter 
- CorsFilter 
-   LogoutFilter
-   RequestCacheAwareFilter
-   SecurityContextHolderAwareRequestFilter
-   AnonymousAuthenticationFilter
-   SessionManagementFilter
-   ExceptionTranslationFilter
-   FilterSecurityInterceptor
-   UsernamePasswordAuthenticationFilter
-   BasicAuthenticationFilter

> **至关重要的类**
>
> `OncePerRequestFilter`和`GenericFilterBean`
>
> `OncePerRequestFilter`顾名思义，能够确保在一次请求只通过一次filter，而不需要重复执行。
> `GenericFilterBean`是javax.servlet.Filter接口的一个基本的实现类
> `GenericFilterBean`将web.xml中filter标签中的配置参数-init-param项作为bean的属性
> `GenericFilterBean`可以简单地成为任何类型的filter的父类
> `GenericFilterBean`的子类可以自定义一些自己需要的属性
> `GenericFilterBean`，将实际的过滤工作留给他的子类来完成，这就导致了他的子类不得不实现doFilter方法
> `GenericFilterBean`不依赖于Spring的ApplicationContext，而是通过访问spring容器（Spring root application context）中的service beans来获取，通常是通过调用filter里面的getServletContext() 方法来获取
>
>
> 原文链接：https://blog.csdn.net/dushiwodecuo/article/details/78913113

---

#### WebAsyncManagerIntegrationFilter

- 根据请求封装获取WebAsyncManager
- 从WebAsyncManager获取/注册SecurityContextCallableProcessingInterceptor

### SecurityContextPersistenceFilter

- SecurityContextHolder->HttpSessionSecurityContextRepository（下面以repo代替）
- 作用：其会从`Session`中取出已认证用户的信息,提高效率,避免每一次请求都要查询用户认证信息。



### HeaderWriterFilter

- 往该请求的Header中添加相应的信息,在http标签内部使用security:headers来控制



### CsrfFilter

- csrf又称跨域请求伪造，攻击方通过伪造用户请求访问受信任站点。
- 对需要验证的请求验证是否包含csrf的token信息，如果不包含，则报错。这样攻击网站无法获取到token信息，则跨域提交的信息都无法通过过滤器的校验。



### LogoutFilter

- 匹配URL,默认为/logout
- 匹配成功后则用户退出,清除认证信息



### RequestCacheAwareFilter

- 通过HttpSessionRequestCache内部维护了一个RequestCache，用于缓存HttpServletRequest



#### SecurityContextHolderAwareRequestFilter

- 针对ServletRequest进行了一次包装，使得request具有更加丰富的API



#### AnonymousAuthenticationFilter

- 当SecurityContextHolder中认证信息为空,则会创建一个匿名用户存入到SecurityContextHolder中。匿名身份过滤器，这个过滤器个人认为很重要，需要将它与UsernamePasswordAuthenticationFilter 放在一起比较理解，spring security为了兼容未登录的访问，也走了一套认证流程，只不过是一个匿名的身份
- pirng Security为了整体逻辑的统一性，即使是未通过认证的用户，也给予了一个匿名身份。而AnonymousAuthenticationFilter该过滤器的位置也是非常的科学的，它位于常用的身份认证过滤器（如UsernamePasswordAuthenticationFilter、BasicAuthenticationFilter、RememberMeAuthenticationFilter）之后，意味着只有在上述身份过滤器执行完毕后，SecurityContext依旧没有用户信息，AnonymousAuthenticationFilter该过滤器才会有意义—-基于用户一个匿名身份



#### SessionManagementFilter

- securityContextRepository限制同一用户开启多个会话的数量
- SessionAuthenticationStrategy防止session-fixation protection attack（保护非匿名用户）



---



## 二. 主要类/组件

### 核心组件

todo

简单回顾一下，Spring Security主要由以下几部分组成的:

- `SecurityContextHolder`, 提供几种访问 `SecurityContext`的方式。
- `SecurityContext`, 保存`Authentication`信息和请求对应的安全信息。
- `Authentication`, 展示Spring Security特定的主体。
- `GrantedAuthority`, 在应用程序范围,你赋予主体的权限。
- `UserDetails`,通过你的应用DAO，提供必要的信息，构建Authentication对象。
- `UserDetailsService`, 创建一个`UserDetails`，传递一个 `String`类型的用户名(或者证书ID或其他).



> 以下举例

### 登陆认证

​	常用认证流程

1. 提示用户输入用户名和密码进行登录。
2. 该系统 (成功) 验证该用户名的密码正确。
3. 获取该用户的环境信息 (他们的角色列表等).
4. 为用户建立安全的环境。
5. 用户进行，可能执行一些操作，这是潜在的保护的访问控制机制，检查所需权限，对当前的安全的环境信息的操作。

---

1. 由用户名和密码进行组合生成–>`UsernamePasswordAuthenticationToken` :` authentication`的实例
2. ==token==  传到`AuthenticationManager`的实例进行验证
3. 该`AuthenticationManager`填充`Authentication`实例返回成功验证
4. 身份信息是通过调用 `SecurityContextHolder.getContext().setAuthentication(…)`, 传递到返回的验证对象建立的。

---

### Web访问认证

1. 你访问首页, 点击一个链接。
2. 向服务器发送一个请求，服务器判断你是否在访问一个受保护的资源。
3. 如果你还没有进行过认证，服务器发回一个响应，提示你必须进行认证。响应可能是HTTP响应代码，或者是重新定向到一个特定的web页面。
4. 依据验证机制，你的浏览器将重定向到特定的web页面。填写安全信息.
5. 浏览器会发回一个响应给服务器。 这将是HTTP POST包含你填写的表单内容，或者是HTTP头部，包含你的验证信息。
6. 下一步，服务器会判断当前的证书是否是有效的。 如果他们是非法的，通常你的浏览器会再尝试一次（所以你返回的步骤二）。
7. 登陆 || 403

---

#### ExceptionTranslationFilter

一个Spring Security过滤器，用来检测是否抛出了Spring Security异常。这些异常会被`AbstractSecurityInterceptor`抛出，它主要用来提供验证服务

####  AuthenticationEntryPoint

`AuthenticationEntryPoint`对应上面列表中的步骤三的执行逻辑

#### 验证机制

在你的浏览器决定提交你的认证证书之后(使用HTTP表单发送或者是HTTP头),服务器部分需要有一些东西来"收集"这些验证信息。现在我们到了上述的第六步。 

`验证机制`从用户代码（通常是浏览器）收集验证信息的功能。实。一旦认证细节已从用户代理收集,建立一个`Authentication` "request"对象，然后提交给`AuthenticationManager`。

验证机制重新获得了组装好的`Authentication`对象时，它会认为请求有效，把`Authentication`放到`SecurityContextHolder`里的，然后导致原始请求重审(第七步)。另一方面,如果`AuthenticationManager`驳回了请求,验证机制会让用户代码重试(第二步)



### Spring Security的访问控制(授权)

负责Spring Security访问控制决策的主要接口是`AccessDecisionManager`。它有一个`decide`方法，一个"和安全元数据属性的列表对象(如一个列表哪些角色需要被访问授权)。

> #### tips:
>
> `SpringAOP` : 保护服务层的方法调用
>
> `AspectJ` 直接保护领域对象

#### 安全对象和AbstractSecurityInterceptor

###  

### Localization

消息本地化





## 三. 核心服务

## 四.  原理/机制