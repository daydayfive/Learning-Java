[toc]
# lecode 字符串题解


## 两个字符串的最长公共字串

问题：有两个字符串str1和str2，求出两个字符串最长公共子串长度。
比如：str1=acbcbcef,str2=abcbced,则str1和str2的最长公共字串长度为bcbce，最长公共子串长度为5.
```java
 class Solution {

    public static void main(String[] args) {
        String str1="acbcbcef";
        String str2="abcbced";

        System.out.println(getMaxLen(str1,str2));
    }

    public static int getMaxLen(String str1,String str2){
        int len1=str1.length();
        int len2=str2.length();
        int[][] dp=new int[len1+1][len2+1];
        int len=0;
        for(int i=1;i<=len1;i++){
            for(int j=1;j<=len2;j++){
                if(str1.charAt(i-1)==str2.charAt(j-1)){
                    dp[i][j]=dp[i-1][j-1]+1;
                    len=Math.max(dp[i][j],len);
                }else {
                    dp[i][j]=0;
                }
            }
        }
        return len;
    }
}
```

若是想知道原本的公共子串，即可用一个变量存储最长时所处的索引。


## 1143. 最长公共子序列
给定两个字符串 text1 和 text2，返回这两个字符串的最长 公共子序列 的长度。如果不存在 公共子序列 ，返回 0 。

一个字符串的 子序列 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。

例如，"ace" 是 "abcde" 的子序列，但 "aec" 不是 "abcde" 的子序列。
两个字符串的 公共子序列 是这两个字符串所共同拥有的子序列。

```java
class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        int len1=text1.length();
        int len2=text2.length();

        int[][] dp=new int[len1+1][len2+1];
        int len=0;
        for(int i=1;i<=len1;i++){
            for(int j=1;j<=len2;j++){
                if(text1.charAt(i-1)==text2.charAt(j-1)){
                    dp[i][j]=dp[i-1][j-1]+1;
                    len=Math.max(dp[i][j],len);
                }else{
                    dp[i][j]=Math.max(dp[i-1][j],dp[i][j-1]);
                }
            }
        }

        return len;
    }
}
```


## 最长上升子序列







## 字符串匹配
给定两个字符串 s1 和 s2，返回匹配字符串的开始下表，未匹配返回-1.

### 暴力法解
```java
public class Solution {
    public static void main(String[] args) {
        String s1="abccabcdabcf";
        String s2="abcd";

    }
    // 返回符合匹配的下标
    public static int stringMatch(String s1,String s2){
        int len1=s1.length();
        int len2=s2.length();
        int i=0;
        int j=0;
        if(len1<len2) return -1;
        while(j<len2&&i<len1){
            if(s1.charAt(i)==s2.charAt(j)){
                i++;
                j++;
            }else{
                i=i-j+1;
                j=0;
            }
        }

        if(j==len2)
            return i-j;

        return -1;


    }
}
```

### kmp算法解
```java
public class KMP {
    public static void main(String[] args) {
        String s1="abccabcabcdabcf";
        String s2="abcd";
        System.out.println(kmp(s1,s2));
    }


    public static int kmp(String s1,String s2){
        int len1=s1.length();
        int len2=s2.length();
        int[] next=getNexts(s2);

        if(len1<len2) return -1;

        int j=0;
        for(int i=0;i<len1;i++){
            while(j>0&&s1.charAt(i)!=s2.charAt(j)){
                j=next[j-1]+1;
            }
            if(s1.charAt(i)==s2.charAt(j)){
                j++;
            }


            if(j==len2){
                return i-j+1;
            }
        }



        return -1;
    }

    // 找到模式串的失效函数,
    // next数组存储了模式串每个前缀的最长可匹配前缀子串的下标。
    public static int[] getNexts(String s2){
        int len=s2.length();
        int[] next=new int[len];
        next[0]=-1;
        int k=-1;
        for(int i=1;i<len;i++){
            while(k!=-1&&s2.charAt(k+1)!= s2.charAt(i)){
                k=next[k];
            }

            if(s2.charAt(k+1)==s2.charAt(i)){
                k++;
            }

            next[i]=k;
        }


        return next;

    }
}

```