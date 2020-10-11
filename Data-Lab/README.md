# README

## Introduction

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

## Puzzles

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
    return !(x + x + 2) & (!!(~x));
}
```

这道题的思路是，要判断这个数是不是最大数，那么首先要知道最大数是多少，然后只需要利用判断两数是否相等的位运算技巧即可。

但是这道题特殊的一点是不允许使用移位运算符，我们其实可以避开去直接实现最大数。考虑到可以使用`+`，假设输入的`x`是`Tmax`，那么`x+1`就是`Tmin`，则`x + (x+1) = -1`，因为规定了只能使用0-255的数，那么没法判断和`-1`是否相等，但是可以`+1`变成0，就可以利用判断相等的技巧了。总共使用了5次运算符。

但是需要注意，上面的方法存在一个问题，就是`-1`也会满足条件，所以需要把`-1`这种情况排除在外。发现对`x`取反，如果是`-1`，则取反后变为0，所以只需要验证数字取反后不为0即可。**运算次数为7**。

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

第一种：因为题目还限定了我们只能使用0-255的数，也就是最大能且满足题目要求的能被使用的数字的二进制表示为`0xAA`，也就是我们可以知道后八位必须满足`1_1_1_1_`，其中`_`代表0或1皆可。根据题目给出的示例，可以认为需要处理的整形数字是32位的，那么就以8位为一个单元进行划分，相当于划分成了4个单元，那么每个单元都需要被检验，于是想到了左移运算，分别左移8，16，24位来生成一个检验数，为了判断对应位是否为1，第一题我们已经分析了与运算`&`用来检验对应位是否为1，恰好题目允许使用`!`运算符，那么只需要利用判断数字是否相等的技巧就可以得到答案。**共进行的9次运算**，满足题目要求。

```c
int allOddBits(int x) {
    int k = 0xAA | (0xAA << 8) | (0xAA << 16) | (0xAA << 24); //生成检验数，|可以被+代替
    int res = k & x;
    return !(res ^ k); //检验两个数字是否相等的技巧
}
```

第二种，考虑`0xAA`的二进制表示：`10101010`，如果我们以4位为单位来看，那么二进制前4位和后4位其实是一样的，每个4位里面，前2位和后2位也是一样的，所以很容易联想到快速幂的处理思路。如果这个数字满足要求，那么它的二进制中的奇数位必须是1，其他位并不影响结果，那么很容易想到分治法，既然必须满足奇数位是1，那么前16位和后16位都需要满足奇数位是1，所以可以考虑把数字`x`的前16位和后16位进行与运算`&`，然后只需要关注结果的后16位，让这16位的前8位和后8位进行与运算，此时就只需要关注后8位了，于是和我们的检验数`0xAA`进行与运算，如果其满足，那么结果必须是`0xAA`，更严格的，如果我们不使用`!`，那么就利用结果后8位里面的前4位和后4位进行运算，然后后两位。注意这里面通过和`0xAA`的与运算消除了算术右移带来的影响。**共进行10次运算**，符合题目要求，但是比上面的方法多了一次运算。

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
**总共使用了2次运算符。**

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

考虑如果`!x = 0`，那么应该返回`y`，那么`!!x = 1`，所以可以巧妙地利用移位，即用`!!x`先左移31位，再右移31位，算术右移会用符号位补齐，由此生成各个位都是1地结果，让这个结果和`y`进行与运算，让`!x = 0`和`z`进行与运算得到0，两个的结果进行或运算即可。因为`0 | x = x`。**总共使用了10次运算符**。

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

**总共使用了22次运算符**。

-----

```c
/* 
 * logicalNeg - implement the ! operator, using all of 
 *              the legal operators except !
 *   Examples: logicalNeg(3) = 0, logicalNeg(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
int logicalNeg(int x) {
    x = (x >> 16) | x;
	x = (x >> 8) | x;
	x = (x >> 4) | x;
	x = (x >> 2) | x;
	x = (x >> 1) | x;

	return (x & 1) ^ 1;
}
```

一种思路是采用类似快速幂的思路，`!`运算符其实验证的就是数字的二进制位里面是否存在1，那么可以把前16位和后16位视为一组，做`|`运算，然后把后16位的前8位视为一组，依次类推，最后就只需要判断最低一位。当然还需要考虑，如果是负数的时候，右移的时候，空出来的位置是用1进行补位的，让其和`1`进行与运算，去除符号位的影响，然后再和`1`做异或运算即可。**总共计算12次**，满足要求。

-----

```c
/* howManyBits - return the minimum number of bits required to represent x in
 *             two's complement
 *  Examples: howManyBits(12) = 5
 *            howManyBits(298) = 10
 *            howManyBits(-5) = 4
 *            howManyBits(0)  = 1
 *            howManyBits(-1) = 1
 *            howManyBits(0x80000000) = 32
 *  Legal ops: ! ~ & ^ | + << >>
 *  Max ops: 90
 *  Rating: 4
 */
