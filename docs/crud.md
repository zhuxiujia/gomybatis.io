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
type ExampleActivityMapperImpl struct {
	SelectByIds       func(ids []string, result *[]Activity) error `mapperParams:"ids"`
	SelectAll         func(result *[]Activity) error
	SelectByCondition func(name string, startTime time.Time, endTime time.Time, page int, size int, result *[]Activity) error `mapperParams:"name,startTime,endTime,page,size"`
	UpdateById        func(session *GoMybatis.Session, arg Activity, result *int64) error                                     //只要参数中包含有*GoMybatis.Session的类型，框架默认使用传入的session对象，用于自定义事务
	Insert            func(arg Activity, result *int64) error
	CountByCondition  func(name string, startTime time.Time, endTime time.Time, result *int) error `mapperParams:"name,startTime,endTime"`
	DeleteById        func(id string, result *int64) error                                         `mapperParams:"id"`
}
//初始化mapper文件和结构体
func InitMapperByLocalSession() ExampleActivityMapperImpl {
	var err error
	//mysql链接格式为         用户名:密码@(数据库链接地址:端口)/数据库名称   例如root:123456@(***.mysql.rds.aliyuncs.com:3306)/test
	engine, err := GoMybatis.Open("mysql", MysqlUri) //此处请按格式填写你的mysql链接，这里用*号代替
	if err != nil {
		panic(err.Error())
	}
	//读取mapper xml文件
	file, err := os.Open("Example_ActivityMapper.xml")
	if err != nil {
		panic(err)
	}
	defer file.Close()

	bytes, _ := ioutil.ReadAll(file)
	var exampleActivityMapperImpl ExampleActivityMapperImpl
	//设置对应的mapper xml文件
	GoMybatis.UseProxyMapperByEngine(&exampleActivityMapperImpl, bytes, engine)
	return exampleActivityMapperImpl
}
```

## 1 插入
定义xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://github.com/zhuxiujia/GoMybatis/blob/master/mybatis-3-mapper.dtd">
<mapper>
   <insert id="insert">
        insert into biz_activity
        <trim prefix="(" suffix=")" suffixOverrides=",">
            <if test="id != ''">id,</if>
            <if test="name != ''">name,</if>
            <if test="pcLink != ''">pc_link,</if>
            <if test="h5Link != ''">h5_link,</if>
            <if test="remark != ''">remark,</if>
            <if test="createTime != ''">create_time,</if>
            <if test="deleteFlag != ''">delete_flag,</if>
        </trim>

        <trim prefix="values (" suffix=")" suffixOverrides=",">
            <if test="id != ''">#{id,jdbcType=VARCHAR},</if>
            <if test="name != ''">#{name,jdbcType=VARCHAR},</if>
            <if test="pcLink != ''">#{pcLink,jdbcType=VARCHAR},</if>
            <if test="h5Link != ''">#{h5Link,jdbcType=VARCHAR},</if>
            <if test="remark != ''">#{remark,jdbcType=VARCHAR},</if>
            <if test="createTime != ''">#{createTime,jdbcType=TIMESTAMP},</if>
            <if test="deleteFlag != ''">#{deleteFlag},</if>
        </trim>
    </insert>
</mapper>
```

```
//插入
func Test_inset(t *testing.T) {
	//初始化mapper文件
	var exampleActivityMapperImpl = InitMapperByLocalSession()
	//使用mapper
	var result int64
	var err = exampleActivityMapperImpl.Insert(Activity{Id: "171", Name: "test_insret", DeleteFlag: 1}, &result)
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
        "https://github.com/zhuxiujia/GoMybatis/blob/master/mybatis-3-mapper.dtd">
<mapper>
      <update id="updateById">
          update biz_activity
          <set>
              <if test="name != ''">name = #{name,jdbcType=VARCHAR},</if>
              <if test="pcLink != ''">pc_link = #{pcLink,jdbcType=VARCHAR},</if>
              <if test="h5Link != ''">h5_link = #{h5Link,jdbcType=VARCHAR},</if>
              <if test="remark != ''">remark = #{remark,jdbcType=VARCHAR},</if>
              <if test="createTime != ''">create_time = #{createTime,jdbcType=TIMESTAMP},</if>
              <if test="deleteFlag != ''">delete_flag = #{deleteFlag},</if>
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
	var updateNum int64 = 0
	var e = exampleActivityMapperImpl.UpdateById(nil, activityBean, &updateNum) //sessionId 有值则使用已经创建的session，否则新建一个session
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
        "https://github.com/zhuxiujia/GoMybatis/blob/master/mybatis-3-mapper.dtd">
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
	var result int64
	var err = exampleActivityMapperImpl.DeleteById("171", &result)
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
        "https://github.com/zhuxiujia/GoMybatis/blob/master/mybatis-3-mapper.dtd">
<mapper>
          <select id="selectByCondition" resultMap="BaseResultMap">
               select * from biz_activity where delete_flag=1
               <if test="name != ''">
                   and name like concat('%',#{name},'%')
               </if>
               <if test="startTime != ''">
                   and create_time >= #{startTime}
               </if>
               <if test="endTime != ''">
                   and create_time &lt;= #{endTime}
               </if>
               order by create_time desc
               <if test="page >= 0 and size != 0">limit #{page}, #{size}</if>
           </select>
</mapper>
```

```
//查询
func Test_select(t *testing.T) {
	//初始化mapper文件
	var exampleActivityMapperImpl = InitMapperByLocalSession()
	//使用mapper
	var result []Activity
	var err = exampleActivityMapperImpl.SelectByCondition("", time.Time{}, time.Time{}, 0, 2000, &result)
	if err != nil {
		panic(err)
	}
	fmt.Println("result=", result)
}
```