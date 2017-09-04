# 理解授权过程

## 介绍

Shiro有指定定义的授权流程，理解整个过程有助于我们拓展和自定义与业务相关的认证逻辑。

## 认证流程

用户登录成功后可以获得认证过的Subject对象，然后使用`isPermitted()`或者`hasRole()`方法就可以进行认证了，而认证请求会调用`SecurityManager.isPermitted()`方法。

这个方法其实也是使用Shiro定义的Authorizer来认证，这里会把Subject的Principal传入作为参数来验证。

```java
public abstract class AuthorizingSecurityManager extends AuthenticatingSecurityManager {

    private Authorizer authorizer;

    public boolean isPermitted(PrincipalCollection principals, String permissionString) {
        return this.authorizer.isPermitted(principals, permissionString);
    }

}
```	

而Shiro默认使用ModularRealmAuthorizer，它可以调用多个Realm来分别验证权限。

```java
public class ModularRealmAuthorizer implements Authorizer, PermissionResolverAware, RolePermissionResolverAware {

    public boolean isPermitted(PrincipalCollection principals, String permission) {
        assertRealmsConfigured();
        for (Realm realm : getRealms()) {
            if (!(realm instanceof Authorizer)) continue;
            if (((Authorizer) realm).isPermitted(principals, permission)) {
                return true;
            }
        }
        return false;
    }
}
```

在quickstart例子中使用了SimpleAccountRealm，它只需要实现`doGetAuthorizationInfo()`方法即可，这个方法是传入Principal然后返回包含Permission和Role即可的Info对象，这里例子就是Account。

```java
public class SimpleAccountRealm extends AuthorizingRealm {
    	protected final Map<String, SimpleAccount> users; //username-to-SimpleAccount

    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        String username = getUsername(principals);
        USERS_LOCK.readLock().lock();
        try {
            return this.users.get(username);
        } finally {
            USERS_LOCK.readLock().unlock();
        }
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

而具体是如何检查Permission的呢，其实是在基类AuthorizingRealm中，它调用子类实现的`getAuthorizationInfo()`方法来获得Info，然后如果需要检查Permission就从Info中获取所有Permission，然后进行Permission比较。

```java
public abstract class AuthorizingRealm extends AuthenticatingRealm
        implements Authorizer, Initializable, PermissionResolverAware, RolePermissionResolverAware {

    public boolean isPermitted(PrincipalCollection principals, Permission permission) {
        AuthorizationInfo info = getAuthorizationInfo(principals);
        return isPermitted(permission, info);
    }

    protected boolean isPermitted(Permission permission, AuthorizationInfo info) {
        Collection<Permission> perms = getPermissions(info);
        if (perms != null && !perms.isEmpty()) {
            for (Permission perm : perms) {
                if (perm.implies(permission)) {
                    return true;
                }
            }
        }
        return false;
    }

    protected Collection<Permission> getPermissions(AuthorizationInfo info) {
        Set<Permission> permissions = new HashSet<Permission>();

        if (info != null) {
            Collection<Permission> perms = info.getObjectPermissions();
            if (!CollectionUtils.isEmpty(perms)) {
                permissions.addAll(perms);
            }
            perms = resolvePermissions(info.getStringPermissions());
            if (!CollectionUtils.isEmpty(perms)) {
                permissions.addAll(perms);
            }

            perms = resolveRolePermissions(info.getRoles());
            if (!CollectionUtils.isEmpty(perms)) {
                permissions.addAll(perms);
            }
        }

        if (permissions.isEmpty()) {
            return Collections.emptySet();
        } else {
            return Collections.unmodifiableSet(permissions);
        }
    }
}
```

在这里例子中，最终会判断下面两个Permission是否匹配，并且使用默认的WildcardPermission。

```java
"lightsaber:*".implies("winnebage:drive:eagle5")
"winnebage:drive:eagle5".implies("winnebage:drive:eagle5")
```

## 总结

如果认证过程是Token到Info的逻辑检查，那么授权就是Principal到Info的逻辑检查，通过认证得到的Subject内部的Principal对象，我们可以获得一个包含多条Permission和多条Role的Info对象，然后Shiro已经实现的方法就是自动检查这些Permission是否足够。
