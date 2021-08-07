# 数据库模型

C语言项目

#### 重要函数说明
#### db_open(const char *pathname, int oflag, ...)

          打开数据库，根据oflag选择打开方式（读、写、创建），如果创建数据库，则传入第三个参数int mode，即文件权限
          
              
#### db_close()

         释放申请的资源
####  db_fetch( DBHANDLE h , const char * key)

        根据key查找data，并返回指向data的指针
     
  1.查找记录：db_find_and_lock()在函数库内部按给定键查找记录，涉及文件操作所有要上锁。只锁散列表开始处的第一个字节，允许多个进程访问索引文件的不同散列表链，根据传进的形参决定是上写锁还是读锁。
     
  2.查找链表下一个节点的偏移量函数：_db_readptr()函数找到并记录的偏移量，类似链表的下一个节点的指针将下一条索引记录的偏移量传给_db_readidx()函数，更新结构体中一些指针。回到db_find_and_lock函数，比较现在结构体中索引缓冲区数据与键是否相同，相同返回0，不同返回则继续调用db_readptr直到找到或最后没有找到
     
  3.若找到该返回数据地址，通过db_readdat()该数据地址
  
  4.文件解锁
  
  5.返回NULL或数据结构指针
  
  6.文件中要注意到的函数
  
      lseek（）//将文件操作起点设置为从开始数到第offset位字符
      struct iovec{void *iov base//内存起始地址; sizeof_t iov len//当前内存的长度}；
      //通过readv()将db->idxfd指向的数据传入到struct iovec结构中
      _db_hash()//通过哈希算法算出哈希值
      
####  db_delete(DBHANDLE h, const char *key)

        删除一条数据
        
  1.通过_db_find_and_lock()查找该值
  
  2.删除记录
  
    通过db_dodelete()删除对应的数据与索引，在空闲链表中添加删除的索引
    
    初始化数据缓冲区，初始化索引缓冲区
    writew_lock()加写锁
    _db_writedat()修改结构体，写一条数据或者从某一数据开始写（覆盖原有的数据）
    _db_readpt()返回空闲链表指针位置
    _db_writeidx()将索引缓冲区清零
    _db_writeptr()将删除的索引记录的偏移量添加至空闲表指针链上
    _db_writeptr()将当前记录指针赋值为下条记录指针得链表指针
    Un_lock()解开写锁
    
#### db_store(DBHANDLE h, const char *key, const char *data, int flag)
  
        插入或修改data
  该函数第四个参数flag有三种情况
  DB_INSERT(插入一条新纪录)
  
  DB_REPLACE(替换一条记录)
  
  DB_STORE(插入或替换)
  

#### void db_rewind(DBHANDLE h)
1
       将索引指针定位到第一个链表指针上
#### char *db_nextrec(DBHANDLE h, char *key)

       读取该链表指针上的数据，并返回该链表上下一套记录的指针

