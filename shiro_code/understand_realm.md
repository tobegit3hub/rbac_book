# 理解Realm

## Realm介绍

Realm是Shiro定义的一种抽象，就是将Shiro定义的Subject、Role等概念与我们系统设计的对象关联起来，起到桥梁的作用。

Shiro实现了基于Ini、LDAP和JDBC的Realm，但我们一般不能直接用，因为username、password这些信息我们会在系统中定义，不一定真好符合Shiro预实现的Realm格式要求，因此我们可以基于已有的类实现自己的Realm。

## Realm源码分析

Shiro提供了AuthenticatingRealm抽象类可以很方便实现认证功能，提供了Authorizer接口可以很方便实现授权功能，如果既需要认证又需要授权可以使用AuthorizingRealm抽象类。

直接查看`AuthorizingRealm.java`源码。

```java
/**
 * An {@code AuthorizingRealm} extends the {@code AuthenticatingRealm}'s capabilities by adding Authorization
 * (access control) support.
 * <p/>
 * This implementation will perform all role and permission checks automatically (and subclasses do not have to
 * write this logic) as long as the
 * {@link #getAuthorizationInfo(org.apache.shiro.subject.PrincipalCollection)} method returns an
 * {@link AuthorizationInfo}.  Please see that method's JavaDoc for an in-depth explanation.
 * <p/>
 * If you find that you do not want to utilize the {@link AuthorizationInfo AuthorizationInfo} construct,
 * you are of course free to subclass the {@link AuthenticatingRealm AuthenticatingRealm} directly instead and
 * implement the remaining Realm interface methods directly.  You might do this if you want have better control
 * over how the Role and Permission checks occur for your specific data source.  However, using AuthorizationInfo
 * (and its default implementation {@link org.apache.shiro.authz.SimpleAuthorizationInfo SimpleAuthorizationInfo}) is sufficient in the large
 * majority of Realm cases.
 *
 * @see org.apache.shiro.authz.SimpleAuthorizationInfo
 * @since 0.2
 */
public abstract class AuthorizingRealm extends AuthenticatingRealm
        implements Authorizer, Initializable, PermissionResolverAware, RolePermissionResolverAware {

    /**
     * Retrieves the AuthorizationInfo for the given principals from the underlying data store.  When returning
     * an instance from this method, you might want to consider using an instance of
     * {@link org.apache.shiro.authz.SimpleAuthorizationInfo SimpleAuthorizationInfo}, as it is suitable in most cases.
     *
     * @param principals the primary identifying principals of the AuthorizationInfo that should be retrieved.
     * @return the AuthorizationInfo associated with this principals.
     * @see org.apache.shiro.authz.SimpleAuthorizationInfo
     */
    protected abstract AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals);
}
```

这个类还需要我们实现`doGetAuthorizationInfo()`方法，还有需要实现父类的`doGetAuthenticationInfo()`抽象方法，定义如下。

```java
public abstract class AuthenticatingRealm extends CachingRealm implements Initializable {
    /**
     * Retrieves authentication data from an implementation-specific datasource (RDBMS, LDAP, etc) for the given
     * authentication token.
     * <p/>
     * For most datasources, this means just 'pulling' authentication data for an associated subject/user and nothing
     * more and letting Shiro do the rest.  But in some systems, this method could actually perform EIS specific
     * log-in logic in addition to just retrieving data - it is up to the Realm implementation.
     * <p/>
     * A {@code null} return value means that no account could be associated with the specified token.
     *
     * @param token the authentication token containing the user's principal and credentials.
     * @return an {@link AuthenticationInfo} object containing account data resulting from the
     *         authentication ONLY if the lookup is successful (i.e. account exists and is valid, etc.)
     * @throws AuthenticationException if there is an error acquiring data or performing
     *                                 realm-specific authentication logic for the specified <tt>token</tt>
     */
    protected abstract AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException;
 }
 ```

## 实现Realm

因此我们如果要自定义带有认证和授权功能的Realm，需要实现上述两个方法，示例如下。

```java
public class UsernamePasswordRealm extends BaseAuthorizingRealm {

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        UsernamePasswordToken upToken = (UsernamePasswordToken) token;
        String username = upToken.getUsername();
        SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(user, user.getPassword(), getName());
        return authenticationInfo;
    }

    @Override
    public AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        User user = (User) principals.getPrimaryPrincipal();
        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
        Set<String> perSet = authorizationService.getShiroPermissionStrings(user.getId());
        authorizationInfo.setStringPermissions(perSet);
        return authorizationInfo;
    }
}
```

