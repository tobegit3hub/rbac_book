# Shiro目录树

## 目录介绍

* aop，Annotation相关工具类，可不了解。
* authc，所有认证相关类，包含Credential、Token、Info定义，常用。
* authz，所有授权相关类，包含Permission、Role定义，常用。
* cache，缓存服务相关类，可不了解。
* codec，常用的编码工具类实现，可不了解。
* concurrent，拓展Java的Executor，非必须。
* config，实现Ini配置读取，可不了解。
* crypto，常用的加密算法工具类，可不了解。
* dao，定义了几个异常工具类，可不了解。
* env，定义了Meta相关工具类，可不了解。
* event，事件相关类，可不了解。
* io，定义了序列化相关工具类，可不了解。
* jndi，对Java的LDAP类的拓展，对接LDAP需要了解。
* ldap，定义了几个异常工具类，可不了解。
* mgt，定义了SecurityManager相关类，需要用它来设置Realm，常用。
* realm，定义了预实现的所有Realm，常用。
* session，定义了Session相关类，可不了解。
* subject，定义了Subject相关类，常用。
* util，定义了其他工具类，可不了解。
* SecurityUtil.java，用户获取Subject等操作，常用。

## 总结

Shiro目录结构算是比较清晰，重点是authc和authz目录，但每个目录都有所有类实现的细节，包含了一些deprecated类，因此不需要对每个文件都理解清楚，根据代码运行逻辑查看对应的类即可。
