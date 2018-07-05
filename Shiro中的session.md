## Shiro中的session

Shiro中也提供了与HTTPSession功能相似的Session接口，它可以以非侵入式的方式在后端任何层上进行调用。且该session同样能获取HTTPSession中设置的Attribute。

```java
Session session = SecurityUtils.getSubject().getSession();
```

```java

Subject.getSession()：即可获取会话；其等价于Subject.getSession(true)，即如果当前没有创建Session 对象会创建一个；Subject.getSession(false)，如果当前没有创建Session 则返回null•session.getId()：获取当前会话的唯一标识•session.getHost()：获取当前Subject的主机地址•session.getTimeout() & session.setTimeout(毫秒)：获取/设置当前Session的过期时间•session.getStartTimestamp() & session.getLastAccessTime()：获取会话的启动时间及最后访问时间；如果是JavaSE应用需要自己定期调用session.touch() 去更新最后访问时间；如果是Web 应用，每次进入ShiroFilter都会自动调用session.touch() 来更新最后访问时间。


session.touch() & session.stop()：更新会话最后访问时间及销毁会话；当Subject.logout()时会自动调用stop 方法来销毁会话。如果在web中，调用HttpSession. invalidate() 也会自动调用ShiroSession.stop方法进行销毁Shiro的会话•session.setAttribute(key, val) & session.getAttribute(key) & session.removeAttribute(key)：设置/获取/删除会话属性；在整个会话范围内都可以对这些属性进行操作
```
