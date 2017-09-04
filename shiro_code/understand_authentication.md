# 理解认证过程
## 介绍


## 登录流程

用户通过`SecurityUtils.getSubject()`获取Subject后，调用`login()`函数。这是会做几件事情，首先是调用`securityManager.login()`，通过Subject获得Principal为username，修改本地的`authenticated`变量，修改host和session。

而`securityManager.login()`会做更具体的操作，默认使用的DefaultSecurityManager，使用了单个IniRealm，先检查Realm的`supports()`函数能否支持UsernamePasswordToken，在doGetAuthenticationInfo方法中返回了Account也就是Info对象。对象认证相关的Info的Principal就是username，Credential就是password，授权相关的Info包含两条oject permission和2条role。

通过Token获取Info的逻辑如下。

```java
public class SimpleAccountRealm extends AuthorizingRealm {
    protected final Map<String, SimpleAccount> users; //username-to-SimpleAccount

    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        UsernamePasswordToken upToken = (UsernamePasswordToken) token;
        SimpleAccount account = getUser(upToken.getUsername());

        if (account != null) {

            if (account.isLocked()) {
                throw new LockedAccountException("Account [" + account + "] is locked.");
            }
            if (account.isCredentialsExpired()) {
                String msg = "The credentials for account [" + account + "] are expired";
                throw new ExpiredCredentialsException(msg);
            }

        }

        return account;
    }

    protected SimpleAccount getUser(String username) {
        USERS_LOCK.readLock().lock();
        try {
            return this.users.get(username);
        } finally {
            USERS_LOCK.readLock().unlock();
        }
    }
}
```



认证过程就是通过Token拿Info，如果系统根据Token的Principal也无法拿到Info而抛出异常，那么认证就失败了。




```

public abstract class AuthenticatingRealm extends CachingRealm implements Initializable {

    public final AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {

        AuthenticationInfo info = getCachedAuthenticationInfo(token);
        if (info == null) {
            //otherwise not cached, perform the lookup:
            info = doGetAuthenticationInfo(token);
            log.debug("Looked up AuthenticationInfo [{}] from doGetAuthenticationInfo", info);
            if (token != null && info != null) {
                cacheAuthenticationInfoIfPossible(token, info);
            }
        } else {
            log.debug("Using cached authentication info [{}] to perform credentials matching.", info);
        }

        if (info != null) {
            assertCredentialsMatch(token, info);
        } else {
            log.debug("No AuthenticationInfo found for submitted AuthenticationToken [{}].  Returning null.", token);
        }

        return info;
    }

}
```




```
public abstract class AuthenticatingRealm extends CachingRealm implements Initializable {

    protected void assertCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) throws AuthenticationException {
        CredentialsMatcher cm = getCredentialsMatcher();
        if (cm != null) {
            if (!cm.doCredentialsMatch(token, info)) {
                //not successful - throw an exception to indicate this:
                String msg = "Submitted credentials for token [" + token + "] did not match the expected credentials.";
                throw new IncorrectCredentialsException(msg);
            }
        } else {
            throw new AuthenticationException("A CredentialsMatcher must be configured in order to verify " +
                    "credentials during authentication.  If you do not wish for credentials to be examined, you " +
                    "can configure an " + AllowAllCredentialsMatcher.class.getName() + " instance.");
        }
    }
}
```


```
public interface CredentialsMatcher {

    /**
     * Returns {@code true} if the provided token credentials match the stored account credentials,
     * {@code false} otherwise.
     *
     * @param token   the {@code AuthenticationToken} submitted during the authentication attempt
     * @param info the {@code AuthenticationInfo} stored in the system.
     * @return {@code true} if the provided token credentials match the stored account credentials,
     *         {@code false} otherwise.
     */
    boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info);

}
```





## 

shiro-web会自动login。

## 总结

认证过程就是Subject先login，根据Realm实现的方法尝试将Token转成Info，转成功也就是业务数据库能找到对象没有抛出异常或者返回null说明认证成功。
