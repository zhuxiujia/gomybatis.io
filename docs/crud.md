> 示例源码https://github.com/zhuxiujia/GoMybatis/tree/master/example

## 先定义模型和结构体
```
package example

import "time"
//模型Activity
type Activity struct {
	Id         string    `json:"id"`
	Uuid       string    `json:"uuid"`
	Name       string    `json:"name"`
	PcLink     string    `json:"pcLink"`
	H5Link     string    `json:"h5Link"`
	Remark     string    `json:"remark"`
	CreateTime time.Time `json:"createTime"`
	DeleteFlag int       `json:"deleteFlag"`
}
//Dao - mapper结构体，包含一系列sql方法，相当于Dao层
type ExampleActivityMapper struct {
	SelectByIds       func(ids []string) ([]Activity, error)       `mapperParams:"ids"`
	SelectByIdMaps    func(ids map[int]string) ([]Activity, error) `mapperParams:"ids"`
	SelectAll         func() ([]map[string]string, error)
	SelectByCondition func(name *string, startTime *time.Time, endTime *time.Time, page *int, size *int) ([]Activity, error) `mapperParams:"name,startTime,endTime,page,size"`
	UpdateById        func(session *GoMybatis.Session, arg Activity) (int64, error)
	Insert            func(arg Activity) (int64, error)
	CountByCondition  func(name string, startTime time.Time, endTime time.Time) (int, error) `mapperParams:"name,startTime,endTime"`
	DeleteById        func(id string) (int64, error)                                         `mapperParams:"id"`
	Choose            func(deleteFlag int) ([]Activity, error)                               `mapperParams:"deleteFlag"`
	SelectLinks       func(column string) ([]Activity, error)                                `mapperParams:"column"`
}
//初始化mapper文件和结构体
func InitMapperByLocalSession() ExampleActivityMapper {
    var engine = GoMybatis.GoMybatisEngine{}.New()
	//mysql链接格式为         用户名:密码@(数据库链接地址:端口)/数据库名称   例如root:123456@(***.mysql.rds.aliyuncs.com:3306)/test
	err := engine.Open("mysql", MysqlUri) //此处请按格式填写你的mysql链接，这里用*号代替
	if err != nil {
		panic(err.Error())
	}
	//读取mapper xml文件
	bytes, _ := ioutil.ReadFile("Example_ActivityMapper.xml")
	var exampleActivityMapper ExampleActivityMapper
	//设置对应的mapper xml文件
	engine.WriteMapperPtr(&exampleActivityMapper, bytes)
	return exampleActivityMapper
}
```

## 1 插入
定义xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://raw.githubusercontent.com/zhuxiujia/GoMybatis/master/mybatis-3-mapper.dtd">
<mapper>
   <insert id="insert">
        insert into biz_activity
        <trim prefix="(" suffix=")" suffixOverrides=",">
            <if test="id != null">id,</if>
            <if test="name != null">name,</if>
            <if test="pcLink != null">pc_link,</if>
            <if test="h5Link != null">h5_link,</if>
            <if test="remark != null">remark,</if>
            <if test="createTime != null">create_time,</if>
            <if test="deleteFlag != null">delete_flag,</if>
        </trim>

        <trim prefix="values (" suffix=")" suffixOverrides=",">
            <if test="id != null">#{id},</if>
            <if test="name != null">#{name},</if>
            <if test="pcLink != null">#{pcLink},</if>
            <if test="h5Link != null">#{h5Link},</if>
            <if test="remark != null">#{remark},</if>
            <if test="createTime != null">#{createTime},</if>
            <if test="deleteFlag != null">#{deleteFlag},</if>
        </trim>
    </insert>
</mapper>
```

```
//插入
func Test_inset(t *testing.T) {
	//初始化mapper文件
	var exampleActivityMapper = InitMapperByLocalSession()
	//使用mapper
	var result, err = exampleActivityMapper.Insert(Activity{Id: "171", Name: "test_insret", DeleteFlag: 1})
	if err != nil {
		panic(err)
	}
	fmt.Println("result=", result)
}
```


## 2 修改
定义xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://raw.githubusercontent.com/zhuxiujia/GoMybatis/master/mybatis-3-mapper.dtd">
<mapper>
      <update id="updateById">
          update biz_activity
          <set>
              <if test="name != null">name = #{name},</if>
              <if test="pcLink != null">pc_link = #{pcLink},</if>
              <if test="h5Link != null">h5_link = #{h5Link},</if>
              <if test="remark != null">remark = #{remark},</if>
              <if test="createTime != null">create_time = #{createTime},</if>
              <if test="deleteFlag != null">delete_flag = #{deleteFlag},</if>
          </set>
          where id = #{id} and delete_flag = 1
      </update>
</mapper>
```

```
//修改
func Test_update(t *testing.T) {
	//初始化mapper文件
	exampleActivityMapperImpl := InitMapperByLocalSession()
	var activityBean = Activity{
		Id:   "171",
		Name: "rs168-8",
	}
	var updateNum, e = exampleActivityMapperImpl.UpdateById(nil, activityBean) //sessionId 有值则使用已经创建的session，否则新建一个session
	fmt.Println("updateNum=", updateNum)
	if e != nil {
		panic(e)
	}
}
```


## 3 删除
定义xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://raw.githubusercontent.com/zhuxiujia/GoMybatis/master/mybatis-3-mapper.dtd">
<mapper>
          <update id="deleteById">
              update biz_activity
              set delete_flag = 0
              where id = #{id}
          </update>
</mapper>
```

```
//删除
func Test_delete(t *testing.T) {
	//初始化mapper文件
	var exampleActivityMapperImpl = InitMapperByLocalSession()
	//使用mapper
	var result, err = exampleActivityMapperImpl.DeleteById("171")
	if err != nil {
		panic(err)
	}
	fmt.Println("result=", result)
}
```

## 4 查询
定义xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://raw.githubusercontent.com/zhuxiujia/GoMybatis/master/mybatis-3-mapper.dtd">
<mapper>
           <select id="selectByCondition" resultMap="BaseResultMap">
                  <bind name="pattern" value="'%' + name + '%'"/>
                  select * from biz_activity
                  <where>
                      <if test="name != null">
                          <!--可以使用bind标签 and name like #{pattern}-->
                          and name like #{pattern}
                      </if>
                      <if test="startTime != null">and create_time >= #{startTime}</if>
                      <if test="endTime != null">and create_time &lt;= #{endTime}</if>
                  </where>
                  order by create_time desc
                  <if test="page != null and size != null">limit #{page}, #{size}</if>
           </select>
</mapper>
```

```
//查询
func Test_select(t *testing.T) {
	//初始化mapper文件
	var exampleActivityMapperImpl = InitMapperByLocalSession()
	//使用mapper
	//使用mapper
    var name = "注册"
    var result, err = exampleActivityMapper.SelectByCondition(&name, nil,nil,nil,nil)
	if err != nil {
		panic(err)
	}
	fmt.Println("result=", result)
}
```
