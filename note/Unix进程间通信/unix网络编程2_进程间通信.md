# UNIX网络编程2 -- 进程间通信

## 第一章 简介

### 1.3 IPC对象的持续性

- 随进程持续的(process-persistent) 

  IPC对象一直存在到打开着该对象的最后一个进程关闭该对象为止

- 随内核持续的(kernel-persistent)

  IPC对象一直存在到内核重新自举或显式删除该对象为止

- 随文件系统持续的(filesystem-persistent)

  IPC对象一直存在到显式删除该对象为止。即使内核重新自举，该对象还是保持其值。



### 1.4 名字空间

当多个进程使用某种IPC对象来彼此通信时, 该IPC对象必须有一个某种形式的**名字(name)**或**标识符(identifier)**

对于一个给定的IPC类型, 其可能的名字的集合称为它的**名字空间(name space)**

| IPC类型                                                      | 名字空间                                                     | 标识符                                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 管道<br/>FIFO                                                | (没有名字)<br/>路径名                                        | 描述符<br/>描述符                                            |
| Posix互斥锁<br/>Posix条件变量<br/>Posix读写锁<br/>fcntl记录上锁 | (没有名字)<br/>(没有名字)<br/>(没有名字)<br/>路径名          | pthread_mutex_t<br/>pthread_cond_t<br/>pthread_rwlock_t<br/>描述符 |
| Posix消息队列<br/>Posix有名信号量<br/>Posix基于内存的信号量<br/>Posix共享内存区 | Posix IPC名字<br/>Posix IPC名字<br/>(没有名字)<br/>Posix IPC名字 | mqd_t<br/>sem_t<br/>sem_t<br/>描述符                         |
| System V消息队列<br/>System V信号量<br/>System V共享内存区   | key_t<br/>key_t<br/>key_t                                    | System V IPC标识符<br/>System V IPC标识符<br/>System V IPC标识符 |
| 门<br/>Sun RPC                                               | 路径名<br/>程序/版本                                         | 描述符<br/>RPC句柄                                           |
| TCP套接字<br/>UDP套接字<br/>Unix域套接字                     | IP地址+TCP端口<br/>IP地址+TCP端口<br/>路径名                 | 描述符<br/>描述符<br/>描述符                                 |



## 第二章 Posix IPC

Posix IPC包括

- Posix消息队列
- Posix信号量
- Posix共享内存

|                     | 消息队列                             | 信号量                                                       | 共享内存                |
| ------------------- | ------------------------------------ | ------------------------------------------------------------ | ----------------------- |
| 头文件              | <mqueue.h>                           | <semaphore.h>                                                | <sys/mman.h>            |
| 创建 打开或删除函数 | mq_open<br/>mq_close<br/>mq_unlink   | sem_open<br/>sem_close<br/>sem_unlink<br/>----------------<br/>sem_init<br/>sem_destroy | shm_open<br/>shm_unlink |
| 控制IPC操作函数     | mq_getattr<br/>mq_setattr            |                                                              | ftruncate<br/>fstat     |
| IPC操作函数         | mq_send<br/>mq_receive<br/>mq_notify | sem_wait<br/>sem_trywait<br/>sem_post<br/>sem_getvalue       | mmap<br/>munmap         |



### 2.3 创建与打开IPC通道

打开或创建Posix IPC对象所用的各种flag值

| 说明                          | mq_open                          | sem_open           | shm_open            |
| ----------------------------- | -------------------------------- | ------------------ | ------------------- |
| 只读<br/>只写<br/>读写        | O_RDONLY<br/>O_WRONLY<br/>O_RDWR |                    | O_RDONLY<br/>O_RDWR |
| 若不存在则创建<br/>排他性创建 | O_CREAT<br/>O_EXCL               | O_CREAT<br/>O_EXCL | O_CREAT<br/>O_EXCL  |
| 非阻塞模式<br/>若已存在则截短 | O_NONBLOCK                       |                    | O_TRUNC             |



创建或打开一个Posix IPC对象的逻辑

| oflag标志       | 对象不存在           | 对象已存在           |
| --------------- | -------------------- | -------------------- |
| 无特殊标志      | 出错, errno = ENOENT | 成功, 引用已存在对象 |
| O_CREAT         | 成功, 创建新对象     | 成功, 引用已存在对象 |
| O_CREAT\|O_EXCL | 成功, 创建新对象     | 出错, errno = EEXIST |



## 第三章 System V IPC

System V IPC:

- System V消息队列
- System V信号量
- System V共享内存区

System V IPC函数汇总

|               | 消息队列          | 信号量      | 共享内存        |
| ------------- | ----------------- | ----------- | --------------- |
| 头文件        | <sys/msg.h>       | <sys/sem.h> | <sys/shm.h>     |
| 创建或打开IPC | msgget            | semget      | shmget          |
| 控制IPC操作   | msgctl            | semctl      | shmctl          |
| IPC操作       | msgsnd<br/>msgrcv | semop       | shmat<br/>shmdt |



### 3.2 key_t键和ftok函数

三种类型的System V IPC使用key_t值作为其标识符名字; key_t被定义为一个**整数**, 包含于头文件**<sys/types.h>**

```c
#include <sys/types.h>
#include <sys/ipc.h>

key_t ftok(const char*pathname, int id);
```

ftok函数将pathname导出的信息与id的低8位组合成一个整数的IPC键

ftok的典型实现调用**stat**函数，然后组合以下三个值:

1. pathname所在的文件系统的信息(stat结构的st_dev成员)
2. 该文件在本文件系统内的索引节点号(stat结构的st_ino成员)
3. id的低序8位(不能为0)

Note: 

1. 不能保证两个不同的路径名与同一个id的组合产生不同的键
2. 索引节点绝不会是0，因此大多数实现把**IPC_PRIVATE**定义为0



### 3.3 ipc_perm结构

内核给每个IPC对象维护一个信息结构

```c
struct ipc_perm {
    uid_t 	uid; 	// 所有者uid
    gid_t 	gid; 	// 所有者group id
    uid_t 	cuid;	// 创建者uid
    gid_t 	cgid; 	// 创建者group id
    mode_t 	mode; 	// 读写权限
    ulong_t seq; 	// 槽序列号
    key_t 	key; 	// IPC key
}
```



### 3.4 创建和打开IPC通道

| oflag参数           | key不存在            | key存在              |
| ------------------- | -------------------- | -------------------- |
| 无特殊标志          | 出错, errno = ENOENT | 成功, 引用已存在对象 |
| IPC_CREAT           | 成功, 创建新对象     | 成功, 引用已存在对象 |
| IPC_CREAT\|IPC_EXCL | 成功, 创建新对象     | 出错, errno = EEXIST |



### 3.7 ipcs 和 ipcrm程序

ipcs 输出有关System V IPC特性的各种信息

ipcrm 删除一个System V消息队列、信号量或共享内存区