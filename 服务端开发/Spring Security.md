# Spring Security

## 简述spring security提供的基本能力有哪些？

* 从web层面

	对每一个用户进行认证

	对每一个URL进行授权

	* 有怎样的角色，才能访问？

* 从方法层面，可以从方法上看用户是否有某些权限？

* 可配置的项目

	![image-20210318193456855](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210318193456855.png)

	用户数据存储方式：内存、数据库

* 安全注解

	在需要授权的方法前加Secured，在角色前面加ROLE变成权限

	![image-20210318193711584](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210318193711584.png)