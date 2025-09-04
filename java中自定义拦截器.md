# java中实现自定义拦截器
在 Java 中实现自定义拦截器，最常见的场景是在 Spring MVC 框架中使用。Spring MVC 的拦截器（Interceptor）可以对请求进行预处理和后处理，常用于日志记录、权限验证、性能监控等场景。

### 核心概念
- **HandlerInterceptor 接口**：:这是Spring MVC 定义的用于创建拦截器的接口，它包含三个核心方法：
    - `preHandle()`：在Controller 方法执行之前执行，返回 `true` 表示继续执行，返回 `false` 则中断请求。
    - `postHandle()`：在Controller 方法执行之后、视图渲染之前执行。
    - `afterCompletion()`：在视图渲染完成后执行。
- **WebMvcConfigurer 接口**：:用于配置Spring MVC 的各种设置，包括**注册**自定义拦截器。

### 实现步骤

**1. 创建拦截器类**
创建一个类，并实现`HandlerInterceptor`接口。在这个类中重写`preHandle()`等方法，实现拦截器业务逻辑。
```java
public class ValidateLoginInterceptor implements HandlerInterceptor {  
    @Override  
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {  
	    // 业务代码
	}
@Override  
public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception { 
		// 在 Controller 方法执行后，视图渲染前执行的逻辑 
		System.out.println("postHandle: Controller 方法执行后"); 
	} 
@Override 
public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {  
		// 在视图渲染完成后执行的逻辑 
		System.out.println("afterCompletion: 视图渲染后");
	}
}
```

**2. 创建配置类，注册拦截器**
- 创建一个配置类，实现`WebMvcConfigurer`接口，并在配置类上添加`@Configuration`注解，标记为配置类。
- 重写 `addInterceptors` 方法，实例化自定义拦截器，并将其添加到拦截器链中。
```java
/**  
 * @Author Ryan  
 * @Date 2025/9/4 17:12  
 */@Configuration  
public class InterceptorConfigurer implements WebMvcConfigurer {  
    /**  
     * 注册权限控制拦截器  
     */  
    @Override  
    public void addInterceptors(InterceptorRegistry registry) {  
        /**  
         * 注册登录核验拦截器  
         */  
         // addPathPatterns("/api/**") 拦截/api路径下的所有请求
         registry.addInterceptor(validateLoginInterceptor()).addPathPatterns("/api/**");  
    }  
    /***  
     * 登录核验拦截器  
     * @return  
     */  
    @Bean  // 注册为Bean，让Spring进行管理
    public ValidateLoginInterceptor validateLoginInterceptor() {  
        return new ValidateLoginInterceptor();  
    }  
}
```

### 如果通过注解的方式定义拦截器
**1. 自定义注解**
```java
@Retention(RetentionPolicy.RUNTIME)  
@Target(ElementType.METHOD)  
@Documented  
public @interface ValidateLogin {  
  
}
```
- `@Retention`：指定被修饰的注解的**保留策略**（即注解在哪个阶段有效）。
	- `SOURCE`：注解仅在**源代码阶段**保留，编译成字节码（.class）后会被丢弃（如 `@Override` 注解）。
	- `CLASS`：注解保留到**字节码阶段**，但运行时（JVM 加载类后）会被丢弃（默认策略）。
	- `RUNTIME`：注解一直保留到**运行时**，可以通过反射机制在程序运行时获取注解信息（最常用，如 Spring 的 `@Autowired`）。
- `@Target`：指定被修饰的注解可以**应用在哪些元素上**（如类、方法、字段）。
	- `ElementType` 是枚举类，常用值包括：
	- `TYPE`：可用于类、接口、枚举
	- `METHOD`：可用于方法
	- `FIELD`：可用于字段（成员变量）
	- `PARAMETER`：可用于方法参数
	- `CONSTRUCTOR`：可用于构造方法
	- 其他（如 `ANNOTATION_TYPE`、`LOCAL_VARIABLE` 等）。
- `@Document`：标记被修饰的注解会被**Javadoc 工具提取到 API 文档中**。
**2. 配置自定义注解的相关拦截器**
```java
public class ValidateLoginInterceptor implements HandlerInterceptor {  
    @Override  
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {  
        HandlerMethod handlerMethod = (HandlerMethod) handler;  
        // 获取拦截到的方法  
        Method method = handlerMethod.getMethod();  
        // 获取方法上的ValidateLogin注解  
        ValidateLogin validateLogin = method.getAnnotation(ValidateLogin.class);  
        if (validateLogin == null) {  
            // 未获取到登录鉴权注解，则表示不需要拦截，直接放行  
            return true;  
        }  
        String userId = SessionHelper.getCurrentUserId();  
        if (StringUtils.isBlank(userId)) {  
            // 未登录，鉴权失败  
            throw new AuthException("未登录！");  
        }  
        return true;  
    }  
}
```

**3. 创建配置类，注册拦截器（如上面所写）**


### 使用场景
- 日志记录：记录请求 URL、参数、处理时间等
- 权限验证：检查用户是否登录或拥有访问权限
- 性能监控：记录请求处理时间
- 字符编码处理：统一设置请求响应编码
- 异常处理：捕获请求过程中的异常
- 等等
### 注意事项
- 拦截器是 Spring MVC 的组件，依赖于 Spring 容器
- 拦截器可以访问 Spring 容器中的 Bean，而过滤器（Filter）不能
- 多个拦截器可以通过`order()`方法指定执行顺序（值越小越先执行）