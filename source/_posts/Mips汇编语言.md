---
title: Mips汇编语言
date: 2022-05-09 15:20:09
tags: 计算机组成Mips
---

## Mips汇编语言学习（预习pre1）

### 大端存储与小端存储

![t](https://note.youdao.com/yws/api/personal/file/8F744AD7AB4441F19CCC744C87C4A077?method=download&shareKey=dcf6d754f1ec4cd68d571f1406060482)
在Mips中使用的是*小端存储*，例如：

```java
.text
li $t0,0x12345678
sw $t0,0($0)
```

则在存储区内第一个格子显示的便是0x12345678，在存储区内，每一个格子从左到右地址增加，在每一个格子内，地址从右到左以此增加，每一个格子4个字节，一个字，正好一个`int`。
再比如，

```java
.data 0x00001000 
num:.byte 1,2,3,4,5,6,7,8 
```

由于使用了小端存储，所以数据存储时，在每一个存储字内，从右到左存储，存储满后再在高地址的字中存储,如下图：

![a](https://note.youdao.com/yws/api/personal/file/CAD044EE12144AF3B7D8A8D4D7CF9D34?method=download&shareKey=eaf54321411ca2b54b806af632e1b3f1)
-----------------------

### 三种类型的指令

- **R型指令**，一般为运算指令，如`add`，`sub`，`sll`等，一般有多个操作数
- **I型指令**，有16位立即数
  - 例如`addi`，`subi`，`ori`
  - 例如`beq`，`bgtz`带标签的跳转指令
  - `sw`，`lw`带`offset`的存取指令
- **J型指令**
  - `j`，`jal`跳转指令

### 系统调用与递归的使用

**本题目内容为：**
![a](https://note.youdao.com/yws/api/personal/file/B871BA8C8FDF40168033616596970863?method=download&shareKey=4ddc8c329883643a20f62a1750c7c0f2)
**先看一段代码：**

```java
.data 
graph:.space 256
visit:.space 40
path:.space 40
stack:.space 800 
```

data空间初始化时不能过大，mips总共的存储空间大概有**4000字节**左右
在data数据段中我们定义了栈stack，栈用于函数调用后返回时获取之前的数据.在函数调用时我们规定用寄存器`$a0`,`$a1``$a2`,`$a3`传4个参，用`$v0`做返回值。在函数中，拿到参数后，**一定要把参数从`$a0`中取出再使用**，否则会发生错误。在调用参数时我们**需要在栈中存储返回地址`$ra`**，以便子函数知道返回到哪里，同时我们如果需要，也会存储一些`$s`，`$t`类型寄存器，同时可能会有多于4个的参数，这时我们还需要在栈内存储这些参数。
我们规定在栈中存储时按照以下顺序：
![a](https://note.youdao.com/yws/api/personal/file/FE343489099E40329445049C3D661C97?method=download&shareKey=08d25854d10eadb542e47596ad8068e6)


**宏定义**

```javascript
.macro getindex(%ans,%i,%j) 
	sll %ans,%i,3
	add %ans,%ans,%j
	sll %ans,%ans,2
.end_macro

.macro getindex2(%ans,%i) //这里运用了宏定义，方便的获取数据存储的地址
	li %ans,0
	add %ans,%ans,%i
	sll %ans,%ans,2
.end_macro
```

下面是主函数，主函数完成了：

- 初始化栈
- 数据读入与存储
- 传参
- 处理返回值
- 最终输出

```java
.text
main:
	la $sp,stack   //让栈指针初始化
	addiu $sp,$sp,800  //栈指针指向栈顶
	li $v0,5
	syscall
	move $s0,$v0 #$s0 is the n
	li $v0,5
	syscall
	move $s1,$v0 #s1 is m
	li $t0,0  #i
	
	
	for_i1_begin:
	beq $t0,$s1,for_i1_end
	
	#read_graph
	li $v0,5
	syscall
	move $t2,$v0
	subi $t2,$t2,1
	li  $v0,5
	syscall
	move $t3,$v0
	subi $t3,$t3,1
	getindex($t4,$t2,$t3)#t4 store address
	li $t5,1 #t5 store the 1
	sw $t5,graph($t4)
	getindex($t4,$t3,$t2)
	sw $t5,graph($t4)
	
	addi $t0,$t0,1
	j for_i1_begin
	
	for_i1_end:
	li $t5,1
	sw $t5,visit($0)
	li $s2,1  #s2 is current

	//调用函数
	#use hamcycle
	move $a0,$s2 #a0 is current //传参
	jal hamcycle
	//函数返回

	#judge
	beq $v0,$0,if_is_0
	li $a0,1
	li $v0,1
	syscall
	j end
	if_is_0:
	li $a0,0
	li $v0,1
	syscall
end:
	li $v0,10
	syscall
```

下面是调用的函数，注意只要不是叶函数，还需往下调用子函数，一定要把该保存的保存到栈空间中，**尤其是返回地址**。

```java
hamcycle:
    //递归底层
	bne $a0,$s0,else
	move $t0,$a0
	subi $t0,$t0,1
	getindex2($t1,$t0)
	lw $t2,path($t1) #t2 is path[current -1]
	getindex($t4,$t2,$0)
	lw $t3,graph($t4)
	beq $t3,$0,if_hasno_verge_to0
	li $v0,1  //返回参数
	jr $ra
	if_hasno_verge_to0:
	li $v0,0   //返回参数
	jr $ra
	
else:
    //这里一定要把参数取出来再使用
	move $t7,$a0 #t7 is current
	li $t0,1 #t0 is i 
	for_i2_begin:
		beq $t0,$s0,for_i2_end
		
		#if visit[i]!=0$$graph[path[current -1]][i]==1
		getindex2($t4,$t0)
		lw $t1,visit($t4)
		bne $t1,$0,else_has_visited 
		move $t1,$t7
		subi $t1,$t1,1
		getindex2($t2,$t1)
		lw $t3,path($t2) #t3 is path[current -1]
		getindex($t4,$t3,$t0)
		lw $t5,graph($t4)
		li $t6,1
		bne $t5,$t6,else_has_visited
		#####if yes
		getindex2($t1,$t0)
		sw $t6,visit($t1)
		getindex2($t1,$t7)
		sw $t0,path($t1)
		####if(hamcycle(current+1)==1)return true

        //重点！！！！！！！！，栈的使用
        //由于这里不是叶函数，所以一定要把返回地址$ra存进来，同时有用的临时变量也要存储起来，栈要从上到下使用
		sw $ra,0($sp)
		subi $sp,$sp,4
		sw $t7,0($sp)   //$t7返回后还要使用
		subi $sp,$sp,4
		sw $t0,0($sp)   //$t0返回后还要使用，因为还要继续循环
		subi $sp,$sp,4
		addi $t7,$t7,1
		move $a0,$t7    //传参
		
		jal hamcycle    //递归
		
		addi $sp,$sp,4   //栈用完了以后一定要回复，防止使用混乱
		lw $t0,0($sp)
		addi $sp,$sp,4
		lw $t7,0($sp)
		addi $sp,$sp,4
		lw $ra,0($sp)
		li $t1,1
		bne $v0,$t1,if_isno_1
		li $v0,1
		jr $ra
		
		if_isno_1:
		getindex2($t1,$t7)
		li $t2,-1
		sw $t2,path($t1)
		getindex2($t1,$t0)
		sw $0,visit($t1)
		
		else_has_visited:
		addi $t0,$t0,1
		j for_i2_begin
	for_i2_end:
		li $v0,0
		jr $ra
		
```

以上代码参考了DFS深度搜索算法[DFS寻找哈密顿回路](https://blog.csdn.net/u011579908/article/details/68943534?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163292490216780255292122%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=163292490216780255292122&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-3-68943534.first_rank_v2_pc_rank_v29&utm_term=%E5%A6%82%E4%BD%95%E5%88%A4%E6%96%AD%E6%9C%89%E6%97%A0%E5%93%88%E5%AF%86%E9%A1%BF%E5%9B%9E%E8%B7%AF&spm=1018.2226.3001.4187)

### 快排算法练习

#### C语言

![a](https://note.youdao.com/yws/api/personal/file/B41D197BDF314F62959468D4D02A9FDC?method=download&shareKey=3b5809c8ebc38fd00e14bb0990342c96)

#### Mips语言

```java
.data 
array:.space 400 #array
stack:.space 800
str_tips1:.asciiz "how many numbers do you want to qsort:"
str_tips2:.asciiz "please enter the numbers: "
str_tips3:.asciiz "these are sorted numbers:"
str_space:.asciiz " "

.macro getindex(%ans,%i) 
	li %ans,0
	add %ans,%ans,%i
	sll %ans,%ans,2
.end_macro

.macro swap(%i,%j)
	getindex($s3,%i)
	getindex($s4,%j)
	lw $s5,array($s3)
	lw $s6,array($s4)
	sw $s5,array($s4)
	sw $s6,array($s3)
.end_macro

.text
main:
	la $sp,stack
	addiu $sp,$sp,800
	la $a0,str_tips1
	li $v0,4
	syscall
	li $v0,5
	syscall
	move $s0,$v0  #$s0 is the amount of numbers
	la $a0,str_tips2
	li $v0,4
	syscall
	read_numbers:
	li $t0,0
	for_i1_begin:
		beq $t0,$s0,for_i1_end
		
		li $v0,5
		syscall
		getindex($t1,$t0)
		sw $v0,array($t1)
		
		addi $t0,$t0,1
		j for_i1_begin
	for_i1_end:
	#a0 is left ,a1 is right
	li $a0,0
	subi $a1,$s0,1
	
	jal qsort #qsort has no return 
	
	print:
	li $t0,0
	for_i2_begin:
		beq $t0,$s0,for_i2_end
		
		getindex($t1,$t0)
		lw $t2,array($t1)
		move $a0,$t2
		li $v0,1
		syscall
		la $a0,str_space
		li $v0,4
		syscall
		
		addi $t0,$t0,1
		j for_i2_begin
	for_i2_end:
		li $v0,10
		syscall
	
	
qsort:
    move $t6,$a0 #left
    move $t7,$a1 #right
	blt $t6,$t7,else_if
	jr $ra
else_if:
	#if left < right
	move $t0,$t6 #t0 is i
	addi $t1,$t7,1 #t1 is j 
	getindex($t2,$t6)
	lw $t2,array($t2) #t2 is pivot
	while1_begin:
		while2_begin:
			addi $t0,$t0,1
			getindex($t3,$t0)
			lw $t3,array($t3) #t3 is k[++i]
			bge $t3,$t2,while2_end
			beq $t0,$t7,while2_end
			j while2_begin
		while2_end:
		
		while3_begin:
			subi $t1,$t1,1
			getindex($t3,$t1)
			lw $t3,array($t3) #t3 is k[--j]
			ble $t3,$t2,while3_end
			beq $t1,$t6,while3_end
			j while3_begin
		while3_end:
		
		if:
			bge $t0,$t1,else
			swap($t0,$t1)
			j while1_begin
		else:
			j while1_end
	while1_end:
		swap($t6,$t1)
		
		########recursion1
		sw $ra,0($sp)#store ra
		subi $sp,$sp,4 
		sw $t6,0($sp)#store left
		subi $sp,$sp,4
		sw $t7,0($sp)#store right 
		subi $sp,$sp,4
		sw $t1,0($sp)#store j
		subi $sp,$sp,4
		
		move $a0,$t6
		subi $t5,$t1,1
		move $a1,$t5
		jal qsort
		
		addi $sp,$sp,4
		lw $t1,0($sp)
		addi $sp,$sp,4
		lw $t7,0($sp)
		addi $sp,$sp,4
		lw $t6,0($sp)
		addi $sp,$sp,4
		lw $ra,0($sp)
		
		########recursion2
		sw $ra,0($sp)#store ra
		subi $sp,$sp,4 
		sw $t6,0($sp)#store left
		subi $sp,$sp,4
		sw $t7,0($sp)#store right 
		subi $sp,$sp,4
		sw $t1,0($sp)#store j
		subi $sp,$sp,4
		
		addi $t5,$t1,1
		move $a0,$t5
		move $a1,$t7
		jal qsort
		
		addi $sp,$sp,4
		lw $t1,0($sp)
		addi $sp,$sp,4
		lw $t7,0($sp)
		addi $sp,$sp,4
		lw $t6,0($sp)
		addi $sp,$sp,4
		lw $ra,0($sp)
		jr $ra
```

笔者在编写swap函数时**出现了错误**，我原来在`swap`中使用的是`$t0`,`$t1`几个寄存器，导致在`swap`时修改了正在程序中运行的`$t0`,`$t1`这几个寄存器的值，导致了重大错误。**其实我在程序中不应该把i，j,pivot这些变量放在临时寄存器中，而应放在`$s`类型寄存器中**，这样临时变量就可以在swap中使用了，这样的程序会更加规范，~~笔者太菜了。。。~~,寄存器千万不能乱用啊！！！！，<font color = red> *程序中重要的的一些量要保存在`$s`中，不要不管啥量都用 `$t`!* </font>


### 其他小知识点

1. 乘法运算`hi`保存高32位，`lo`保存低32位。除法运算`hi`保存余数，`lo`保存商
2. load指令,`lw`，`lb`分别为从存储器加载一个字，从存储器加载一个字节。store指令,`sw`，`sb`，`sh`分别为在存储器内存储一个字，一个字节，半个字（两个字节）
3. `.space`申请空间时尽量申请4的倍数，防止用指令sw，lw时出错

