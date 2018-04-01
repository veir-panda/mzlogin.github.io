> 本文系学习SSH框架整合时的学习笔记，学习的内容为慕课网的[基于SSH实现员工管理系统之案例实现篇](http://www.imooc.com/learn/679)


----------


准备
--

 1. 新建Web项目 	
 2. 引入SSH开发包 	
 3. 配置文件
 
		1. Web.xml:里面包括Struts2过滤器，Spring监听器
		2. ApplicationContext.xml：注意版本的问题
		3. Struts.xml
		4. db.properties：修改项目所用的数据库
		5. lo4j.properties
 4. 创建包结构

		1. *.action
		2. *.domain
		3. *.service
		4. *.service.impl
		5. *.dao
		6. *.dao.impl


开始
--

 1. 创建实体类：实体类ID一般设置为Integer引用类型
**理由**：如果数据库查询字段为null，返回给int型，会报错，Integer则不会，Java里面的int型默认为0，数据库里面的int默认为null

 2. 创建实体类映射文件

 3. 引入页面
>  注意：
> 		1. 对于可能出现的img、css、js的路径找不到问题，应该在路径前面加上：`${pageContext.request.contextPath}/+路径`
> 		2. 引入Struts2页面标签库:`<%@taglib uri="/struts-tags" prefix="s"%>`
> 		3. 对于页面的表单元素，以Struts2代替：`<s:form></form>`
> 		4. 表单里面的属性字段，name属性要与后台实体类保持一致

 4. 依次创建`action`、`service`、`serviceImpl`、`Dao`、`DaoImpl`
 5. 创建`action`时，一般要继承`ActionSupport`，实现`ModelDriven` ,重写`getModel()`方法
 6. 一般要对`service`和`dao`类封装借口，然后在实现包*.impl里面实现
 7. 创建`dao`类时一般要继承`HibernateDaoSupport`类
 8. 在`Struts.xml`文件中配置`action`
	    > 注意：action里面的class属性要在Spring的配置文件中配置好Action类的bean之后再把id写到此处；

 9. 将各个bean装配好
> 例如：在action中需要调用`serviceImpl`中的方法，需要先实例化一个类,就用到了`Spring`的依赖注入，这里采用`set`方法注入，先在`action`中声明`serviceImpl`变量，再为这个变量提供一个set方法，不要忘了在配置文件中装配`bean`

 10. 完了。
 

----------


**开发中遇到的坑**
-------

	

 - `userAction`中继承了`ModelDriven`的类，要重写`getModel()`方法，而该方法不能直接返回user类，可以在方法里面写上：
```java
  if(user == null){ 
	  user = new User(); 	
	} 
	return user;
```
**改正**：后面证明，在声明实体的时候new一个该类对象出来，即可在该方法里面直接 `return user；`

 - hql语句里面"`from 实体类名`",一定要注意实体类名通常第一个字母一般要大写，不是数据库的表名，而是实体类名。

	

 - 实体类`entity`中，对于`set`集合的属性，重写`toString`方法时，不能有该属性
   **猜测理由**是：打印`Column`类时，会打印`Article`类，而打印`Article`又会打印`Column`类（因为双向关联），成了死循环

	

 - 对于`Hibernate`延迟加载策略中出现的问题，通常是由于在`Hibernate`已经关闭了`session`的情况下继续执行数据库的操作而发生的错误，`Spring`中提供了`OpenSessionViewFilter`来支持此操作
 
	 **配置方法**：在web.xml里面配置过滤器，拦截的`urlparteen`为`/*`, class为该版本的`OpenSessionViewFilter`
	 **特别注意**：按照上面的配置之后，出现错误，检查是否在多对一的关联关系配置文件里面，lazy的属性值是否写了不允许的值。
 - Structs2从一个Action转到另外一个Action时，有两种方式： 		
	 1. 修改`<result type="redirectAction">ActionName</result>`,重定向，参数不能通过`request.attribute`传过去,原理同jsp/servlet类似
	 
	也可以传递参数，通过`<param name="参数名">${参数}</param>`或者直接`<result type="redirectAction">ActionName?参数名=${参数}</result>`
	 2. `<result type="chain">ActionName</result>`
		 **特别注意**，里面的`ActionName`名字前不能加上根路径"/",原因不知