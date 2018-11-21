# GoMybatis
> Github地址 https://github.com/zhuxiujia/GoMybatis

> 文档地址 https://zhuxiujia.github.io/gomybatis.io/
![Image text](https://github.com/zhuxiujia/gomybatis.io/blob/master/docs/logo.png)
GoMybatis 是根据java版 Mybatis3 的实现,基于Go标准库sql驱动和govaluate表达式及反射实现。
GoMybatis 内部在初始化时反射分析mapper xml生成golang的func代码，默认支持绝大部分的Java版的mybatis标签和规范

> 支持标签`<select>,<update>,<insert>,<delete>,<trim>,<if>,<set>,<foreach>`

> 支持本地事务

> 支持远程的伪分布式事务

> GoMybatis的目标是方便快捷的使用SQL，我们认为SQL并不会被ORM框架所替代，使用SQL使项目逻辑更加规范，比orm框架灵活，当数据库运行缓慢时 dba或者开发 可分析慢sql

