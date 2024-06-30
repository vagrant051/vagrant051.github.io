---
layout: post
title: "KMP字符串算法"
categories: algorithm
tags: [KMP]
---

KMP算法解释

## 背景

KMP算法（Knuth-Morris-Pratt算法）是一种用于字符串匹配的高效算法，由Donald Knuth、Vaughan Pratt和James H. Morris在1977年共同提出。KMP算法的主要优点是在进行字符串匹配时，它避免了重复检查已经匹配的部分，从而提高了匹配效率。

## 基本思想

KMP算法的核心思想是利用已经匹配的信息来加快匹配过程。当在主串（original）中发现一个字符与模式串（target）中的字符不匹配时，不需要回退主串的指针，而是利用部分匹配表（pi）来决定下一步应该跳转到模式串中的哪个位置继续匹配。

## KMP算法引出的子问题：如何求得部分匹配表（pi）

### 什么是部分匹配表？（表格请竖着看）

索引 | 字符（target） | π[i]
------ | ------ | ------ 
0 | A | 0
1 | B | 0
2 | A | 1
3 | B | 2
4 | C | 0
5 | A | 1
6 | B | 2
7 | A | 3
8 | B | 4
9 | A | 3

不理解KMP可以，我们不妨先理解子问题：

前缀函数表（也叫部分匹配表、失配函数）记录了模式串中每个位置之前的最长相同前缀和后缀的长度。

- 前缀（Prefix）：字符串从开始位置到某一位置之间的子串，不包括最后一个字符。例如，对于字符串 "ABC"，它的前缀包括 ""、"A"、"AB"。
- 后缀（Suffix）：字符串从某一位置到末尾之间的子串，不包括第一个字符。例如，对于字符串 "ABC"，它的后缀包括 ""、"C"、"BC"。

比如：

索引为8的地方：

0~3为"ABAB"，是长度为4的前缀，5~8为"ABAB"，是长度为4的后缀，而且前缀后缀的字符相等，经观察可以发现相等的前缀后缀的字符最大长度就是4，所以π[8]=4

### 解决子问题的思想

初始状态：pi[0]=0;

那接下来肯定是比较A和B，发现没有共同前缀后缀，currentFitLength = 0

再看"ABA"有共同前缀后缀吗？有长度为1的"A"。那我怎么知道这个是最长的同前缀后缀？因为pi[1]=0;那么pi[2]顶多是0+1=1，不可能超过这个值。又比如当我想要通过pi[8]=4来得到pi[9]是多少？我只知道pi[9]<=4+1，因为最好情况就是str[9]==str[4]，此时pi[9]=4+1=5。（当然上面的例子不符合最好的情况）

那理解上面的话之后，我知道pi[8]=4后，我怎么知道pi[9]=3？我们可以看到当前的currentFitLength=4。那我们不妨将str[9]翻折到str[4]的位置上。为什么可以这么做？因为str[0~3]和str[5~8]的4位都是一样的啊，那么此时问题就变成求"ABABA"的最长公共前后缀，显然答案是3，因为pi[3]=2，也就是str[0~1]和str[2~3]是一样的，那么此时我们就要比较str[2]和str[4]，发现一样，那么pi[4]=pi[3]+1=3。

请注意"翻折"（对应代码：currentFitLength = pi[currentFitLength - 1];）可能不止有一次，可能是两次三次......

{% highlight ruby %}
std::vector<int> CalculateCollectiveFrontAndBack(std::string target)
{
	int lengthOfTarget = target.size();
	std::vector<int> pi(lengthOfTarget, 0);

	int currentFitLength = 0;
	for (int i = 1; i < lengthOfTarget; i++)
	{
		while (target[i] != target[currentFitLength] && currentFitLength != 0)
		{
			currentFitLength = pi[currentFitLength - 1];
		}
			
		if (target[i] == target[currentFitLength])
		{
			currentFitLength++;
		}

		pi[i] = currentFitLength;
	}

	return pi;
}
{% endhighlight %}

## 最后一步：实现kmp算法

基本思路是匹配不成功时将target向右移动，但移动几格是有技巧的：

- pi[currentEqualLength - 1] == 0时移动1格
- pi[currentEqualLength - 1] > 0时移动pi[currentEqualLength - 1]格，因为target前缀后缀有相等的，通过跳过这么多格来免去不必要的比较过程，比如target="ABABC"（pi:00120），当我们比到第五位（str[4]）时，发现和original的一部分（比如...ABABA）不一样，此时target向右移动两格，因为target[2~3]==original[n+2~n+3]，那么根据pi[currentEqualLength - 1]=pi[3]=2，我们知道target[0~1]也==original[n+2~n+3]，当然移动两格。

{% highlight ruby %}
int kmpSearch(std::string original, std::string target)
{
	int index = -1;
	int lengthOfOriginal = original.size();
	int lengthOfTarget = target.size();
	std::vector<int> pi = CalculateCollectiveFrontAndBack(target);
	int currentEqualLength = 0;
	int targetMoveStep;//目前target向右移动的距离
	for (targetMoveStep = 0; targetMoveStep < lengthOfOriginal - lengthOfTarget + 1; targetMoveStep++)
	{
		while (original[targetMoveStep + currentEqualLength] == target[currentEqualLength] && currentEqualLength != lengthOfTarget)
		{
			currentEqualLength++;
		}

		if (currentEqualLength == lengthOfTarget)
		{
			return targetMoveStep;
		}
		else
		{
			if (pi[currentEqualLength - 1] > 0)
			{
				currentEqualLength = pi[currentEqualLength - 1];
				targetMoveStep = targetMoveStep + currentEqualLength - 1;
			}
			else
			{
				continue;
			}
		}
	}

	return index;
}
{% endhighlight %}

注意"-1"是为了抵消for循环的影响：targetMoveStep = targetMoveStep + currentEqualLength - 1;