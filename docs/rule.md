## 限制和规则
### 定义mapper
```
type ActivityMapperImpl struct {
//定义mapper文件的接口和结构体，也可以只定义结构体就行
//mapper.go文件 函数参数（自定义结构体参数（属性必须大写）,*GoMybatis.Session作为该sql执行的session） error 为返回错误
	SelectByIds       func(ids []string) ([]Activity, error)                                                            `mapperParams:"ids"`
	SelectAll         func() ([]Activity, error)
	SelectByCondition func(name string, startTime time.Time, endTime time.Time, page int, size int) ([]Activity, error) `mapperParams:"name,startTime,endTime,page,size"`
	UpdateById        func(session *GoMybatis.Session, arg Activity) (int64, error) //参数中包含有*GoMybatis.Session的类型，用于自定义事务
	Insert            func(arg Activity) (int64, error)
	CountByCondition  func(name string, startTime time.Time, endTime time.Time) (int, error)                            `mapperParams:"name,startTime,endTime"`
	DeleteById        func(id string) (int64, error)                                                                    `mapperParams:"id"`
	Choose            func(deleteFlag int) ([]Activity, error)   
}
```

* func 名称必须是可导出的，首字母大写（xml中首字母可小写，框架可忽略xml中的函数名大小写）
* 其中 func 返回值必须有一个error
* 多个基本类型参数请tag注解  `mapperParams:"*,*,*"`,逗号隔开，参数对应xml中出现的‘#{*}’字段
* 参数中如果传入 *GoMybatis.Session，框架则使用该session，否则框架将新建一个session操作数据库
* 如果xml没有出现struct定义的func，框架会在扫描阶段 painc，并且提示缺少实现的 函数名称

### 定义Mapper xml
xml文件案例:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"https://raw.githubusercontent.com/zhuxiujia/GoMybatis/master/mybatis-3-mapper.dtd"
>
<mapper>
    <!-- 查询活动数据集，if表达式里的 != '' 表示 判断参数是否为golang 零值 -->
    <select id="SelectByCondition" >
        select
        <trim prefix="" suffix="" suffixOverrides=",">
            <if test="Name != ''">name,</if>
        </trim>
        from biz_activity where delete_flag=1
        <if test="Name != ''">
            and name like concat('%',#{Name},'%')
        </if>
        <if test="StartTime != 0">
            and create_time >= #{StartTime}
        </if>
        <if test="EndTime != 0">
            and create_time &lt;= #{EndTime}
        </if>
        order by create_time desc
        <if test="Page != 0 and Size != 0">limit #{Page}, #{Size}</if>
    </select>
</mapper>
```

* if表达式判断参数是否0值，注意 time.Time的零值为0，日期和数值的零值为0，string的零值为''
* 对于"<，>" 符号可能需要转义字符，例如 '<' 对应 '&lt';
* 为了正常显示dtd定义xml智能提示，推荐使用GoLand或者intellij idea打开和编辑xml文件。dtd链接请指向例子中的 "https://raw.githubusercontent.com/zhuxiujia/GoMybatis/master/mybatis-3-mapper.dtd"
