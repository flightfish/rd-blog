# 字符串匹配

#BF 算法
BF 算法中的 BF 是 Brute Force 的缩写，中文叫作暴力匹配算法，也叫朴素匹配算法。
时间复杂度O(n*m)
![](img/bf.png)

#RK 算法

我们通过哈希算法对主串中的 n-m+1 个子串分别求哈希值，然后逐个与模式串的哈希值比较大小。如果某个子串的哈希值与模式串相等，那就说明对应的子串和模式串匹配了
时间复杂度O(n)
![](img/rk_string.png)
![](img/rk_hash.png)

#BM 算法
##坏字符规则（bad character rule）
![](img/bm_bad_character.png)
根据 si-xi 计算出来的移动位数

##好后缀规则（good suffix shift）

