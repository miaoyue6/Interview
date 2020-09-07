# BFS

//向四周进行扩散，一层一层，利用队列；必要时使用visited记录是否遍历过；

``` c++
279.//最短路径，一层一层的找
class Solution {
public:
    int numSquares(int n) {
        vector<int> circle;
        
        for (int i = 1; i <= n;i++){
            if(i*i<=n)
                circle.push_back(i * i);
            if(i*i>n)
                break;
        }
        int count = circle.size();

        queue<int> que;
        que.push(n);
        vector<int> sum(count,n);
        int step = 1;

        while(!que.empty()){
            int len = que.size();
            for (int i = 0; i < len;i++){
                int cur = que.front();
                que.pop();
                if(cur==0||find(circle.begin(),circle.end(),cur)!=circle.end())
                    return step;
                for (int i = 0; i < count;i++){
                    int sum;
                    sum = cur - circle[i];
                    que.push(sum);
                }
            }
            step++;
        }
        return -1;
    }
};
```

``` c++
127.
    /*
 * @lc app=leetcode.cn id=127 lang=cpp
 *
 * [127] 单词接龙
 */

// @lc code=start
class Solution {
public:
	bool connect(const string& word1,const string &word2)//判断两个单词是否仅一个字母不同
	{
		int dif=0; //不同的字母数量
		for(int i=0;i<word1.length();i++)
			if(word1[i]!=word2[i]) 
				dif++;
		return dif==1;
	 } 
	//构建图,graph必须为引用 
	void construct_graph(string &beginWord,vector<string> &wordList,unordered_map<string,vector<string> > &graph)
	{
		wordList.push_back(beginWord);
		for(int i=0;i<wordList.size();i++)
		for(int j=i+1;j<wordList.size();j++)//避免重复
		{
			if(connect(wordList[i],wordList[j])==true)
			{
				graph[wordList[i]].push_back(wordList[j]);
				graph[wordList[j]].push_back(wordList[i]);
			}
		} 	
	} 
    int ladderLength(string beginWord, string endWord, vector<string>& wordList) 
    {
    	unordered_map<string,vector<string> > graph;
    	construct_graph(beginWord,wordList,graph);
        queue<string> Q;
        unordered_set<string> visit;//记录已经访问过的元素
		Q.push(beginWord); 
		visit.insert(beginWord);
		int result=1;
		while(!Q.empty())
		{
			int size=Q.size();
			for(int i=0;i<size;i++) //每一次循环代表宽度增1 
			{
				string tmp=Q.front();
				Q.pop();
				vector<string> neighbors=graph[tmp]; //邻居们
				for(int i=0;i<neighbors.size();i++)
				{
					if(visit.find(neighbors[i])==visit.end())  //还没加入 
					{
						Q.push(neighbors[i]);
						visit.insert(neighbors[i]);
					}
                    if(neighbors[i]==endWord) return result+1;
				 }			
			} 
            result++;  //遍历一层则+1		
		} 
		return 0;
		 
    }
};

// @lc code=end


```

