# 理解Info

## AuthenticationInfo介绍

前面提到，Matcher其实就是从Token和AuthenticationInfo中分别获取Credential对象进行比较，那么AuthenticationInfo是什么呢？其实是服务端Realm通过Subject信息构造的Info对象，里面可以包含任意的Credential对象而已。


## Info源码分析

我们看一下`AuthenticationInfo.java`的源码。

```java
public interface AuthenticationInfo extends Serializable {

    /**
     * Returns all principals associated with the corresponding Subject.  Each principal is an identifying piece of
     * information useful to the application such as a username, or user id, a given name, etc - anything useful
     * to the application to identify the current <code>Subject</code>.
     * <p/>
     * The returned PrincipalCollection should <em>not</em> contain any credentials used to verify principals, such
     * as passwords, private keys, etc.  Those should be instead returned by {@link #getCredentials() getCredentials()}.
     *
     * @return all principals associated with the corresponding Subject.
     */
    PrincipalCollection getPrincipals();

    /**
     * Returns the credentials associated with the corresponding Subject.  A credential verifies one or more of the
     * {@link #getPrincipals() principals} associated with the Subject, such as a password or private key.  Credentials
     * are used by Shiro particularly during the authentication process to ensure that submitted credentials
     * during a login attempt match exactly the credentials here in the <code>AuthenticationInfo</code> instance.
     *
     * @return the credentials associated with the corresponding Subject.
     */
    Object getCredentials();

}
```

AuthenticationInfo和Token类似，定义了两个接口用户获取Pricipal和Credential，我们只需要实现这两个接口即可。

下面有一个Shiro已经实现的SimpleAuthenticationInfo类，只需要创建时传入Pricipal和Credential信息即可。

```java
public class SimpleAuthenticationInfo implements MergableAuthenticationInfo, SaltedAuthenticationInfo {

    public SimpleAuthenticationInfo(Object principal, Object credentials, String realmName) {
        this.principals = new SimplePrincipalCollection(principal, realmName);
        this.credentials = credentials;
    }
}
```

这个接口也实现了MergableAuthenticationInfo，在同时使用多个Realm时可以合并用户信息。

## 总结

这里我们知道，通过创建Token可以传入Pricipal和Credential，而Realm定义中可以通过创建Info来传入Pricipal和Credential，两者通过Realm设置的Matcher就可以比较Credential来判断是否认证通过了。
