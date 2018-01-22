# mysql相关操作：

## 参考：

http://go-database-sql.org/index.html

https://github.com/go-sql-driver/mysql

https://www.cnblogs.com/tsiangleo/p/4483657.html

https://golang.org/pkg/database/sql/

## 准备：

```go
go get github.com/go-sql-driver/mysql

import "database/sql"
import _ "github.com/go-sql-driver/mysql"
```

## 连接：

```go
package main

import "database/sql"
import (
	"fmt"
	_ "github.com/go-sql-driver/mysql"
)

func main() {
	db, err := sql.Open("mysql", "root:123456@tcp(127.0.0.1:3306)/d_ec_crm?charset=utf8mb4")

	if err != nil {
		panic(err.Error())
	}
	//保证关闭
	defer db.Close()

	err = db.Ping()
	if err != nil {
		panic(err.Error())
	}
	fmt.Println("链接DB成功")

}

```

```
user@unix(/path/to/socket)/dbname
root:pw@unix(/tmp/mysql.sock)/myDatabase?loc=Local
user:password@tcp(localhost:5555)/dbname?tls=skip-verify&autocommit=true
user:password@/dbname?sql_mode=TRADITIONAL
user:password@tcp/dbname?charset=utf8mb4,utf8&sys_var=esc@ped 
```

## 直接操作：

主要涉及两个方法：Query,QueryRow，Exec执行

```go
rows, err := db.Query("SELECT f_crm_id,f_tmp_id,f_name,f_company,f_contact_time FROM d_ec_crm.t_eccrm_detail WHERE f_crm_id = ?", "230740537")
	if err != nil {
		log.Fatal(err)
	}

	for rows.Next() {
		var crmid int64
		var tmpid sql.NullInt64
		var name string
		var company string
		var contact_time string

		if err := rows.Scan(&crmid, &tmpid, &name, &company, &contact_time); err != nil {
			log.Fatal(err)
		}

		fmt.Println(crmid, tmpid, name, company, contact_time) //230740537 {0 false} 晏子阿斯顿333  2015-06-11 16:03:09

	}

	if err := rows.Err(); err != nil {
		log.Fatal(err)
	}


row := db.QueryRow("SELECT f_crm_id,f_name FROM d_ec_crm.t_eccrm_detail WHERE f_crm_id = ?", "230740537")
	var crmid int
	var crmname string
	err = row.Scan(&crmid, &crmname) //必须和获取的参数个数匹配
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(crmid, crmname) //230740537 晏子阿斯顿333

//直接插入或更新
result, err := db.Exec(
    "INSERT INTO users (name, age) VALUES (?, ?)",
    "gopher",
    27,
)
```

## 查询：

- rows.Close()必须确保关闭，否则会耗尽资源。
- rows.Scan()的时候要注意检查错误，防止异常数据。
- 不要再loop里面是有defer.延迟等操作，这样会占用连接资源不释放。
- rows.Scan()可以完成自动类型转换工作Scan error on column index 2: converting driver.Value type []uint8 ("") to a int64: invalid syntax
  exit status 1 但是感觉不靠谱，如果出现不能转换的就直接报错了，如果可以转换的还是可以正常的。

```Go
	//自动类型转换
	row1s, err1 := db.Query(`
SELECT f_crm_id,f_name,f_mobile FROM d_ec_crm.t_eccrm_detail
 WHERE f_mobile > '0' ORDER BY f_crm_id DESC LIMIT 10
`)
	if err1 != nil {
		log.Fatal(err1)
	}
	//注意关闭
	defer row1s.Close()
	var crmid1 int64
	var crmname1 string
	var crmmobile1 int64 //数据库中为字符串
	for row1s.Next() {
		if err := row1s.Scan(&crmid1, &crmname1, &crmmobile1); err != nil {
			log.Fatal(err)
		}
		fmt.Println(crmid1, crmname1, crmmobile1)
	}
	//检查是否有错误 防止循环中有错误抛出，需要了解
	err = row1s.Err()
	if err != nil {
		log.Fatal(err)
	}
```





