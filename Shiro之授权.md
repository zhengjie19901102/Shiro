## Shiro之授权

Shiro中授权功能通过继承`AuthorizingRealm`实现类或实现`Realm`接口，实现`doGetAuthorizationInfo`抽象方法来实现授权功能。

`AuthorizingRealm`与`AuthenticatingRealm`间关系如图:

<img src="img/1.png" width="60%"/>

从上图可知，`AuthorizingRealm`是`AuthenticatingRealm`的子类，且他们都为`CachingRealm`的子类。而`CachingRealm`实现了`Realm`接口。

从图中得出结论，授权实际是扩展了认证。

授权实现示例代码:

```java
//实现`doGetAuthorizationInfo`抽象方法
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
	/**
	 * the primary principal used to uniquely identify the owning account/Subject
	 */
	Object primaryPrincipal = principals.getPrimaryPrincipal();
	//权限集合
	Set<String> roles = new HashSet<>();
	//判断当前登录用户名
	if("admin".equals(primaryPrincipal)) {
		//如果当前用户登录的是"admin",则拥有访问user的权限
		roles.add("user");
	}
	SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo(roles);
	return simpleAuthorizationInfo;
}
```

`doGetAuthorizationInfo`返回的`AuthorizationInfo`会被shiro通过调用`ModularRealmAuthorizer`或`AuthorizingRealm`的`hasRole`方法进行权限校验。

通常: 权限并非是定死的，而是从数据库中查询获取到的。所以可以通过工厂类的工厂方法，通过修改`ShiroFilterFactoryBean`的`filterChainDefinitionMap`属性来从数据库中查询出权限。

配置文件:

```xml
<!-- shiro权限过滤:id要和web.xml中的ShiroFilter的filter-name一致[默认情况下，
如果不一致，spring会在filer的init-param中的targetBeanName配置参数中获取到对应的bean名称，
如果没有在容器中配置该bean，则会抛出NoSuchBeanDefinitionException异常] -->
<bean id="shiroFilter"
	class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
	<property name="securityManager" ref="securityManager" />
	<!-- 登录页面 -->
	<property name="loginUrl" value="/login.jsp" />
	<!-- 登录成功页面 -->
	<property name="successUrl" value="/success.jsp" />
	<!-- 无权限页面 -->
	<property name="unauthorizedUrl" value="/unauthorized.jsp" />
	<!-- 
	Shiro会通过Web.xml中的ShiroFilter过滤器调用该
	id为shiroFilter的Bean,然后通过该Bean的property name为filterChainDefinitions中设置的value来
	判断页面是否需要验证授权才能访问。
	默认情况下，没有配置到value中的路径shiro不会拦截，而路径被标识为anon的则为
	可以通过匿名访问[也就是可以直接访问],标识为authc则为需要授权才能访问的页面。
	 -->
	<!-- <property name="filterChainDefinitions">
		<value>
			anon:表示可以匿名访问
			 = anon
			/user.jsp = roles[user]
			/shiroRequest/login = anon
			/shiro/logout = logout
			authc:表示需要授权才可以访问
			/** = authc
		</value>
	</property> -->
	<property name="filterChainDefinitionMap" ref="filterChainDefinition"></property>
</bean>
	
<bean id="filterChainDefinition" factory-bean="filterChainDefinitionBuilder" factory-method="FilterChainFactoryBeanBuilder"></bean>
<bean id="filterChainDefinitionBuilder" class="com.heiketu.shiro.factorybean.FilterChainDefinitionFactory"></bean>
```

示例工厂类`filterChainDefinitionBuilder `

```java
/**
 * Shiro过滤链工厂类
 * @author zhengjie
 *
 */
public class FilterChainDefinitionFactory {
	/**
	 * FilterChainFactoryBeanBuilder
	 */
	public LinkedHashMap<String, String> FilterChainFactoryBeanBuilder(){
		LinkedHashMap<String,String> perm = new LinkedHashMap<String,String>();
		perm.put("/login.jsp","anon");
		perm.put("/user.jsp","roles[user]");
		perm.put("/shiroRequest/login","anon");
		perm.put("/**", "authc");
		return perm;
	}
}
```

**`注: 查询出来的权限由LinkedHashMap类型返回，且注意先后顺序。`**