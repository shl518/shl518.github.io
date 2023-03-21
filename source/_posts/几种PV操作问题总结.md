---
title: 几种PV操作问题总结
date: 2022-05-09 15:50:25
tags: 操作系统
---

## OS Homework 4

--------------------

### 1.1 读者写者问题（写者优先）

#### 实现思路

- 首先读者写者问题肯定需要一个`mutex`信号量保护缓冲区。

- 用read变量实现写优先，初值是1：当至少有一个写进程准备访问数据区时，用于禁止所有的读进程。
- `readcount,writecount`两个int变量记录读者写者数量，用于改变`read`信号量，相应的有`rcmutex，wcmutex`两个信号量保护他们读写的互斥。

```c#
//实现读者写者问题（写者优先）
int readcount = 0 ;//记录当前读者数量 
int writecount = 0; //记录写者数量
semaphore rcmutex = 1; //保护更新readcount变量时的互斥 
semaphore wcmutex = 1;//保护更新writecount变量的互斥 
semaphore read = 1;//信号量read，初值是1：当至少有一个写进程准备访问数据区时，用于禁止所有的读进程。
semaphore filemutex = 1;//临界资源互斥 

//写者 
writer() {
	while(1) {
		P(wcmutex);//申请writecount资源 
		if(!writecount) //说明这是第一个写者，这时申请read用于阻塞后续读者 
			P(read);//阻塞读者
		writecount++;
		V(wcmutex) //释放writecount
            
		//进入临界区
		P(filemutex)
		//临界区代码
		writing();
		V(filemmutex)
		//出临界区
            
		P(wcmutex);
		writecount--;//该写者操作完毕，计数器减一
		if(!writecount)//如果是当前等待队列最后一个写者，释放read锁，不再阻塞读者
			V(read);
		V(wcmutex) 
	}
}

//读者
reader() {
	while(1) {
		P(read);//如果有写者在等待，则轮不到读者，读者一直被阻塞
		P(rcmutex);
		if(!readcount) //只有第一个读者申请临界资源锁，因为要实现共享读，后续读者无需申请，直接进入临界资源读取
			P(filemutex);
		readcount++;
		V(rcmutex);
		V(read);//这是一个理解难点，我们需要及时释放read锁，如果这时来了一个写者就能够继续及时阻塞读者，实现写者优先
		//临界代码 
		reading();
		
		P(rcmutex);
		readcount--;
		if(!readcount)//如果读者队列已空，释放临界区资源
			V(file);
		V(rcmutex); 
	}
} 
```

### 1.2 寿司店问题

#### 实现思路：

- 通过eating，waiting记录当前正在eat的线程个数和等待的线程个数。eating用于改变must_wait变量（eating==5使，must_wait = true，eating==0时 为false）。waiting用于放人进来eat，当eating == 0时，放n = min(5,waiting)个人进来。对这两个变量需要一个mutex即可，因为二者是一个整体，在操作这两个变量是我们需要同时操作。
- must_wait线程在申请时首先判断该变量，该变量为true说明已经有5人

```C
int eating = 0, waiting = 0;
Semaphore mutex = 1, queue = 1;
bool must_wait = false;

Customer(){
  P(mutex);
  if (must_wait){
    waiting++;
    V(mutex); //对waiting变量的保护可以释放
    P(queue);	// 被阻塞，坐着等待排队，等待被唤醒
  }
  else {
    eating++;
    must_wait = (eating == 5) 
    // 一旦我是最后一个来坐下吃导致人满的就要等所有人一起吃完，好难过
    V(mutex);	// 对eating变量的保护可以释放
  }
  // 上一部分已经解决了进店后是等待还是吃的问题
  Eat_sushi();// else的人和被唤醒的排队者成功进入这一步
  P(mutex);   // 开启对eating, waiting变量保护
  eating--;		// 吃的人-1,如果5个没全吃完，不可以换下一批人吃
  if (eating == 0){ // 最后一个吃完的人离开才可以进顾客
    int n = min(5, waiting);	// 放顾客进来的数量，不超过5个
    waiting -= n;
    eating +=n;
    must_wait = (eating == 5)
    for(int i = 0; i<n; i++)
      V(queue);  // 唤醒排队的n个人继续进程
  }
  V(mutex);	// 允许下一个吃完的人对变量和队列进行操作
}

```

