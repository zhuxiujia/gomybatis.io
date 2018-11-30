mapper.go 文件案例
# 使用教程,代码文件请移步Github文件夹https://github.com/zhuxiujia/GoMybatis/tree/master/example
[使用xml](#1)


```
go get github.com/zhuxiujia/GoMybatis
go get github.com/go-sql-driver/mysql
```
mapper.go 文件案例
```
//定义mapper文件的接口和结构体，也可以只定义结构体就行
//mapper.go文件 函数必须为2个参数（第一个为自定义结构体参数（属性必须大写），第二个为指针类型的返回数据） error 为返回错误
type ExampleActivityMapperImpl struct {
	SelectAll         func(result *[]Activity) error
	SelectByCondition func(name string, startTime time.Time, endTime time.Time, page int, size int, result *[]Activity) error `mapperParams:"name,startTime,endTime,page,size"`
	UpdateById        func(session *GoMybatis.Session, arg Activity, result *int64) error //只要参数中包含有*GoMybatis.Session的类型，框架默认使用传入的session对象，用于自定义事务
	Insert            func(arg Activity, result *int64) error
	CountByCondition  func(name string, startTime time.Time, endTime time.Time, result *int) error                            `mapperParams:"name,startTime,endTime"`
}
```

xml文件案例:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.spc.platform.dao.activity.ActivityMapper">
    <resultMap id="BaseResultMap" type="com.spc.platform.domain.model.Activity">
        <id column="id" property="id" jdbcType="VARCHAR"/>
        <result column="name" property="name" jdbcType="VARCHAR"/>
        <result column="pc_link" property="pcLink" jdbcType="VARCHAR"/>
        <result column="h5_link" property="h5Link" jdbcType="VARCHAR"/>
        <result column="remark" property="remark" jdbcType="VARCHAR"/>
        <result column="create_time" property="createTime" jdbcType="TIMESTAMP"/>
        <result column="delete_flag" property="deleteFlag" jdbcType="INTEGER"/>
    </resultMap>
    <!--id-->

    <!--List<Activity> selectByCondition(@Param("name") String name,@Param("startTime") Date startTime,@Param("endTime") Date endTime,@Param("index") Integer index,@Param("size") Integer size);-->
    <!-- 后台查询产品 -->
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
    <!--int countByCondition(@Param("name")String name,@Param("startTime") Date startTime, @Param("endTime")Date endTime);-->
    <select id="countByCondition" resultType="java.lang.Integer">
        select count(id) from biz_activity where delete_flag=1
        <if test="name != ''">
            and name like concat('%',#{name},'%')
        </if>
        <if test="startTime != ''">
            and create_time >= #{startTime}
        </if>
        <if test="endTime != ''">
            and create_time &lt;= #{endTime}
        </if>
    </select>
    <!--List<Activity> selectAll();-->
    <select id="selectAll" resultMap="BaseResultMap">
        select * from biz_activity where delete_flag=1 order by create_time desc
    </select>
    <!--Activity selectByUUID(@Param("uuid")String uuid);-->
    <select id="selectByUUID" resultMap="BaseResultMap">
        select * from biz_activity
        where uuid = #{uuid}
        and delete_flag = 1
    </select>
    <select id="selectById"  resultMap="BaseResultMap">
        select * from biz_activity
        where id = #{id}
        and delete_flag = 1
    </select>
    <update id="deleteById" >
        update biz_activity
        set delete_flag = 0
        where id = #{id}
    </update>
    <update id="updateById" >
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
    <insert id="insert" useGeneratedKeys="true" keyProperty="id" keyColumn="id"
            parameterType="com.spc.platform.domain.model.Activity">
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
实际使用Example_test.go
```

import (
	_ "github.com/go-sql-driver/mysql"
	"testing"
	"time"
	"os"
	"fmt"
	"io/ioutil"
	"github.com/zhuxiujia/GoMybatis"
)

//定义mapper文件的接口和结构体，也可以只定义结构体就行
//mapper.go文件 函数必须为2个参数（第一个为自定义结构体参数（属性必须大写），第二个为指针类型的返回数据） error 为返回错误
type ExampleActivityMapperImpl struct {
	SelectAll         func(result *[]Activity) error
	SelectByCondition func(name string, startTime time.Time, endTime time.Time, page int, size int, result *[]Activity) error `mapperParams:"name,startTime,endTime,page,size"`
	UpdateById        func(session *GoMybatis.Session, arg Activity, result *int64) error //只要参数中包含有*GoMybatis.Session的类型，框架默认使用传入的session对象，用于自定义事务
	Insert            func(arg Activity, result *int64) error
	CountByCondition  func(name string, startTime time.Time, endTime time.Time, result *int) error                            `mapperParams:"name,startTime,endTime"`
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
* [代码示例](#1)
```
//本地GoMybatis使用例子
func Test_main(t *testing.T) {
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

* [本地事务](#2)

```
//本地事务使用例子
func Test_local_Transation(t *testing.T) {
	//初始化mapper文件
	exampleActivityMapperImpl := InitMapperByLocalSession()
	//使用事务
	session := *GoMybatis.DefaultSessionFactory.NewSession()
	session.Begin() //开启事务
	var activityBean = Activity{
		Id:   "170",
		Name: "rs168-8",
	}
	var updateNum int64 = 0
	var e = exampleActivityMapperImpl.UpdateById(&session, activityBean, &updateNum)//sessionId 有值则使用已经创建的session，否则新建一个session
	fmt.Println("updateNum=", updateNum)
	if e != nil {
		fmt.Println(e)
	}
	session.Commit() //提交事务
	session.Close()  //关闭事务
}
```

* [远程事务](#3)

```
//远程事务示例，可用于分布式微服务(单数据库，多个微服务)
func Test_Remote_Transation(t *testing.T) {

	//启动GoMybatis独立节点事务服务器，通过msgpackrpc调用(msgpack是一种多语言序列化格式，类似json但是速度快于go默认的gob编码)
	var addr = "127.0.0.1:17235"
	go GoMybatis.ServerTcp(addr, MysqlDriverName, MysqlUri)

	var TransationRMClient = GoMybatis.TransationRMClient{
		RetryTime: 3,
		Addr:      addr,
	}

	//这里是关键，使用原程的transationRMSession（即Transation Resource Manager）替换LocalSession本地的session调用
	var transationRMSession = *GoMybatis.TransationRMSession{}.New("",&TransationRMClient,GoMybatis.Transaction_Status_NO)

	//初始化mapper文件
	var exampleActivityMapperImpl = InitMapperByLocalSession()

	//开启远程事务
	transationRMSession.Begin()
	//使用mapper
	var activityBean = Activity{
		Id:   "170",
		Name: "rs168-11",
	}
	var updateNum int64 = 0
	var err = exampleActivityMapperImpl.UpdateById(&transationRMSession, activityBean, &updateNum)
	if err != nil {
		panic(err)
	}
	//提交远程事务
	transationRMSession.Commit()
	//关闭远程事务
	//transationRMSession.Rollback()
}
```

