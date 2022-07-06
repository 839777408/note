---
title: KMP算法
tags:
  - KMP算法
  - 朴素匹配算法
categories:
  - Java
  - 数据结构和算法
top_img: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp1.jpg'
cover: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp1.jpg'
abbrlink: 8c51
date: 2020-08-21 12:52:18
updated: 2020-08-21 12:52:18
---

# 朴素匹配算法

## 基本介绍

朴素匹配算法也叫**暴力匹配算法(Brute Force)**，有一个文本串S，和一个模式串P，现在要查找P在S中的位置，怎么查找呢？

如果用暴力匹配的思路，并假设现在文本串S匹配到 i 位置，模式串P匹配到 j 位置，则有：

- 如果当前字符匹配成功（即S[i] == P[j]），则i++，j++，继续匹配下一个字符；
- 如果失配（即S[i]! = P[j]），令i = i - j + 1，j = 0。相当于每次匹配失败时，i 回溯，j 被置为0。

## 思路图解

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706215637.png)

---

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706215638.png)

---

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706215639.png)

---

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706215640.png)

---

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706215633.png)

---

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706214959.png)

---

## 代码实现

```java
package com.nanzx.kmp;

public class ViolenceMatch {

	public static void main(String[] args) {
		String str1 = "硅硅谷 尚硅谷你尚硅 尚硅谷你尚硅谷你尚硅你好";
		String str2 = "尚硅谷你尚硅你";
		int index = violenceMatch(str1, str2);
		System.out.println("index=" + index);
	}

	private static int violenceMatch(String str1, String str2) {
		char[] str1Char = str1.toCharArray();
		char[] str2Char = str2.toCharArray();

		int i = 0;
		int j = 0;
		while (i < str1Char.length && j < str2Char.length) {
			if (str1Char[i] == str2Char[j]) {
				i++;
				j++;
			} else {
				i = i - j + 1;
				j = 0;
			}
		}

		if (j == str2Char.length) {
			return i - j;
		} else {
			return -1;
		}
	}
}
```





# KMP算法

## 基本介绍

Knuth-Morris-Pratt 字符串查找算法，简称为 “KMP算法”，常用于在一个文本串S内查找一个模式串P的出现位置，这个算法由Donald Knuth、Vaughan Pratt、James H. Morris三人于1977年联合发表，故取这3人的姓氏命名此算法。

暴力匹配算法在模式串中有多个字符和主串中的若干个连续字符比较都相等，但最后一个字符比较不相等时，主串的比较位置需要回退。KMP算法在上述情况下，**主串位置不需要回退**，从而可以大大提高效率 。

## 算法思路

当空格与D不匹配时，其实已经知道前面六个字符是”ABCDAB”。KMP 算法的想法是，设法利用这个已知信息，不要把”搜索位置“移回已经比较过的位置。

为此，为**模式串P**计算出一张《部分匹配表》，也称前缀表：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706214946.png)

根据《部分匹配表》求 next 数组：

| 搜索词 | A    | B    | C    | D    | A    | B    | D    |
| ------ | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| next   | -1   | 0    | 0    | 0    | 0    | 1    | 2    |

next数组是把《部分匹配表》的**部分匹配值往右右移一格，然后第一个位置补为 -1**。

next数组的作用是方便我们运算。

---

当空格与D不匹配时，**保持文本串S的 i 位置不变，继续指向空格**，模式串P的 j 位置原本指向D，现在指向失配字符(D)对应的next值，也就是 j = next[ j ]; 即 j = 2， j 指向模式串中下标为2的C。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706215633.png)

---

发现空格与C不匹配时，**保持文本串S的 i 位置不变，继续指向空格**，模式串P的 j 位置指向失配字符(C)对应的next值，即 j = 0， j 指向模式串中下标为0的A。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706214953.png)

---

此时又发现空格与A不匹配，**这时模式串P的 j = 0，当 j=0 且不匹配时，文本串S的 i 位置要向右移动一位。**

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706214951.png)

---

D与C 失配，模式串P的 j 位置指向失配字符(D)对应的next值，即 j = 2， j 指向模式串中下标为2的C。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706214956.png)

---

匹配成功，过程结束，相较于暴力算法的不停回溯 i 位置，此算法效率较高。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706215631.png)





## 前缀表

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706214946.png)

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706214952.png)

看了这张表不难发现部分匹配值其实就是子串的**前缀和后缀的最大公共元素长度**。这里的**前缀和后缀并不包括子串本身**。

那这个前缀和后缀的最大公共元素长度的有什么用呢？

> 看下图，绿色部分代表前缀和后缀的最长公共元素，红色代表失配位置，当失配时，前缀的最长公共元素直接移动到原后缀最长公共元素位置，失配位重新匹配，避免模式串从头开始，也就是绿色前缀不用再匹配了。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20200821235030.png)





## next数组

next数组是把《部分匹配表》的**部分匹配值往右右移一格，然后第一个位置补为 -1**。

相应地，**next的值就是返回失配位之前（不包括失配位）的最长公共前后缀。这是重点！！！**

---

| 搜索词 | A    | B    | C    | D    | A    | B    | D    |
| ------ | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| next   | -1   | 0    | 0    | 0    | 0    | 1    | 2    |

**用代码求next数组的实现思路：**

1. next[ 0 ] = -1; 

2. 前一个字符的next值是0时，只需将当前字符的前一个字符与子串第一个字符比较，若相等，说明当前字符的next值就是1了。
3. 前一个字符的next值是1时，只需将当前字符的前一个字符与子串第二个字符进行比较，如果当前字符的前一个字符又与子串第二个字符相等了，说明当前字符的next值就是2了。

3. 按照上面的推理，如果一直相等，next值就一直累加：

   **k = next [ j ]，p[k] == p[j]，则next[ j + 1 ] = next [ j ] + 1；**

4. 若p[k] ≠ p[j]，如果此时p[ next[k] ] == p[j]，则next[ j + 1 ] =  next[ k ] + 1，否则继续递归前缀索引 **k = next[k]**，而后重复此过程。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706215003.png)





## KMP算法的代码实现

```java
package com.nanzx.kmp;

import java.util.Arrays;

public class KmpAlgorithm {

	public static void main(String[] args) {
		String text = "BBC ABCDAB ABCDABCDABDE";
		String pattern = "ABCDABD";
		int[] next = getNext(pattern);
		System.out.println(Arrays.toString(next));
		int index = search(text, pattern, next);
		System.out.println("index=" + index);
	}

	public static int[] getNext(String pattern) {
		int[] next = new int[pattern.length()];
		next[0] = -1;
		int k = -1;
		int j = 0;
		while (j < pattern.length() - 1) {//这里要注意减一
			if (k == -1 || pattern.charAt(k) == pattern.charAt(j)) {
				k++;
				j++;
				next[j] = k;
			} else {
				k = next[k];
			}
		}
		return next;
	}

	public static int search(String text, String pattern, int[] next) {
		for (int i = 0, j = 0; i < text.length(); i++) {
			while (j > 0 && text.charAt(i) != pattern.charAt(j)) {
				j = next[j];
			}
			if (text.charAt(i) == pattern.charAt(j)) {
				j++;
			}
			if (j == pattern.length()) {
				return i - j + 1;
			}
		}
		return -1;
	}
}
```

**运行结果：**

[-1, 0, 0, 0, 0, 1, 2]

index=15

