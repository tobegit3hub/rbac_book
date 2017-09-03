# 理解Credential

## Credential介绍

前面介绍了Principal，在token对象中还包含了用户认证的Credential，我们查看`UsernamePasswordToken`源码继续了解。

```java
 public Object getCredentials() {
    return getPassword();
}

public char[] getPassword() {
    return password;
}
```

如果我们直接打印token的Credential信息，发现直接返回加密后的符串，从代码中也可以看到UsernamePasswordToken这个token的Credential其实就是password，但会经过Shiro的加密过程。

后面我们还会介绍自定义token，这时的Credential就不一定是字符串类型，例如在AKSK认证的过程，Credential就是用户在HTTP请求中的header信息，类型就是自定义的Java类。

这里我们需要知道，Credential就是Subject要认证的“密码”，当然不一定是字符串密码，而如果不是密码那如何进行比较和验证呢，其实就是Matcher实现的。
