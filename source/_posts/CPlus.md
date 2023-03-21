---
title: C Plus
date: 2022-05-09 15:01:57
tags: C Plus
---

## 北航C++复习

#### 3.7

```c++
#include <iostream>
using namespace std
//tip:封装继承多态(行为关系)
--size control
#pragma pack(1)压缩为17字节
struct node{
    int a:1; //a occupy 1 B ,bitwise struct应用于与硬件通信
    int x;
    int y;
    double d;
    char c; //all 32 byte ,以最大字节数分配4*8=32
};
//tip OS -> API ->application background(database dll multiple-thread) ->GUI
//tip c plus 封装
//tip 访问控制 public private 
class Student{//对象占的空间就是其属性所占空间 sizeof(Student)=8B
private://不写default private
    int age;
    char* name;
    int *p = new int();
public:
    void initialize(int aage,char* aname);//method  declare inside
    //tip constructor 构造器，可以写多个
    Student(int aage,char* aaname);
    //tip default constructor
    Student();
    //tip stack and heal 
    //tip
    ~Student();//destructor 析构函数 对象消亡时自动调用，只能有一个，不能写多个重载的析构函数，保证内存的正常分配
}
//method define outside
Student::~Student(){
    delete p;
}
Student::Student(){
    
}
void Student::initialize(int age,char* aname ){
    age = aage;//hiden this pointer === this->age = aage;  this is address of s,point to s
    name = aname;
}
Student::Student(int age,char* aname ){
    age = aage;//hiden this pointer === this->age = aage;  this is address of s,point to s
    name = aname;
}
int main(){
    struct Student s(10,"a");//use constructor，强制初始化
    //or Student* s = new Student(10,'a'); 
    //Student s2(); 这种写法是错的
    //Student *s2 = new Student();  right
    //s.initialize(20,"aa");
    //int i =4 === int i(4);
}

```

- hm: 自学写udp socked
- hm:属性什么时候用值，什么时候用指针
- hm:构造析构例子

#### 3.14

```c++
#include <iostream>
using namespace std
void fun(int i) {
    i++;
}
void fun2(int& i) {
    i++;
}
class Test {
    int i;
    int *j;
public:
    Test(int ai,int aj);
    Test(const Test& t);
}
class Test {
    int i;
    int *j;
    Test(const Test& t);//tip5 将拷贝构造私有化，所有拷贝构造的行为被编译器拦截，必须进行
public:
    Test(int ai,int aj);
    
}
Test::Test(int ai,int aj){
    i=ai;
    this->j = aj;
}
Test(const Test& t) {
    this->i = t.i;
    //this->j = t.j;//浅拷贝
    this->j = new int(*t.j)//深拷贝，深拷贝
        
}
void fun3(Test t){
    
}

int main() {
    //tip1:引用
    int a = 20;
    int &r = a;//引用必须初始化，是一个安全的指针，对于引用操作等价于对啊的操作
    r++;//== a++
    fun(a);//a = 20,not changed C++编程思想
    fun2(a);//a = 21 引用，借值之名，行指针之实
    //引用不可修改
    int b = 30;
    r = b;//wrong ,can not be changed === 20==30
    //tip3
    Test t1(1,2);
    Test t2(t1);//深拷贝logic copy和浅拷贝 bitwise copy.默认浅拷贝
    fun3(t1);//只要传值就会发生拷贝，传参是把值复制一份，即拷贝
    //tip4 自定义数据类型不要传值，传指针或者引用
}



```

##### tip2:内存分配

代码区（常量等），全局变量区（在main函数运行之前全部清零），runtime memory (stack and heap)

- stack 局部变量，函数局部变量
- heap 存储malloc动态内存分配，malloc常用于
  - 人为控制生命周期
  - 不确定变量有多少个，例如链表的构造

### 3.21

```c#
#include <iostream>
using namespace std
//tip1 static 1.将变量声明为此文件全局？永久保存 2.将函数声明为此文件可见 3.实现类之间的数据通信4.实现函数属于类
//区别 static首次使用时构造,全局变量main函数之前构造，两者都在main结束后析构
extern int global_i;//声明此量不在本文件中
extern void fun1();//函数默认其他文件也可见，可以不写extern，但变量外连接必须写
static void fun2() {//该函数只在本文件内使用
    
}
class Static
{
    static int i; //3.实现类之间的数据通信
    int j;
public:
    //3Static(int ai,int aj);
    Static(int aj);
    static void fun4();//4此函数属于类，不属于对象，可以通过类直接调用  Static::fun4();
}
int Static::i = 100;//3类内部的静态变量外部一次初始化
Static::fun4(){
    i++;//4 静态类内函数只能操作静态类内变量，因为函数可以通过类直接调用，不能定位非静态j
    //j++;
}
Static::Static(int aj) {
    //3i = ai;//wrong static不能重复初始化
    j = aj;
}
void fun(){
    const static i = 0; 
    i++;
    cout << i<< endl;
}
int main() {
    Static::fun4();
}
```

```C#
#file2
int global_i = 100;

```

- static loacl var

- static fun

- static datamember

- static funmember

##### tip2:设计模式

**单件模式（实现一个类全局只能存在一个对象）**

```c#
#include <iostream>
using namespace std

Class Single 
{
    static Single* self;
	Single();//构造函数私有化，不能构造    
public:
    static Singlr* get_instance();
}

Single* Single::self = NULL;
Single* Single:: get_instance()
{
    if(self == NULL) {
        self = new Single();
    }
    return self;
}

int main() {
    Single * s = Single::get_instance();
}
```

