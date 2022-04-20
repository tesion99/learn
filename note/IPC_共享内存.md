# 共享内存

共享内存(shared memory)是进程间通信方式的一种, 常见的共享内存方式有4种:

1. 基于传统SYS V的共享内存
2. 基于POSIX mmap文件映射实现共享内存
3. 通过memfd_create()与fd跨进程共享实现共享内存
4. 多媒体 图形领域广泛使用的基于dma-buf的共享内存

共享内存 /dev/shm

### (一) SYS V 共享内存
通过 **ipcs** 查看到的共享内存即是SYS V 共享内存

此种方式需要执行shared memory的key, 通常 key 由 **ftok()** 生成

```c
// 所需头文件
#include <sys/types.h>  // key_t
#include <sys/ipc.h>    // key_t ftok()
#include <sys/shm.h>    // shmXXX()

// 分配共享内存
int shmget(key_t key, size_t size, int shmflg);
// attach到指定对象
void *shmat(int shmid, const void *shmaddr, int shmflg);
// detach共享内存与对象
int shmdt(const void *shmaddr);
// 控制共享内存, 执行删除等相关操作
int shmctl(int shmid, int cmd, struct shmid_ds *buf);
```

### (二) POSIX 共享内存
POSIX通过mmap实现共享内存
```c
#include <sys/mman.h>
#include <sys/stat.h>        /* For mode constants */
#include <fcntl.h>           /* For O_* constants */

// 创建共享内存
int shm_open(const char *name, int oflag, mode_t mode);
// 删除共享内存
int shm_unlink(const char *name);
// 创建映射
void *mmap(void *addr, size_t length, int prot, int flags,
           int fd, off_t offset);
// 解除映射
int munmap(void *addr, size_t length);
```

### (三) memfd_create 共享内存

### (四) dma-buf 共享内存


### 共享内存相关命令

#### 查看信息 - ipcs
ipcs命令用于显示进程间通信(inter-process communication)信息, 默认情况下其展示信息来源有3种

1. 共享内存(shared memory)
2. 信号量(semaphore)
3. 消息队列(message queue)

```shell
ipcs [options]
# 参数说明
-i, --id id      # 查看某个特定ID对应的源信息
# 指定信息源
-m, --shmems     # 查看共享内存信息
-q, --queues     # 查看消息队列信息
-s, --semaphores # 查看信号量信息
# 查看所有
-a, --all
# 格式化
-c, --creator   # 显示创建者
-p, --pid       # 显示创建者PID
-t, --time      # 显示最近操作时间
-u, --summary   # 显示概要信息
# 显示
-b, --bytes     # 以字节显示大小
--human         # 以人类可读形式显示大小
```

#### 创建 - ipcmk
ipcmk命令用于创建IPC资源, 允许创建共享内存 信号量 消息队列

```shell
-M, --shmem size # 指定共享内存大小
-Q, --queue # 创建消息队列
-S, --semaphore number # 创建信号量数组, number指定元素个数

-p, --mode mode # 执行权限
```

#### 删除 - ipcrm
ipcrm命令用于删除IPC资源, 消息队列与信号量会立即删除, 共享内存需要在所有对象都detach之后, 才能删除
```shell
# 用法
ipcrm [options] # 新式, 支持按key与id删除
ipcrm {shm|msg|sem} id... # 老式方式, 仅支持id删除
# 选项
-a, --all [shm] [msg] [sem]
-M, --shmem-key shmkey
-m, --shmem-id shmid
-Q, --queue-key msgkey
-q, --queue-id msgid
-S, --semaphore-key semkey
-s, --semaphore-id semid
```

#### 其他
其他相关函数接口
**ipc生成key**
ftok
