---
title: 全排列去重
date: 2024-09-22
categories:
  - Algorithm
nolastmod: true
cover: /img/pic-1.jpg
---
链接：https://www.acwing.com/problem/content/47/
输入一组数字（可能包含重复数字），输出其所有的排列方式。

如果是不重复的，就是一个常规的深度优先遍历，当可重复时，我们需要人为的区分重复数字，做法是指定重复数字首次要放入某个框位时，只使用重复数字的第一个。
```java
class Solution {
    public List<List<Integer>> result = new ArrayList<>();
    public List<List<Integer>> permutation(int[] nums) {
        // 排序
        Arrays.sort(nums);
        int n = nums.length;
        boolean[] used = new boolean[n];
        List<Integer> temp = new LinkedList<>();
        dfs(nums,used,0,temp);
        return result;
    }
    public void dfs(int[] nums,boolean[] used,int index,List<Integer> temp){
        int n = nums.length;
        if(index==n){
            result.add(new ArrayList<>(temp));
            return;
        }
        for(int i=0;i<n;i++){
            // 过滤
            if(i>0&&nums[i]==nums[i-1]&&!used[i-1])continue;
            if(!used[i]){
                temp.add(nums[i]);
                used[i]=true;
                dfs(nums,used,index+1,temp);
                temp.remove(temp.size()-1);
                used[i]=false;
            }
        }
    }
}
```