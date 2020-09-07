# DFS

## 连通器

#### 岛屿最大面积

``` c++
/*
 * @lc app=leetcode.cn id=695 lang=cpp
 *
 * [695] 岛屿的最大面积
 */

// @lc code=st   art
class Solution {
public:

    int maxAreaOfIsland(vector<vector<int>>& grid) {
        int row=grid.size();
        int col=grid[0].size();
        int res=0;
        for(int i=0;i<row ;i++){
             for(int j=0;j<col ;j++){
                 if(grid[i][j]==1 )
                   res=max(res,dfs(grid,i,j,0 ));
                 }
        }
        return res;
    }
         
int dfs(vector<vector<int>>& grid,int i,int j,int cnt) {
         grid[i][j]=0;
        cnt++;
        if(i-1>=0&&grid[i-1][j]==1){
             cnt=dfs(grid,i-1,j,cnt);
           }
        if(i+1<grid.size ()&&grid[i+1][j]==1){
            cnt= dfs(grid,i+1,j,cnt);
           } 
        if(j-1>=0&&grid[i][j-1]==1){
            cnt= dfs(grid,i,j-1,cnt);
           }
        if(j+1<grid[0].size()&&grid[i][j+1]==1){
             cnt=dfs(grid,i,j+1,cnt);
             }
            return cnt;
    }
};
 
 

```

#### 朋友圈

``` c++
/*
 * @lc app=leetcode.cn id=547 lang=cpp
 *
 * [547] 朋友圈
 */

// @lc code=start
 class Solution {
 public:
     void DFS(vector<vector<int>>& M,vector<int> &visit,int n){
         visit[n]=1;
         for(int j=0;j<M.size();++j)
             if(M[n][j] && !visit[j])
                 DFS(M,visit,j);
     }
     int findCircleNum(vector<vector<int>>& M) {
         int N=M.size();
         vector<int> visit(M.size(),0);
         int count=0;
         for(int i=0;i<N;++i){
             if(!visit[i]){
                 DFS(M,visit,i);
                 count++;
             }
         }
         return count;
     }
 };

// @lc code=end

//M矩阵用于查找谁是谁的好友
//建立一维vector，先找第一个人的朋友圈，把第一个人的朋友圈都标记

```

#### 130填充封闭区域

``` c++
//从四边找到不符合的
//遍历，把visit作为判断条件进行修改
```

131.分割字符串得到回文字符

``` c++
//substr截取字符串
class Solution {
public:
    vector<vector<string>> res;
    vector<vector<string>> partition(string s) {
        if(s.empty())
            return res;
        vector<string> path;
        dfs(s, 0, path);
        return res;
    }
    void dfs(string str,int pos,vector<string> path)
    {
        if(pos>=str.size())
        {
             res.push_back(path);
             return;
        }
        for (int i = 1; i+pos <= str.size();i++)
        {
            string tmp;
            tmp = str.substr(pos, i);
            if(isPalindrome(tmp))
            {
                path.push_back(tmp);
                dfs(str, pos + i, path);
                path.pop_back();
            }
        }
        return;
    }

    bool isPalindrome(string str)
    {
        int low = 0, high = str.size() - 1;
        while(low<=high)
        {
            if(str[low]!=str[high])
                return false;
            low++;
            high--;
        }
        return true;
    }
};
```

