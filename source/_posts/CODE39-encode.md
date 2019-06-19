---
title: CODE39编码
date: 2018-12-15 00:05:33
tags:
- C/C++
- 编码
- 条形码
- 穿戴类
---

![](https://i.bmp.ovh/imgs/2019/06/544ba965d4dcd24c.jpg)


#### 0x00 产线很重要

今年赶手环项目的两个月加班加到飘飘欲仙，好在项目产品代理商客户都很给力，我一度觉得这个项目要凉，8月初拿到UI逻辑，timeline到中期了协议都还没敲定，最后竟然在春哥打鸡血般的带领下顺(惊)利(险)定版，虽然多少还有些问题，也算得上是个小奇迹了。有阵子每天凌晨一两点提交完代码大家就一起勾结去老细吃砂锅粥，一晃眼都年底了，项目组跑路的跑路换部门的换部门。人生よ……

<!-- more -->

第一次做穿戴类的订单软件，算一算从大二下学期开始快三年没有写过纯C的项目了。除去SDK提供的MCU和外设驱动部分，把蓝牙协议、存储协议、UI显示效果、操作逻辑从上到下摸了个遍，收获还是挺大的。另一方面，非常感谢这个项目，真正和产线有了一次完整的接触，让我在设计项目框架的时候会更多地思考，从生产线的角度去看待产品。

项目的PCBA和组包装分包给了两个不同的工厂，设备蓝牙mac地址在PCBA阶段已经烧录进去，并且没有预留其他口从板子上抓出来。而组包装工厂需要在出货产品的盒子上印mac地址信息。如何保证蓝牙mac地址不错乱且生产简单高效就相当重要了。刚好手环是有屏幕的，对于产线和工人来说，用扫码枪滴一下就很简单快捷啦，没有额外的物料成本或学习成本，也基本不会出错。那么终于引出了主题——CODE39条码

听说唯乐在Neo系列使用了条形码，能够在产线各环节一一对应到个人。找志刚大佬要了Neo的BarCode代码片段，基本上~~换成我们方案的绘图函数就ok了~~，不过Neo用的是电子墨水屏，当时还不确定我们的辣鸡 128x32 LCD大点阵能不能画得出来 ┑(￣Д ￣)┍

#### 0x01 聊聊CODE39
##### Wikipedia
> **Code 39** (also known as **Alpha39**, **Code 3 of 9**, **Code 3/9**, **Type 39**, **USS Code 39**, or **USD-3**) is a variable length, discrete [barcode](https://en.wikipedia.org/wiki/Barcode "Barcode") symbology.
The Code 39 specification defines 43 characters, consisting of uppercase letters (A through Z), numeric digits (0 through 9) and a number of special characters (-, ., $, /, +, %, and [space](https://en.wikipedia.org/wiki/Space_(punctuation) "Space (punctuation)")). An additional character (denoted '*') is used for both start and stop delimiters. Each character is composed of nine elements: five bars and four spaces. Three of the nine elements in each character are wide (binary value 1), and six elements are narrow (binary value 0). The width ratio between narrow and wide is not critical, and may be chosen between 1:2 and 1:3.

摘几个重点
+ 编码了大写字母、阿拉伯数字、特殊字符以及额外的“*”号
+ 每个字符由九个元素组成：五个条型和四个空格
+ 九个元素是三宽（二进制值的1）六窄（二进制值的0）
+ 宽窄比例并不严格，介于1:2与1:3之间即可

![Code_39_barcode](https://i.bmp.ovh/imgs/2019/06/17708573716c297f.png)

最终在代码实现中抽象出的数据结构与上图还是有些不同，如维基百科中所描述的，条形和空格的粗细的概念就需要具现出来。

#### 0x03 Let's code
首先抽象数据结构，编码过程字符映射为条码，将字符编为二进制编码，再将编码转化为黑白相间的图案。解码则为编码的逆过程，扫码枪扫描到图案后解析黑白的二进制值，将这个值转化为编码内容，继而对应到字符内容。

以A为例
![A.png](https://i.bmp.ovh/imgs/2019/06/662b850047aa3581.png)

100001001 （条码编码，1宽0窄）
取窄宽比为1:2
110101001011 （条码图案，黑白相间**1黑0白** 这很重要）
所以可以定义：
```C
typedef struct {
	char name;
	unsigned short symbol;
}BAR_CODE_T;

BAR_CODE_T code_39[] = {
		{'A',	0B110101001011},//0
		{'B',	0B101101001011},
		{'C',	0B110110100101},
		{'D',	0B101011001011},
		{'E',	0B110101100101},
		{'F',	0B101101100101},
		{'G',	0B101010011011},
		{'H',	0B110101001101},
		{'I',	0B101101001101},
		{'J',	0B101011001101},
		{'K',	0B110101010011},
		{'L',	0B101101010011},
		{'M',	0B110110101001},
		{'N',	0B101011010011},
		{'O',	0B110101101001},
		{'P',	0B101101101001},
		{'Q',	0B101010110011},
		{'R',	0B110101011001},
		{'S',	0B101101011001},
		{'T',	0B101011011001},
		{'U',	0B110010101011},
		{'V',	0B100110101011},
		{'W',	0B110011010101},
		{'X',	0B100101101011},
		{'Y',	0B110010110101},
		{'Z',	0B100110110101},
		{'0',	0B101001101101},//26
		{'1',	0B110100101011},
		{'2',	0B101100101011},
		{'3',	0B110110010101},
		{'4',	0B101001101011},
		{'5',	0B110100110101},
		{'6',	0B101100110101},
		{'7',	0B101001011011},
		{'8',	0B110100101101},
		{'9',	0B101100101101},
		{'+',	0B100101001001},//36
		{'-',	0B100101011011},
		{'*',	0B100101101101},
		{'/',	0B100100101001},
		{'%',	0B101001001001},
		{'$',	0B100100100101},
		{'.',	0B110010101101},
		{' ',	0B100110101101},
};
```
有几个宏
```C
// 窄宽比取1:2时单个字符需要的像素宽度
#define BAR_CODE_LEN			(12)
// 字符A在code_39数组的索引
#define BAR_CODE_A_INDEX		(0)
// 字符0在code_39数组的索引
#define BAR_CODE_0_INDEX		(26)
// 符号+在code_39数组的索引
#define BAR_CODE_SYMBOL_INDEX	(36)
// 符号字符的个数
#define BAR_CODE_SYMBOL_CNT		(7)
```

总体的流程是传入需要显示的字符数组，找到各个字符对应的索引，调接口画图。我这里用的方法是先点亮满屏全白，画黑线留白空。

##### 具体函数实现
###### 获取字符的黑白图像排列
```C
/**
 * a function to get character encode
 * @param char: character to get code39 encode
 * return BAR_CODE_T index, error return -1
 */
signed char find_barcode_char_index(char chr)
{
	if (chr >= 'A' && chr <= 'Z')
	{
		return chr - 'A' + BAR_CODE_A_INDEX;
	}
	else if (chr >= '0' && chr <= '9')
	{
		return chr - '0' + BAR_CODE_0_INDEX;
	}
	else
	{
		for (int i = BAR_CODE_SYMBOL_INDEX; i < BAR_CODE_SYMBOL_INDEX + BAR_CODE_SYMBOL_CNT; i++)
		{
			if(chr == code_39[i].name)
			{
				return i;
			}
		}
	}
	return -1;
}
```

###### 画一个字符的CODE39条码
```C
/**
 * a function to draw barcode in screen
 * @param chr: draw character
 * @param pen_size: single bar code line width
 * @param heigh: bar code height
 * @param x: draw position x
 * @param y: draw position y
 */
void show_barcode_char(char chr, unsigned char pen_size, int heigh, int x, int y)
{
	signed char index = find_barcode_char_index(chr);
	if (index < 0)
	{
		return;
	}
	char buf[BAR_CODE_LEN];
	memset(buf, 0x0, sizeof(buf));
	for(int i = BAR_CODE_LEN; i > 0 ; i--)
	{
		if(code_39[index].symbol & (0x1 << (i - 1)))
		{
			// draw black line at white back screen
			// TODO add your draw line function here
			printf("[bar code dbg] draw line (%d,%d)~(%d,%d)\n", x, y, x + heigh, y);
		}
		buf[BAR_CODE_LEN-i] = y;
		y += pen_size;
		printf("[bar code dbg] ---> symbol is %d \n", code_39[index].symbol & (0x1 << (i - 1)) ? 1 : 0);

	}
	printf("\n");
	for(int i = 0; i < BAR_CODE_LEN; i++)
	{
		printf("[bar code dbg] ---> buf is %d \n", buf[i]);
	}
	printf("\n");
}
```

###### 画完整的条码
这里就包含了开始和结束的两个"*"号，相邻的字符之间要隔一个空格
```C
/**
 * a function prepare to draw whole code39 bar code, include 2 "*"
 * @param *str: character array
 * @param len: total character length
 * @param pen_size: single bar code line width
 * @param heigh: bar code height
 */
void show_barcode(char *str, int len, unsigned char pen_size, int heigh)
{
	int y = 13; // change to your coordinate
	int x = (OLED_LCD_Y_WIDTH - heigh) / 2;

	show_barcode_char('*', pen_size, heigh, x, y); // begin with "*"
	y += pen_size * (BAR_CODE_LEN + 1); //need a space line
	for(int i = 0; i < len; i++)
	{
		printf("[bar code dbg] this char is %c.\r\n", *(str + i));
		show_barcode_char(*(str + i), pen_size, heigh, x, y);
		y += pen_size*(BAR_CODE_LEN + 1);//need a space line
	}
	show_barcode_char('*', pen_size, heigh, x, y); // end with "*"
}
```
有些打印没有关闭，替换成方案里的画图接口就可以了

主函数再调一下，show_barcode传要显示的字符数组。上下各留了一个像素宽度
```  C
// 这个参数是我的案子里用的
show_barcode((char*)bar_code, 6, 1, 27);
```

#### 0x04 0B26A5
图上的条码是随机生成蓝牙MAC地址的后三个字节，应该可以扫得出来⑧