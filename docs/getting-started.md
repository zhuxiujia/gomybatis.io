## 开始安装
选择数据库驱动类型
```
 Mysql: github.com/go-sql-driver/mysql
 MyMysql: github.com/ziutek/mymysql/godrv
 Postgres: github.com/lib/pq
 Tidb: github.com/pingcap/tidb
 SQLite: github.com/mattn/go-sqlite3
 MsSql: github.com/denisenkom/go-mssqldb
 MsSql: github.com/lunny/godbc
 Oracle: github.com/mattn/go-oci8
 ```
> 1 设置好GoPath,用go get 在命令行工具执行以下命令下载GoMybatis
```
go get github.com/zhuxiujia/GoMybatis
```
> 2 设置好GoPath,用go get 在命令行工具执行以下命令下载数据库驱动
如果要使用Postgres,请go get github.com/lib/pq
如果要使用Tidb,请go get github.com/pingcap/tidb
这里展示Mysql驱动 例如:
```
go get github.com/go-sql-driver/mysql
```
> 3 在你的代码中需要载入你选择的驱动
这里展示Mysql驱动 例如:
```
import (
	"fmt"
	_ "github.com/go-sql-driver/mysql" //Mysql驱动程序，必须使用 _ 下划线导入
	"github.com/zhuxiujia/GoMybatis"
	)
```