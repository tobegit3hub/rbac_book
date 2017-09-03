# 理解Principal

## Principal介绍

前面我们看quickstart项目源码时，用到了Subject，其实也用到了Credential，这也是我们需要理解的基础概念。

```java
if (!currentUser.isAuthenticated()) {
    UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");
    token.setRememberMe(true);
    try {
        currentUser.login(token);
    } catch (UnknownAccountException uae) {
        log.info("There is no user with username of " + token.getPrincipal());
    } catch (IncorrectCredentialsException ice) {
        log.info("Password for account " + token.getPrincipal() + " was incorrect!");
    } catch (LockedAccountException lae) {
        log.info("The account for username " + token.getPrincipal() + " is locked.  " +
                "Please contact your administrator to unlock it.");
    }
    // ... catch more exceptions here (maybe custom ones specific to your application?
    catch (AuthenticationException ae) {
        //unexpected condition?  error?
    }
}
```

如果登录失败，那么我们会打印下`token`对象的Principal，如果修改代码在登录前后打印Subject和Token分别的Principal。

```java
System.out.println("Subject principal: " + currentUser.getPrincipal());
System.out.println("Token principal " + token.getCredentials());
```

发现在调用`login()`函数前，Subject的Principal是`null`，而登录后Subject和Token的Principal都是lonestarr，也就是说在默认的Realm下Principal就是username字符串。

## Principal源码

那么Principal究竟是什么呢，其实在Subject的源码中定义了Principal，就是用户的唯一标识，可以返回任意的Java对象，在前面的场景下返回的就是字符串类型的username。

```java
/**
 * Returns this Subject's application-wide uniquely identifying principal, or {@code null} if this
 * Subject is anonymous because it doesn't yet have any associated account data (for example,
 * if they haven't logged in).
 * <p/>
 * The term <em>principal</em> is just a fancy security term for any identifying attribute(s) of an application
 * user, such as a username, or user id, or public key, or anything else you might use in your application to
 * identify a user.
 * <h4>Uniqueness</h4>
 * Although given names and family names (first/last) are technically considered principals as well,
 * Shiro expects the object returned from this method to be an identifying attribute unique across
 * your entire application.
 * <p/>
 * This implies that things like given names and family names are usually poor
 * candidates as return values since they are rarely guaranteed to be unique;  Things often used for this value:
 * <ul>
 * <li>A {@code long} RDBMS surrogate primary key</li>
 * <li>An application-unique username</li>
 * <li>A {@link java.util.UUID UUID}</li>
 * <li>An LDAP Unique ID</li>
 * </ul>
 * or any other similar suitable unique mechanism valuable to your application.
 * <p/>
 * Most implementations will simply return
 * <code>{@link #getPrincipals()}.{@link org.apache.shiro.subject.PrincipalCollection#getPrimaryPrincipal() getPrimaryPrincipal()}</code>
 *
 * @return this Subject's application-specific unique identity.
 * @see org.apache.shiro.subject.PrincipalCollection#getPrimaryPrincipal()
 */
Object getPrincipal();
```

而前面我们用的是`UsernamePasswordToken`，查看源码看到这个Token确实返回的是username字符串，然后`login()`时会回写到Subject对象中。

```java
public Object getPrincipal() {
    return getUsername();
}

public String getUsername() {
    return username;
}
```

这里我们需要理解，Principal就是一个用户的标识，可以是username或者是自定义的User对象，Subject和Token都会包含这个信息，但不一定是同一个或者同种类型的对象。
