# 理解授权过程



securityManager.isPermitted()




```java
public abstract class AuthorizingSecurityManager extends AuthenticatingSecurityManager {

private Authorizer authorizer;

    public boolean isPermitted(PrincipalCollection principals, String permissionString) {
        return this.authorizer.isPermitted(principals, permissionString);
    }

}

    ```

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
}
    ```



    "lightsaber:*".implies("winnebage:drive:eagle5")
"winnebage:drive:eagle5".implies("winnebage:drive:eagle5")

## 总结

授权过程就是Subject调用isPermitted或者hasRole等方法，系统会通过Subject的Principal来获取Realm提供的Info对象，然后根据Info对象取出多条Permission与请求内容进行比较，通过implies方法对比通过即可。


