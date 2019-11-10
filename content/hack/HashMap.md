---
title: LeetCode | 利用HashMap实现O(n)时间的算法
date: 2017-12-08 22:11:44
showDate: true
tags: ["LeetCode", "HashMap"] 
mathjax: true
---
一日，$E^{6}X$和我说，“你看着越来越没有程序猿气质了”。我顿时感到警觉，翻了翻之前的Blog，尽是书评影评小说摘抄，像初三女生的QQ空间，连非主流自拍都没有的那种。摸摸头顶，与宇宙演进截然相反的熵减趋势时刻提醒我作为码农的阶级属性。

“得写点技术博客了。”我想。

<!--more-->

这篇算是个开头，至于能写多少、内容深度，主要取决于我和懒癌斗争的结果。我估摸主要还是记录下我做过的算法题。

---

LeetCode大名鼎鼎的高频题2Sum很可能是大部分人AC的第一道题目，它的核心之处在于利用`HashMap`实现$O(n)$时间复杂度的算法，通过对key-value pair的巧妙设置，达到每向前遍历一步，都能利用map查看之前所得结果的效果。在这里，把这类题目整理一下：

## [325.Maximum Size Subarray Sum Equals k](https://leetcode.com/problems/maximum-size-subarray-sum-equals-k/description/)

问题： 求sum=k的最长子数组的长度。

`HashMap` structure: `<sum of all elements from 0 to index i, i>`

Solution：若当前sum为k，更新结果max；若map中存在key为sum-k，说明之前存在子数组sum_pre，使得sum_cur - sum_pre = k，更新结果max。若map中不存在当前sum，将其添加到map中。

```java
public int maxSubArrayLen(int[] nums, int k) {
        
        int sum = 0;
        int max = 0;
        
        //map stores the key-value pair of <sum of all elements from 0 to index i, i>
        HashMap<Integer, Integer> map = new HashMap<>();
        
        for(int i = 0;i < nums.length;i++)
        {
            sum += nums[i];
            
            if(sum == k) max = i+1; //return the length of current array index
            else if(map.containsKey(sum-k)) //sum - (sum-k) = k, 减掉之前的某个sum得到k
            {
                max = Math.max(max, i - map.get(sum-k));
            }
            
            if(!map.containsKey(sum)) map.put(sum, i);  //子数组有相同sum的话，取index小的存入map，这样结果的长度最长
        }
        
        return max;
        
    }
```

---

