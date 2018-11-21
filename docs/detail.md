# 实现原理

## 当我们定义mapper
<pre>
type ActivityMapperImpl struct {
	SelectAll         func(result *[]model.Activity) error
	SelectByCondition func(name string, startTime time.Time, endTime time.Time, page int, size int, result *[]model.Activity) error `mapperParams:"name,startTime,endTime,page,size"`
	UpdateById        func(arg model.Activity, result *int64) error
	Insert            func(arg model.Activity, result *int64) error
	CountByCondition  func(name string, startTime time.Time, endTime time.Time, result *int) error                                  `mapperParams:"name,startTime,endTime"`
}
</pre>

> 当我们使用 GoMybatis.UseProxyMapper(&activityMapper, bytes, engine) 实例化ActivityMapperImpl中的函数时，即重写了ActivityMapperImpl中的所有func

> 可以看到 ActivityMapperImpl 定义了几个 func,func 首字母大写代表可导出的函数，这里我们可以使用反射重写此方法

> 可以看到 ActivityMapperImpl 定义了几个 tag,`mapperParams:"name,startTime,endTime,page,size"` 反射根据tag 确定参数名和实际值,然后写入到xml定义的#{name}，#{startTime}值中自动替换.
