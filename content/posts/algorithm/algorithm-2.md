---
title: 骰子的点数
date: 2024-09-24
categories:
  - Algorithm
nolastmod: true
cover: /img/pic-2.jpg
draft: true
---
### description
链接：https://www.acwing.com/problem/content/76/
将一个骰子投掷n次，获得的总点数为s，s的可能范围为n～6n。求出投掷n次，掷出n～6n
点分别有多少种掷法。

### code
拿到题目我首先想到的是回溯：
```java
class Solution {
    public Map<Integer,Integer> map = new HashMap<>();
    public int[] numberOfDice(int n) {
        int index = 0;
        dfs(n,0);
        int N = n*6-n+1;
        int[] result = new int[N];
        for(int i=0;i<N;i++){
            result[i]=map.get(n+i);
        }
        return result;
    }
    public void dfs(int n,int temp){
        if(n==0){
            map.put(temp,map.getOrDefault(temp,0)+1);
            return;
        }
        for(int i=1;i<=6;i++){
            dfs(n-1,temp+i);
        }
    }
}
```
链接：https://www.acwing.com/solution/content/80458/
后面看到递归的写法：
```java
class Solution {
    public int[][] dict;
    public int[] numberOfDice(int n) {
        List<Integer> result = new ArrayList<>();
        for(int i=n;i<=6*n;i++){
            result.add(dfs(n,i));
        }
        return result.stream().mapToInt(Integer::intValue).toArray();
    }
    public int dfs(int n,int sum){
        if(sum<0)return 0;
        if(n==0){
            return sum==0?1:0;
        }
        int ans = 0;
        for(int i=1;i<=6;i++){
            ans+=dfs(n-1,sum-i);
        }
        return ans;
    }
}
```
上面这种做法会有很多重复计算，于是采用下面记忆化搜索的方式优化：
```java
class Solution {
    public int[][] dict;
    public int[] numberOfDice(int n) {
        dict = new int[n+1][6*n+1];
        List<Integer> result = new ArrayList<>();
        for(int[] row:dict){
            Arrays.fill(row,-1);
        }
        for(int i=n;i<=6*n;i++){
            result.add(dfs(n,i));
        }
        return result.stream().mapToInt(Integer::intValue).toArray();
    }
    public int dfs(int n,int sum){
        if(sum<0)return 0;
        if(n==0){
            return sum==0?1:0;
        }
        if(dict[n][sum]!=-1)return dict[n][sum];
        int ans = 0;
        for(int i=1;i<=6;i++){
            ans+=dfs(n-1,sum-i);
        }
        dict[n][sum]=ans;
        return ans;
    }
}
```
dp写法：
```java
class Solution {
    public int[] numberOfDice(int n) {
        int[][] dp = new int[n+1][6*n+1];
        dp[0][0]=1;
        // 投掷次数从1开始
        for(int i=1;i<=n;i++){
            // sum为0不存在，所以从1开始
            for(int j=1;j<=6*n;j++){
                // 有1~6个点数
                for(int k=1;k<=6;k++){
                    if(j>=k){
                        dp[i][j]=dp[i][j]+dp[i-1][j-k];
                    }
                }
            }
        }
        int[] result = new int[6*n-n+1];
        for(int i=n;i<=6*n;i++){
            result[i-n]=dp[n][i];
        }
        return result;
    }
}
```
### summery
1. 经典的递归优化
2. List转int[]可以使用这种写法：`list.stream().mapToInt(Integer::intValue).toArray()`