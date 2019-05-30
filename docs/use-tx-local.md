> 示例源码https://github.com/zhuxiujia/GoMybatis/tree/master/example

普通事务
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

//定义mapper文件的接口和结构体
//支持包含基本类型的结构体和基本类型(int,string,time.Time,float...且需要指定参数名称`mapperParams:"name"以逗号隔开，且位置要和实际参数相同)
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
    	
    	//新建session方法,  参数：config，传nil为本地session,传值则为远程 remote session
        NewSession func(config *GoMybatis.TransationRMClientConfig) (GoMybatis.Session, error)
        //NewSession      func() (GoMybatis.Session, error)    //NewSession也可以无参数写法
}


//初始化mapper文件和结构体
func InitMapperByLocalSession() ExampleActivityMapperImpl {
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

//本地事务使用例子
func Test_local_Transation(t *testing.T) {
	//初始化mapper文件
	exampleActivityMapperImpl := InitMapperByLocalSession()
	
	//使用事务
    var session, err = exampleActivityMapper.NewSession(nil)
    if err != nil {
    	t.Fatal(err)
    }
    session.Begin() //开启事务
    var activityBean = Activity{
    	Id:   "170",
    	Name: "rs168-8",
    }
    var updateNum, e = exampleActivityMapper.UpdateById(&session, activityBean) //sessionId 有值则使用已经创建的session，否则新建一个session
    fmt.Println("updateNum=", updateNum)
	if e != nil {
		panic(e)
	}
	session.Commit() //提交事务
	session.Close()  //关闭事务
}

```


AOP事务-用户只需关注业务逻辑框架自动处理回滚（嵌套事务/带有传播行为的事务）
```
//定义服务
type TestService struct {
	exampleActivityMapper *ExampleActivityMapper //服务包含一个mapper操作数据库，类似java spring mvc

	//类似拷贝spring MVC的声明式事务注解
	//rollback:回滚操作为error类型(你也可以自定义实现了builtin.error接口的自定义struct，框架会把自定义的error类型转换为string，检查是否包含，是则回滚
	//tx:"" 开启事务，`tx:"PROPAGATION_REQUIRED,error"` 指定传播行为为REQUIRED(默认REQUIRED))
	UpdateName   func(id string, name string) error   `tx:"" rollback:"error"`
	UpdateRemark func(id string, remark string) error `tx:"" rollback:"error"`
}

//初始化服务实现类（Impl）
func init() {
	var testService TestService
	testService = TestService{
		exampleActivityMapper: &exampleActivityMapper,
		UpdateRemark: func(id string, remark string) error {
			var activitys, err = testService.exampleActivityMapper.SelectByIds([]string{id})
			if err != nil {
				panic(err)
			}
			//TODO 此处可能会因为activitys长度为0 导致数组越界 painc,painc 为运行时异常 框架自动回滚事务
			var activity = activitys[0]
			activity.Remark = remark
			updateNum, err := testService.exampleActivityMapper.UpdateTemplete(activity)
			if err != nil {
				panic(err)
			}
			println("UpdateRemark:", updateNum)
			if id == "167" {
				return errors.New("e")
			}
			return nil
		},
		UpdateName: func(id string, name string) error {
			var activitys, err = testService.exampleActivityMapper.SelectByIds([]string{id})
			if err != nil {
				panic(err)
			}
			var activity = activitys[0]
			activity.Name = name
			updateNum, err := testService.exampleActivityMapper.UpdateTemplete(activity)
			if err != nil {
				panic(err)
			}
			println("UpdateName:", updateNum)
			testService.UpdateRemark("172", "p2")
			testService.UpdateRemark("167", "p1")
			return nil
		},
	}
	GoMybatis.AopProxyService(&testService, &engine)
}


//嵌套事务/带有传播行为的事务
func TestTestService(t *testing.T) {
	if MysqlUri == "" || MysqlUri == "*" {
		fmt.Println("no database url define in MysqlConfig.go , you must set the mysql link!")
		return
	}
	var testService = initTestService()

	//go testService.UpdateName("167", "updated name1")
	testService.UpdateName("167", "updated name2")

	time.Sleep(3 * time.Second)
}


```