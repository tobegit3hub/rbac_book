# 理解Token

## Token介绍

Shiro中Token就是用户用于登录的信息，就是Subject调用`login()`函数的参数，一般是用户名密码，也可以是我们自定义的格式。

## Token源码

我们可以看`AuthenticationToken.java`源码，定义了Token的接口，只需要实现这两个接口即可。

```java
public interface AuthenticationToken extends Serializable {
  Object getPrincipal();
  Object getCredentials();
}
```

其中`UsernamePasswordToken.java`就实现了这两个接口，直接返回username和password。

```java
public Object getPrincipal() {
    return getUsername();
}

public Object getCredentials() {
    return getPassword();
}

public String getUsername() {
    return username;
}

public void setPassword(char[] password) {
    this.password = password;
}
```

而我们想想，什么时候我们会创建一个新的UsernamePasswordToken来登录呢，实际上`shiro-web`这个库替我们实现了。在AuthenticatingFilter中，`execute()`方法会调用`createToken()`方法来创建一个UsernamePasswordToken来登录。

## 自定义Token

要自定义Token非常简单，只需要继承已有的AuthenticationToken或者UsernamePasswordToken类即可，然后实现两个方法返回特定对象。

```java
public class AkskAuthenticationToken implements AuthenticationToken {
    @Override
    public Object getPrincipal() {
        return "";
    }

    @Override
    public Object getCredentials() {
        return "";
    }
}
```

一般实现Token时，我们也需要实现对应的Filter，在请求过来是根据请求内容构造我们自定义的Token，并且实现Realm时重载`supports()`方法这样我们根据Token类型只需要处理自己定义的Token即可。

这里我们需要知道Token就是用户请求过来是构建的对象，如果是默认的UsernamePasswordToken，那么从Token中获取的Principal就是username，获取的Credential就是加密后的password，当然我们也可以定制Token来返回任意的Java对象。至于这个Token怎样来验证是否通过认证，后面继续看Matcher详细介绍。
