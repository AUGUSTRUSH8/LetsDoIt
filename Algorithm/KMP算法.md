### KMP算法

#### kmp算法背景

具体的背景可以参看leetcode 28题的描述

#### 什么是kmp算法

通过前缀表的预处理,加速模式串移动的速度,达到时间复杂度的缩减.

#### kmp算法的直观解释

[跳转链接](<http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html>)

#### 利用反证法证明kmp算法的有效性

[跳转链接](<https://blog.csdn.net/qq_43188744/article/details/102505839>)

#### 根据kmp算法的思想进行编码

```java
//创建最大前缀后缀数组
public static int[] createPartMatchTable(String needle) {
        int[] res =new int[needle.length()];
        for (int i = 0; i < needle.length(); i++) {
            int prefixLength = i;
            while (prefixLength > 0) {
                if (needle.substring(0, prefixLength).equals(needle.substring(i +1 - prefixLength, i+1))) {
                    break;
                }
                prefixLength--;
            }
            if (prefixLength > 1) {
                res[i]=prefixLength;
            }else {
                if (i >= 1) {
                    if (needle.charAt(0) == needle.charAt(i)) {
                        res[i]=1;
                    }
                }
            }
        }
        return res;
    }
    
//利用前后缀备查数组进行字符串匹配
    public static int strStr(String haystack, String needle) {
        if (needle.isEmpty()) {
            return 0;
        }
        if (haystack.isEmpty()) {
            return -1;
        }
        if (haystack.length() < needle.length()) {
            return -1;
        }
        int[] lookuptable = createPartMatchTable(needle);
        int haysatckIndex=0;
        int needleIndex=0;
        while (haysatckIndex < haystack.length()-needle.length()+1) {
            while (needleIndex < needle.length()) {
                if (haystack.charAt(haysatckIndex + needleIndex) != needle.charAt(needleIndex)) {
                    break;
                }
                needleIndex++;
            }
            if (needleIndex == needle.length()) {
                return haysatckIndex;
            } else if (needleIndex == 0) {
                haysatckIndex++;
            } else {
                haysatckIndex += needleIndex - lookuptable[needleIndex - 1];
                needleIndex = lookuptable[needleIndex - 1];
            }
        }
        return -1;
    }
```