```go
	stmt, err := db.Prepare(`
	SELECT f_id,f_crm_id,f_time,f_content FROM d_ec_crm.t_crm_visit
	WHERE f_id IN (?) LIMIT 100
	`)
	if err != nil {
		log.Fatal(err)
	}
	//关闭
	defer stmt.Close()
	ids := []string{"1", "81006116", "81006117"}
	rows, err := stmt.Query(strings.Join(ids, ","))
	fmt.Println(strings.Join(ids, ","))
	if err != nil {
		log.Fatal(err)
	}

	//确认可以关闭
	defer rows.Close()
	type visit struct {
		Id      int64
		Crmid   int64
		Time    int64
		Content string
	}
	Myvisit := visit{}
	for rows.Next() {
		err := rows.Scan(&Myvisit.Id, &Myvisit.Crmid, &Myvisit.Time, &Myvisit.Content)
		if err != nil {
			log.Fatal(err)
		}
		fmt.Println(Myvisit)

	}
	//确认循环没有问题
	if err = rows.Err(); err != nil {
		log.Fatal(err)
	}
```

发现真实执行的是1,2,3,4。https://stackoverflow.com/questions/45351644/golang-slice-in-mysql-query-with-where-in-clause

```mysql
2018-01-04T08:52:49.903327Z	  262 Query	SELECT @@max_allowed_packet
2018-01-04T08:52:49.903516Z	  262 Query	SET NAMES utf8mb4
2018-01-04T08:52:49.903832Z	  262 Prepare	SELECT f_crm_id from d_ec_crm.t_crm_lock where f_crm_id IN (?)
2018-01-04T08:52:49.903929Z	  262 Execute	SELECT f_crm_id from d_ec_crm.t_crm_lock where f_crm_id IN ('1,2,3,4')
2018-01-04T08:52:49.904312Z	  262 Close stmt
2018-01-04T08:52:49.904381Z	  262 Quit
```

理解为mysql底层预处理机制就是如此。解决方案如下

```go
//方案1 不用预处理
fmt.Sprintf("SELECT * from table2 where id in (%s)", asID)


//方案2 
ids := []string{"1", "2", "3", "4"}
	args := make([]interface{}, len(ids))
	for i, id := range ids {
		args[i] = id
	}
	//多个参数就多个占位符?
	sql := "SELECT f_crm_id from d_ec_crm.t_crm_lock where f_crm_id IN (?" + strings.Repeat(",?", len(args)-1) + ")"
	rows, err := db.Query(sql, args...)
	defer rows.Close()
	if err != nil {
		log.Fatal(err)
	}
	var id int
	for rows.Next() {
		err := rows.Scan(&id)
		if err != nil {
			log.Fatal(err)
		}
		fmt.Println("CRMID:", id)
	}
	if err := rows.Err(); err != nil {
		log.Fatal(err)
	}
```



### 单条查询

```go
var name string
err = db.QueryRow("select name from users where id = ?", 1).Scan(&name)
if err != nil {
	log.Fatal(err)
}
fmt.Println(name)

stmt, err := db.Prepare("select name from users where id = ?")
if err != nil {
	log.Fatal(err)
}
var name string
err = stmt.QueryRow(1).Scan(&name)
if err != nil {
	log.Fatal(err)
}
fmt.Println(name)
```

### 不确定查询获取的字段的时候

```go
rows, err := db.Query("SELECT * FROM d_ec_crm.t_crm_lock WHERE f_crm_id < ?", 5)
	if err != nil {
		log.Fatal(err)
	}
	defer rows.Close()
	//当不清楚有多少列的时候
	cols, err := rows.Columns() //获取列
	if err != nil {
		log.Fatal(err)
	}
	count := len(cols)
	vals := make([]string, count)
	ptr := make([]interface{}, count)
	for i := 0; i < count; i++ {
		ptr[i] = &vals[i] //很重要 不然scan不通过
	}

	var data []map[string]interface{}
	entry := make(map[string]interface{}, count)
	for rows.Next() {
		err = rows.Scan(ptr...)
		if err != nil {
			log.Fatal(err)
		}
		for i, col := range cols {
			val := vals[i] //很重要不然val得到的是一个结果
			entry[col] = val
		}
		//追加
		data = append(data, entry)
	}
	//这样就得到一个类似php的数组
	fmt.Println(data)
	if err = rows.Err(); err != nil {
		log.Fatal(err)
	}
```



## 插入更新删除：

