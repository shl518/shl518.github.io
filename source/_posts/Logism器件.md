---
title: Logism器件
date: 2022-05-09 15:19:45
tags: 计算机组成Logism
---

## 常用器件 [^参考文献]

`Spliter,Pin,Probe,Tunnel,MUX`均比较简单，不做叙述。

### Wire

- `Bit Extender`
  ![a](https://note.youdao.com/yws/api/personal/file/80D166054146401BB764F043AC9F30DB?method=download&shareKey=40688ad8de5a9265ecff929eadf1c8de)

### Gate

- 可以在`Negate`中选择在哪一个输入前加一个非门，可以省去手动加`非门`的过程。
- `Oddparity`与`Evenparity`奇偶校验门，校验输入中1的个数。

### Plexers

- `MUX`,`DMX`一个从**多个输入**里选一个，一个从**多个输出**里选一个。
- `Decoder` 是译码器，把select信号编译为独热码，例如：2位的select信号可以翻译成四种独热码，0000,0100，0010，,0001
- `Priority Encoder`,判断有无输入是1，输出是1的最高位输入是哪一个(从上到下输入端依此从0开始编号)
  ![a](https://note.youdao.com/yws/api/personal/file/F3FC88B72EB343AD8E7124445C04A171?method=download&shareKey=113072c76b93582bcea5d9189e1484cb)
- `Bit selecter`顾名思义将输入的某一位输出

### Arithmatic

前几个都是加减乘除，去负数，比较器，不做介绍。

- `shifter`位移器,关键的是位移，有逻辑位移，算数位移，循环位移，其具体含义如下：
  ![a](https://note.youdao.com/yws/api/personal/file/1BB34F2E841B45DB96592928253A7671?method=download&shareKey=d5d494b791056a5ea92f305d17fd6ce5)
- `Bit Adder` 数输入中1的个数，`input=10001010，output=3`
- `Bit Finder`找最低位的0或1，或找最高位的0或1，查找时可能会需要，不难理解，其含义如下图：
  ![a](https://note.youdao.com/yws/api/personal/file/30C68C2FDE834574A938196A613B7148?method=download&shareKey=37947dc9e97be5e694a3403b51325855)

### Memory

前几个不常用，主要用的还是`register`，`counter`,`RAM`和`ROM`.

- Register 注意这里默认的reset是异步复位，并不管是不是上升沿，只要reset=1就复位，在实际的电路中，reset为1时寄存器和counter都要清零，以便再一次初始化（参照常用电路---寄存器的初始化）。如果要异步清零，我们**把reset信号同时接在counter和寄存器上**，寄存器**立刻异步复位**。要构成同步复位，我们**只将reset接在counter上**，在上升沿时，寄存器**将实现同步复位**。

- `RAM`:`RAM` 是一个可读可写的存储器，在我们的实验中，我们采用的是读与写相互分离的类型，所以在选择 RAM 时，请将数据接口选择为`Separate load and store port`,地址从上到下，从左往右依次递增，储存时以十六进制存储。

- `ROM`：​ROM 是一个只读类型的存储器，顾名思义，在使用过程中只能对其进行读取操作，而不能进行写操作，所以 ROM 在创建时，必须一次性将所有的信息全部导入，之后不可再进行更改。

- 对`RAM`，`ROM`存储方式分为

  - 手动存储
  - txt文件存储，文件第一行写v2.0 raw，接下来每一行为一个十六进制数据

  ```
    v2.0 raw
    10*01  //这里相当于写十个0x01
    ff
    ac
  ```

---------------------

## 有限状态机(FSM)

### Mealy

输出与输入和当前状态都有关
![a](https://note.youdao.com/yws/api/personal/file/A80337FDFC0D463985FB1C88F050A463?method=download&shareKey=7405ef00856ef72dbdf1d9b98eefd117)
上图中三个主要部件分别问**状态转移**（运用组合逻辑计算下一个状态），**状态存储**（存储下一个状态，输出当前状态），**状态输出**（运用组合逻辑计算输出）。

### Moore

输出至于当前状态有关，只要把上图中input连向状态输出的线删去即可。**Moore 和 Mealy 的区别在于, Moore 需要等待状态转移完成后才输出结果 (因此会晚一个周期), 而 Mealy 在输入的时候可以直接响应.**

-------------------

## 常用电路---寄存器的初始化

**实现寄存器的initialize**，我们需要搭配使用计数,器`Counter`与复用器`MUX`,我们设置寄存器达到最大值时`Stay at value`保持在最大值，`Max Value`设置为`0x1`,复用器设置为`select=0`时选择初始化值，`select=1`时选择输入值。这样我们就可以在最开始的一个时钟周期内将寄存器初始化。
如下图：
![a](https://note.youdao.com/yws/api/personal/file/7D798C40039A4654B54FBC623781C307?method=download&shareKey=6d164014a8e6d5a0b17a705e7e5e2de3)
该图片中红色部分即为初始化的电路，在时钟第一个周期时，`counter=0`把寄存器初始化为1,之后`counter`保持在1，寄存器开始正常的进行输入。

### 附：不同颜色线的含义

![a](https://note.youdao.com/yws/api/personal/file/9DE608A8E9B34906A4EB1FA7BD7BF7DE?method=download&shareKey=293fb4543b622941b8b6e16577d1367d)

[^参考文献]: [Library Reference](http://www.cburch.com/logisim/docs/2.7/en/html/libs/index.html)**多看英文的官方文档！**

-----------

### 一个Logisim组合电路小练习（可以自己试一下）

题目描述

- 输入一个 32 位的串, 由前置 0 + 串 10101 + 需要处理的串 u 三部分拼接而成 (串u的长度小于等于27)

- 若串的长度小于 27 位, 则在后面补 1000… 直至达到 27 位

- 结果为补全后的串 u’ 加上串 u 的长度
  ![a](https://note.youdao.com/yws/api/personal/file/8E23405E343D4327A423FECBC31A518C?method=download&shareKey=a400371a419a052a9efe35d69ded827f)

### logisim易错点汇总

1. comparator比较器是比较补码，比如1111和0111比较不是15和7比较，而是-1和7比较，所以可以位拓展一下，就可以正常比较了

