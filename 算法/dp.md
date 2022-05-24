## 剑指offer 12.矩阵中的路径

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220309090118359.png" alt="image-20220309090118359" style="zoom:50%;" />

- 题解：我们可以参考左神的dp思路，只需要关心一开始的情况，各种情况会返回什么值就可以。同时在截枝方面，可以将判断条件同意在一起，避免重复写大量的判断条件。

  ![image-20220309090619490](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220309090619490.png)

```java
class Solution {
    public boolean exist(char[][] board, String word) {
        int i_size = board.length;
        int j_size = board[0].length;
        boolean res = false;
        for(int i=0;i<i_size;i++){
            for(int j=0;j<j_size;j++){
                if(board[i][j]==word.charAt(0)){
                    if(dp(board,word,i,j,0)){
                        return true;
                    }
                }
            }
        }
        return res;
    }
    public boolean dp(char[][] board, String word,int cur_i,int cur_j,int index){
        if(cur_i<0 || cur_i>board.length-1 || cur_j<0 || cur_j>board[0].length-1 || board[cur_i][cur_j] != word.charAt(index)) 
          return false;
        if(index==word.length()-1)
            return true;
        board[cur_i][cur_j] = '0';
        boolean dfs = dp(board,word,cur_i,cur_j-1,index+1) || dp(board,word,cur_i,cur_j+1,index+1) ||dp(board,word,cur_i-1,cur_j,index+1) || dp(board,word,cur_i+1,cur_j,index+1);
        board[cur_i][cur_j] = word.charAt(index);
        return dfs;
    }
}
```

```java
//改进前的代码
public boolean dp(char[][] board, String word,int cur_i,int cur_j,int index){
  if(index==word.length())
    return true;
  boolean res;
  if(board[cur_i][cur_j+1]!='0'&&board[cur_i][cur_j+1]==word.charAt(index)){
    char temp = board[cur_i][cur_j+1];
    board[cur_i][cur_j+1]='0';
    res = dp(board,word,cur_i,cur_j+1,index+1);
    board[cur_i][cur_j+1]=temp;

  }else if(board[cur_i][cur_j-1]!='0'&&board[cur_i][cur_j-1]==word.charAt(index)){
    char temp = board[cur_i][cur_j-1];
    board[cur_i][cur_j-1]='0';
    res = dp(board,word,cur_i,cur_j-1,index+1);
    board[cur_i][cur_j-1]=temp;
  }else if(board[cur_i+1][cur_j]!='0'&&board[cur_i+1][cur_j]==word.charAt(index)){
    char temp = board[cur_i+1][cur_j];
    board[cur_i+1][cur_j]='0';
    res = dp(board,word,cur_i+1,cur_j,index+1);
    board[cur_i+1][cur_j]=temp;
  }else if(board[cur_i-1][cur_j]!='0'&&board[cur_i-1][cur_j]==word.charAt(index)){
    char temp = board[cur_i-1][cur_j];
    board[cur_i-1][cur_j]='0';
    res = dp(board,word,cur_i+1,cur_j,index+1);
    board[cur_i-1][cur_j]=temp;
  }else{
    return false;
  }
  return res;
}
```



## 剑指offer 13.机器人的运动范围

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220309093808097.png" alt="image-20220309093808097" style="margin-left:0px;zoom:50%;" />

- 深度优先搜索

  ```java
  class Solution {
      public int movingCount(int m, int n, int k) {
          boolean[][] visit = new boolean[m][n];
          return dfs(visit,0,0,m,n,k);
  
      }
      public int digit_sum(int n){
          int count=0;
          while(n!=0){
              count += n%10;
              n = n/10;
          }
          return count;
      }
      public int dfs(boolean[][] visit, int i,int j,int m,int n,int k){
        	//关键还是理解这个截枝条件
        	//很明显，这个条件是在当前结点作出的判断
          if(i<0 || i>m-1 || j<0 || j>n-1 || digit_sum(i)+digit_sum(j)>k || visit[i][j]){
              return 0;
          }
          visit[i][j] = true;
          return 1+dfs(visit,i-1,j,m,n,k)+dfs(visit,i+1,j,m,n,k)+dfs(visit,i,j-1,m,n,k)+dfs(visit,i,j+1,m,n,k);
  
      }
  }
  ```

  

- 广度优先搜索

