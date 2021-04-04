### data lab

直接给出ans

```c
/* 
 * CS:APP Data Lab 
 * 
 * <Please put your name and userid here>
 * 
 * bits.c - Source file with your solutions to the Lab.
 *          This is the file you will hand in to your instructor.
 *
 * WARNING: Do not include the <stdio.h> header; it confuses the dlc
 * compiler. You can still use printf for debugging without including
 * <stdio.h>, although you might get a compiler warning. In general,
 * it's not good practice to ignore compiler warnings, but in this
 * case it's OK.  
 */

#if 0
/*
 * Instructions to Students:
 *
 * STEP 1: Read the following instructions carefully.
 */

You will provide your solution to the Data Lab by
editing the collection of functions in this source file.

INTEGER CODING RULES:
 
  Replace the "return" statement in each function with one
  or more lines of C code that implements the function. Your code 
  must conform to the following style:
 
  int Funct(arg1, arg2, ...) {
      /* brief description of how your implementation works */
      int var1 = Expr1;
      ...
      int varM = ExprM;

      varJ = ExprJ;
      ...
      varN = ExprN;
      return ExprR;
  }

  Each "Expr" is an expression using ONLY the following:
  1. Integer constants 0 through 255 (0xFF), inclusive. You are
      not allowed to use big constants such as 0xffffffff.
  2. Function arguments and local variables (no global variables).
  3. Unary integer operations ! ~
  4. Binary integer operations & ^ | + << >>
    
  Some of the problems restrict the set of allowed operators even further.
  Each "Expr" may consist of multiple operators. You are not restricted to
  one operator per line.

  You are expressly forbidden to:
  1. Use any control constructs such as if, do, while, for, switch, etc.
  2. Define or use any macros.
  3. Define any additional functions in this file.
  4. Call any functions.
  5. Use any other operations, such as &&, ||, -, or ?:
  6. Use any form of casting.
  7. Use any data type other than int.  This implies that you
     cannot use arrays, structs, or unions.

 
  You may assume that your machine:
  1. Uses 2s complement, 32-bit representations of integers.
  2. Performs right shifts arithmetically.
  3. Has unpredictable behavior when shifting an integer by more
     than the word size.

EXAMPLES OF ACCEPTABLE CODING STYLE:
  /*
   * pow2plus1 - returns 2^x + 1, where 0 <= x <= 31
   */
  int pow2plus1(int x) {
     /* exploit ability of shifts to compute powers of 2 */
     return (1 << x) + 1;
  }

  /*
   * pow2plus4 - returns 2^x + 4, where 0 <= x <= 31
   */
  int pow2plus4(int x) {
     /* exploit ability of shifts to compute powers of 2 */
     int result = (1 << x);
     result += 4;
     return result;
  }

FLOATING POINT CODING RULES

For the problems that require you to implent floating-point operations,
the coding rules are less strict.  You are allowed to use looping and
conditional control.  You are allowed to use both ints and unsigneds.
You can use arbitrary integer and unsigned constants.

You are expressly forbidden to:
  1. Define or use any macros.
  2. Define any additional functions in this file.
  3. Call any functions.
  4. Use any form of casting.
  5. Use any data type other than int or unsigned.  This means that you
     cannot use arrays, structs, or unions.
  6. Use any floating point data types, operations, or constants.


NOTES:
  1. Use the dlc (data lab checker) compiler (described in the handout) to 
     check the legality of your solutions.
  2. Each function has a maximum number of operators (! ~ & ^ | + << >>)
     that you are allowed to use for your implementation of the function. 
     The max operator count is checked by dlc. Note that '=' is not 
     counted; you may use as many of these as you want without penalty.
  3. Use the btest test harness to check your functions for correctness.
  4. Use the BDD checker to formally verify your functions
  5. The maximum number of ops for each function is given in the
     header comment for each function. If there are any inconsistencies 
     between the maximum ops in the writeup and in this file, consider
     this file the authoritative source.

/*
 * STEP 2: Modify the following functions according the coding rules.
 * 
 *   IMPORTANT. TO AVOID GRADING SURPRISES:
 *   1. Use the dlc compiler to check that your solutions conform
 *      to the coding rules.
 *   2. Use the BDD checker to formally verify that your solutions produce 
 *      the correct answers.
 */


#endif
/* 
 * bitAnd - x&y using only ~ and | 
 *   Example: bitAnd(6, 5) = 4
 *   Legal ops: ~ |
 *   Max ops: 8
 *   Rating: 1
 */
int bitAnd(int x, int y) {
  return ~((~x)|(~y));
}
/* 
 * getByte - Extract byte n from word x
 *   Bytes numbered from 0 (LSB) to 3 (MSB)
 *   Examples: getByte(0x12345678,1) = 0x56
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 6
 *   Rating: 2
 */ 
int getByte(int x, int n) {

	int tmp = ((x << ((4 + (~n)) << 3)) >> 24);
	
	return tmp&0x000000ff;

}
/*
获取某个Byte，先左移使有效位到最左边，然后使其右移24位到最右边。将其移动到最左边的位数可通过n计算，由于不可以使用减号，通过使(3 - n = 3 + ~n + 1) << 3的到左移的位数。
*/
/* 
 * logicalShift - shift x to the right by n, using a logical shift
 *   Can assume that 0 <= n <= 31
 *   Examples: logicalShift(0x87654321,4) = 0x08765432
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 20
 *   Rating: 3 
 */
int logicalShift(int x, int n) {
  int t = (0x1 << 31) ^ ((!n) << 31);
	return (x >> n) & ~(t >> (n + ~1 + 1));
}
/*
逻辑移位，右移补零，当负数时，应当消去前面的1，特殊情况时当n = 0的时候，会误消，因此需要特别注意。
*/
/*
 * bitCount - returns count of number of 1's in word
 *   Examples: bitCount(5) = 2, bitCount(7) = 3
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 40
 *   Rating: 4
 */
int bitCount(int x) { // 使用二分法，从小到大依次获取2、4、8、16、32个01位中1的个数。 
    int bitcount;
    
    int tmp1 = (0x55)|(0x55<<8);
    int mask1 = (tmp1)|(tmp1<<16);  //得到掩码： 01010101……01010101
    
    int tmp2 = (0x33)|(0x33<<8);
    int mask2 = (tmp2)|(tmp2<<16);  //得到掩码： 00110011……00110011
    
    int tmp3 = (0x0f)|(0x0f<<8);
    int mask3 = (tmp3)|(tmp3<<16);  //得到掩码： 00001111……00001111
    int mask4 = (0xff)|(0xff<<16);          //得到掩码： 0000 0000 1111 1111 0000 0000 1111 1111
    int mask5 = (0xff)|(0xff<<8);           //得到掩码： 0000 0000 0000 0000 1111 1111 1111 1111
    
    bitcount = (x & mask1) + ((x>>1) & mask1);       //分别计算每组2位中，低位的1的个数；再移位求每组2位中，高位的1的个数，下面类似 
    bitcount = (bitcount & mask2) + ((bitcount >> 2) & mask2);
    bitcount = (bitcount + (bitcount >> 4)) & mask3;
    bitcount = (bitcount + (bitcount >> 8)) & mask4;
    bitcount = (bitcount + (bitcount >> 16)) & mask5;
    return bitcount;
}
/* 
 * bang - Compute !x without using !
 *   Examples: bang(3) = 0, bang(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
int bang(int x) {
	int t = ~x + 1;
	return ((t | x) >> 31) + 1;
}
//只有0取反+1后会变成0，然后或上x即0之后，右移31位还是0，+1后就得到了1. 而其它的任何数，由于一个数与其负数相或，结果一定是负数，因此右移31位后，一定是-1，+1后即变成了0.
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
  return 1 << 31;
}
// 返回最小的二进制数，将1直接左移31位，把最小的正就变成了最小的负数了。
/* 
 * fitsBits - return 1 if x can be represented as an 
 *  n-bit, two's complement integer.
 *   1 <= n <= 32
 *   Examples: fitsBits(5,3) = 0, fitsBits(-4,3) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 2
 */
int fitsBits(int x, int n) { // x能否只用n位表示？ 左移32-n位再右移32-n位，然后和原数比较 
	int shift = 32 + ~n + 1;
	int t = x << shift >> shift;
	return !(t ^ x);
}
// x能否只用低n位表示？ 左移32-n位再右移32-n位，然后使用异或操作和原数比较。
/* 
 * divpwr2 - Compute x/(2^n), for 0 <= n <= 30
 *  Round toward zero
 *   Examples: divpwr2(15,1) = 7, divpwr2(-33,4) = -2
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 2
 */

int divpwr2(int x, int n) { // 因为是有符号数（补码）的移位操作，所以，需要对符号位进行一下判断，防止出现-5/2 = -3的情况 
    int sign = x >> 31; // 符号 
//    cout << (1 << n) + ~0 << endl; 
//    cout << "(sign & ((1 << n) + ~0)) = " << (sign & ((1 << n) + ~0)) << endl;
    return (x + (sign & ((1 << n) + ~0))) >> n;
}
// 如果是正数或者0，可以直接移位；如果是负数，为避免出现-5/2 = -3的情况（应该向0舍入），对符号进行一下判断，如果是负奇数，则加一然后移位，否则直接移位。
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
  return ~x + 1;
}
// 求负数，先求反然后+1.
/* 
 * isPositive - return 1 if x > 0, return 0 otherwise 
 *   Example: isPositive(-1) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 8
 *   Rating: 3
 */
int isPositive(int x) {
  return (!((x >> 31) & 1) & !!(x)); // 取符号位，同时判断是否为0
 // 还要小心正偶数，因为是判断最低位
}
// 取符号位，同时判断是否为0
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */

int isLessOrEqual(int x, int y) { // 小心溢出！分情况讨论
int xl = (x >> 31) & 1;
int yl = (y >> 31) & 1;
int zl = ((y + ~x + 1) >> 31) & 1;
  return ((xl ^ yl) & xl) | (!(xl ^ yl) & !zl);
}
// 小于或等于，不可以简单地调用isPositive来实现，因为相减时会溢出，需要分情况讨论。
// 首先取他们的符号位，判断是否是x<0且y>=0,如果是则直接返回1；否则如果同号，则判断y-x的符号，如果为0，说明x小于等于y；

/*
 * ilog2 - return floor(log base 2 of x), where x > 0
 *   Example: ilog2(16) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 90
 *   Rating: 4
 */
int ilog2(int x) { // 二分法找最高位1，注意+的优先级比<<要高，括号要打清楚 
	int id;
	id = (!!(x >> 16)) << 4;
	id = id + ((!!(x >> (id + 8))) << 3);
	id = id + ((!!(x >> (id + 4))) << 2);
	id = id + ((!!(x >> (id + 2))) << 1);
	id = id + ((!!(x >> (id + 1))) << 0);
	return id;
}
// 使用二分法求log2，log2n的结果就是最高位的1，那么这道题就是找这个最高位的1的位。首先找第16位为分界点，然后依次缩小范围，调整id，最后得到的id就是结果。注意+的优先级比<<要高，括号要打清楚 
/* 
 * float_neg - Return bit-level equivalent of expression -f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representations of
 *   single-precision floating point values.
 *   When argument is NaN, return argument.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 10
 *   Rating: 2
 */
unsigned float_neg(unsigned uf) {
    unsigned tmp = (uf << 1);
    if (((tmp & 0xFF000000) == 0xFF000000) && (tmp != 0xFF000000)) {
        return uf;
    }
    return (uf + 0x80000000);
}
// 1、首先判断是不是NAN，如果是，则直接返回NAN
// 2、如果不是，那么将其符号位取反之后返回。由于浮点数存储数据不是补码形式，所以只要给最高位（符号位）+1即可反转。或者使用以活操作。

/* 
 * float_i2f - Return bit-level equivalent of expression (float) x
 *   Result is returned as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point values.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned float_i2f(int x) { // 符号位、指数位、尾数位分别获取，关键是舍入部分，向偶数舍入 
	int sign, id, shr, mid;
	unsigned tmp; 
	if (x == 0) return 0;
	sign = x & 0x80000000;
	if (sign) x = -x;
	id = -1;
	tmp = x;
	while (tmp) {
		tmp >>= 1;
		id++; // x = 5, id = 2, 获取最高位的1的位 
	}
	tmp = x << (32 - id); // 23位尾数移到最左端 
	shr = (tmp & 0x1ff) >> 1; // 判断舍入 
	tmp = tmp >> 9; // 移到尾数位
	mid = (127 + id) << 23; // 指数位
	if (shr > 128 || (shr == 128 && (tmp & 1) == 1)) {
		tmp = tmp + 1;
	}
	return (tmp + mid + sign);
}
/* 
 * float_twice - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interfrpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned float_twice(unsigned uf) {
  int s = uf & 0x80000000;
  int e = uf & 0x7f800000;
  int m = uf & 0x007fffff;
  int ans = 0;
  if (e == 0x7f800000) return uf;
  if (e == 0) ans = s + (m << 1);
  else  ans = s + e + 0x00800000 + m;
  return ans;
}
// 分别获取符号位、指数位、M位，然后分情况讨论。
```

（难的雅痞，查了好多资料