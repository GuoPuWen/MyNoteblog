本系列要求：争取一两句话把题目说清楚，力争一句话把思路说清楚，力争一句话把关键点说清楚，关键点指的是上一次做哪里没有想到

# 1. 反转链表

- 题目：给你一个链表，输出这个链表反转的结果
- 思路：三个指针，prev，curr，next

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode prev = null;
        ListNode curr = head;
        while (curr != null) {
            ListNode next = curr.next;
            curr.next = prev;
            prev = curr;
            curr = next;
        }
        return prev;
    }
}

```

# 2.数组中第k大的数

- 题意：topk问题
- 思路：使用快速选择算法，paration函数将该基准数放到正确位置，同时返回该基准数的位置，select函数判断该基准数的位置i与位置k进行比对，判断

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        int n = nums.length;
        select(nums, n - k, 0, n - 1);
        return nums[n - k];
    }

    public void select(int[] nums, int k, int left, int right){
        int i = paration(nums, left, right);
        if(i == k) return;
        else if(i < k)  select(nums, k, i + 1, right);
        else select(nums, k, left, right - 1); 
    }

    public int paration(int[] nums, int left, int right){
        int base = nums[left];
        int i = left, j = right;
        while(i < j){
            while(i < j && nums[j] >= base)    j--;
            while(i < j && nums[i] <= base)    i++;
            if(i < j){
                int temp = nums[i];
                nums[i] = nums[j];
                nums[j] = temp;                
            }
        }
        nums[left] = nums[i];
        nums[i] = base;
        return i;
    }

}
```

# 3.无重复字符的最长子串

- 题意：给你一个字符串，求出最长的连续子串，其中该子串是没有重复自负的
- 思路：滑动窗口，如果该窗口内有有重复字符则该窗口向右滑动，如果没有重复字符则记录，得到最大值
- 要点：使用队列做双端滑动窗口，addFirst向队列首加元素，removeLast向队列尾移除元素；使用while，可能多个重复的值

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        int res = 0;
        Deque<Character> help = new LinkedList<>();
        for(int i = 0;i < s.length();i++){
            // help.addLast(help.charAt(i));
            while(help.contains(s.charAt(i))){
                help.removeLast();
            }
            help.addFirst(s.charAt(i));
            res = Math.max(res, help.size());
        }
        return res;

    }
}
```

# 4.买卖股票的最佳时机

- 题意：在数组中选两个数使得，后面选的数与前面的数的差值最大，该值最小为0
- 思路：设置一个变量用于记录最小值min，遍历数组，当price[i] 大于最小值时可以减记录结果，小于该值时把最小值给min

```java
class Solution {
    public int maxProfit(int[] prices) {
        int min = Integer.MAX_VALUE;
        int res = 0;
        for(int i = 0;i < prices.length;i++){
            if(prices[i] >= min){
                res = Math.max(res, prices[i] - min);
            }else{
                min = prices[i];
            }
        }
        return res;
    }
}
```

# 5.螺旋矩阵

- 题意：按照螺旋的方式将矩阵打印出来
- 思路：设置一个辅助数组用来判断是否遍历过，然后遇到遍历过的需要改变方向，设置方向数组用来控制方向，同时如果res的长度等于数组大小则便利结束

```java
class Solution {
    public List<Integer> spiralOrder(int[][] matrix) {
        List<Integer> res = new ArrayList<>();
        if(matrix == null || matrix.length == 0|| matrix[0].length == 0)  return res;
        int row = matrix.length, colume = matrix[0].length;
        //帮助数组
        int[][] help = new int[row][colume];
        int total = row * colume;
        row = 0;
        colume = 0;
        int[][] dir = new int[][]{{0,1}, {1,0}, {0,-1}, {-1,0}};    //方向数组。核心！！！！
        int dirIndex = 0;
        for(int i = 0;i < total;i++){
            res.add(matrix[row][colume]);
            help[row][colume] = 1;
            //下一个位置
            int nextRow = row + dir[dirIndex][0], nextColume = colume + dir[dirIndex][1];
            //判断是否合法
            if(nextRow < 0 || nextRow >= matrix.length || nextColume < 0 || nextColume >= matrix[0].length || help[nextRow][nextColume] == 1){
                //换方向
                dirIndex = (dirIndex + 1) % 4;
            }
            //如果合法 则下一个位置
            row += dir[dirIndex][0];
            colume += dir[dirIndex][1];
        }
        return res;
    }
}
```

# 6.岛屿数量

- 题意：求出一个矩阵中岛屿的数量
- 思路：岛屿常见的DFS模版问题，关键在于如何求出岛屿数量？dfs遍历遇到1就变为0，DFS的次数就是岛屿的数量

```java
class Solution {
    public int numIslands(char[][] grid) {
        if(grid == null) return 0;
        int count = 0;
        for(int i = 0;i < grid.length;i++){
            for(int j = 0;j < grid[0].length;j++){
                if(grid[i][j] == '1'){
                    dfs(grid, i, j);
                  //岛屿数量
                    count++;
                }
            }
        }
        return count;
    }

