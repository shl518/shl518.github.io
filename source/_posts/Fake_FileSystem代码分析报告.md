---
title: 2022 C++课程大作业—Fake_FileSystem 代码分析报告
date: 2022-06-20 15:57:42
tags: Cpp
---

## C++课程大作业——Fake_FileSystem 代码分析报告

## :label: 前言

文件系统的内容是本学习我的一门核心专业课操作系统中，重点学习的理论知识，我也在操作系统课下实验中利用C和Linux实现过一个简单的文件系统。在翻阅`Github`上面的源码时，我发现了这一有趣的项目Fake_FileSystem ，其利用`cpp`的架构实现了一个伪文件系统，逻辑清晰，涉及了许多cpp的oop知识例如封装，继承，多态，构造，解析，重载，虚函数，工厂模式等等知识，是一个不错的学习模板和范例，因此本次作业选用这个源码来作分析和学习。

![image-20220620160922473](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220620160922473.png)

可以看到虽然Fake_FileSystem 的star星数不多，这可能是因为其里面的内容**涉及到许多操作系统有关的知识**，比如数据块，Block0引导块，FAT文件分配表，FCB文件控制块，等等这些文件系统的底层知识可能为没有学习过操作系统的读者带来一定的难度。总而言之，幸运的是本学期我刚刚学过操作系统，所以勉强有了阅读这份代码的“资格”。

`Fake_Sysytem`大体的逻辑就是用`Cpp+面向对象`的思想，实现了包括文件系统底层涉及到的Block0（文件系统引导块）、FAT（文件分配表）、File（文件）、Document（目录）、UserOpen（打开文件表）等一系列基本类，**FileSystem这一核心文件系统类**、CommandFactory这一工厂模式的命令行类（内部实现了多种文件系统操作命令），和最终与用户进行交互的Bash（终端）类。

- 实现的主要命令如下

可以看到由于是一个简单的伪文件系统，不能实现所有操作系统中文件系统的功能，但一些常见的文件操作功能可以很好地实现，比如目录操作：mkdir,rmdir,cd,ls;文件操作:create,open,close,write,rm等;

| 命令名   | 参数1     | 参数2              |
| -------- | --------- | ------------------ |
| `mkdir`  | 绝对路径  |                    |
| `rmdir`  | 绝对路径  |                    |
| `cd`     | 绝对路径  |                    |
| `ls`     |           |                    |
| `create` | 绝对路径  |                    |
| `open`   | 绝对路径  |                    |
| `write`  | -c -co -a | 标识符（open返回） |
| `close`  | 标识符    |                    |
| `rm`     | 绝对路径  |                    |
| `save`   |           |                    |

