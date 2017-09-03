# 理解Matcher

## Matcher介绍

Matcher是Shiro定义的一种抽象，表示用户请求时生成的Token与后端认证服务Realm返回的Info是否能匹配的关系，Info的将在后面详细介绍。

一般情况下，用户登录使用UsernamePasswordToken，后端服务使用预实现的Realm，用户不需要修改Matcher也可以，只有在我们自定义Token和Realm的情况下想实现自定义的Matcher方法才需要修改，例如AKSK认证可能需要加入签名判断逻辑等。

## Matcher源码分析

我们可以直接查看`CredentialsMatcher.java`源码，用户只需要实现一个方法即可。

```java
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

这里首先会传入一个Token，可以通过Token来获取Principal和Credential信息，然后比较时根据Principal返回对应的Info信息，一般来说Info也包含Credential信息可以直接取出比较。

我们再看一个简单的实现类`SimpleCredentialsMatcher.java`，也是通过Token和Info来获取Credential对比。

```java
public class SimpleCredentialsMatcher extends CodecSupport implements CredentialsMatcher {

    public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {
        Object tokenCredentials = getCredentials(token);
        Object accountCredentials = getCredentials(info);
        return equals(tokenCredentials, accountCredentials);
    }

	protected boolean equals(Object tokenCredentials, Object accountCredentials) {
	    if (log.isDebugEnabled()) {
	        log.debug("Performing credentials equality check for tokenCredentials of type [" +
	                tokenCredentials.getClass().getName() + " and accountCredentials of type [" +
	                accountCredentials.getClass().getName() + "]");
	    }
	    if (isByteSource(tokenCredentials) && isByteSource(accountCredentials)) {
	        if (log.isDebugEnabled()) {
	            log.debug("Both credentials arguments can be easily converted to byte arrays.  Performing " +
	                    "array equals comparison");
	        }
	        byte[] tokenBytes = toBytes(tokenCredentials);
	        byte[] accountBytes = toBytes(accountCredentials);
	        return MessageDigest.isEqual(tokenBytes, accountBytes);
	    } else {
	        return accountCredentials.equals(tokenCredentials);
	    }
	}
```

一般情况下，我们讲Token和Info得到的Credential都转成byte array，然后使用Java自带的数组比较返回结果即可，如果是字符串或者其他类型则可以用Java的默认比较策略。

## 自定义Matcher

在AKSK的场景下，因为我们自己定义的Token是包含Header的所有信息和签名的，而我们指定定义的Info对象，里面的Credential则是secret key，因此必须重写Matcher加入AKSK签名逻辑才可以比较。

```java
public class AkskMatcher extends CodecSupport implements CredentialsMatcher {
    public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {
        AkskCredentials tokenCredentials = (AkskCredentials) token.getCredentials();
        String secretKey = (String) info.getCredentials();
        signature = SignatureUtil.computeSignature(secretKey);
        return signature.equals(tokenCredentials.getSignature());
    }
}
```

代码实现也比较简单，从Token中获取Header的信息和签名，从Info中获取secret key，重新签名后进行比较即可。

如果要使用这个Matcher，在Realm类中调用下面的方法即可。

```java
Realm.setCredentialsMatcher(customedMatcher);
```

## 总结

这里我们知道，用户可以自定义Realm的Matcher来实现任意的Credential对比，而一般情况下使用默认的byte array对比也足够了。
