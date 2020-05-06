# README

# Introduction

实验材料都在名为`datalab-handout.tar`的文件内。解压缩：

```bash
$ tar xvf datalab-handout.tar
```

在名为`bits.c`的文件内包含13个编程问题，任务就是在指定规则下完成代码，只允许使用下面的8个运算符：

```
! ~ & ^ | + << >>
```

`dlc`文件是用来检测 `bits.c` 里面的函数是否  是按照要求编写的，有没有使用非法的数据类型等。 使用方法：`./dlc bits.c`

检测成功后，使用` btest` 测试 每一个函数功能方面是否正确无误。使用方法：`./btest`，如果某个函数错误，会显示错误的数据，以及正确的数据。

# Puzzles

## 1. Bit Manipulations

下面的表格里`Rating`代表难度等级，`Max ops`代表使用运算符完成每个程序的最大次数。

| Name                 | Description                             | Rating | Max Ops |
| -------------------- | --------------------------------------- | ------ | ------- |
| `bitXor(x, y)`       | `x ^ y` using only `&` and `~`          | 1      | 14      |
| `allOddBits(x)`      | Are all odd-number bits set to 1?       | 2      | 12      |
| `isAsciiDigit(x)`    | Is `x` an ASCII digit?                  | 3      | 15      |
| `conditional(x,y,z)` | Same as C's `x ? y : z`                 | 3      | 16      |
| `logicalNeg(x)`      | Compute `!x` without using `!` operator | 4      | 12      |

```c
//1
/* 
 * bitXor - x^y using only ~ and & 
 *   Example: bitXor(4, 5) = 1
 *   Legal ops: ~ &
 *   Max ops: 14
 *   Rating: 1
 */
int bitXor(int x, int y) {
  	return ~(x & y) & ~(~x & ~y);
}
```

`x & y`是取出`x`和`y`里面同是1的位，那么`~(x & y)`取出的就是同为0或不同的位；同理，`~x & ~y`取出的是`x`和`y`里面同为0的位，那么`~(~x & ~y)`取出的就是`x`和`y`里面同为1或不同的位，所以取交集就是取出不同的位。

另外实现运算符的时候还可以考虑下德摩根定律，不一定用来解决本题，但也是思考的一个角度。

-----

在`bits.c`里这道题的后面还多了一个函数，所以也一同实现一下：

```c
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
    return (1 << 31);
}
```

这道题目其实和自己在《位运算》里面的总结是重合的，考虑在Lecture里面带符号整数的计算规则：
$$
x = -2^{w-1} + \sum_{i = 0}^{w-2} 2 ^i
$$
所以带符号整数的最小值一定是符号位为1，其余位为0最小，所以左移即可（标准些其实应该写成下面的形式）

```c++
#include <iostream>
#include <climits>

using namespace std;

int getMax()
{
    int mask = (sizeof(int) * CHAR_BIT) - 1;
    return ~(1 << mask); //也可以写为(1 << mask) - 1
}

int getMin()
{
    int mask = (sizeof(int) * CHAR_BIT) - 1;
    return 1 << mask;
}

```

顺手也实现了带符号整数最大值的实现方法。总共使用了1次运算符。

-----

```c
//2
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise 
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 2
 */
int isTmax(int x) {
    int y = x + 1;
    int res = x + y + 1;
    return !(res ^ 0);
}
```

这道题的思路是，要判断这个数是不是最大数，那么首先要知道最大数是多少，然后只需要利用判断两数是否相等的位运算技巧即可。

但是这道题特殊的一点是不允许使用移位运算符，我们其实可以避开去直接实现最大数。考虑到可以使用`+`，假设输入的`x`是`Tmax`，那么`x+1`就是`Tmin`，则`x + (x+1) = -1`，因为规定了只能使用0-255的数，那么没法判断和`-1`是否相等，但是可以`+1`变成0，就可以利用判断相等的技巧了。总共使用了5次运算符。

----

```c++
/* 
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 */
```

