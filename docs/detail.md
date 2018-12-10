# 事务原理
![Image text](https://zhuxiujia.github.io/gomybatis.io/assets/tx.png)
## mapper(Struct)代理原理
<pre>
type ActivityMapperImpl struct {
	SelectByIds       func(ids []string) ([]Activity, error)                                                            `mapperParams:"ids"`
	SelectAll         func() ([]Activity, error)
	SelectByCondition func(name string, startTime time.Time, endTime time.Time, page int, size int) ([]Activity, error) `mapperParams:"name,startTime,endTime,page,size"`
	UpdateById        func(session *GoMybatis.Session, arg Activity) (int64, error) //参数中包含有*GoMybatis.Session的类型，用于自定义事务
	Insert            func(arg Activity) (int64, error)
	CountByCondition  func(name string, startTime time.Time, endTime time.Time) (int, error)                            `mapperParams:"name,startTime,endTime"`
	DeleteById        func(id string) (int64, error)                                                                    `mapperParams:"id"`
	Choose            func(deleteFlag int) ([]Activity, error)   
}
</pre>

> 当我们使用 GoMybatis.UseProxyMapper(&activityMapper, bytes, engine) 实例化ActivityMapperImpl中的函数时，即重写了ActivityMapperImpl中的所有func

> 可以看到 ActivityMapperImpl 定义了几个 func,func 首字母大写代表可导出的函数，这里我们可以使用反射重写此方法

> 可以看到 ActivityMapperImpl 定义了几个 tag,`mapperParams:"name,startTime,endTime,page,size"` 反射根据tag 确定参数名和实际值,然后写入到xml定义的#{name}，#{startTime}值中自动替换.