    public void dfs(char[][] grid, int i, int j){
      //判断边界情况
        if(i < 0 || i >= grid.length || j < 0 || j >= grid[0].length || grid[i][j] == '0') return;
      //将1变为0
        grid[i][j] = '0';
        dfs(grid, i + 1, j);
        dfs(grid, i - 1, j);
        dfs(grid, i, j + 1);
        dfs(grid, i, j - 1);
    }
}
```

# 7.最长回文子串

- 题意：找出字符串中最长的回文串
- 思路：dp[l] [r]表示从l到r是回文串，关键是怎么转移，当s[l] == s[r]时并且dp[l + 1] [r - 1]是回文串的时候那么dp[l] [r]也是回文串，那么转移方程就好了，同时需要考虑边界情况，当s[l] == s[r]时如果2个以下元素，那么是回文串

```java
class Solution {
    public String longestPalindrome(String s) {
        int[][] dp = new int[s.length()][s.length()];
        int start = 0, end = 0, max = 0;
        for(int r = 1;r < s.length();r++){
            for(int l = 0;l <= r;l++){
                if(s.charAt(l) == s.charAt(r) && (r -l <= 2 || dp[l + 1][r - 1] == 1)){
                    dp[l][r] = 1;
                    if(max < r - l + 1){
                        max = r - l + 1;
                        start = l;
                        end = r;
                    }
                }
            }
        }
        return s.substring(start, end + 1);
    }
}
```

# 8.最长递增子序列

- 题意：找出数组中最长的递增序列
- 思路：dp[i]代表以0，i为序号的最长子序列，求解dp[i]的时候，向前遍历j当遇到num[j]比num[i]更小时，dp[i] = max(dp[i], dp[j] + 1)

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        int[] dp = new int[nums.length];
        dp[0] = 1;
        int max = dp[0];
        for(int i = 1;i < nums.length;i++){
            dp[i] = 1;
            for(int j = 0;j <= i;j++){
                if(nums[j] < nums[i]){
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }
            max = Math.max(dp[i], max);
        }
        return max;
    }
}
```

# 9.最长公共子串

- 题意：在两个字符串中找到一个公共的子串，注意子串是连续的
- 思路：dp[i] [j]表示字符串str1[0, i ]和字符串str2 [0, j]中最长的连续子串长度，那么dp方程为：
  -  如果str[i] == str[j] ， 那么(dp[i] [j] = dp[i - 1] [j - 1] + 1) && (i > 0 && j > 0)
  - 如果str[i]  != str[j]，那么 dp[i] [j] =0
  - 边界情况很好判断，当 i== 0或者 j == 0的时候，如果str[i] == str[j] ，那么dp[i] [j] = 1，否则dp[i] [j] = 0

