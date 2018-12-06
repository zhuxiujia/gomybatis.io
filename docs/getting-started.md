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
设置好GoPath,用go get 在命令行工具执行以下命令下载GoMybatis和数据库驱动
```
go get github.com/zhuxiujia/GoMybatis
go get github.com/go-sql-driver/mysql
```
在你的代码中需要载入你选择的驱动，这里展示Mysql驱动
```
import (
	"fmt"
	_ "github.com/go-sql-driver/mysql" //Mysql驱动程序
	"github.com/zhuxiujia/GoMybatis"
	)
```