这道题目让判断一个数字的奇数位是否全是1，限定了运算符和运算次数。这道题有两种思考方式：

第一种：因为题目还限定了我们只能使用0-255的数，也就是最大能且满足题目要求的能被使用的数字的二进制表示为`0xAA`，也就是我们可以知道后八位必须满足`1_1_1_1_`，其中`_`代表0或1皆可。根据题目给出的示例，可以认为需要处理的整形数字是32位的，那么就以8位为一个单元进行划分，相当于划分成了4个单元，那么每个单元都需要被检验，于是想到了左移运算，分别左移8，16，24位来生成一个检验数，为了判断对应位是否为1，第一题我们已经分析了与运算`&`用来检验对应位是否为1，恰好题目允许使用`!`运算符，那么只需要利用判断数字是否相等的技巧就可以得到答案。共进行的9次运算，满足题目要求。

```c
int allOddBits(int x) {
    int k = 0xAA | (0xAA << 8) | (0xAA << 16) | (0xAA << 24); //生成检验数，|可以被+代替
    int res = k & x;
    return !(res ^ k); //检验两个数字是否相等的技巧
}
```

第二种，考虑`0xAA`的二进制表示：`10101010`，如果我们以4位为单位来看，那么二进制前4位和后4位其实是一样的，每个4位里面，前2位和后2位也是一样的，所以很容易联想到快速幂的处理思路。如果这个数字满足要求，那么它的二进制中的奇数位必须是1，其他位并不影响结果，那么很容易想到分治法，既然必须满足奇数位是1，那么前16位和后16位都需要满足奇数位是1，所以可以考虑把数字`x`的前16位和后16位进行与运算`&`，然后只需要关注结果的后16位，让这16位的前8位和后8位进行与运算，此时就只需要关注后8位了，于是和我们的检验数`0xAA`进行与运算，如果其满足，那么结果必须是`0xAA`，更严格的，如果我们不使用`!`，那么就利用结果后8位里面的前4位和后4位进行运算，然后后两位。注意这里面通过和`0xAA`的与运算消除了算术右移带来的影响。共进行10次运算，符合题目要求，但是比上面的方法多了一次运算。

```c
int allOddBits(int x) {
    x = (x >> 16) & x; //数字前16位和后16位进行与运算
    x = (x >> 8) & x; //后16位的前8位和后8位进行与运算
    x = x & 0xAA; //后8位和检验数0xAA进行与运算，同时消除算术右移的影响
    x = (x >> 4) & x; //数字后8位的前4位和后4位进行与运算
    x = (x >> 2) & x; //数字后4位的前2位和后2位进行与运算，如果满足，此时x的二进制为表示为00...0010
    return x >> 1; //最后两位是10，所以满足条件返回1
}
```

-----

```c
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
    return (~x) + 1;
}
```

这个技巧在《位运算》里面总结过，即：
$$
\sim n = -(n+1), \quad n \geq 0\\
 \sim n = -n - 1, \quad n < 0\\
 \therefore \sim n = -n-1\\
$$
总共使用了2次运算符。

------

```c
/* 
 * isAsciiDigit - return 1 if 0x30 <= x <= 0x39 (ASCII codes for characters '0' to '9')
 *   Example: isAsciiDigit(0x35) = 1.
 *            isAsciiDigit(0x3a) = 0.
 *            isAsciiDigit(0x05) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 3
 */
int isAsciiDigit(int x) {
    int part1 = x >> 8; //前24位
    int part2 = (x >> 4) & 0xF; //第25位到28位
    int part3 = x & 0xF; //最后四位
    return !(part1 ^ 0) & !(part2 ^ 3) & (!((part3 >> 1) ^ 4) | !(part3 >> 3));
}
```

这道题目是让判断输入的数字是否是符合ASCII要求的数字，那么不妨写出看一看符合要求的数字的二进制表示有什么特点，只取后8位。

