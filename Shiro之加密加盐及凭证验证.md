## Shiro之加密加盐及凭证验证

MD5加密在Shiro中使用极其简单:

shiro中工具类:**`SimpleHash`**

```java
//SimpleHash构造器
SimpleHash(String algorithmName, Object source, Object salt, int hashIterations)
```

参数解释:

参数名  |  参数解释 | 参数数据类型
-------|----------|--------
algorithmName | 加密类型[MD5、Md2、Sha1、Sha256等] | String
source | 要加密的对象 | Object
salt | 加盐对象，如果不打算加密时进行加盐则传null | Object
hashIterations | 对目标对象加密次数,次数越多可靠性越高。同时越复杂 | int

使用示例:

```java
//SimpleHash加密
SimpleHash simpleHash2 = new SimpleHash("MD5", "123456", salt, 0);
//输出加密后结果[直接输出对象，或调用toString方法后就是加密结果]
System.out.println(simpleHash2);
```
Shrio中可以通过修改实现了`Realm`接口的自定义`Realm`中的`credentialsMatcher`属性所对应的证书匹配器来使用加密设置。常用的是其实现类`HashedCredentialsMatcher`对象,在整合了Spring之后Spring的配置中如下:

```xml
<!--  配置自定义Realm,实现Realm接口或继承其实现类[例如:AuthenticatingRealm]  -->
<bean id="jdbcRealm" class="com.heiketu.shiro.realm.ShiroRealm">
	<!-- 配置自定义的凭证匹配器 -->
	<property name="credentialsMatcher">
		<!-- 
			内部构造:new SimpleHash(hashAlgorithmName, credentials, salt, hashIterations)对象
			通过创建SimpleHash对象来进行加密
		 -->
		<bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
			<!-- 采用MD5算法加密 -->
			<property name="hashAlgorithmName" value="MD5"></property>
		</bean>
	</property>
</bean>


<!-- 配置shiro的安全管理器 -->
<bean id="securityManager"
	class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
	<property name="cacheManager" ref="cacheManager" />
	<!-- 
	shiro中session相关配置
	<property name="sessionMode" value="native" />
	 -->
	<property name="realm" ref="jdbcRealm" />
</bean>


<!-- shiro生命周期Post处理器 -->
<bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor" />

<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
	depends-on="lifecycleBeanPostProcessor" />
<bean
	class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
	<property name="securityManager" ref="securityManager" />
</bean>

<bean id="secureRemoteInvocationExecutor"
	class="org.apache.shiro.spring.remoting.SecureRemoteInvocationExecutor">
	<property name="securityManager" ref="securityManager" />
</bean>

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
	<property name="filterChainDefinitions">
		<value>
			<!-- anon:表示可以匿名访问 -->
			/login.jsp = anon
			/shiroRequest/login = anon
			/shiro/logout = logout
			<!-- authc:表示需要授权才可以访问 -->
			/** = authc
		</value>
	</property>
</bean>
```
注:如果自定义`Realm`是继承自`AuthenticatingRealm`实现类，则需要实现:`AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken arg0)`抽象方法。

`Shiro中doGetAuthenticationInfo方法的示例代码如下:`

```java
@Override
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken arg0) throws AuthenticationException {
	//强转UsernamePasswordToken
	UsernamePasswordToken upToken = (UsernamePasswordToken)arg0;
	
	//用户名
	String username = upToken.getUsername();
	if("unkown".equals(username)) {
		throw new UnknownAccountException("未知用户");
	}

	//principal
	Object principal = username;
	//密码:加盐
	ByteSource bytes = ByteSource.Util.bytes("admin");
	String pass = new SimpleHash("MD5", "123456", bytes, 0).toString();
	String name2 = getName();
	//不加盐的设置
	//AuthenticationInfo info = new SimpleAuthenticationInfo(principal, pass, name2);
	AuthenticationInfo info = new SimpleAuthenticationInfo(username, pass, bytes, getName());
	return info;
}
```

