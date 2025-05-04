## Leet Code 
- https://blog.algomaster.io/p/15-leetcode-patterns


## About String 
字符串的子串必须是连续的，如"abcd"的子串有"ab","bcd"等，但是"ac"不是"abcd"的子串。因此，对于长度为n的字符串，共有1+2+...+n个非空子串，即(n+1)*n/2个，是O(n^2)级别的。



为什么子串的不同种的数量级是O(n^2)?
子串是连续的，所以一共是n+n-1+...+1=n(n+1)/2个非空子串，所以是On^2的

Fibonacci sequence
