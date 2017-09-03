# 配置Shiro

## 介绍

用户使用Shiro框架时，可以使用不同的Realm，配置不同的User和Role。

配置方式一般分为两种，通过配置文件或者通过代码配置。

## 配置文件

Shiro支持Ini等配置文件格式，例如在`quickstart`项目中，使用简单易读的Ini配置文件即可。

参考[shiro.ini](https://github.com/apache/shiro/blob/master/samples/quickstart/src/main/resources/shiro.ini)文件。

```
[users]
# user 'root' with password 'secret' and the 'admin' role
root = secret, admin
# user 'guest' with the password 'guest' and the 'guest' role
guest = guest, guest
# user 'presidentskroob' with password '12345' ("That's the same combination on
# my luggage!!!" ;)), and role 'president'
presidentskroob = 12345, president
# user 'darkhelmet' with password 'ludicrousspeed' and roles 'darklord' and 'schwartz'
darkhelmet = ludicrousspeed, darklord, schwartz
# user 'lonestarr' with password 'vespa' and roles 'goodguy' and 'schwartz'
lonestarr = vespa, goodguy, schwartz

[roles]
# 'admin' role has all permissions, indicated by the wildcard '*'
admin = *
# The 'schwartz' role can do anything (*) with any lightsaber:
schwartz = lightsaber:*
# The 'goodguy' role is allowed to 'drive' (action) the winnebago (type) with
# license plate 'eagle5' (instance specific id)
goodguy = winnebago:drive:eagle5
```

大家可能发现，这个完整的配置并没有选择使用的Realm，确实是这样的，这里默认会使用TextConfigurationRealm，当然我们可以在配置文件中选择其他预定义或者自己实现的Realm类。

## 代码配置

如果使用Spring等框架，可以在代码中配置Shiro的相关参数。

例如创建ShiroConfig类，加上`@Configuration`和`@Bean`标注，在这里选择预实现或者自定义的Realm类。

```
@Configuration
public class ShiroConfig {
    @Bean(name = "shiroFilter")
    public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        Map<String, String> filterChainMap = new LinkedHashMap<>();
        filterChainMap.put("/**", "anon");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainMap);
        return shiroFilterFactoryBean;
    }

    @Bean
    public SecurityManager securityManager(UsernamePasswordRealm usernamePasswordRealm,
                                           SessionManager sessionManager) {
        DefaultSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(usernamePasswordRealm);
        securityManager.setSessionManager(sessionManager);
        return securityManager;
    }
```
