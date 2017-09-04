# 理解认证过程

## 介绍

Shiro有自己定义的认证流程，理解整个流程有助于我们熟悉Shiro并且更好地拓展和自定义认证功能。

## 登录流程

用户通过`SecurityUtils.getSubject()`获取Subject后，调用`login()`函数。这是会做几件事情，首先是调用`S
ecurityManager.login()`，这里通过Subject获得Principal为username，修改本地的`authenticated`变量，根据Token类型来修改登录Host和系统Session。

而`SecurityManager.login()`会做更具体的操作，默认会使用DefaultSecurityManager，使用了唯一的Realm就是IniRealm，先检查Realm的`supports()`函数能否支持UsernamePasswordToken，然后在`doGetAuthenticationInfo()`方法中返回了Account也就是Info对象。对象认证相关的Info的Principal就是username，Credential就是password，授权相关的Info还包含两条oject permission和两条role。

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

这里使用的Account就是包含认证Info和授权Info，这个Realm必须实现`doGetAuthenticationInfo()`方法，根据Token从Ini文件中获取对应的Info返回出去。

实际上认证过程就是通过Token拿Info，如果系统根据Token的Principal也无法拿到Info而抛出异常，那么认证就失败了。除此之外，还需要检查Token拿到的Info是否是匹配了，默认就是取得两者的Credential检查password是否相等，在基类中定义了final方法必须使用Matcher进行检查，具体代码如下。

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

这里调用了`assertCredentialsMatch()`方法，就是检查Token和Info的Credential是否相等，使用默认的Matcher进行比较。


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

而Matcher是一个接口，Shiro已经预实现了一些方法，如果我们需要自定义Matcher也可以实现这个接口，然后在这个Realm中配置即可。

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

## 自动登录

前面quickstart的项目例子中，我们需要手动调用`login()`方法，后面的请求才会认证通过，那么在Spring或者其他Web项目中我们希望在HTTP请求中就能自动登录。

实际上在Shiro提供的Filter中，就包含AnonymousFilter和AuthenticatingFilter，我们在配置Shiro时可以指定不同URL经过不同Filter，一般login路径会指定"anon"，而其他路径都必须是"authc"。

```
public abstract class AuthenticatingFilter extends AuthenticationFilter {

    protected boolean executeLogin(ServletRequest request, ServletResponse response) throws Exception {
        AuthenticationToken token = createToken(request, response);
        if (token == null) {
            String msg = "createToken method implementation returned null. A valid non-null AuthenticationToken " +
                    "must be created in order to execute a login attempt.";
            throw new IllegalStateException(msg);
        }
        try {
            Subject subject = getSubject(request, response);
            subject.login(token);
            return onLoginSuccess(token, subject, request, response);
        } catch (AuthenticationException e) {
            return onLoginFailure(token, e, request, response);
        }
    }

    protected abstract AuthenticationToken createToken(ServletRequest request, ServletResponse response) throws Exception;

    protected AuthenticationToken createToken(String username, String password,
                                              ServletRequest request, ServletResponse response) {
        boolean rememberMe = isRememberMe(request);
        String host = getHost(request);
        return createToken(username, password, rememberMe, host);
    }

    protected AuthenticationToken createToken(String username, String password,
                                              boolean rememberMe, String host) {
        return new UsernamePasswordToken(username, password, rememberMe, host);
    }
}
```

这些都在shiro-web库中实现的，如果使用shiro-spring也会自动包含在里面，这样我们就不需要在每个API请求都显式调用`login()`了。

## 总结

认证过程就是Subject进行login，将请求时传入Token作为参数，自己实现Realm的方法通过Token获取Info对象，并且检查Token和Info的Credential是否符合Matcher的匹对，如果Token到Info的过程中抛出异常或者返回null说明认证失败了。
