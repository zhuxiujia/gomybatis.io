mapper.go 文件案例
# 使用教程,代码文件请移步Github文件夹https://github.com/zhuxiujia/GoMybatis/tree/master/example
```
go get github.com/zhuxiujia/GoMybatis
go get github.com/go-sql-driver/mysql
```
mapper.go 文件案例
```
//定义mapper文件的接口和结构体
type ExampleActivityMapper interface {
	SelectAll(result *[]Activity) error
	SelectByCondition(Name string, StartTime time.Time, EndTime time.Time, Page int, Size int, result *[]Activity) error
	UpdateById(arg Activity, result *int64) error
	Insert(arg Activity, result *int64) error
	CountByCondition(name string, startTime time.Time, endTime time.Time, result *int) error
}
//定义mapper文件的接口和结构体，也可以只定义结构体就行
//mapper.go文件 函数必须为2个参数（第一个为自定义结构体参数（属性必须大写），第二个为指针类型的返回数据） error 为返回错误
type ExampleActivityMapperImpl struct {
	ExampleActivityMapper
	SelectAll         func(result *[]Activity) error
	SelectByCondition func(name string, startTime time.Time, endTime time.Time, page int, size int, result *[]Activity) error `mapperParams:"name,startTime,endTime,page,size"`
	UpdateById        func(arg Activity, result *int64) error
	Insert            func(arg Activity, result *int64) error
	CountByCondition  func(name string, startTime time.Time, endTime time.Time, result *int) error                            `mapperParams:"name,startTime,endTime"`
}
```

xml文件案例:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="ActivityMapperImpl">
    <resultMap id="BaseResultMap" type="model.Activity">
        <id column="id" property="id" jdbcType="VARCHAR"/>
        <result column="name" property="name" jdbcType="VARCHAR"/>
        <result column="pc_link" property="pcLink" jdbcType="VARCHAR"/>
        <result column="h5_link" property="h5Link" jdbcType="VARCHAR"/>
        <result column="remark" property="remark" jdbcType="VARCHAR"/>
        <result column="create_time" property="createTime" jdbcType="TIMESTAMP"/>
        <result column="delete_flag" property="deleteFlag" jdbcType="INTEGER"/>
    </resultMap>
    <!--SelectAll(result *[]Activity)error-->
    <select id="selectAll" resultMap="BaseResultMap">
        select * from biz_activity where delete_flag=1 order by create_time desc
    </select>
</mapper>
```
实际使用mapper
```
import (
	_ "github.com/go-sql-driver/mysql"
	"testing"
	"time"
	"github.com/zhuxiujia/GoMybatis/lib/github.com/go-xorm/xorm"
	"os"
	"fmt"
	"io/ioutil"
	"github.com/zhuxiujia/GoMybatis"
)
func main() {
  var err error
  	//mysql链接格式为         用户名:密码@(数据库链接地址:端口)/数据库名称   例如root:123456@(***.mysql.rds.aliyuncs.com:3306)/test
  	engine, err := xorm.NewEngine("mysql", "*?charset=utf8&parseTime=True&loc=Local") //此处请按格式填写你的mysql链接，这里用*号代替
  	if err != nil {
  		panic(err.Error())
  	}

  	file, err := os.Open("Example_ActivityMapper.xml")
  	if err != nil {
  		panic(err)
  	}
  	defer file.Close()

  	bytes, _ := ioutil.ReadAll(file)
  	var exampleActivityMapperImpl ExampleActivityMapperImpl
  	//设置对应的mapper xml文件
  	GoMybatis.UseProxyMapper(&exampleActivityMapperImpl, bytes, engine)

  	//使用mapper
  	var result []Activity
  	exampleActivityMapperImpl.SelectAll(&result)

  	fmt.Println(result)
}
```