int howManyBits(int x) {
	iint sign, b16, b8, b4, b2, b1;
	sign = (x >> 31);
	x = (sign & ~x) | (~sign & x);
	b16 = !!(x >> 16) << 4;
	x = x >> b16;
	b8 = !!(x >> 8) << 3;
	x = x >> b8;
	b4 = !!(x >> 4) << 2;
	x = x >> b4;
	b2 = !!(x >> 2) << 1;
	x = x >> b2;
	b1 = !!(x >> 1);
	x = x >> b1;
	
	return b16 + b8 + b4 + b2 + b1 + x + 1;
}
```

一个数字用补码最少需要多少位才能表示，数字12的二进制是`1100`，需要四位，加上符号位共需5位。数字`-1`需要符号位一位，数字`-5`需要的补码可以写为`10101`，因为从第4位往前的1可以被忽略。

正数则需要判断最高位的1的位置，然后结果加1；负数则需要找到其补码的最高位的0，然后+1，相当于补码取反后找最高位的1。那么取反还是不取反取决于符号位，所以用`sign`来判断符号位。判断最高位1可以采用二分法来解决，先判断在前16位还是后16位，如果在前16位，在必然需要低位的16位，也就是`!!(x >> 16) << 4`，其中`!!(x >> 16)`，如果`x >> 16`为0，则`!!(x >> 16) << 4`也为0，意味着不在最高的16位上面。一直判断到还剩下最后两位，也就是`b1`的结果，注意此时还需要记得如果是数字`1`这种特殊的情况，所以最后的结果需要加上`x`。另外需要统一对变量进行声明，不然在用`dlc`检测的时候会报错。

**总共使用了36次运算符。**

-----

```c
//float
/* 
 * floatScale2 - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned floatScale2(unsigned uf) {
	int exp, frac, sign;
	exp = (uf & 0x7f800000) >> 23;
	sign = uf & (1 << 31);
	frac = (uf << 9) >> 9;

	if (exp == 0 && frac == 0) return uf;
	else if (exp == 0) return (uf << 1) | sign;
	else if (exp == 255) return uf;
	else ++exp;

	if (exp == 255 && frac > 0) return uf;

	return sign | (exp << 23) | frac;
}
```

这个实验考察的是对于IEEE浮点数的理解，题目限定了是单精度浮点数，对于32位表示的情况，0-22位是`frac`部分，`23-30`是`exp`部分，最高位是符号位`s`。我们需要计算的是这个浮点数乘以2以后的情况。需要考虑的是各种边界条件：

如果`uf`是0，则直接返回。

如果`exp`部分全是0，属于非规格化，那么后23位右移一位即可并补上符号位。

如果`exp`全是1，也就是特殊值的情况，如果`frac`为0，那就是无穷的情况，乘以2还是无穷，无需处理，直接返回；如果`frac`不为0，属于`NaN`，也是无需处理。

剩下就是属于规格化数，但是需要考虑如果乘以2，也就是`exp`部分+1变为`exp`全是1，就成了特殊值，还是直接返回。

**总共使用了19次运算符**

-----

```c
/* 
 * floatFloat2Int - Return bit-level equivalent of expression (int) f
 *   for floating point argument f.
 *   Argument is passed as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point value.
 *   Anything out of range (including NaN and infinity) should return
 *   0x80000000u.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
int floatFloat2Int(unsigned uf) {
    int sign, E, M;
	sign = uf & (1 << 31);
	E = ((uf & 0x7f800000) >> 23) - 127;

	if (E > 31) return 0x80000000;
	if (E < 0) return 0;

	M = (uf & 0x7fffff) | 0x8fffff;
	if (E >= 23) M <<= (E - 23);
	else M >>= (23 - E);

	if (!sign) {
		if (M >> 31) return 0x80000000;
		else return M;
	}

	return ~M + 1;
}
```

这里是将浮点数转为整数，还是考察对于单精度浮点数各个位的理解。首先去判断最高位`sign`，做一个记录。然后根据$E = e - \text{Bias}$可以得到$2^E$的指数部分$E$，设小数部分是`frac`，其实表达式写为：
$$
(-1)^{s} \times (1 + \frac{frac}{2^{23}}) \times 2^{E}
$$
如果E > 31，已经溢出，所以直接返回，如果`E < 0`，因为括号里的部分大于等于1，小于2，所以`E < 0`，则可以直接返回0。然后需要和23比较来决定左移还是右移。这里还是得注意一个，如果是左移，还是有溢出风险的，所以需要判断移位后的`M`的最高位是否成为了1，因为如果原来`sign`是0，`M`的高位是1，则必然是发生了溢出。如果`sign`是1，意味着是负数，也就是需要得到$-M$，很显然用到`~M = -M - 1`的规则。

**共使用计算符18次**

------

```c
/* 
 * floatPower2 - Return bit-level equivalent of the expression 2.0^x
 *   (2.0 raised to the power x) for any 32-bit integer x.
 *
 *   The unsigned value that is returned should have the identical bit
 *   representation as the single-precision floating-point number 2.0^x.
 *   If the result is too small to be represented as a denorm, return
 *   0. If too large, return +INF.
 * 
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. Also if, while 
 *   Max ops: 30 
 *   Rating: 4
 */
