---
layout: post
title: 动态分配空间并模拟任意尺寸二维数组的方法
---

# 动态分配空间并模拟任意尺寸二维数组的方法 #

## Easy and simple ##
C 语言中, 动态分配一维数组是很简单的. 例如, 动态分配一个长度为 `n` 的数组, 其中每个元素类型为 `int`, 只需:

```c
int *pti;
int n=5;

pti=(int *)malloc(n*sizeof(int));
free(pti);
```

动态分配*列数已知的*二维数组也很简单. 例如, 动态分配一个 `row*COL` 的数组, 其中 `COL` 是已知的常数, 数组中每个元素为 `int`, 只需:

```c
#define COL 10
int row=5;
int (*ptar)[COL];

ptar=(int (*)[COL])malloc(row*COL*sizeof(int));
free(ptar);
```

此后 `ptar` 就能像普通二维数组的名字那样一模一样地使用.

以上构造的这个数组可以说和真正的数组在结构上和使用上都没有区别. 唯一的区别只是真正的数组是在编译过程中就处理好的, 这也就是为什么声明数组时只能用常量类的表达式.

可这种二维数组令我不爽的是它的列必须是*已知的*. 我怎么可能总能预先知道需要多少列呢?

## 我不知道有多少列 ##
如果有多少行多少列预先都不知道 (重点是不知道列), 而是两个在运行过程中才能确定的变量 `row,col`, 那么用 `malloc(row*col*sizeof(int))` 分配空间虽不是问题, 相应的指针类型却无法确定, 因此无法声明 (`int (*ptar)[?]`). 

为了解决这个问题, C99 提供了 variable length array (VLA) 这个概念. 从此以后, 数组声明时, 方括号里可以使用变量了, 不再局限于常量. 凡是用变量声明的数组都会自动看作 VLA. 但由于声明时用了变量, VLA 只能是动态变量, 即在 runtime 才知道该分配多少空间, 进而也不允许对 VLA 进行初始化.

这下一切都简单了, 一个

```c
int row=10;
int col=20;
int ptarr[row][col];
```

就搞定.

## C99 以前的人都不活了么 ##
是啊, 以前都是怎么解决这个问题的呢? _Mathematica_ 里面随便生成任意行列的矩阵可不是 1999 年以后的事. (_Mathematica_ kernel 是用 C 写的.)

然后 boxsun 同学指给我了[这个链接][sohu]. 挺好, 精神领会到了. 但我觉得那哥们的 implementation 稍微有点绕. 好多个 `void*`, `void**` 之类的东西, 看得都快晕了.

[sohu]: http://tsindahui.blog.sohu.com/84512010.html?nsukey=kR94oA26%2BjfoFbysgfuQ0IROkMUGZao%2BLEIXJdr0Z5YXmxQP%2BuW6NlgWKfBWrt5ohBSDW8c7N4edejCE3R6YPA%3D%3D

所以我按照他的精神, 自己写了一个方案, 应用到了一个小程序中. 其实要动态分配任意行列数的数组, 关键就 3 行. 但是这里面的逻辑稍微有点绕. 此外, 这样的数组与一般的数组还有点区别, 只能算是 "模拟" 的二维数组. 这在后面会提到.

## 方法 ##
设程序在运行中通过某个过程获得了变量 `row` 和 `col` 的值, 要用它们来模拟一个 `int` 型的二维数组需要以下代码:

```c
int **num; 				//the name of the simulated array

num=(int**)malloc(row*sizeof(int*)+row*col*sizeof(int));
for( i=0 ; i<row ; i++ )
{
	num[i]=(int*)(num+row)+i*col;
}

free(num);
```

若分配其他类型的数组, 把 `int` 改成相应类型即可.

## 解释 ##
大致想法是: 既然无法预先声明一个类型合适的指针 (由于不知道所要分配的数组的列数), 也无法预先声明一个具有足够数量指针的指针数组 (由于不知道所要分配的数组的行数), 那么干脆不预先声明指针, 而是把数组所占的空间和多个指针所占的空间一起创造出来. 然后让每个指针分别指向数组中每一行的行首. (也就是把每行行首的地址保存在一个地址类的内存空间里.)

现在要分配 `row` 个 pointer-to-int 变量的空间和 `row*col` 个 int 变量的空间, `malloc()` 部分就干了这件事.

```c
num=(int**)malloc(row*sizeof(int*)+row*col*sizeof(int));
```

注意到 pointer-to-int 的类型是 `int*`, 也就是一个地址变量, 在 64-bit 的机器上占 8 bytes. 为什么 `num` 要声明为 `int**` 型呢? 这有两
个目的, 一个是为了像二维数组的名字那样, 保证要两次 dereference 才得到 `int` 数据; 另一个是为了让 `num+i` 以指针为单位 (8 bytes) 进行
位移.

接下来, `for` 循环里这一步

```c
num[i]=(int*)(num+row)+i*col;
```

很有意思. 每次都要先跳过 `num+row` 这些区域. 而 `num` 指向 `int*`, `num+row` 就跳过了 row 个指针. 这暗示着起始处有 `row` 个地址类的数据, 而且它们应该分别被赋值为各行的第一个元素的地址. 左边的 `num[i]` 恰好是从 `num` 处起第 i 个地址变量. 注意 `num+row` 应该先 typecast 为 `int*` 型, 再加 `i*col`, 才能以 `int` 数据为单位移动地址, 得到每行的起始地址.

