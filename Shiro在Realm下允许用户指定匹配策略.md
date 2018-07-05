## Shiro在Realm下允许用户指定匹配策略

查看源代码得出:

Shiro中默认已经有以下匹配策略:

- AllSuccessfulStrategy
- AtLeastOneSuccessfulStrategy
- FirstSuccessfulStrategy

Shiro与spring整合后在多Realm下修改匹配策略如下:

```xml
<bean id="authenticator" class="org.apache.shiro.authc.pam.ModularRealmAuthenticator">
	<property name="realms">
		<list>
			<ref bean="jdbcRealm"/>
			<ref bean="secondRealm"/>
		</list>
	</property>
	
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

在`ModularRealmAuthenticator`下修改`authenticationStrategy`属性。