# 作业

1. 我们在数据库操作的时候，比如 dao 层中当遇到一个 `sql.ErrNoRows` 的时候，是否应该 Wrap 这个 error，抛给上层。为什么，应该怎么做请写出代码？

## 我的理解
跟第三方或者底层交互的时候，我们需要把error通过wrap处理，保留异常信息，然后返回给上一层，但当我们遇到一个具体的异常的时候，比如 `sql.ErrNoRows`,我们不应该往上抛出，因为我们依赖的第三方库是有可能会别替换掉的，如果上层处理了这个问题，当我们的底层换成另外一个第三方库的时候，就会有问题了，所以应该返回普通的error即可。

## 伪代码现实如下

### dao层
```go
type Dao struct{
    db *Db
}

func  NewDao() *Dao {
    return &Dao{ db : &Db{...}}
}

func (this *Dao) QueryList() ([]User, error){
    data,err := this.db.query("select * from users where ...")
    if err != nil{
        if errors.Is(sql.ErrNoRows)  {
            return nil,errors.New("query users errors : "+ sql.ErrNoRows.Error())
        }
        return errors.Wrap(err,"query users errors")
    }
    return data,err
}
```

### service层
```go
type Service {
    dao *Dao
}

func NewService(){
    dao := NewDao()
    return &Service{dao:dao}
}

func (this *Service) QueryList() ([]User,error) {
    return this.dao.QueryList()
}

```

### main函数入口
```go
func main(){
    service := NewService()
    data,err := service.QueryList()
    if err != nil {
        fmt.Printf("error info %v ", err)
        return
    }
    fmt.Printf("user data : %v ", data)
}
```
