# SQL mapper framework for Golang
[![Build Status](https://travis-ci.com/zhuxiujia/GoMybatis.svg?branch=master)](https://travis-ci.com/zhuxiujia/GoMybatis)

![Image text](https://zhuxiujia.github.io/gomybatis.io/assets/vuetify.png)
# 官方网站/文档
https://zhuxiujia.github.io/gomybatis.io/info.html
# 优势
-**多种数据库支持**,理论上支持mysql和pg的协议以及支持(标准库"database/sql")都支持<br>
-**高并发**，假设数据库响应时间为0，在6核16Gpc上可框架可以压出 246982Tps,耗时仅仅0.4s<br>
-**支持事务**，session灵活插拔，兼容过渡期微服务<br>
-**动态SQL**，在xml中可灵活运用if判断，foreach遍历数组，resultMap,bind等等java框架Mybatis包含的实用功能<br>
-**无需生成.go等等中间代码**，xml读取后可直接写入到自定义的Struct,Func属性中调用函数<br>
### 已支持绝大部分标签
### 已支持本地和远程事务,方便处于 单数据库(Mysql,postgresql)-分布式数据库（TiDB,cockroachdb...）过渡期间的微服务
![Image text](https://zhuxiujia.github.io/gomybatis.io/assets/tx.png)


数据库驱动列表
```
 Mysql: github.com/go-sql-driver/mysql
 MyMysql: github.com/ziutek/mymysql/godrv
 Postgres: github.com/lib/pq
 Tidb: github.com/pingcap/tidb
 SQLite: github.com/mattn/go-sqlite3
 MsSql: github.com/denisenkom/go-mssqldb
 MsSql: github.com/lunny/godbc
 Oracle: github.com/mattn/go-oci8
 CockroachDB(Postgres): github.com/lib/pq
 ```
 
## 使用教程

> 示例源码https://github.com/zhuxiujia/GoMybatis/tree/master/example

设置好GoPath,用go get 命令下载GoMybatis和对应的数据库驱动
```
go get github.com/zhuxiujia/GoMybatis
go get github.com/go-sql-driver/mysql
```
实际使用mapper
```
import (
	_ "github.com/go-sql-driver/mysql"
	"fmt"
	"github.com/zhuxiujia/GoMybatis"
)
//定义xml内容，建议以ActivityMapper.xml文件存于项目目录中,这样可以享受GoLand等ide渲染和智能提示。这里string直接定义
//生产环境可以使用statikFS把xml文件打包进程序里
var xmlBytes = []byte(`
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "https://raw.githubusercontent.com/zhuxiujia/GoMybatis/master/mybatis-3-mapper.dtd">
<mapper namespace="ActivityMapperImpl">
    <resultMap id="BaseResultMap" type="example.Activity">
        <id column="id" property="id" langType="string"/>
        <result column="name" property="name" langType="string"/>
        <result column="pc_link" property="pcLink" langType="string"/>
        <result column="h5_link" property="h5Link" langType="string"/>
        <result column="remark" property="remark" langType="string"/>
        <result column="create_time" property="createTime" langType="time.Time"/>
        <result column="delete_flag" property="deleteFlag" langType="int"/>
    </resultMap>
    <!--SelectAll(result *[]Activity)error-->
    <select id="selectAll" resultMap="BaseResultMap">
        select * from biz_activity where delete_flag=1 order by create_time desc
    </select>
</mapper>
`)


//定义mapper文件的接口和结构体
// 支持基本类型(int,string,time.Time,float...且需要指定参数名称`mapperParams:"name"以逗号隔开，且位置要和实际参数相同)
//自定义结构体参数（属性必须大写）
//参数中除了session指针外，为参数数据
// 函数return必须有error 为返回错误信息
type ExampleActivityMapperImpl struct {
	SelectByIds       func(ids []string) ([]Activity, error)                                                            `mapperParams:"ids"`
	SelectAll         func() ([]Activity, error)
	SelectByCondition func(name *string, startTime *time.Time, endTime *time.Time, page *int, size *int) ([]Activity, error) `mapperParams:"name,startTime,endTime,page,size"`
	UpdateById        func(session *GoMybatis.Session, arg Activity) (int64, error) //参数中包含有*GoMybatis.Session的类型，用于自定义事务
	Insert            func(arg Activity) (int64, error)
	CountByCondition  func(name string, startTime time.Time, endTime time.Time) (int, error)                            `mapperParams:"name,startTime,endTime"`
	DeleteById        func(id string) (int64, error)                                                                    `mapperParams:"id"`
	Choose            func(deleteFlag int) ([]Activity, error)                                                          `mapperParams:"deleteFlag"`
}

func main() {
	var err error
	//mysql链接格式为         用户名:密码@(数据库链接地址:端口)/数据库名称   例如root:123456@(***.mysql.rds.aliyuncs.com:3306)/test
	engine, err := GoMybatis.Open("mysql", "*?charset=utf8&parseTime=True&loc=Local") //此处请按格式填写你的mysql链接，这里用*号代替
	if err != nil {
		panic(err.Error())
	}
	var exampleActivityMapperImpl ExampleActivityMapperImpl
	//挂载mapper xml内容
	GoMybatis.WriteMapperPtrByEngine(&exampleActivityMapperImpl, xmlBytes, engine, true)

	//使用mapper
    result,error:=exampleActivityMapperImpl.SelectAll(&result)
    fmt.Println(error)
	fmt.Println(result)
}
```

