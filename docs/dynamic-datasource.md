> 动态多数据源

> 有时我们会使用多个数据库作为我们的数据源，以达到分库分表或者是高可用的效果。此时就需要多数据源和路由的功能


>  1多数据源：
```
//初始化引擎
var engine = GoMybatis.GoMybatisEngine{}.New()

//第一个数据源
err := engine.Open("mysql", MysqlUri) //此处请按格式填写你的mysql链接，这里用*号代替
	if err != nil {
		panic(err.Error())
	}
//第二个数据源
err := engine.Open("mysql", MysqlUri) //此处请按格式填写你的mysql链接，这里用*号代替
	if err != nil {
		panic(err.Error())
	}
//第三个数据源
err := engine.Open("mysql", MysqlUri) //此处请按格式填写你的mysql链接，这里用*号代替
	if err != nil {
		panic(err.Error())
	}
//更多的数据库
```



> 2路由功能：

```
   //动态数据源路由器
	var router = GoMybatis.GoMybatisDataSourceRouter{}.New(func(mapperName string) *string {
		//根据包名路由指向数据源
		if strings.Contains(mapperName, "example.") {
			var url = MysqlUri//第二个mysql数据库,请把MysqlUri改成你的第二个数据源链接
			fmt.Println(url)
			return &url
		}
		//如果返回nil，则框架会选择已存在的默认数据源作为连接
		return nil
	})
	//下一步指定我们的路由器
	engine.SetDataSourceRouter(&router)
```