##### hm

//SQLlite 实现借书还书系统

//ORMapping

##### tips3:const

```C#
#include <iostream>
using namespace std
//1 const parameter
void fun(const int* a) {
    (*a) = 10;//1 error 不能修改常量
}
//2 const return value

class Const
{
    const int i;//定义类内常量
    enum//定义类内常量
    {
        tcp,udp
    };
    public:
    const Test* fun2(); //3返回值是常量
    void fun() const;
    
}

Test* TEst::fun2(){
    
}
void Const::fun() const{
    //此函数内只读不可写
}
int fun3(){
    return 1;
}


class Account
{
    public:
    Account& operater + (int money);
    Account& operater ++();
    Account operater ++(int);//强加占位符以区分，必须写int占位
}
//++a
Account& Account::operater ++(){
    this->balance ++;
    return *this;
}
//a++
Account Account::operater ++(int){
    Account a = *this;
    this->balance++;
    return a;
}
Account& Account::operater+(int money) {
    this.balance +=money;
    return *this;
}
int main() {
    Const c;
    c.fun2()->fun2();//error 返回值是const不允许做左值、
    //函数返回BUILD_IN常量，自动为常量
    fun3()++;//error
    //返回自定义，不为常量,需要自己定义
    const Const c;//4const 对象
    c.fun();//error 常量对象不能直接调用函数，加上const就可以调用
    
    //tips4 new and delete
    Test* p = new Test();
    //new =  =malloc + constructor
    //delete = destructor + free
    
    
    //tip5 运算符重载
    //类内部人为定义加减乘除
    Account a(10);
    a = a+100;//=相当于拷贝，需要返回&
}
```



### 3.28

//effective C++

//继承与组合

```C++
#include <iostream>
using namespace std

class Computer
{
    int i;
    int price;
    char* brand;
public:
    int get_price();
    void maintain();
    Computer(int ai);//有了这个就没有默认构造
}
int Computer::get_price(){
    return price;
}
void Computer::maintain(){
    cout<<"service"<<endl;
}
Computer::Computer(int ai){
    i=ai;
}
class Mac : public Computer  //tip1继承写法
{
    int j;
    int year;
	public：
    void maintain();
    Mac(int aj);
}
Mac::Mac(int ai,int aj):Computer(ai)//写，间隔多个
    
{
    j = aj; //tip3子类构造调用父类构造
}
//tip4 外壳的构造首先调用内部成员的构造，和上面语法一样
//tip5 内部成员构造顺序与类内成员声明顺序相符
//tip6 私有继承，父类的所有东西都继承为私有   class Mac: private Computer  私有继承
//私有继承削弱了父类（私有继承，只重写父类一个重载）
void Mac::maintain()
{
    if(year <= 1)
        cout<<"new"<<endl;
    else
        Computer::maintain();//super() C++一个类可以有多个父类
    
    
}
//tip2 父类方法有重载，子类如果需要重写其中一个，必须把其余重载方法也重写，否则其他重载方法不可使用
//tip7 多重继承
int main(){
    
    
}
```

### 4.11

### 多态

- tip1：向上类型转换

- tip2:binding 绑定 将函数的调用绑定对应的过程，之前所学的函数都是前绑定。后绑定（动态的绑定）实现之后才能实现多态，可以通过虚函数实现
  - 多态的前提是继承
  - 没有向上类型转换就没有多态
  - 多态的实现通过一个类和为其创建的虚函数表vtable实现，虚函数会在对象内创建一个虚指针vptr指向虚函数表，大小会增加一个指针的大小4bytes。
  - 纯虚函数，一个类含有一个纯虚函数，被称为抽象类abstract class，不允许实例化
- 抽象类的两个用途
  - 规定本类家族的共性行为
  - 连接本不相关的多个类家族
- 构造没有多态（父类指针可以指向子类，子类指针不能指向父类），析构函数通常需要实现多态，前面加上virtual

```c#
class Pet
{
public :
	virtual void speak()=0; //父类加上virtual 虚函数实现后绑定，父类的虚函数可以定义为纯虚函数
}
void Needle(Pet& pet){ //改成传引用
	pet.speak();//此处是多态
}
class Dog:public Pet
{
public:
	void speak();//这里重写
}
int main(){
	Dog dog;
	Needle(dog);//向上类型转换
}
```

### 4.18

```c#
//STL standard template library
//vector 
//动态增长的万能容器
template <class T>
class Stack
{
    int pool[100];
    int top;
public:
    void push(int i);
    int pop();
    Stack();
}
//int stack -----> double stack 
template <class T>
class Stack
{
    T pool[100];
    T top;
public:
    void push(T i);
    T pop();
    Sta n  ck();
}
//如何支持动态增长，通过拷贝构造，空间不够了找另一个空间拷贝构造
class Test 
{
    static int cnt;//static本类所有对象共享
    public:
 	Test(const Test& t);//拷贝构造
}
//tip3 迭代器模式，统一对不同容器的存取
int main(){
    list<int> v;
    list<int>::iterator it = v.begin;//list 类中的迭代器类
    while(it!= v.end()){
        cout << *it<<endl;//运算符重载
        it++;
    }
}
```

### 4.25

```c#
void Fun(int m) throw int
{
    if(m = 4)
        throw 1;
}
```