unsigned floatPower2(int x) {
    if (x < -126) return 0;
    if (x > 127) return (0xff << 23);

    x += 127;
    return x << 23;
}
```

考察的是浮点数的阶码部分的计算。阶码计算公式为$E = e - \text{Bias}$，单精度是`-126 - 127`，那么如果小于`-126`，则就是规则里指的过小，直接返回0；如果大于127，就是规则里指的过大，直接返回正无穷，也就是阶码位全是1，其余位置全是0，直接位运算即可。满足范围要求后需要还原得到`e`的值，也就是需要加上`Bias`的值，然后移位即可。

**总共使用了6次运算符。**

## Autograding

### 正确性验证

```bash
$ make
# btest用来验证程序的正确性
$ ./btest
Score   Rating  Errors  Function
 1      1       0       bitXor
 1      1       0       tmin
 1      1       0       isTmax
 2      2       0       allOddBits
 2      2       0       negate
 3      3       0       isAsciiDigit
 3      3       0       conditional
 3      3       0       isLessOrEqual
 4      4       0       logicalNeg
 4      4       0       howManyBits
 4      4       0       floatScale2
 4      4       0       floatFloat2Int
 4      4       0       floatPower2
Total points: 36/36
```

第一次执行`make`的时候报错：

```
/usr/include/stdio.h:27:10: fatal error: bits/libc-header-start.h: No such file or directory
 #include <bits/libc-header-start.h>
          ^~~~~~~~~~~~~~~~~~~~~~~~~~
```

问题是安装GCC的时候环境没有安装完善，解决办法是执行：

```bash
$ sudo apt-get install gcc-multilib
```

出现的第二个问题：

```bash
$ ./fshow 2080374784
zsh: exec format error: ./fshow
```

原因是自己的系统是64位的，现在需要按照32位来编译，那么就需要交叉编译工具，参考了[win10 linux子系统运行32位程序(32bit交叉编译工具链)](https://blog.csdn.net/fangye945a/article/details/105777266?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param)和[Ubuntu18.04使用x86（32位）交叉编译工具链](https://blog.csdn.net/fangye945a/article/details/86568216)

更简单的办法是去修改`Makefile`里的编译选项，将`-m32`改为`-m64`即可。

### 运算符次数计算

```bash
./dlc -e bits.c
dlc:bits.c:147:bitXor: 7 operators
dlc:bits.c:156:tmin: 1 operators
dlc:bits.c:167:isTmax: 7 operators
dlc:bits.c:180:allOddBits: 9 operators
dlc:bits.c:190:negate: 2 operators
dlc:bits.c:205:isAsciiDigit: 11 operators
dlc:bits.c:217:conditional: 10 operators
dlc:bits.c:232:isLessOrEqual: 22 operators
dlc:bits.c:250:logicalNeg: 12 operators
dlc:bits.c:279:howManyBits: 36 operators
dlc:bits.c:307:floatScale2: 19 operators
dlc:bits.c:338:floatFloat2Int: 18 operators
dlc:bits.c:359:floatPower2: 6 operators
```

### 综合验证

`driver.pl`会验证正确性和运算符次数是否满足最大运算符次数的要求，并给出得分。

```bash
./driver.pl
Correctness Results     Perf Results
Points  Rating  Errors  Points  Ops     Puzzle
1       1       0       2       7       bitXor
1       1       0       2       1       tmin
1       1       0       2       7       isTmax
2       2       0       2       9       allOddBits
2       2       0       2       2       negate
3       3       0       2       11      isAsciiDigit
3       3       0       2       10      conditional
3       3       0       2       22      isLessOrEqual
4       4       0       2       12      logicalNeg
4       4       0       2       36      howManyBits
4       4       0       2       19      floatScale2
4       4       0       2       18      floatFloat2Int
4       4       0       2       6       floatPower2

Score = 62/62 [36/36 Corr + 26/26 Perf] (160 total operators)
```









