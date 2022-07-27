参考：1、[知乎：Redis怎样学？](https://www.zhihu.com/question/61315701)

#### 单线程

Redis说是单线程的，但是在遇到像RDB/AOF操作的时候，往往是先**<font color="red">fork出来一个子进程</font>**来处理，延迟时间仅仅是fork子进程的时间。fork出子进程之后，仍然可以继续处理请求。

---

