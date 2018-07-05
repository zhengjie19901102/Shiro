## Shiro多Realm认证

查看源代码，发现:

```java
assertRealmsConfigured();
Collection<Realm> realms = getRealms();
if (realms.size() == 1) {
    return doSingleRealmAuthentication(realms.iterator().next(), authenticationToken);
} else {
    return doMultiRealmAuthentication(realms, authenticationToken);
}
```

shiro支持使用多Realm，来执行认证。
shiro通过将多个Realm封装到一个集合中一起使用。
当集合中的Realm有多个时，就采用多Realm的使用方式。

此时需要准备多个Realm用来测试,多个Realm继承`Realm`接口的实现类或直接实现`Realm`接口。

此时在spring的配置文件如下:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
	
	<!-- 配置shiro的securityManager -->
	<bean id="securityManager"
		class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
		<property name="cacheManager" ref="cacheManager" />
		<!-- 
		shiro中session相关配置与单个Realm属性配置
		<property name="sessionMode" value="native" />
		<property name="realm" ref="jdbcRealm" />
		 -->
		 <!-- 使用多Realm时使用该配置配置多realm认证匹配器 -->
		 <property name="authenticator" ref="authenticator"/>
	</bean>
	
	
	<!-- 添加多Realm认证匹配器 -->
	<bean id="authenticator" class="org.apache.shiro.authc.pam.ModularRealmAuthenticator">
		<property name="realms">
			<list>
				<ref bean="jdbcRealm"/>
				<ref bean="secondRealm"/>
			</list>
		</property>
	</bean>
	
	
	<!-- shiro缓存管理器，采用第三方缓存 -->
	<bean id="cacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager">
		<!-- 
		采用bean方式，也可以采用指定类路径下的xml文件
		<property name="cacheManager" ref="ehCacheManager"/>
		 -->
		<property name="cacheManagerConfigFile" value="classpath:ehcache.xml"/>
	</bean>
	
	<!-- 第一个Realm -->
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
	
	<!-- 第二个Realm -->
	<bean id="secondRealm" class="com.heiketu.shiro.realm.SecondRealm">
		<!-- 配置自定义的凭证匹配器 -->
		<property name="credentialsMatcher">
			<!-- 
				内部构造:new SimpleHash(hashAlgorithmName, credentials, salt, hashIterations)对象
				通过创建SimpleHash对象来进行加密
			 -->
			<bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
				<!-- 采用MD5算法加密 -->
				<property name="hashAlgorithmName" value="SHA1"></property>
			</bean>
		</property>
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
</beans>
```

使用`ModularRealmAuthenticator`来配置多个Realm，然后通过`DefaultWebSecurityManager`的`authenticator `属性将指向`ModularRealmAuthenticator`。
