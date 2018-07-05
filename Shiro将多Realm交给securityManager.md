## Shiro将多Realm交给securityManager

```java
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
	 <!-- 授权时需要用到securityManager的realms
	 所以要使用shiro授权则需要将realms配置到securityManager下.
	 同样，运行时securityManager会将realms交给ModularRealmAuthenticator
	 所以ModularRealmAuthenticator同样能使用securityManager的realms集合-->
	 <property name="realms">
		<list>
			<ref bean="jdbcRealm"/>
			<ref bean="secondRealm"/>
		</list>
	</property>
</bean>
	
	
<!-- 添加多Realm认证匹配器 -->
<bean id="authenticator" class="org.apache.shiro.authc.pam.ModularRealmAuthenticator">
	<!-- 配置认证策略 -->
	<property name="authenticationStrategy">
		<!-- 所有Realm皆匹配成功才算成功 -->
		<bean class="org.apache.shiro.authc.pam.AllSuccessfulStrategy"></bean>
		<!-- 最后一个匹配成功才算成功 -->
		<!-- <bean class="org.apache.shiro.authc.pam.AtLeastOneSuccessfulStrategy"></bean> -->
		<!-- 最后一个匹配成功才算成功 -->
		<!-- <bean class="org.apache.shiro.authc.pam.AtLeastOneSuccessfulStrategy"></bean> -->
		<!-- 第一个匹配成功即算匹配成功 -->
		<!-- <bean class="org.apache.shiro.authc.pam.FirstSuccessfulStrategy"></bean> -->
	</property>
</bean>
```