```go
stmt, err := db.Prepare(`
INSERT INTO d_ec_crm.t_crm_lock(f_crm_id,f_corp_id,f_task_id,f_type,f_do_userid,f_time)
VALUES (?,?,?,?,?,?)
`)
	if err != nil {
		log.Fatal(err)
	}
	//注意时区
	res, err := stmt.Exec(1, 2, 2, 1, 1, time.Now())
	if err != nil {
		log.Fatal(err)
	}
	//获取影响行数
	rowCnt, err := res.RowsAffected()
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("影响行数：", rowCnt)

	//获取自增ID，没有返回0
	lastId, err := res.LastInsertId()
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("自增ID:", lastId)
```

```go
2018/01/03 17:26:09 Error 1062: Duplicate entry '1' for key 'PRIMARY'
exit status 1
```

同理：**删除和修改**。

## 事务：

- 当prepare的时候就会获取一个连接。
- stmt会记住当前使用的是哪个连接。
- 当执行stmt的时候，如果连接不可用，会重新获取一个，**重新prepare**。这又可能导致高并发下，会导致大量连接繁忙，从而创建大量准备好的语句。
- db和db.Begin创建的连接不是同一个，事务有自己的一套。

```go
	tx, err := db.Begin()
	if err != nil {
		log.Fatal(err)
	}
	defer tx.Rollback() //回滚

	stmt, err := tx.Prepare(`
	INSERT INTO d_ec_crm.t_crm_lock(f_crm_id,f_corp_id,f_task_id,f_type,f_do_userid,f_time)
VALUES (?,?,?,?,?,?)
`)
	if err != nil {
		log.Fatal(err)
	}
	//defer stmt.Close() 危险
	for i := 10; i < 20; i++ {
		_, err = stmt.Exec(i, i, 1, 1, 1, time.Now())
		if err != nil {
			//log.Fatal(err)
		}
		if i == 12 {
			log.Fatal("强制错误")//exit了
		}
	}
	err = tx.Commit()
	if err != nil {
		log.Fatal(err)
	}
	stmt.Close()
```

## 连接池：

**db.SetMaxOpenConns(n int)** 设置连接池中**最多保存打开多少个数据库连接**。注意，它包括在使用的和空闲的。如果某个方法调用需要一个连接，但连接池中没有空闲的可用，且打开的连接数达到了该方法设置的最大值，该方法调用将堵塞。默认限制是0，表示最大打开数没有限制。

**db.SetMaxIdleConns(n int)** 设置**连接池中能够保持的最大空闲连接的数量**。**默认值是2**
上面的两个设置，可以用程序实际测试。比如通过下面的代码，可以验证 MaxIdleConns 是 2：

自白点就是：一个是现在真实的连接数量，一个限制的是连接池的数量。

show processlist

```go
db, err := sql.Open("mysql", "root:123456@tcp(127.0.0.1:3306)/d_ec_crm?charset=utf8mb4")

	if err != nil {
		panic(err.Error())
	}
	db.SetMaxOpenConns(10)
	//show processlist
	db.SetMaxIdleConns(3) //以这个为主
	//保证关闭
	defer db.Close()

	for i := 0; i < 10; i++ {
		go func() {
			db.Ping()
		}()
	}
	time.Sleep(20 * time.Second)
	fmt.Println("链接DB成功")
```

结论：当SetMaxOpenConns比SetMaxIdleConns小的时候则为 会现在为SetMaxOpenConns。当SetMaxIdleConns的时候限制为SetMaxIdleConns。即谁小限制为谁。

## 注意事项：

- 如果获取的数据有可能是null必须定义为sql.NullString sql.NullInt64。

```go
SELECT first_name, COALESCE(age, 0) FROM person;//
SELECT first_name, IFNULL(age, 0) FROM person;//
```



- 获取的参数必须和scan的参数个数匹配。
- 获取数据不存在的时候会报错sql: no rows in result set exit status 1 只针对QueryRow()

```go
  switch {
    case err == sql.ErrNoRows://数据为空判断
    case err != nil:
        fmt.Println(err)
    }
//标准
var name string
err = db.QueryRow("select name from users where id = ?", 1).Scan(&name)
if err != nil {
	if err == sql.ErrNoRows {
		// there were no rows, but otherwise no error occurred
	} else {
		log.Fatal(err)
	}
}
fmt.Println(name)
```

