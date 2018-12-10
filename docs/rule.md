# 限制和规则

# 定义mapper
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

> 包含一系列func的struct 为mapper定义文件,其中 func 返回值必须为error。多个参数请定义tag注解  `mapperParams:"***"`,逗号隔开，参数为你的xml中sql出现的‘#{***}’字段

# 定义Mapper xml
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

> 这里可以看到，if表达式判断参数是否0值，注意 time.Time的零值为0，数值的零值为0，string的零值为''