👉源代码请您移步这里：[Fake_FileSystem](https://github.com/laCorse/Lacorse_Fake_Filesystem)

## :label: 1.1 整体架构

首先先放一张整体架构的类图，接下来进行分析。

![image-20220620180616138](https://note.youdao.com/yws/api/personal/file/8BF122BE36974C4C8A1839D745F7ABAD?method=download&shareKey=9f278f414fe0a6f80cca2155d39a20d2)

可以看到整个架构被明显的分为了三个部分，其实这便是伪文件系统的三个组成部分。

- 左边是命令工厂，其中与Bash终端直接交互的是CommandFactory命令工厂，采用工厂模式，在类内部自己决定使用哪种工厂命令，这些工厂命令包括ls，exit，open，close等等，这些工厂命令**继承**一个共同的基类CommandBase实现多态，之后会在1.2中详细讲到。
- 右边是伪文件系统，这一部分基本都与操作系统的理论知识相关，包括记录打开文件的UserOpen（打开文件表），记录数据块分配情况的FAT（文件分配表），初始化磁盘的引导块Block0等。其中FileSystem是最终实现的核心类，这个类与Bash终端进行交互。
- 最下边是Bash类，**main.cpp运行这个类中的Run函数**，开始运行整个文件系统，Bash类“类如其名”，就是终端的意思。

![2](https://note.youdao.com/yws/api/personal/file/7B0EE40946274D2A92198871E496D3D1?method=download&shareKey=b3584f8b400d4ac0dc4bb7479a6338a0)

最后，我们将整个文件系统运行逻辑简化如下，可以更加清晰的理解。

![image-20220620184310512](https://note.youdao.com/yws/api/personal/file/47E8D7DF35AE4AD18E58FC42F45F0A88?method=download&shareKey=ed65bdafd5177f124644ca43d7155118)

## :label: 1.2 细节分析

接下来我们将运用课上所学的各种知识来分析代码中的**封装，多态，继承，构造，析构，拷贝构造，重载，重写等等知识点**，主要结合代码片段来分析。

### 1.2.1 封装性

封装性几乎在该项目中处处都能得到体现，因为本项目毕竟是一个面向对象逻辑的项目，肯定处处都透露着封装性这一面向对象的基本思想。我就以整体架构中的**伪文件系统**这一部分来阐述封装性在代码中的体现。

```C++
//Filesystem核心类节选
/**
 * @class 核心文件系统 
 */

class Filesystem {
private:
    //!是否已经初始化
    bool initialized_;
    //!文件块的起始地址,分配SIZE(1024000字节)大小的空间给该指针(存储也利用该指针)。
    char *ptrToContent;
    //!分别指向各个块地址，第一块为引导块，2~3块为FAT1,4~5为FAT2，剩余995块为数据区。
    char *blocks[BLOCKNUM];
    //!打开的文件列表
    Useropen *openFiles[MAXOPENFILE];
    int fd = 0;
    //!引导块,托管blocks[0]
    BLOCK0 *pblock0;
    //!FAT1/2
    Fat *pfat1;
    Fat *pfat2;

    //!root dir
    Document *proot;
    Document *plastdir;
    //!保存当前目录的信息
    currentData currentData;
public:
    Document() {};
    Document(char *block)
    {
        pos = block;
        constexpr int size = BLOCKSIZE/sizeof(Fcb);
        for(int i=0;i<size;i++)
        {
            char *tmp = pos+i*sizeof(Fcb);
            Fcb * tmpFcb = (Fcb *)tmp;
            //memcpy(&tmpFcb,tmp,sizeof(Fcb));
            if (tmpFcb->free == 1)
            {
                fcbList.push_back(*tmpFcb);
            }
        }

    };
};
//..............................................................
```

上面是Filesystem核心类代码的节选，可以看到内部有private的私有成员，和一部分public的方法，这些成员的私有化便很好的体现了封装性，我认为其作用有以下2点。

- **隐藏实现细节，提供公共的访问方式**：**使用面向对象封装性可以很好的实现文件系统的“隐藏细节”这一思想**。在实际的文件系统中，它对程序员隐藏实现细节，程序员在日常工作时感受不到他的存在，但文件系统默默的在幕后为我们“当牛做马”，此外程序员在使用文件系统时可以执行一些文件系统提供的、封装好的系统调用接口来间接实现操作系统的功能。而在这里我们实现的的“伪文件系统”Filesystem便利用封装性模拟了这一点，我们将一些功能封装到Filesystem中，而不需要知道类中的这个方法的逻辑原理，文件系统Filesystem只需要给User一个对外的接口，User只需要能够调用这个方法就可以实现文件系统的操作，同时User也无法对FileSystem内部的成员进行直接更改。
- **提高安全性**：文件系统本身就是一个安全性很高的系统，我们程序员一般不能对操作系统内核进行随意的修改，否则我们的电脑系统就“乱了套”。**封装性也很好的提供了安全性**，我们不能对Filesystem类内部的成员变量进行随意的修改，只能通过调用Filesystem内的public方法，而这些方法由于是Filesystem自己提供的，所以保证文件系统是安全的。

### 1.2.2 继承性和多态性

继承性和多态性主要在命令工厂这一部分得到实现。接下来节选了命令工厂中的命令基类，和继承他的几个命令类。这里命令子类可以继承符类的参数parameter属性，和fakeFs文件系统的智能指针。

同时命令子类中，也重写了Execute()方法，以实现对应的命令执行方法，多态性使命令子类可以表现出不同的Execute()行为，这里我们注意到**使用了虚函数进行了方法重写**，这里的原因我们之后在1.2.5中会讲到。

```C++
//基类
class CommandBase
{
protected:
    vector<string> parameter;
    shared_ptr<Filesystem> fakeFs;
public:
    CommandBase(vector<string> cmdPara,shared_ptr<Filesystem> fs):parameter(cmdPara),fakeFs(fs)
    {}

    virtual void Execute()=0;  //这里用了虚函数！！！！！！！！！！！！！！！！！！
    virtual ~CommandBase(){};

    /**
     * @brief 在指定目录下创建文件
     * @param filename 名字
     * @param attribute 指定文件类型是文件还是目录
     * @return
     */
    int touch(string & filename, int attribute);

    /**
     * @brief 打开一个文件并读入到当前打开文件中,并分配一个openfilelist的下标作为描述符
     */
    int open(string & filename);

};

//命令子类继承基类CommandBase
//mkdir命令
class Mkdir :public CommandBase
{

public:
    Mkdir(vector<string> cmdPara,shared_ptr<Filesystem> fs):CommandBase(cmdPara, fs)
    {}

    virtual void Execute();
};
//cd 命令
class Cd :public CommandBase
{

public:
    Cd(vector<string> cmdPara,shared_ptr<Filesystem> fs):CommandBase(cmdPara, fs)
    {}

    virtual void Execute();
};
//.............................
```

### 1.2.3 运算符重载

运算符重载也在很多地方得到了体现，这里节选FAT类里的一处。

```C++
//FAT文件分配表代码节选
class Fat
{
        typedef unsigned short Filelist;

public:
    Filelist filelist[BLOCKNUM];
    char * postion;
    /**
     * @brief 重载了下[]，便于直接访问
     * @param id
     * @return
     */
    unsigned short& operator[](int id)
    {
        return filelist[id];
    }
};
```

由于我们访问FAT文件分配表其实是访问其内部的`filelist`这一数组成员。如果不进行运算符重载，我们还需要编写对应的方法进行访问，比较麻烦。进行运算符重载之后可以直接`FAT[i]`进行访问，十分方便。

### 1.2.4 虚函数实现重写

指向基类的指针在操作它的多态类对象时，会根据不同的类对象，调用其相应的函数，这个函数就是虚函数。这点在命令基类CommandBase和他的许多命令子类中的Execute()虚函数中得到很好地体现。

为什么要用虚函数进行实现呢？首先因为Bash调用命令工厂解析命令后，返回的是指向命令子类的父类指针（CommandBase*）。

```C++
class CommandFactory
{
public:

    unique_ptr<CommandBase> SmartCreateCmd(string command,vector<string> parameter,shared_ptr<Filesystem> fs)
    {
        if (command == "mkdir")
            return unique_ptr<CommandBase>(new Mkdir(parameter,fs)); //返回的是指向命令子类的父类CommandBase指针
        else if(command == "rmdir")
            return unique_ptr<CommandBase>(new Rmdir(parameter,fs));
        else if(command == "cd")
            return unique_ptr<CommandBase>(new Cd(parameter,fs));
```

而我们在执行命令时`cmd -> Execute()`采用的是指针来调用函数，这时必须要求我们在命令类中将Execute()函数定义为虚函数来实现多态，这样才能保证`cmd -> Execute()`执行的是子类重写好的Execute函数，而不是父类中的。

```C++
void LacorseBash::Run()
{
    alwaysTrue = true;
    while (alwaysTrue)
    {
        Show();
        Read();
        //执行
        cmd -> Execute(); //cmd是解析好的父类CommandBase指针，但他指向对应的命令子类,这时必须要求我们在命令类中将Execute（）函数定义为虚函数,否则执行的是父类的Execute（）
    }
}
```

### 1.2.5 拷贝构造

拷贝构造在Document文件目录类中有很好的体现。在我们对目录文件进行拷贝构造时，由于功能实现要求，我们希望只将需要拷贝的Document类中的Free的FCB块进行拷贝。这需要我们重写拷贝构造函数如下。

```C++

class Document {
//节选
vector<Fcb> fcbList;
char *pos;
    
Document(Document & doc)
    {
        doc.save();
        pos = doc.pos;

        constexpr int size = BLOCKSIZE/sizeof(Fcb);
        for(int i=0;i<size;i++)
        {
            char *tmp = pos+i*sizeof(Fcb);
            Fcb * tmpFcb = (Fcb *)tmp;
            if (tmpFcb->free == 1)
            {
                fcbList.push_back(*tmpFcb);//只拷贝Free的FCB文件控制块，其他的跳过！
            }
        }
    }
}
```

### 1.2.6 析构函数

析构函数很多类中都有写到。这里主要以文件系统类Filesystem的析构函数为例进行介绍。这里重写析构函数的原因有两个：

- Filesystem内部有许多成员变量也分配了内存，这些内存在Filesystem被销毁后不会自动释放，所以非常有必要再添加一个析构函数，专门用来释放已经分配的内存。
- 同时我们实现的伪文件系统额外实现了一个功能，就是**将系统运行结束后文件系统Filesystem中的内容记录到文件中**，作为最终的输出记录结果，这也要求我们在析构Filesystem前实现这一功能。

```C++
~Filesystem(){ //析构函数
    SaveFileSys();//系统运行结束后文件系统Filesystem中的内容记录到文件中
    //释放成员内存
    delete pblock0;
    delete pfat1;
    delete pfat2;
    delete proot;
    delete []ptrToContent;
}

void Filesystem::SaveFileSys()
{
    ofstream outFile("./lacorse_Fs_bak1.dat", ios::out | ios::binary);
    outFile.write(ptrToContent, SIZE);
    outFile.close();
    cout << "[Save]Save File System succeed!" << endl;
}
```

### 1.2.7 工厂模式

```c++
class CommandFactory
{
public:

    unique_ptr<CommandBase> SmartCreateCmd(string command,vector<string> parameter,shared_ptr<Filesystem> fs)
    {
        if (command == "mkdir")
            return unique_ptr<CommandBase>(new Mkdir(parameter,fs));
        else if(command == "rmdir")
            return unique_ptr<CommandBase>(new Rmdir(parameter,fs));
        else if(command == "cd")
            return unique_ptr<CommandBase>(new Cd(parameter,fs));
        else if(command == "ls")
            return unique_ptr<CommandBase>(new Ls(parameter,fs));
        else if(command == "create")
            return unique_ptr<CommandBase>(new Create(parameter,fs));
        else if(command == "rm")
            return unique_ptr<CommandBase>(new Rm(parameter,fs));
        else if(command == "open")
            return unique_ptr<CommandBase>(new Open(parameter,fs));
        else if(command == "close")
            return unique_ptr<CommandBase>(new Close(parameter,fs));
        else if(command == "write")
            return unique_ptr<CommandBase>(new Write(parameter,fs));
        else if(command == "save")
            return unique_ptr<CommandBase>(new Exit(parameter,fs));
        else
            return unique_ptr<CommandBase>(new Empty(parameter,fs));

    }
};
```

工厂模式在命令工厂里也得到了很好的体现。**工厂方法模式一种创建对象的模式,应用在超类和多个子类之间的情况，这种模式将创建对象的责任转移到工厂类**。这里我们的工厂类就是CommandFactory，Bash中端调用CommandFactory类解析命令，并**把创建对应命令子类（Mkdir，Rmdir，Cd等）的任务交给了工厂类**。可以看到我们在CommandFactory类中利用if-else语句逻辑解析对应命令，并创建对应的命令子类。

这是工厂模式的体现，好处是通过工厂模式，我们把对象创建的具体逻辑给隐藏起来了，交给工厂统一管理，这样不仅减少了代码量，以后如果想改代码的话，只需要改一处即可，也方便我们日常的维护。

## :label: 1.3 总结

最后对这一学期的课程和本次作业做一个总结。对于本次作业而言，在阅读代码的过程中我复习了许多课上的知识，封装继承多态，拷贝构造函数，重写重载，析构构造等等知识点，也体会到了工厂模式带来的解耦，利于维护等优点。总而言之，这个Fake_FileSystem项目利用了许多cpp面向对象的基本思想，自底向上很好的实现了文件系统的一些功能，不仅模拟了文件系统底层的实现，还封装了上层对应的命令接口，二者通过Bash终端进行交互，架构和实现都很完美，是一份不错的学习范例，我学到了很多，也同时复习了课上的许多知识，这再次印证了理论与实践相互结合、相辅相成的魅力。