```java
public class Solution{
      public String LCS (String str1, String str2) {
        // write code here
        if(str1 == null || str2 == null){
            return null;
        }
        int index = 0, max = 0;
        int[][] dp = new int[str1.length()][str2.length()];
        for(int i = 0;i < str1.length();i++){
            for(int j = 0;j < str2.length();j++){
                if(str1.charAt(i) != str2.charAt(j)){
                    dp[i][j] = 0;
                }else{
                    if(i == 0 || j == 0) dp[i][j] = 1;
                    else{
                        dp[i][j] = dp[i - 1][j - 1] + 1;
                    }
                }
                if(dp[i][j] > max){
                    max = dp[i][j];
                    index = i;
                }
            }
        }
        //左边是包含，右边不包含
        return  str1.substring(index - max + 1, index + 1);
    }
}
```

# 10.最长公共子序列

- 题意：在两个字符串中找出最长的公共子序列，子序列的意思就是字符可以不连续，与上一个题目的差距就在于是不连续的
- 思路：同样的建立dp方程，只不过这个时候定义需要改变dp[i] [j] 表示str1[0, i - 1]与str2[0, j - 1]的最长公共子序列，为什么需要改变定义呢？因为当str1[0] ==  str2[0]并且str[1] != str[0]的时候例如ae和ac，dp[0] [1] = 1才对如果定义为str[0, i] 那么dp[0] [1] = 0，定义明白了之后来看dp方程如何推进：
  - 如果str1[ i -1] == str[j - 1]，那么dp[i] [j] = dp[ i - 1] [j - 1] + 1
  - 如果str1[ i -1] != str[j - 1]，那么dp[i] [j] = max(dp[i - 1] [j], dp[i] [j - 1])，因为不需要连续
  - 当dp的定义更改之后无需边界情况了

```java
class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        int[][] dp = new int[text1.length() + 1][text2.length() + 1];
        for(int i = 1;i <= text1.length();i++){
            for(int j = 1;j <= text2.length();j++){
                if(text1.charAt(i - 1) == text2.charAt(j - 1)){
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                }else{
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);   
                }
            }
        }
        return dp[text1.length()][text2.length()];
    }
}
```

# 11.缺失的第一个正数

- 题意：给定一个未排序的数组，找出这个数组中第一个缺失的正整数，要求时间复杂度为O(n)，空间复杂度为O(1)
- 思路：对于使用hash表，这个题目思路很明确，但是使用hash表空间复杂度为O(n)，如果不使用额外的数组，那么时间复杂度为O(n2)，所以这个题目没有办法在时间复杂度为O(n)，空间复杂度为O(1)中实现的，但是可以在原数组中进行修改，这样就相当于使用了O(n)的空间复杂度。将元数组改为hash表，首先需要思考的是对于缺失的第一个正数的值一定出现在[1, N + 1]，N为数组的长度，为N+1是因为[1, N]都出现了。对于hash需要做标记，但是应该如何做上标记呢？可以先将整个数组中小于0的数变为N +1，接着遍历数组，如果值为x，那么将nums[x - 1] = -nums[x - 1]，注意不能重复了，最后判断数组中大于0的位置，那么就是缺失的第一个正数，否则就是N + 1

```java
class Solution {
    public int firstMissingPositive(int[] nums) {
        int n = nums.length;
        for(int i = 0;i < nums.length;i++){
            if(nums[i] <= 0){
                nums[i] = nums.length + 1;
            }
        }
        for(int i = 0;i < nums.length;i++){
            int val = Math.abs(nums[i]);
            if(val < n + 1){
                nums[val - 1] = -Math.abs(nums[val - 1]);
            }
        }
        for(int i = 0;i < nums.length;i++){
            if(nums[i] > 0){
                return i + 1;
            }
        }
        return n + 1;
    }
}