- 有自己的连接池不用担心，连接问题，自动回重试10次，但是如果语句在mysql中被kill，也会触发这个机制重试10次。
- http://go-database-sql.org/surprises.html 重点阅读
- 事务的连接中尽量保持语句的独立性，否则就会掉坑里

```go
rows, err := db.Query("select * from tbl1") // Uses connection 1
for rows.Next() {
	err = rows.Scan(&myvariable)
	// The following line will NOT use connection 1, which is already in-use
  //出现两个连接，如果处理不当则会资源耗尽
	db.Query("select * from tbl2 where id = ?", myvariable)
}

tx, err := db.Begin()
rows, err := tx.Query("select * from tbl1") // Uses tx's connection
for rows.Next() {
	err = rows.Scan(&myvariable)
	// ERROR! tx's connection is already busy!
  //事务的连接拥有独立性，
	tx.Query("select * from tbl2 where id = ?", myvariable)
}
```

- 假如直接用time.Now().Local()写入到DB，会遇到问题，需要调整mysql的连接时区https://github.com/go-sql-driver/mysql/blob/master/dsn.go#L39 Loc为time.Local 否则默认为time.UTC ?charset=utf8mb4&loc="+**time.Local.String**()
- 如何写入null?

```go
func NewNullString(s string) sql.NullString {
    if len(s) == 0 {
        return sql.NullString{}
    }
    return sql.NullString{
         String: s,
         Valid: true,
    }
}
db.Exec(`
  insert into
      users first_name, last_name, email
      values (?,?,?)`,
  firstName,
  lastName,
  NewNullString(email),
)
//即写入null要写入对应类型的NullXX类型
```



## 深入理解：

1）sql.Open("mysql", "username:pwd@/databasename")

功能：返回一个DB对象，DB对象对于多个goroutines并发使用是安全的，DB对象内部封装了连接池。

实现：**open函数并没有创建连接**，它只是验证参数是否合法。然后开启一个单独goroutines去监听是否需要建立新的连接，当有请求建立新连接时就创建新连接。

注意：open函数应该被调用一次，通常是没必要close的。



（2）DB.Exec()

功能：执行不返回行（row）的查询，比如INSERT，UPDATE，DELETE

实现：DB交给内部的exec方法负责查询。exec会首先调用DB内部的conn方法从连接池里面获得一个连接。然后检查内部的driver.Conn实现了Execer接口没有，如果实现了该接口，会调用Execer接口的Exec方法执行查询；否则调用Conn接口的Prepare方法负责查询。



（3）DB.Query()

功能：用于检索（retrieval），比如SELECT

实现：DB交给内部的query方法负责查询。query首先调用DB内部的conn方法从连接池里面获得一个连接，然后调用内部的queryConn方法负责查询。

（4）DB.QueryRow()

功能：用于返回单行的查询

实现：转交给DB.Query()查询

 （5）db.Prepare()

功能：返回一个Stmt。Stmt对象可以执行Exec,Query,QueryRow等操作。

实现：DB交给内部的prepare方法负责查询。prepare首先调用DB内部的conn方法从连接池里面获得一个连接，然后调用driverConn的prepareLocked方法负责查询。

Stmt相关方法：st.Exec()，st.Query()，st.QueryRow()，st.Close()



（6）db.Begin()

功能：开启事务，返回Tx对象。调用该方法后，这个TX就和指定的连接绑定在一起了。一旦事务提交或者回滚，该事务绑定的连接就还给DB的连接池。

实现：DB交给内部的begin方法负责处理。begin首先调用DB内部的conn方法从连接池里面获得一个连接，然后调用Conn接口的Begin方法获得一个TX。

TX相关方法：

//内部执行流程和上面那些差不多，只是没有先去获取连接的一步，因为这些操作是和TX关联的，Tx建立的时候就和一个连接绑定了，所以这些操作内部共用一个TX内部的连接。tx.Exec() tx.Query()，tx.QueryRow()，tx.Prepare()，tx.Commit()，tx.Rollback()，tx.Stmt()

//用于将一个已存在的statement和tx绑定在一起。一个statement可以不和tx关联，比如db.Prepare()返回的statement就没有和TX关联。