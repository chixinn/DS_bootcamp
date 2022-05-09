# OS1-一学习就饿的飞快

Ref：

- https://www.bilibili.com/video/BV1Cm4y1d7Ur?spm_id_from=333.999.0.0

- http://jyywiki.cn/OS/2022/

## 理解操作系统

### OS服务谁？

- 程序=状态机
- 课程：多线程linux程序

### 设计/应用视角：OS提供什么服务？

- **操作系统=服务对象+API(程序视角)**
- 课程：POSIX+部分linux

### 实验/硬件：如何实现OS提供的服务？

- **操作系统=C程序（硬件视角）**
  - 【存疑】完成初始化后就成为interrupt/trap/fault handler
- 课程： xv6,自制迷你OS

>机器永远是对的
>
>没有测试的代码永远是错的
>
>STFW/RTFW

![截屏2022-05-06 15.53.02](https://tva1.sinaimg.cn/large/e6c9d24egy1h1yqrsq8u2j20k0030q37.jpg)

代码长了才自然而然需要代码，在上课需要编更多的程序然后拥有全世界。

### 书

- 教科书：https://pages.cs.wisc.edu/~remzi/OSTEP/

  并发->虚拟化->持久化

- 参考书：CSAPP 

### 少说废话，多写代码

![截屏2022-05-06 16.00.11](https://tva1.sinaimg.cn/large/e6c9d24egy1h1yqrs3q1bj20wq0kcmzn.jpg)

![截屏2022-05-06 16.00.40](https://tva1.sinaimg.cn/large/e6c9d24egy1h1yqs38adwj20xe0kigoo.jpg)

## Prerequisites

- Linux 云主机（嫖学院的）/ macOS ITerm2 ssh登陆
  - 能够显示图形界面：macOS 就通过 XQuartz 
  - 配置![截屏2022-05-06 16.40.04](https://tva1.sinaimg.cn/large/e6c9d24egy1h1ysmsdcjxj208405qjrk.jpg)

###   XQuartz+图形界面

1. 安装XQuartz，安装好后iTerm2就出现如下错误

> The `osx` plugin is deprecated and has been renamed to `macos`. Please update your .zshrc to use the `macos` plugin instead.

解决：

​	修改iTerm 中的`~/.zshrc`，Ref:[Rename ops to macos](https://dev.to/asdflkjh/rename-osx-to-macos-318h)

2. 启动XQuartz，其实brew就能装 但我是下载的pkg安装,强行Alfred启动，这波操作好睿智Orz![截屏2022-05-06 16.38.57](OS1.assets/截屏2022-05-06 16.38.57.png)
3. 在云主机中输入：我的云主机不能用yum，ref上的[macOS使用XQuarts支持图形化显示](https://wsgzao.github.io/post/x11/)一些命令需要我自己改![截屏2022-05-06 16.43.02](https://tva1.sinaimg.cn/large/e6c9d24egy1h1ysj5mpx9j20ka030gls.jpg)
   - `sudo apt-get install xauth`，然后抱错，后面试着输入什么`asmca`也不行，后来看提示了，输入`x11-apps`发现ok了![截屏2022-05-06 16.43.21](https://tva1.sinaimg.cn/large/e6c9d24egy1h1ysj6vbz9j20vk08w3zz.jpg)![截屏2022-05-06 16.44.22](https://tva1.sinaimg.cn/large/e6c9d24egy1h1ysja4n35j20oq02wq38.jpg)

   -	在终端中输出我想现实的app就行了～

# 第一周阅读材料

 ![截屏2022-05-08 15.08.32](https://tva1.sinaimg.cn/large/e6c9d24egy1h210igai9hj20pc07mt9q.jpg)

## CRUX of the problem

1. How OS virtualize resources: virtual CPUS(seemingly infinite )allow concurrency
2. How to build correct concurrent programs.
3. How to manage persistent data: correct/high performance/reliability.

- 冯诺依曼的模型：Von Neumann :
  - Fetch isntruction
  - Decode instruction
  - Execute
- OS ：operate correctly/ in an easy-to-use manner
- Sys call/library/API:run programs, access memory and devices etc. related actions
- `ctrl+C`:UNIX 系统中 terminate the program running in the foreground
- 为了 read memory, one must specify the address able to access the data stored there.
- Instruction is stored in memory ,so we need to fetch.
- 书上2.2的这个例子我跑的时候没有公用一个地址。
	![image-20220508152743886](https://tva1.sinaimg.cn/large/e6c9d24egy1h2112dcgbbj20m40jw0ud.jpg)

 ![截屏2022-05-08 15.25.07](https://tva1.sinaimg.cn/large/e6c9d24egy1h210zpfmh5j20v606wgoh.jpg)

- 如何理解线程：a **function** running within the same memory space as other **functions,**with more than one of them active at a time.

- 2.3 并发的例子

  ```c
  #include <stdio.h>
  #include <stdlib.h>
  #include "common.h"
  #include "common_threads.h"
  volatile int counter = 0;
  int loops;
  void *worker(void *arg) {
      int i;
      for (i = 0; i < loops; i++) {
  	counter++;
      }
      return NULL;
  }
  int main(int argc, char *argv[]) {
      if (argc != 2) {
  	fprintf(stderr, "usage: threads <loops>\n");
  	exit(1);
      }
      loops = atoi(argv[1]);
      pthread_t p1, p2;
      printf("Initial value : %d\n", counter);
      Pthread_create(&p1, NULL, worker, NULL);
      Pthread_create(&p2, NULL, worker, NULL);
      Pthread_join(p1, NULL);
      Pthread_join(p2, NULL);
      printf("Final value   : %d\n", counter);
      return 0;
  }
  ```

  

  ![image-20220508152736483](https://tva1.sinaimg.cn/large/e6c9d24egy1h21128sufwj20l205yaas.jpg)

  【So why is this happening?】

  > 原因来源于how instructions are executed.在这里`counter`变量是shared的。
  >
  > 代码里的counter++是怎么做到的呢？其instruction分为三步：1. Load the value of the counter from memory into a register. 2. Increase the value 3. Store it back
  >
  > 又因为instruction不是atomically，所以错误就发生了。

- Persistent 的理解和volatile相反。
- Users share information in files rather than have their own private and virtualized disk for each application as the OS does for CPU and memory.

## Design Goals for 操作系统

1. Build up abstractions to make the system convenient and easy to use.
2. Provide high performance
3. Minimize overheads of the OS
4. Provide protection between applications -> allow many programs to run at the same time-> isolation
5. Reliability/energy-efficiency/security/mobility,OS on smaller devices.
6. Batch processing ,sit idle 
7. OS 总要有一些额外的syscall权限嘛：![截屏2022-05-08 20.53.48](https://tva1.sinaimg.cn/large/e6c9d24egy1h21am3xip9j20uu04ijt7.jpg)

8. Syscall v.s. procedure:
   1. Sys call transfer control; user mode总有level 的限制
   2. 我懒：![截屏2022-05-08 20.56.07](https://tva1.sinaimg.cn/large/e6c9d24egy1h21am2qrmqj20uu0h2n4q.jpg)

# OS2

## 状态机

### 数字逻辑电路：模拟器

定义好任何一个寄存器再更新回去

````c
#include<stdio.h>
#include<unistd.h>
#define REGS_FOREACH(_)  _(X) _(Y)
#define RUN_LOGIC        X1 = !X && Y; \
                         Y1 = !X && !Y;
#define DEFINE(X)        static int X, X##1;
#define UPDATE(X)        X = X##1;
#define PRINT(X)         printf(#X " = %d; ", X);

int main() {
  REGS_FOREACH(DEFINE);
  while (1) { // clock
    RUN_LOGIC;
    REGS_FOREACH(PRINT);
    REGS_FOREACH(UPDATE);
    putchar('\n'); sleep(1);
  }
}
````

> 在vim中直接编译：`	:!gcc %`
>
> 如何理解这里的宏函数？gcc预编译宏展开。`gcc -E <filename>`

![截屏2022-05-07 15.48.33](https://tva1.sinaimg.cn/large/e6c9d24egy1h1zw1t2ab1j20ok0acdgh.jpg)

### 数码管

可以模拟任何数字系统： ![截屏2022-05-07 16.27.44](https://tva1.sinaimg.cn/large/e6c9d24egy1h1zx6ku9j6j20b201ka9w.jpg)

## 什么是程序:源代码视角-程序是状态机

**程序也是状态机，数字系统本身也是状态机。**

```c
//递归汉诺塔hanoi-r.c
void hanoi(int n, char from, char to, char via) {
  if (n == 1) printf("%c -> %c\n", from, to);
  else {
    hanoi(n - 1, from, via, to);
    hanoi(1,     from, to,  via);
    hanoi(n - 1, via,  to,  from);
  }
  return;
}
```

C程序也是状态机，那么其状态是什么？是堆和栈。

![截屏2022-05-07 16.36.40](https://tva1.sinaimg.cn/large/e6c9d24egy1h20u4cp3rij20ca072t8s.jpg)

`#include`的形式语义就是复制和粘贴。



## 什么是C语言对应的状态机？-- GDB工具

- 先编译`gcc test-hanoi.c`
- 使用GDB工具`gdb a.out`
- 切换到源代码视角`layout src`![截屏2022-05-07 19.58.08](https://tva1.sinaimg.cn/large/e6c9d24egy1h2039xjwyqj20s60lywhb.jpg)

### Gdb Hanoi-r.c

报错，`Unable to find Mach task port for process-id 71884: (os/kern) failure (0x5).
 (please check gdb is codesigned - see taskgated(8))`

解决：很好 ，加了sudo还是卡在这了

![截屏2022-05-08 11.51.15](https://tva1.sinaimg.cn/large/e6c9d24egy1h20ut6nbvzj20za07amz0.jpg)

函数返回？状态机的状态变成啥了？

C语言程序由很多栈桢组成![截屏2022-05-07 20.01.00](https://tva1.sinaimg.cn/large/e6c9d24egy1h203cg6gdfj20ok074t91.jpg)

PC是我们在这看到的黑色的条子。

函数调用：创建一个新的栈桢append到

**非递归的汉诺塔**

````c
typedef struct {
  int pc, n;
  char from, to, via;
} Frame;

#define call(...) ({ *(++top) = (Frame) { .pc = 0, __VA_ARGS__ }; })# 创建一个新的frame,所有的变参数丢进去，pc=0
#define ret()     ({ top--; })#栈顶的那桢pop掉
#define goto(loc) ({ f->pc = (loc) - 1; })

void hanoi(int n, char from, char to, char via) {
  Frame stk[64], *top = stk - 1;
  call(n, from, to, via);
  for (Frame *f; (f = top) >= stk; f->pc++) {
    switch (f->pc) {
      case 0: if (f->n == 1) { printf("%c -> %c\n", f->from, f->to); goto(4); } break;
      case 1: call(f->n - 1, f->from, f->via, f->to);   break;
      case 2: call(       1, f->from, f->to,  f->via);  break;
      case 3: call(f->n - 1, f->via,  f->to,  f->from); break;
      case 4: ret();                                    break; 
      default: assert(0);
    }
  }
}
````

![截屏2022-05-08 11.21.52](https://tva1.sinaimg.cn/large/e6c9d24egy1h20tymubu9j20hg0ji75f.jpg)



## 什么是程序:二进制视角-可执行文件和源代码有什么关系呢-还是状态机

**gdb**可以连接两种，既可以汇编语言也可以看栈桢。`layout asm`

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h20yhjfrc4j20ww0n6ju4.jpg)

当有随机数时，状态机产生了分叉。【non-deterministic】![截屏2022-05-08 14.03.50](https://tva1.sinaimg.cn/large/e6c9d24egy1h20yn45afmj20i60ku0tg.jpg)

无论是deterministic还是non-deterministic的指令，所有指令都只能计算。如果只执行计算的话，程序无法退出。`nemu-trap`指令，程序退出，状态机终止。

但这些指令对OS来说，它无法干任何事，譬如退出自己，譬如printf里输出东西。

## OS上程序特殊的指令`syscall`

以状态机的视角view我们的程序，大部分的指令都是计算型的，即从`(M,R)`中计算得到`(M',R')`

syscall把当前正在运行程序的所有状态`(M,R)`都交给OS，由OS决定`(M',R')`，`(M',R')`就作为返回值了。**即程序调用一条指令请求我们的操作系统将我们的状态机改一改**。由于OS可以访问所有的硬件资源，所以syscall指令实现了程序和操作系统中其他对象的交互。

在操作系统这门课的视角：程序=计算+syscall![截屏2022-05-08 14.14.48](https://tva1.sinaimg.cn/large/e6c9d24egy1h20yyj8fjsj20hc05amx8.jpg)

程序的本质虽然还是计算，比如说我有一个程序显示一个图片（RGB数组)，这个时候我通过计算指令把这个二维数组（图片在计算机中的存储形式）已经算好了，我需要显示这个图片，这个时候我需要syscall来帮我显示。【应用眼中的操作系统也不care这个操作系统是怎么显示这张图片的】

## 如何构造最小的`hello,world`

- `gcc --verbose`可以看出所有链接过程
- `gcc -static`不依赖动态链接库变异的输出`a.out`会很大![截屏2022-05-08 14.24.48](https://tva1.sinaimg.cn/large/e6c9d24egy1h20z8zh8ipj20q6022aa9.jpg)![截屏2022-05-08 14.25.27](https://tva1.sinaimg.cn/large/e6c9d24egy1h20z9leozlj20q6022t8x.jpg)

- `gcc -c`只是compile以后用`objdump`查看![截屏2022-05-08 14.27.49](../../Library/Application Support/typora-user-images/截屏2022-05-08 14.27.49.png)