(好吧我知道我说的很乱. 这几步都是相互呼应的一个整体, 真的很难线性地说清楚. 反正我是很艰难地说成了这样啦.
其实你也不需要看我的解释嘛, 就那三行代码, 瞪一瞪你就明白了. 你看三体人不用说话多好.)

到这里你应该能够看出, 这样构造的其实应该算是二维数组的"模拟", 而不与普通的二维数组在使用上完全等价. 很重要的一点是, 在数值上, `(num+i)!=num[i]`. 左右表达式的意义必须清楚.

基于以上分析容易知道, 一个函数若要以这样分配的数组为自变量, prototyping 时应该写为:

```c
SomeType func(int **num)
```

现在模拟数组的首地址的类型是 `int**`, 不再是 `int (*pt)[COL]` 之类的东西.

## gdb 看内存 ##
下面主要是秀 gdb, 顺带证明以上扯的蛋真实有效. 首先是一个小程序: 从命令行输入所需的行列数, 随机给数组的每个元素赋值, 然后输出到屏幕.

```c
/*--------egg.c-------------*/
#include <stdio.h>
#include <time.h>
#include <stdlib.h>

void InitializeAndPrint(int **num, int row, int col);

int main(int argc, char **argv)
{
	if( argc!=3 )			//usage: $./egg row col
	{
		exit(EXIT_FAILURE);
	}

	int **num;
	int row=atoi(argv[1]);		//get row and col
	int col=atoi(argv[2]);
	int i;

	num=(int**)malloc(row*sizeof(int*)+row*col*sizeof(int));	//allocate memory
	for( i=0 ; i<row ; i++ )					//construct pointers to simulate 2D array
	{
		num[i]=(int*)(num+row)+i*col;
	}

	InitializeAndPrint(num, row, col);

	free(num);			//Dare you forget it
	return 0;
}

void InitializeAndPrint(int **num, int row, int col)
{
	int i,j;

	srand((unsigned int) time(0));	//set random seed by CPU time
	for( i=0 ; i<row ; i++ )
	{
		for( j=0 ; j<col ; j++ )
		{
			num[i][j]=rand()%10;	//initialize random number and print to stdout
			printf( "%d " , num[i][j] );
		}
		putchar('\n');
	}
}
```

让这个名叫 `egg` 的程序在 gdb 下运行, 让它生成 `10*20` 的矩阵:

```
$ gdb --args ./egg 10 20
```

设置在第 28 行 break, 这样可以看见输出, 并检查内存情况. 然后 run:

```
(gdb) break 28
Breakpoint 1 at 0x40084a: file egg.c, line 28.
(gdb) run
Starting program: /home/naitree/Documents/1/c/egg 10 20
6 3 6 3 9 4 0 8 2 8 6 6 2 7 2 7 3 4 1 6 
8 9 4 1 0 3 0 9 7 9 0 3 5 9 8 4 3 9 4 7 
7 0 6 2 8 8 9 3 5 2 1 3 1 5 6 4 0 6 3 8 
7 5 3 2 4 2 8 9 1 2 7 0 5 3 2 5 3 3 8 8 
8 0 1 9 7 9 5 0 5 8 0 5 6 5 9 0 7 8 0 0 
2 9 1 7 4 5 2 7 1 1 8 9 3 1 8 2 1 4 2 8 
4 4 3 0 0 3 3 9 3 5 0 5 4 1 3 0 8 5 7 9 
8 7 8 1 9 9 6 2 5 0 0 1 5 6 4 7 1 7 6 4 
2 6 1 8 9 4 8 8 2 7 9 0 5 0 4 6 9 0 8 6 
2 0 7 7 8 1 6 9 0 3 5 2 9 7 2 1 3 0 1 5 

Breakpoint 1, main (argc=3, argv=0x7fffffffde08) at egg.c:28
28		free(num);			//Dare you forget it
```

我们看见一个 `10*20` 的矩阵正常输出了. 下面看看内存分配的情况是否符合预期:

```
(gdb) p num
$1 = (int **) 0x602010
(gdb) p num[0]
$2 = (int *) 0x602060
(gdb) p num[0][0]
$3 = 6
(gdb) p &num[0]
$4 = (int **) 0x602010
(gdb) p &num[0][0]
$5 = (int *) 0x602060
```

我们看见 `num` 是 `int**` 型, 地址是 `0x602010`, `num[0]` 地址是 `0x602060`, 两者相差 `0x50`. 10 个指针变量占用 80 bytes, 即十六进制 0x50 bytes. 这说明从 `num` 处到数组的第一行之间确实有 10 个指针. `$4` 的输出说明 `num=&num[0]`, 即存储 `num[0]` 这个地址的地址在 `num` 处. `$5` 的输出说明 `num[0]=&num[0][0]`, 当然这是废话.

还有点不放心, 还是数以下到底这些信息占了多少 bytes 吧.

```
(gdb) p num[9][19]
$6 = 5
(gdb) p &num[9][19]
$7 = (int *) 0x60237c
```

最后一个元素的地址是 `0x60237c`. `0x60237c - 0x602010 = 0x36c = 876`, 加上最后那个元素的空间总共 880 bytes. `10*8 + 10 * 20 * 4 = 880`. `malloc()` 分配的和我们看见的大小一致.

Well, time well wasted.