```
0 0 1 1		0 0 0 0
0 0 1 1		0 0 0 1
0 0 1 1		0 0 1 0
0 0 1 1		0 0 1 1
0 0 1 1		0 1 0 0
0 0 1 1		0 1 0 1
0 0 1 1		0 1 1 0
0 0 1 1		0 1 1 1
0 0 1 1		1 0 0 0
0 0 1 1		1 0 0 1
```

于是发现可以分为三部分来判断，前24位，第25位到28位，以及后四位。

* 前24位必须全是0，于是相当可以数字左移8位判断是否为0
* 第25到28位必须表示成`0 0 1 1`，于是只需取出这四位，利用上一题的判断相等的技巧即可
* 最后四位就需要分类讨论，但是题目不允许用`if, else`，那么就利用或运算，后四位其实可以看成两类，后四位中的最高位为1的情况，那么必须后四位里面的前三位为`1 0 0`，最后一位任意；后四位的最高位为0，后面三位任意。于是程序只需要把这些翻译成代码即可。

**总体恰好15次运算**，满足要求。

写完发现其实还可以进一步优化，将前28位看成一组，后4位看成一组：

```c
int isAsciiDigit(int x) {
    int part1 = x >> 4; //前28位
    int part2 = x & 0xF; //最后四位
    return !(part1 ^ 3) & (!((part3 >> 1) ^ 4) | !(part3 >> 3));
}
```

这样就只需要**进行11次运算**了。

-----

```c
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
int conditional(int x, int y, int z) {
    int mask1 = !x << 31 >> 31;
    int mask2 = !!x << 31 >> 31;
    return (mask1 & z) | (mask2 & y);
}
```

题目要求用位运算来实现三元运算符，如果`x`是0，那么就返回`z`，否则返回`y`。判断`x`是否为0很容易，只需要利用`!`即可。那么`!x`的结果只能是0或1，所以问题取决于如何区分0和1。

考虑如果`!x = 0`，那么应该返回`y`，那么`!!x = 1`，所以可以巧妙地利用移位，即用`!!x`先左移31位，再右移31位，算术右移会用符号位补齐，由此生成各个位都是1地结果，让这个结果和`y`进行与运算，让`!x = 0`和`z`进行与运算得到0，两个的结果进行或运算即可。因为`0 | x = x`。总共使用了10次运算符。

-----

```c
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) {
    int isSameSign = (x >> 31) ^ (y >> 31); //相同为0，不同为1，相当于conditional里的x
    int trueVal = !(0 ^ (y >> 31)); //异号下判断y是否大于等于0
    int falseVal = !(((y + 1 + ~x) >> 31) ^ 0); //同号下作差比较是否大于等于0
    int mask1 = !isSameSign << 31 >> 31;
    int mask2 = !!isSameSign << 31 >> 31;
    return (mask1 & falseVal) | (mask2 & trueVal);
}
```

这道题目最先想到的是计算`y`和`x`的差，判断最后结果是否大于等于0即可。那么就需要计算`-x`，这个时候会产生问题，因为如果`x = Tmin`，那么`-x`会溢出。如果把`x = Tmin`单独考虑呢？也不行，因为如果`x`本身小于0，而`y`本身是很大的正数，那么作差也会产生溢出。

所以第一步应该先思考两个数是否同号。如果同号，那么作差即可。如果不同号，那么就判断`y`的最高位是否为0即可。所以相当于实现一次三元运算符。注意作差的时候判断`x - y <= 0`，如果同号`y = Tmin`，计算`-y = ~y + 1`，结果还是`Tmin`，如果`x = Tmin`，结果就是0，如果`x`大于`Tmin`，那么最终结果会大于0，无论哪种情况，`y`是`Tmin`都不会影响结果。

```
同号 isSameSign = 0，判断x - y <= 0，即!(((y + 1 + ~x) >> 31) ^ 0)
异号 isSameSign = 1, 判断y >= 0，即!(0 ^ (y >> 31))
然后利用三元运算符conditional的具体表达式即可。
```

总共使用了22次运算符。















