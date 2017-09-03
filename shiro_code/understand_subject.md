# 理解Subject

## 介绍

Subject是Shiro定义的一种抽象，可以认为是一个用户，但User在业务中有特殊含义所以在Shiro中使用Subject来代替User。

我们可以阅读`Subject.java`源码。

```java
/**
 * A {@code Subject} represents state and security operations for a <em>single</em> application user.
 * These operations include authentication (login/logout), authorization (access control), and
 * session access. It is Shiro's primary mechanism for single-user security functionality.
 * <h3>Acquiring a Subject</h3>
 * To acquire the currently-executing {@code Subject}, application developers will almost always use
 * {@code SecurityUtils}:
 **/

public interface Subject {
  void login(AuthenticationToken token) throws AuthenticationException;
  Object getPrincipal();
  boolean isPermitted(String permission);
  ......
}
```

从注释了解到，Subject可以认为就是一个User，一般我们都需要通过`SecurityUtils`来获取Subject，为什么？因为不同的部署环境获取Subject的方法可能不同，例如本地Java应用可能在ThreadLocal中获取对象，而Spring等可能在容器中获取对象。

通过代码我们也了解到，Subject对象本身就包含登录登出功能，而且有认证和授权的接口功能，下面来详细看一下。

## 使用Subject

如何使用Subject，我们可以直接看`quickstart`项目中的[Quickstart.java](https://github.com/apache/shiro/blob/master/samples/quickstart/src/main/java/Quickstart.java).

```java
Subject currentUser = SecurityUtils.getSubject();

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

if (currentUser.isPermitted("lightsaber:wield")) {
    log.info("You may use a lightsaber ring.  Use it wisely.");
} else {
    log.info("Sorry, lightsaber rings are for schwartz masters only.");
}
```

首先，我们可以通过`isAuthenticated()`方法来判断用户是否登录，如果没有登录，就使用一个新创建的UsernamePasswordToken来登录，至于Token的细节将在后面介绍。然后用户可以使用`isPermitted()`等方法来检查该用户是否有权限。

实际上这个简单的例子使用了UsernamePasswordToken、SimpleAccount、SimpleAuthenticationInfo、SimpleAuthorizationInfo和TextConfigurationReal等功能，如何比较用户的密码还有是否有权限，这些细节也将会更加细致得展开介绍。

这里我们需要知道，我们通过SecurityUtils可以获得一个Subject对象，这个Subject有可能一开始什么都没有所有认证和授权都会失败，然后我们可以用它来login，登录后就有身份可以进行认证和授权了，检查是否有认证和授权通过这个Subject就可以做到了。