### 1.3 三个进程的奇偶数消费者-生产者问题

#### 实现思路：

- P1为生产者，P2，P3分别为奇偶数消费者，三者共享一个缓冲区，首先肯定需要一个mutex信号量互斥整个缓冲区，接着需要一个empty信号量让生产者可以申请空缓冲区用于生产，目前为止与普通的生产者消费者模式相同。
- 但是由于P2只取奇数，P3只取偶数，因此原先我们用的full信号量相当于要分成两个，odd与even，用于代表当前装有奇数、偶数缓冲区的个数，其跟原先我们使用的full的作用一样。

```C
#define N 100
Semaphore empty = N;  // 假设初始条件下缓冲区有N个空位
Semaphore mutex = 1;
Semaphore odd = 0;
Semaphore even = 0;

void P1(){
  int integer;
  while(true){
    integer = produce(); 	// 生成一个整数
    P(empty); 						// down(empty)，若empty为0则会被阻塞（等待别人拿走）
    P(mutex);							// 开始互斥，down(mutex)
    put();								// 放入缓冲区
    V(mutex);							// 访问临界区结束，up(mutex)
    if(integer %2 == 0){
      V(even);						// 是偶数
    } else {
      V(odd);							// 是奇数
    }
  }
}

void P2(){
  while(true){
    P(odd);							// 请求一个奇数，down(odd)
    P(mutex);						// 互斥
    getodd();
    V(mutex);
    V(empty);						// 缓冲区多一个位置，up(empty)
    countodd();
  }
}

void P3(){
  while(true){
    P(even);						// 请求一个偶数，down(even)
    P(mutex);						// 互斥
    geteven();
    V(mutex);
    V(empty);						// 缓冲区多一个位置，up(empty)
    counteven();
  }
}
```

### 1.4 搜索-插入-删除问题(类似读者写者问题)

#### 实现思路:

- 定义信号量No_search & No_insert表示此时没有搜索和插入线程，用于删除线程与插入线程，删除线程与搜算线程互斥，同时由于一个删除线程P(No_insert)以后，其他删除线程再次P(No_insert)便会阻塞，所以No_insert顺便也可以作为删除进程之间互斥。
- 信号量`insertMutex， searchMutex`用于保护searcher,inserter这两个int变量，他俩的作用是记录插入搜索线程的数量，以改变No_search & No_insert（如果数量为零则释放）。

```C
Semaphore insertMutex =1, searchMutex = 1; // 保护searcher,inserter变量
Semaphore No_search = 1; // 顾名思义，为1时没有搜索进程访问
Semaphore No_insert = 1; // 为1时没有插入进程访问
//当上述两个信号量同时为1，删除者才可以进行删除操作

int searcher = 0, inserter = 0;
void Search(){
  P(searchMutex);
    searcher++;
    if (searcher == 1)	// 第一个进来的搜索者加锁
      P(No_search)
  V(searchMutex);
  Searching(); // 访问临界区，多个搜索无需互斥
  P(searchMutex);
  	searcher--;
  	if (searcher == 0)
      V(No_search);	 // 表示此时没有搜索线程在进行，解锁
  V(searchMutex);
}

void Insert(){
  P(insertMutex);
  	inserter++;
  	if (inserter == 1)
      P(No_insert);
  V(insertMutex);
  
  P(insertMutex); // 既然可以和搜索线程并行，那么不用管Searcher
  	Inserting();	// 访问临界区，多个插入者要互斥访问，一次一个insert
  V(insertMutex);
  
  P(insertMutex);
  	inserter--;
  	if (inserter == 0)
      V(No_insert); // 解锁，可唤醒删除者
  V(insertMutex);
}

void Delete(){	  // 删除线程与其他任何线程互斥
  P(No_search);
  	P(No_insert); // 若为1则可进入，这个信号量顺便也可以当作删除者的互斥保护
  		Deleting();	// 搜索和插入线程都没，成功进入临界区
  	V(No_insert);
  V(No_search);	
}
```

