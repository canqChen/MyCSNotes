# 数学

## [计数质数](https://leetcode-cn.com/problems/count-primes/)

## 判断一个数是否2的幂次方

* 2的幂次方的数的二进制表示，都是只有一个1，后面跟n个0，所以只需判断num & (num-1)是否等于0即可

## [分数到小数](https://leetcode-cn.com/problems/fraction-to-recurring-decimal/)

```c++
//小数部分如果余数重复出现两次就表示该小数是循环小数了
string fractionToDecimal(int numerator, int denominator) {
    if(denominator == 0) return "";//边界条件，分母为0
    if(numerator == 0) return "0";//边界条件，分子为0
    string result;

    //转换为longlong防止溢出
    long long num = static_cast<long long>(numerator);
    long long denom = static_cast<long long>(denominator);

    //处理正负号，一正一负取负号
    if((num>0)^(denom>0))
        result.push_back('-');

    //分子分母全部转换为正数
    num=llabs(num);
    denom=llabs(denom);

    //处理整数部分
    result.append(to_string(num/denom));

    //处理小数部分
    num %= denom;                         //获得余数
    if(num == 0)
        return result;             //余数为0，表示整除了，直接返回结果
    result.push_back('.');              //余数不为0，添加小数点
    int index = result.size() - 1;          //获得小数点的下标
    unordered_map<int, int> record;      //map用来记录出现重复数的下标，然后将'('插入到重复数前面就好了
    while(num && record.count(num) == 0) {   //小数部分：余数不为0且余数还没有出现重复数字
        record[num] = ++index;
        num *= 10;                        //余数扩大10倍，然后求商，和草稿本上运算方法是一样的
        result += to_string(num/denom);
        num %= denom;
    }
    if(record.count(num) == 1) {           //出现循环余数，我们直接在重复数字前面添加'(',字符串末尾添加')'
        result.insert(record[num],"(");
        result.push_back(')');
    }
    return result;
}
```

## [两数相除](https://leetcode-cn.com/problems/divide-two-integers/)

## pow(x, n)快速幂

```c++
double myPow(double x, int n) {
    bool nagative = n < 0;
    if(n == 0) {
        return 1;
    }
    n = abs(n);
    double ret = 1;
    while(n > 0) {
        if(n & 1) {
            ret *= x;
        }
        x *= x;
        n /= 2;
    }
    if(nagative) {
        return 1 / ret;
    }
    return ret;
}
```

# 数组

## 表示数值的字符串

* 题目描述

  * 请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串"+100","5e2","-123","3.1416"和"-1E-16"都表示数值。 但是"12e","1a3.14","1.2.3","+-5"和"12e+4.3"都不是

* 代码实现

  ```c++
  bool isNumeric(char* string)
  {
      char *s = string;
      while(*s==' ')
          s++;
      if(*s=='+'||*s=='-')
          s++;
      int pointNum=0;
      int num=0;
      while(*s>='0' && *s<='9' || *s=='.'){
          *s=='.' ? pointNum++:num++;
          s++;
      }
      // 如果小数点的数目大于1  或者数字为0,则不是数字（包含全空格）
      if(num < 1 || pointNum > 1) {
          return false;
      }   
      //科学计数法
      if(*s=='e'||*s=='E'){   
          s++;
          if(*s=='+'||*s=='-'){
              s++;
          }
          // 统计e后面数字，e后面只能包含数字
          int eNum=0;  
          while(*s>='0' && *s<='9'){
              eNum++;
              s++;
          }
          if(eNum==0) {
           	return false;   
          }
      }
      // 去除后面的空格
      while(*s==' '){
          s++;
      }
      // 没到字符串结尾，必然不是合法数字表示
      return *s=='\0';
  }
  ```

# 排序

## [最大数](https://leetcode-cn.com/problems/largest-number/)

* 思路

  * 要想组成最大的整数，**一种直观的想法是把数值大的数放在高位**。于是我们**可以比较输入数组的每个元素的最高位，最高位相同的时候比较次高位，以此类推，完成排序，然后把它们拼接起来**。这种排序方式对于输入数组 没有相同数字开头的时候是有效的

  <img src="imgs/algorithm/max_num.jpg" alt="max_num" style="zoom: 80%;" />

* 代码实现

  ```c++
  string largestNumber(vector<int>& nums) {
      // 
      auto comp = [](int & x, int & y)->bool{
          long sx = 10, sy = 10;
          while(sx <= x){
              sx *= 10;
          }
          while(sy <= y){
              sy *= 10;
          }
          return x*sy + y > y*sx + x;
      };
  
      sort(nums.begin(), nums.end(), comp);
      if(nums[0] == 0){
          return "0";
      }
      string ret;
      for(auto &num: nums){
          ret += to_string(num);
      }
      return ret;
  }
  ```

## 数组中的逆序对

* 题目描述

  * 在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组,求出这个数组中的逆序对的总数P。并将P对1000000007取模的结果输出。 即输出P%1000000007

* 解题思路

  * 利用归并排序，自底向上归并过程，如果出现待归并左子数组中left下标数字大于右子数组中right下标数字，由于左右数组都是正序，则左子数组left右边的数都与right所指的数组成逆序对，逆序对总数增加mid-left+1对

* 代码实现

  ```c++
  class Solution {
  public:
      int count = 0;
      int InversePairs(vector<int> data) {
          if(data.empty())
              return 0;
          vector<int> tmp (data);
          division(data, tmp, 0, data.size() - 1);
          return count;
      }
       
      void division(vector<int> & array, vector<int> & tmp, int left, int right) {
          if(left >= right) {
              return;
          }
          int mid = left + (right - left) / 2;
           
          division(array, tmp, left, mid);
          division(array, tmp, mid+1, right);
   
          int idx = left, rightIdx = mid+1, leftIdx = left;
          // merge
          while(leftIdx <= mid && rightIdx <= right){
              if(array[leftIdx] <= array[rightIdx]){
                  tmp[idx++] = array[leftIdx++];
              }
              else { 
                  // 逆序，则由于array[left...mid]已经是正序，
                  // left右边都与array[right]组成逆序对
                  count += mid - leftIdx + 1;
                  count %= 1000000007;
                  tmp[idx++] = array[rightIdx++];
              }
          }
          while(leftIdx <= mid)
              tmp[idx++] = array[leftIdx++];
          while(rightIdx <= right)
              tmp[idx++] = array[rightIdx++];
          for(int i = left; i <= right; ++i)
              array[i] = tmp[i];
      }
  };
  ```

# 快速选择(Quick Select)

## [数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

<img src="imgs/algorithm/k-th.jpg" alt="k-th" style="zoom:80%;" />

* 代码实现

  ```c++
  int findKthLargest(vector<int>& nums, int k) {
      int n = nums.size();
      int left = 0, right = n-1;
      int ret = 0;
      int target = n - k;
      while(left <= right) {
          int idx = patition(nums, left, right);
          if(idx == target) {
              return nums[target];
          }
          else if(idx < target) {
              left = idx + 1;
          }
          else {
              right = idx - 1;
          }
      }
      return ret;
  }
  
  int patition(vector<int> & nums, int left, int right){
      int pivot = nums[left];
      while(left<right) {
          while(left<right && nums[right] >= pivot)
              right--;
          nums[left] = nums[right];
          while(left<right && nums[left]<=pivot)
              left++;
          nums[right] = nums[left];
      }
      nums[left] = pivot;
      return left;
  }
  ```

# 分治

## [至少有 K 个重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-with-at-least-k-repeating-characters/)

* 思路：

  * 遍历找到哪些字符出现次数少于k
  * 再次遍历找到哪些不满足条件的位置（就是字符出现次数少于k），就把字符串拆成更小的一部分
  * 更小一部分里找满足条件的长度
  * 调用结束：找不到任何不满足条件位置，就是当前字符串整个就是满足条件的
  * 返回结果： 在每一部分调用里记录最大的长度，就是结果

* 代码实现

  ```c++
  class Solution {
  public:
      int longestSubstring(string s, int k) {
          int n = s.size();
          if(k == 1) {
              return n;
          }
          int maxLen = 0;
          dfs(s, 0, n-1, maxLen, k);
          return maxLen;
      }
  
      void dfs(string &s, int left, int right, int &maxLen, int k) {
          if(left > right) {
              return;
          }
          int len = right - left + 1;
          // 剪枝
          if(maxLen >= len || len < k) {
              return;
          }
          // 统计当前字串字符类型及数量
          vector<int> count(26);
          for(int i = left; i <= right; ++i) {
              ++count[s[i] - 'a'];
          }
          // 寻找分割点(字符数<k)
          int sep = -1;
          for(int i = left; i <= right; ++i) {
              if(count[s[i] - 'a'] < k) {
                  sep = i;
                  break;
              }
          }
          // 不存在分割点，字串满足条件，更新最大长度
          if(sep == -1) {
              maxLen = max(maxLen, len);
              return;
          }
          // 根据分割点分治
          dfs(s, left, sep - 1, maxLen, k);
          dfs(s, sep + 1, right, maxLen, k);
      }
  };
  ```


# 链表

## [ K 个一组翻转链表](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/)

* 代码实现

  ```c++
  ListNode* reverseKGroup(ListNode* head, int k) {
      if(head == nullptr || k <= 1) {
          return head;
      }
      int nodeNum = 0;
      ListNode * cur = head;
  	// 统计节点总数
      while(cur != nullptr) {
          ++nodeNum;
          cur = cur->next;
      }
      // 有多少组节点需要翻转
      int group = nodeNum / k;
      cur = head;
      ListNode * prev = nullptr, *next = nullptr, dum;
      dum.next = head;	// dummy头，方便处理
      ListNode * lastTail = &dum;	// 记录上一组的尾节点
      while(group--) {
          ListNode * tmp = cur;
          prev = nullptr;	// 需要置为nullptr，使得每组翻转后尾节点为nullptr
          // reverse
          for(int i = 0; i < k; ++i) {
              next = cur->next;
              cur -> next = prev;
              prev = cur;
              cur = next;
          }
          // 将上一组尾节点连接到本组首节点
          lastTail->next = prev;
          // 更新尾节点为本组尾节点
          lastTail = tmp;
          // 连接下组首节点
          lastTail->next = cur;
      }
      return dum.next;
  }
  ```

## 拷贝有随机指针的链表

* 代码实现

  ```c++
  class Solution {
  public:
      RandomListNode *copyRandomList(RandomListNode *head) {
          if(head==nullptr) return nullptr;
          cloneNodes(head);
          reconnectRandomNodes(head);
          return splitList(head);
      }
      void cloneNodes(RandomListNode * head)
      {
          while(head!=nullptr)
          {
              RandomListNode * tmp = new RandomListNode(head->label);
              tmp->next = head->next;
              head->next = tmp;
              head = tmp->next;
          }
      }
      void reconnectRandomNodes(RandomListNode *head)
      {
          while(head!=nullptr)
          {
              if(head->random!=nullptr)
                  head->next->random = head->random->next;
              else
                  head->next->random = nullptr;
              head = head->next->next;
          }
      }
      RandomListNode * splitList(RandomListNode*head)
      {
          RandomListNode * newHead = head->next;
          RandomListNode *cur = newHead->next;
          head->next = cur;
          RandomListNode *newCur = newHead;
          while(cur!=nullptr)
          {
              newCur->next = cur->next;
              newCur = newCur->next;
              cur->next = newCur->next;
              cur = cur->next;
          }
          return newHead;
      }
  };
  ```


# 栈

## [字符串解码](https://leetcode-cn.com/problems/decode-string/)

> 给定一个经过编码的字符串，返回它解码后的字符串。编码规则为: k[encoded_string]，表示其中方括号内部的 encoded_string 正好重复 k 次。注意 k 保证为正整数。你可以认为输入字符串总是有效的；输入字符串中没有额外的空格，且输入的方括号总是符合格式要求的。此外，你可以认为原始数据不包含数字，所有的数字只表示重复的次数 k ，例如不会出现像 3a 或 2[4] 的输入

* 解题思路：

  * 本题难点在于括号内嵌套括号，需要**从内向外**生成与拼接字符串，这与栈的**先入后出**特性对应
  * 构建辅助栈 `stk`， 遍历字符串 `s` 中每个字符 `c`
    * 当 `c` 为数字时，将数字字符转化为数字 `num`，用于后续倍数计算
    * 当 `c` 为字母时，在 `ret` 尾部添加 `c`
    * 当 `c` 为 `[` 时，将当前 `num`和 `ret` 入栈，并分别置空置0，用于重新记录
      * 记录此`[` 前的临时结果 `ret` 至栈，用于发现对应 `]` 后的拼接操作；
      * 记录此 `[ `前的倍数 `num`至栈，用于发现对应 `]` 后，获取 `num`× [...] 字符串
      * 进入到新 `[` 后，`ret` 和` num` 重新记录
    * 当 `c` 为 `] `时，`stk` 出栈，拼接字符串 `ret = last_ret + num* ret`，其中:
    * last_ret 是上个 [ 到当前 [ 的字符串，例如 "3[a2[c]]" 中的 a；
    * `num`是当前 `[ `到 `]` 内字符串的重复倍数，例如 "3[a2[c]]" 中的 2
    * 返回字符串 `ret`

* 代码实现

  ```c++
  string decodeString(string s) {    
      int n = s.size();    
      string ret, next;    
      stack<pair<int, string> > stk;    
      int num = 0;    
      for(int i = 0; i < n; i++) {        
          if(s[i]<='9' && s[i]>='0'){           
              num = num*10 + s[i]-'0';        
          }        
          else if(s[i] == '[') {            
              stk.emplace(num, ret);
              num = 0;	// 置零置空
              ret = "";
          }        
          else if(s[i] == ']') {            
              auto p = stk.top();            
              stk.pop();            
              while(p.first--) {                
                  p.second = p.second + ret;
              }            
              ret = std::move(p.second);
          }        
          else {            
              ret += s[i];        
          }    
      }    
      return ret;
  }
  ```

## [基本计算器 II](https://leetcode-cn.com/problems/basic-calculator-ii/)

* 题目描述

  <img src="imgs/algorithm/problem227.png" alt="problem227" style="zoom:80%;" />

* 解题代码

  ```c++
  int calculate(string s) {
      int n = s.size();
      int num = 0;
      char sign = '+';
      vector<int> stk;
      for(int i = 0; i < n; ++i) {
          char ch = s[i];
          if(isdigit(ch)) {
              num = num * 10 + (ch - '0');
          }
          if(!isdigit(ch) && ch != ' ' || i == n-1) {
              switch (sign) {
                  case '+':
                      stk.push_back(num);
                      break;
                  case '-':
                      stk.push_back(-num);
                      break;
                  case '*':
                      stk.back() *= num;
                      break;
                  case '/':
                      stk.back() /= num;
              }
              sign = ch;
              num = 0;
          }
      }
      return accumulate(stk.begin(), stk.end(), 0);
  }
  ```

## [括号的分数](https://leetcode-cn.com/problems/score-of-parentheses/)

```c++
int scoreOfParentheses(string s) {
    stack<int> stk;
    stk.push(0);
    for(auto ch : s) {
        if(ch == '(') {
            stk.push(0);
        }
        else {
            int cur = stk.top();
            stk.pop();
            int last = stk.top();
            stk.pop();
            stk.push(max(cur * 2, 1) + last);
        }
    }
    return stk.top();
}
```

# 单调栈

## [移掉 K 位数字](https://leetcode-cn.com/problems/remove-k-digits/)

* 题目描述：给定一个以字符串表示的非负整数  num，移除这个数中的 k 位数字，使得剩下的数字最小

> 注意:
>
> num 的长度小于 10002 且  ≥ k。
> num 不会包含任何前导零。
>
>
> 示例 1 :
>
> 输入: num = "1432219", k = 3
> 输出: "1219"
> 解释: 移除掉三个数字 4, 3, 和 2 形成一个新的最小的数字 1219。

* 解题思路：

  * 以题目中的 `num = 1432219， k = 3` 为例，我们需要返回一个长度为 4 的字符串，问题在于： 我们怎么才能求出这四个位置依次是什么呢？
  * 暴力法的话，我们需要枚举$C_n^{n - k}$ 种序列（其中 n 为数字长度），并逐个比较最大。这个时间复杂度是指数级别的，必须进行优化
  * 一个思路是利用数学知识
    * 对于两个数 `123a456` 和 `123b456`，如果 `a > b`， 那么数字 `123a456` 大于 数字 `123b456`，否则数字 `123a456` 小于等于数字 `123b456`。也就说，**两个相同位数的数字大小关系取决于第一个不同的数的大小**
    * 从左到右遍历
    * 对于每一个遍历到的元素，决定是**丢弃**还是**保留**
    * **遍历过程中，如果左边数字比当前数字小，则不能丢弃左边数字，否则导致数字变大**
  * 以`1432219`为例，于是得到解题流程为
    * 当前数字为1，左边没有元素，保留
    * 当前数字为4，左边为1，保留左边元素
    * 以此类推
  * 然而需要注意的是，**如果给定的数字是一个单调递增的数字，那么我们的算法会永远选择不丢弃**。这与题目中要求的，我们要永远确保丢弃 k 个矛盾
  * 一个简单的思路就是：
    * 每次丢弃一次，k 减去 1。当 k 减到 0 ，我们可以提前终止遍历
    * **而当遍历完成，如果 k 仍然大于 0。不妨假设最终还剩下 x 个需要丢弃，那么我们需要选择删除末尾 x 个元素**
    * **刚才我们的关注点一直是丢弃，题目要求我们丢弃 k 个。反过来说，就是让我们保留 n - k个元素，其中 n 为数字长度。 那么我们只需要按照上面的方法遍历完成之后，再截取前n - k个元素即可**

* 代码实现

  ```c++
  string removeKdigits(string num, int k) {	
      if(k<=0)    	
          return num;    
      string ret;    
      for(auto & ch : num) {        
          while(k > 0 && !ret.empty() && ch < ret.back()) {	
              // 保持ret中元素从栈底到栈顶递增            
              ret.pop_back();            
              --k;	// 剩余应丢弃数-1        
          }        
          ret.push_back(ch);    
      }    
      while(k-- > 0 && !ret.empty())  // 清除尾部多余数字，并防止k>num.size()的情况        
          ret.pop_back();    
      int idx = 0;    
      while(idx < ret.size() && ret[idx] == '0')  // 清除头部的'0'        
          ++idx;    
      ret = ret.substr(idx);    
      return ret.empty() ? "0" : ret; 
  }
  ```

## [去除重复字母](https://leetcode-cn.com/problems/remove-duplicate-letters/)

* 题目描述：给你一个仅包含小写字母的字符串，请你去除字符串中重复的字母，使得每个字母只出现一次。需保证返回结果的字典序最小（要求不能打乱其他字符的相对位置）

> 示例 1:
>
> 输入: "bcabc"
> 输出: "abc"
> 示例 2:
>
> 输入: "cbacdcbc"
> 输出: "acdb"

* 解题思路

  * 与上面题目不同，这道题没有一个全局的删除次数 k。而是对于每一个在字符串 s 中出现的字母 c 都有一个 k 值。这个 k 是 c 出现次数 - 1
  * 沿用上面的知识的话，我们首先要做的就是计算每一个字符的 k，可以用一个字典来描述这种关系，**其中 key 为 字符 c，value 为其出现的次数**
  * 具体算法：
    * **建立一个字典。其中 key 为 字符 c，value 为其出现的剩余次数**
    * 从左往右遍历字符串，**每次遍历到一个字符，其剩余出现次数 - 1**
    * 对于每一个字符，**如果其对应的剩余出现次数大于 1，我们可以选择丢弃（也可以选择不丢弃），否则不可以丢弃**
    * 是否**丢弃的标准和上面题目**类似。遍历过程中，如果**栈顶元素字典序比当前元素大且该元素剩余次数大于1，那么我们选择丢弃该栈顶元素**
    * 由于题目是要求字母出现一次，我们可以在遍历的时候简单地判断其是否在栈上即可

* 代码实现

  ```c++
  string removeDuplicateLetters(string s) {    
      unordered_map<char, int> remainder, visited;	// 记录剩余字母数和字母是否在栈中    
      for(auto & ch: s) {        
          remainder[ch]++;	// 统计剩余数    
      }    
      string ret;    
      for(auto &ch: s) {        
          if(visited[ch] == 0) {	// 当前字母不在栈中，可能可以入栈            
              // 栈顶元素大于当前元素(逆序)，且栈顶元素剩余数大于0，则出栈。出栈将visited置0            
              while(!ret.empty() && ch < ret.back() && remainder[ret.back()] > 0) {                 
                  visited[ret.back()] = 0;                
                  ret.pop_back();            
              }            
              visited[ch] = 1;	// 入栈将visited置1            
              ret.push_back(ch);        
          }        
          remainder[ch]--;	// 每遍历一次，剩余数减一    
      }    
      return ret;
  }
  ```

## [ 拼接最大数](https://leetcode-cn.com/problems/create-maximum-number/)

> 给定长度分别为 m 和 n 的两个数组，其元素由 0-9 构成，表示两个自然数各位上的数字。现在从这两个数组中选出 k (k <= m + n) 个数字拼接成一个新的数，要求从同一个数组中取出的数字保持其在原数组中的相对顺序。
>
> 求满足该条件的最大数。结果返回一个表示该最大数的长度为 k 的数组。

* 解题思路

  * 为了找到长度为 k 的最大数，需要从两个数组中分别选出最大的子序列，这两个子序列的长度之和为 k，然后将这两个子序列合并得到最大数。两个子序列的长度最小为 0，最大不能超过 k 且不能超过对应的数组长度
  * 令数组 $\textit{nums}1$的长度为 m，数组 $\textit{nums}2$的长度为 n，则需要从数组$ \textit{nums}1$中选出长度为 x 的子序列，以及从数组$ \textit{nums}2$ 中选出长度为 y 的子序列，其中 x+y=k，且满足 $0 \le x \le m$ 和 $0 \le y \le n$。**需要遍历所有可能的 x 和 y 的值**，**对于每一组 x 和 y的值，得到最大数**。在整个过程中维护可以通过拼接得到的最大数
  * 对于每一组 x 和 y 的值，得到最大数的过程分成两步，**第一步是分别从两个数组中得到指定长度的最大子序列，第二步是将两个最大子序列合并**
  * **第一步可以通过单调栈实现**。单调栈满足**从栈底到栈顶的元素单调递减，从左到右遍历数组，遍历过程中维护单调栈内的元素**，需要**保证遍历结束之后单调栈内的元素个数等于指定的最大子序列的长度**。遍历结束之后，将从栈底到栈顶的元素依次拼接，即得到最大子序列
  * **第二步需要自定义比较方法**。首先比较两个子序列的当前元素，**如果两个当前元素不同，则选其中较大的元素作为下一个合并的元素**，**否则需要比较后面的所有元素才能决定选哪个元素作为下一个合并的元素**

* 代码实现

  ```c++
  vector<int> maxNumber(vector<int>& nums1, vector<int>& nums2, int k) {    
      int len1 = nums1.size();    
      int len2 = nums2.size();    
      if(k <= 0 || k > len1 + len2)        
          return {};    
      vector<int> ret(k, 0);    
      int len = max(0, k - len2), maxlen = min(len1, k);	
      // 拆分子串    
      for(; len <= maxlen; len++) {	// 暴力枚举所有可能        
          auto subvec1 = std::move(getMaxSub(nums1, len));	        
          auto subvec2 = std::move(getMaxSub(nums2, k-len));        
          auto subvec = std::move(merge(subvec1, subvec2));        
          if(comp(subvec, 0, ret, 0) > 0) {            
              ret.swap(subvec);        
          }    
      }    
      return ret;
  }
  // 与“移除k个数字”算法相同
  vector<int> getMaxSub(vector<int> & vec, int k) {    
      if(k == 0)        
          return {};    
      vector<int> ret;    
      int n = vec.size();    
      int remainder = n - k;    
      for(auto & num: vec) {        
          while(!ret.empty() && num > ret.back() && remainder > 0) {            
              ret.pop_back();            
              remainder--;        
          }        
          if(ret.size() < k) {            
              ret.push_back(num);        
          }        
          else            
              remainder--;    
      }    
      return ret;
  }
  // 根据comp比较两个数大小的规则，合并两个数
  vector<int> merge(vector<int> & vec1, vector<int> & vec2) {    
      int m = vec1.size(), n = vec2.size();    
      int idx1 = 0, idx2 = 0;    
      vector<int> ret(m+n);    
      for(int i = 0; i < m+n; i++) {        
          if(comp(vec1, idx1, vec2, idx2) > 0) {            
              ret[i] = vec1[idx1++];        
          }        
          else {            
              ret[i] = vec2[idx2++];        
          }    
      }    
      return ret;
  }
  // 比较两个数大小，比较规则：先比较两个数从左到右首个不相同的数字，若全相同，位数多的大
  int comp(vector<int> & vec1, int idx1, vector<int> & vec2, int idx2) {    
      int m = vec1.size(), n = vec2.size();   
      while(idx1 < m && idx2 < n ) {        
          int diff = vec1[idx1++] - vec2[idx2++];        
          if(diff != 0 ) {            
              return diff;        
          }    
      }    
      return (m - idx1) - (n - idx2);
  }
  ```

## [132 模式](https://leetcode-cn.com/problems/132-pattern/)

* 题目描述

  <img src="imgs/algorithm/problem456.png" alt="problem456" style="zoom:80%;" />

* 解题思路：

  * 三个下标未定，枚举三个下标，复杂度O(n^3)可能会超时
  * 可以枚举1，2，3的方式，使用数据结构维护寻找另两个数

* 解题代码

  ```c++
  // 枚举2
  bool find132pattern(vector<int>& nums) {
      int n = nums.size();
      if(n < 3) {
          return false;
      }
      int minI = nums[0];
      multiset<int> rightK;
      for(int k = 2; k < n; ++k) {
          rightK.insert(nums[k]);
      }
      // 枚举2，j从1开始
      for(int j = 1; j < n-1; ++j) {
          // 满足13，从右边找2
          if(nums[j] > minI) {
              auto it = rightK.upper_bound(minI);
              if(it != rightK.end() && *it < nums[j]) {
                  return true;
              }
          }
          // 更新minI，删除不合题意的k
          minI = min(minI, nums[j]);
          rightK.erase(rightK.find(nums[j+1]));
      }
      return false;
  }
  
  // 从后往前，枚举1
  bool find132pattern(vector<int>& nums) {
      int n = nums.size();
      stack<int> stk;
      int maxK = INT_MIN;
      stk.push(nums[n-1]);
      for(int i = n-2; i >= 0; --i) {
          // nums[i] 作为 1
          if(nums[i] < maxK) {
              return true;
          }
          // 维护 i...n-1的单调递减
          while(!stk.empty() && nums[i] > stk.top()) {
              maxK = stk.top();   // 记录i右侧能充当2的最大值，nums[i]可作为3
              stk.pop();
          }
          // 如果nums[i]<=maxK，可以不用进栈，因为未来被弹出也不会将maxK更新为更大的值
          if(nums[i] > maxK) {
              stk.push(nums[i]);
          }
      }
      return false;
  }
  ```

# 双指针

## **下一个排列**

* 题目描述：实现获取 下一个排列 的函数，算法需要将给定数字序列重新排列成字典序中下一个更大的排列。如果不存在下一个更大的排列，则将数字重新排列成最小的排列（即升序排列）。必须原地修改，只允许使用额外常数空间。

* 思路：注意到下一个排列总是比当前排列要大，除非该排列已经是最大的排列。我们希望找到一种方法，能够找到一个大于当前序列的新序列，且变大的幅度尽可能小。具体地：

  * 我们需要将一个**左边的「较小数」与一个右边的「较大数」交换**，以能够让当前排列变大，从而得到下一个排列
  * 同时我们**要让这个「较小数」尽量靠右，而「较大数」尽可能小**。当交换完成后，**「较大数」右边的数需要按照升序重新排列**。这样可以在保证新排列大于原来排列的情况下，**使变大的幅度尽可能小**
  * 以排列 \[4,5,2,6,3,1] 为例：

    * 我们能找到的符合条件的一对「较小数」与「较大数」的组合为 2 与 3，满足「较小数」尽量靠右，而「较大数」尽可能小
    * 当我们完成交换后排列变为 \[4,5,3,6,2,1]，此时我们可以**重排「较小数」右边的序列**，序列变为 \[4,5,3,1,2,6]

* 代码实现

  ```c++
  void nextPermutation(vector<int> & nums) {    
      if(nums.size()<=1) {        
          return;    
      }    
      int n = nums.size();    
      int smaller = n - 2;    
      while(smaller >= 0 && nums[smaller] >= nums[smaller + 1]) {        
          --smaller;    
      }    
      if(smaller >= 0) {        
          int bigger = n - 1;        
          while(bigger >= 0 && nums[bigger] <= nums[smaller]) {            
              --bigger;        
          }        
          swap(nums[smaller], nums[bigger]);    
      }    
      reverse(nums.begin() + smaller + 1, nums.end());
  }
  ```

## [三数之和](https://leetcode-cn.com/problems/3sum/)

* 代码实现

  ```c++
  vector<vector<int>> threeSum(vector<int>& nums) {    
      if(nums.empty()) {        
          return {};    
      }    
      vector<vector<int> > ret;    
      int len = nums.size();    
      sort(nums.begin(), nums.end());    
      for(int i = 0; i < len; ++i) {        // 最小大于0，三数相加必大于0        
          if(nums[i] > 0) {            
              break;        
          }        
          // 去重        
          if(i > 0 && nums[i] == nums[i-1]) {            
              continue;        
          }        
          int target = 0 - nums[i];        // 双指针右指针，往左移动        
          int k = len - 1;        
          for(int j = i + 1; j < len; ++j) {            
              // 去重            
              if(j > i + 1 && nums[j] == nums[j-1]) {                
                  continue;            
              }            
              // 指针往左移动，直到满足条件，或者不存在三数组合            
              while(k > j && nums[j] + nums[k] > target) {                
                  --k;            
              }            
              // 没有满足条件的三数，跳出本次循环            
              if(k == j) {                
                  break;            
              }            
              // 满足条件            
              if(nums[j] + nums[k] == target) {                
                  ret.push_back({nums[i], nums[j], nums[k]});            
              }        
          }    
      }    
      return ret;
  }
  ```

## [接雨水](https://leetcode-cn.com/problems/trapping-rain-water/)

* 本题可以用双指针和单调栈做，单调栈横向统计，而双指针则纵向统计。

* 代码实现：

  ```c++
  // 单调栈解法
  int trap(vector<int>& height) {    
      int len = height.size();    
      if(len <= 2) {        
          return 0;    
      }    
      stack<int> idxs;    
      int volume = 0;    
      for(int i = 0; i < len; ++i) {        
          // 当前值大于栈顶，出栈        
          while(!idxs.empty() && height[i] > height[idxs.top()]) {            
              // 低水位            
              int low = height[idxs.top()];            
              idxs.pop();            
              // 出栈后，栈空，不能储水            
              if(idxs.empty()) {                
                  break;            
              }            
              // 计算与低水位差值，为储水矩形的高            
              int h = min(height[idxs.top()], height[i]) - low;            
              // 储水矩形的宽            
              int width = i - idxs.top() - 1;            
              volume += h * width;        
          }        
          idxs.push(i);    
      }    
      return volume;
  }
   // 双指针：统计当前遍历柱子是否能储水，即左右两边同时有没有更高的柱子，有就能储水
  int trap(vector<int> & height) {    
      int len = height.size();    
      if(len <= 2) {        
          return 0;    
      }    
      int volume = 0;    
      int leftMax = 0, rightMax = 0;    
      int left = 0, right = len - 1;    
      while(left < right) {        
          // 记录左边最高柱子高度        
          leftMax = max(leftMax, height[left]);        
          // 记录右边最高柱子高度        
          rightMax = max(rightMax, height[right]);        
          // 当前遍历左柱子小于右柱子，leftMax>=height[left]都满足下式        
          if(height[left] < height[right]) {            
              volume += leftMax - height[left]; // >= 0            
              ++left;        
          }        
          else {  // 当前遍历左柱子大于等于右柱子，rightMax>=height[right]都满足下式            
              volume += rightMax - height[right]; // >= 0           
              --right;        
          }    
      }    
      return volume;
  }
  ```

## [至少有 K 个重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-with-at-least-k-repeating-characters/)

* 思路

  <img src="imgs/algorithm/solution3.png" alt="solution3" style="zoom:67%;" />

  <img src="imgs/algorithm/solution4.png" alt="solution4" style="zoom:67%;" />

* 代码实现

  ```c++
  int longestSubstring(string s, int k) {    
      int n = s.size();    
      int maxLen = 0;    // 枚举字符类型数目，限定滑动窗口中能出现的最大字符类型数，数目为1到26    
      // 在每个字符类型数目限定下找到局部最优，更新全局最优    
      for(int alphaLimit = 1; alphaLimit <= min(n, 26); ++alphaLimit) {        
          vector<int> alpha(26, 0); // 统计窗口中字符类型和数目        
          int left = 0, right = 0; // 滑动窗口初始值        
          int total = 0; // 窗口总字符类型数量        
          int satisfied = 0;  // 窗口中满足数量>=k的字符类型数目        
          while(right < n) {            
              int idx = s[right] - 'a';            
              // 向右移动，计数+1            
              ++alpha[idx];            
              // 若+1后，数目为1，为新增字符，total+1            
              if(alpha[idx] == 1) {                
                  ++total;            
              }            
              // 若+1后，数目为k，为新的满足条件的字符，satisfied + 1            
              if(alpha[idx] == k) {                
                  ++satisfied;            
              }            
              // 在字符数约束下，左移left，使满足total <= alphaLimit            
              while(total > alphaLimit) {                
                  idx = s[left++] - 'a';                
                  --alpha[idx];                
                  // 从k减为k - 1, satisfied - 1                
                  if(alpha[idx] == k - 1) {                    
                      --satisfied;                
                  }                
                  // 减为0，total - 1                
                  if(alpha[idx] == 0) {                    
                      --total;                
                  }            
              }            
              // 在满足字符约束，且满足每种字符数量都满足>=k情况下，更新长度            
              if(satisfied == total) {                
                  maxLen = max(maxLen, right - left + 1);            
              }            
              ++right;        
          }    
      }    
      return maxLen;
  }
  ```

## [最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)

* 给你一个字符串 `s` 、一个字符串 `t` 。返回 `s` 中涵盖 `t` 所有字符的最小子串。如果 `s` 中不存在涵盖 `t` 所有字符的子串，则返回空字符串 `""`
  * **注意：**
    - 对于 `t` 中重复字符，我们寻找的子字符串中该字符数量必须不少于 `t` 中该字符数量
    - 如果 `s` 中存在这样的子串，我们保证它是唯一的答案

* 代码实现

  ```c++
  string minWindow(string s, string t) {    
      unordered_map<char, int> window, target;    
      int n = s.size();    
      // 记录t中所有字符，用于快速查询    
      for(auto &ch:t) {        
          ++target[ch];    
      }    
      int left = 0, right = 0, minLen = INT_MAX;
      int count = 0;	// 统计窗口中有多少在t中的字符
      int start = 0;    // 记录字串起点
      while(right < n) {        
          char ch = s[right];        
          right++;        
          if(target.count(ch) > 0) {  // 在t中出现           
              window[ch]++;           // 当前窗口中统计出现次数+1
              if(target[ch] == window[ch]) {   // 当前窗口满足t中ch字符数量，count+1             
                  ++count;            
              }
              // left右移，缩减窗口大小，同时统计最小长度和起点
              while(count == target.size()) {                
                  if(right - left < minLen) {                    
                      minLen = right - left;                    
                      start = left;                
                  }                
                  --window[s[left]];    // 将s[left]移出window 
                  // s[left]移除后，不满足window中s[left]大于等于t中s[left]数目
                  if(target.count(s[left]) > 0 && window[s[left]] < target[s[left]]) {                 
                      --count;                
                  }                
                  ++left;            
              }        
          }    
      }    
      return minLen == INT_MAX? "": s.substr(start, minLen);
  }
  ```
  

## [环形数组是否存在循环](https://leetcode-cn.com/problems/circular-array-loop/)

## [快乐数](https://leetcode-cn.com/problems/happy-number/)

* 题目描述

  <img src="imgs/algorithm/problem202.png" alt="problem202" style="zoom:67%;" />

* 快慢指针判环

  ```c++
  bool isHappy(int n) {
      auto getNext = [](int num)->int{
          int ret = 0;
          while(num != 0) {
              int tmp = num % 10;
              ret += tmp * tmp;
              num /= 10;
          }
          return ret;
      };
  
      int slow = getNext(n), fast = getNext(slow);
      while(fast != 1 && slow != fast) {
          slow = getNext(slow);
          fast = getNext(getNext(fast));
      }
      return fast == 1;
  }
  ```

## [等差数列划分](https://leetcode-cn.com/problems/arithmetic-slices/)

* 解题代码

  ```c++
  int numberOfArithmeticSlices(vector<int>& nums) {
      int len = nums.size();
      if(len < 3) {
          return 0;
      }
      // left记录起点，right记录终点
      int left = 0, right = 1;
      int ret = 0;
      int diff = nums[1] - nums[0];
      while(right < len) {
          // 不满足等差数组条件，统计left...right等差数列数
          if(nums[right] - nums[right - 1] != diff) {
              // 长度大于等于3才记录
              if(right - left >= 3) {
                  int tmpLen = right - left;
                  int count = (2 + tmpLen - 1) * (tmpLen - 2) / 2;
                  ret += tmpLen * (tmpLen - 2) - count;
              }
              // 更新diff和left
              left = right - 1;
              diff = nums[right] - nums[left];
          }
          ++right;
      }
      // 最后可能right==len，长度大于3，则记录
      if(right - left >= 3) {
          int tmpLen = right - left;
          int count = (2 + tmpLen - 1) * (tmpLen - 2) / 2;
          ret += tmpLen * (tmpLen - 2) - count;
      }
      return ret;
  }
  
  // dp
  // 思路：dp[i]为以nums[i]结尾的等差数列个数
  // 若nums[i] - nums[i-1]==nums[i-1] - nums[i-2], dp[i] = dp[i-1] + 1
  // 表示nums[i-1]结尾的等差数列都可以以nums[i]结尾，并且加+1(nums[i-2],nums[i-1],nums[i])
  // 否则dp[i] = 0。最终结果为dp数组的和
  // 优化dp[i]只与dp[i-1]有关，可以使用一个变量记录之前状态
  int numberOfArithmeticSlices(vector<int>& nums) {
      int len = nums.size();
      if(len < 3) {
          return 0;
      }
      int diff = nums[1] - nums[0];
      int pre = 0;
      int ret = 0;
      for(int i = 2; i < len; ++i) {
          if(nums[i] - nums[i-1] == nums[i-1] - nums[i-2]) {
              ++pre;
          }
          else {
              pre = 0;
              diff = nums[i] - nums[i-1];
          }
          ret += pre;
      }
      return ret;
  }
  ```


# DFS+回溯

## [括号生成](https://leetcode-cn.com/problems/generate-parentheses/)

* 题目描述：数字 `n` 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 **有效的** 括号组合

* 解题思路：如果左括号数量不大于 n，我们可以放一个左括号。如果**右括号数量小于左括号的数量，我们可以放一个右括号**

* 代码实现

  ```c++
  vector<string> generateParenthesis(int n) {
      if(n <= 0)
          return {};
      vector<string> ret;
      string tmp = string("");
      dfs(ret, tmp, n, n);
      return ret;
  }
  
  void dfs(vector<string> & ret, string &str, int open, int close)
  {
      if(open ==0 && close ==0) {
          ret.push_back(str);
          return;
      }
      if(open > 0) {
          str.push_back('(');
          dfs(ret, str, open-1, close);
          str.pop_back();
      }
      if(close > open) {
          str.push_back(')');
          dfs(ret, str, open, close - 1);
          str.pop_back();
      }
  }
  ```

## [删除无效的括号](https://leetcode-cn.com/problems/remove-invalid-parentheses/)

* 题目描述：给你一个由若干括号和字母组成的字符串 `s` ，删除最小数量的无效括号，使得输入的字符串有效。返回所有可能的结果。答案可以按 **任意顺序** 返回

* 例子

  ```
  输入: "()())()"输出: ["()()()", "(())()"]
  ```

* 思路：

  * **如果当前遍历到的左括号的数目严格小于右括号的数目则表达式无效**
  * 因此，我们可以遍历一次输入字符串，统计「左括号」和「右括号」出现的次数
    * 当遍历到「右括号」的时候
      * 如果此时「左括号」的数量不为 0，因为 「右括号」可以与之前遍历到的「左括号」匹配，此时「左括号」出现的次数 -1
      * 如果此时「左括号」的数量为 0，「右括号」数量加 1
    * 当遍历到「左括号」的时候，「左括号」数量加 1
  * 通过这样的计数规则，最后「左括号」和「右括号」的数量就是**各自最少应该删除的数量**

* 代码实现

  ```c++
  vector<string> removeInvalidParentheses(string s) {
      if(s.empty())
          return {""};
      set<string> ret;	// hashset去重
      int leftRemove = 0, rightRemove = 0;    // 记录最少删除多少左括号和右括号
      int n = s.size();
  
      for(auto & ch: s){
          if(ch == '(')
              leftRemove++;
          else if(ch == ')') {
          	if(leftRemove == 0) // 多余右括号
              	rightRemove++;
              if(leftRemove > 0)  // 可以抵消一个左括号
                  leftRemove--;
          }
      }
      string path;
      dfs(ret, s, path, 0, 0, 0, leftRemove, rightRemove);
      return vector<string> (ret.begin(), ret.end());
  }
  
  void dfs(set<string> & ret, string &s, string &path, int idx, 
           int left, int right, int leftRemove, int rightRemove) 
  {
      if(idx == s.size()){
          if(leftRemove == 0 && rightRemove == 0) {
              ret.insert(path);
          }
          return;
      }
          
      char ch = s[idx];
      // 删除当前字符
      if(ch == '(' && leftRemove > 0)
          dfs(ret, s, path, idx+1, left, right, leftRemove-1, rightRemove);
      else if(ch == ')' && rightRemove > 0)
          dfs(ret, s, path, idx + 1, left, right, leftRemove, rightRemove-1);
  
      // 保留当前字符
      path.push_back(ch);
      if(ch != '(' && ch!=')')    // 字母直接保留
          dfs(ret, s, path, idx+1, left, right, leftRemove, rightRemove);
      else if(ch == '(')      // 保留左括号，left+1
          dfs(ret, s, path, idx+1, left+1, right, leftRemove, rightRemove);
      else if(left > right)     // 左括号数大于右括号数时才能保留右括号，right+1
          dfs(ret, s, path, idx+1, left, right + 1, leftRemove, rightRemove);
  
      path.pop_back();	// 回溯
  }
  ```

## [被围绕的区域](https://leetcode-cn.com/problems/surrounded-regions/)

## [串联字符串的最大长度](https://leetcode-cn.com/problems/maximum-length-of-a-concatenated-string-with-unique-characters/)

* 代码实现(妙)

  ```c++
  int maxLength(vector<string>& arr) {
      if(arr.empty())
          return 0;
      // 统计本身不含重复字符的字符串，每个元素表示字符串含有的字符
      vector<int> masks;
      for(auto str : arr){
          int mask = 0;
          for(auto ch : str) {
              ch -= 'a';
              // 重复，跳出
              if((mask >> ch) & 1) {
                  mask = 0;
                  break;
              }
              // 记录字符
              mask |= 1 << ch;
          }
          if(mask > 0) {
              masks.emplace_back(mask);
          }
      }
      
      // 计算mask中有多少个1
      auto count = [] (int mask)->int {
          int ret = 0;
          while(mask != 0) {
              if(mask & 1) {
                  ret++;
              }
              mask >>= 1;
          }
          return ret;
      };
      // dfs + 回溯
      int ret = 0;
      // mask记录不重复的字符
      function<void(int, int)> dfs = [&](int pos, int mask) {
          if(pos == masks.size()) {
              ret = max(ret, count(mask));
              return;
          }
          // 如果当前字符串的字符在之前没出现，可以选择当前字符串
          if((mask & masks[pos]) == 0) {
              dfs(pos + 1, mask | masks[pos]);
          }
          // 不选当前字符串
          dfs(pos + 1, mask);
      };
      
      dfs(0, 0);
      return ret;
  }
  ```

## [二叉搜索树序列](https://leetcode-cn.com/problems/bst-sequences-lcci/)

* 代码实现：本质上**求BST的任意遍历方式得到的序列集合**。隐含条件：**遍历了父节点才能遍历其子节点**

  ```c++
  vector<vector<int>> BSTSequences(TreeNode* root) 
  {
      if(root == nullptr)
          return {};
      vector<vector<int> > ret;
      vector<int> path;
      deque<TreeNode*> que;
      que.push_front(root);
      dfs(que, ret, path);
      return ret;
  }
  
  void dfs(deque<TreeNode*> &que, vector<vector<int> > &ret, vector<int> &path) {
      if(que.empty()) {
          ret.push_back(path);
          return;
      }
      // 当前层能选择遍历的节点数，同一层的节点插入顺序任意
      int n = que.size();
      while(n--) {
          auto node = que.front();
          que.pop_front();
          path.emplace_back(node->val);
          // 记录下一轮的所有可能选择
          if(node->left!=nullptr)
              que.push_back(node->left);
          if(node->right!=nullptr)
              que.push_back(node->right);
  		
          // 进入下一轮
          dfs(que, ret, path);
          
  		// 本轮结束，将回溯，需要复原返回
          if(node->right != nullptr)
              que.pop_back();
          if(node->left != nullptr)
              que.pop_back();
          // 此处不能push_front原因是：上面出队是front，已经被先选择遍历了，就要从队尾入队，防止重复遍历
          que.push_back(node);
          path.pop_back();
      }
  }
  ```

## [单词转换](https://leetcode-cn.com/problems/word-transformer-lcci/)

* 题目与“单词接龙”相似，但是本题要求路径，而“单词接龙”求最短路径，所以此题用dfs解决，且需要剪枝，不然会超时

* 解题代码

  ```c++
  vector<string> findLadders(string beginWord, string endWord, vector<string>& wordList) {
      // 判断两个单词能否互相转换
      auto check = [](string & str1, string &str2)->bool {
          if(str1.size() != str2.size()) {
              return false;
          }
          int diff = 0;
          for(int i = 0; i < str1.size(); ++i) {
              if(str1[i] != str2[i]) {
                  ++diff;
              }
              if(diff > 1) {
                  return false;
              }
          }
          return diff == 1;
      };
      vector<string> ret;
      int n = wordList.size();
      // 记录已经遍历过的单词
      vector<bool> visited(n, false);
      function<bool(string &)> dfs = [&](string &word) {
          if(word == endWord) {
              ret.push_back(word);
              return true;
          }
          ret.push_back(word);
          for(int i = 0; i < n; ++i) {
              if(visited[i]) {
                  continue;
              }
              if(check(word, wordList[i])) {
                  visited[i] = true;
                  if(dfs(wordList[i])) {
                      return true;
                  }
                  // 此处没有将visited[i]设为false是为了剪枝
                  // 因为代码执行到此处表明沿着wordList[i]搜索到不了endWord
                  // 所以后面无需再转换到此分支进行搜索
              }
          }
          ret.pop_back();
          return false;
      };
  
      dfs(beginWord);
      return ret;
  }
  ```

## [单词拆分 II](https://leetcode-cn.com/problems/word-break-ii/)

* 题目描述

  <img src="imgs/algorithm/problem140.png" alt="problem140" style="zoom:80%;" />

* 解题思路：dfs+记忆化搜索+回溯

  <img src="imgs/algorithm/solution140.png" alt="solution140" style="zoom:80%;" />

* 解题代码

  ```c++
  vector<string> wordBreak(string s, vector<string>& wordDict) {
      int n = s.size();
      unordered_set<string> dict(wordDict.begin(), wordDict.end());
      // 用于记录int下标开始的子串，所有可能的划分情况
      unordered_map<int, vector<string> > rec;
      function<void(int)> dfs = [&](int idx) {
          // 已经搜索过，剪枝返回
          if(rec.count(idx)) {
              return;
          }
          // 到达最后，注意要把rec[idx]设为空串，返回上层拼接
          if(idx == n) {
              rec[idx] = {""};
              return;
          }
          for(int i = idx + 1; i <= n; ++i) {
              string word = std::move(s.substr(idx, i - idx));
              if(dict.count(word)) {
                  dfs(i); // 获得i后所有可能，自顶向下
                  // 返回后组装
                  for(auto & str : rec[i]) {
                      rec[idx].push_back(str.empty() ? word : word + " " + str);
                  }
              }
          }
      };
      dfs(0);
      return rec[0];
  }
  ```

# BFS

## [单词接龙](https://leetcode-cn.com/problems/word-ladder/)

* 题目描述

  <img src="imgs/algorithm/problem127.png" alt="problem127" style="zoom:80%;" />

* 解题思路

  <img src="imgs/algorithm/solution127-1.png" alt="solution127-1" style="zoom:80%;" />

  <img src="imgs/algorithm/solution127-2.png" alt="solution127-2" style="zoom:80%;" />

* 解题代码

  ```c++
  class Solution {
  public:
      //确定俩个单词能否互相转换
      bool exchange_word(const string& a, const string& b) {
          int misMatch = 0;
          int len = a.size();
          for (int i = 0; i < len; ++i) {
              if (a[i] != b[i]) 
                  ++misMatch;
              if (misMatch > 1) return false;     //如果>1直接返回
          }
          return misMatch == 1;
      }
  
      int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
          
          unordered_set<string> word_set(wordList.begin(), wordList.end());
          if (word_set.count(endWord) == 0) 
              return 0;         
  
          queue<string> q;
          q.push(beginWord);
  
          //必须消除已经是树节点的单词
          word_set.erase(beginWord);
          int steps = 1;
  
          while (!q.empty()) {
              int size = q.size();                //记住当前队列的长度
              while(size--) {    //for循环pop出队列所有元素
                  auto str = q.front(); 
                  q.pop();  
                  vector<string> tmp;             //保存str能够转换的单词
                  for (const string& word : word_set) {
                      if (exchange_word(str, word)) {
                          if (word == endWord)
                              return steps + 1;
                          tmp.push_back(word);
                          q.push(word);
                      }
                  }
                  // tmp中的单词已经是树中的节点,我们需要erase它使得后续的树节点不重复
                  // 在这里删除是防止上面遍历迭代器失效
                  for (const auto& str : tmp) 
                      word_set.erase(str);
              }
              ++steps;
          }
          return 0;
      }
  };
  ```

## [被围绕的区域](https://leetcode-cn.com/problems/surrounded-regions/)

* 题目描述

  <img src="imgs/algorithm/problem130.png" alt="problem130" style="zoom: 80%;" />

* 解题思路

  <img src="imgs/algorithm/solution130.png" alt="solution130" style="zoom:80%;" />

* 代码实现

  ```c++
  void solve(vector<vector<char>>& board) {
      if(board.empty())
          return;
      int rows = board.size(), cols = board[0].size();
      queue<pair<int, int> > pos;
      for(int i = 0; i < rows; ++i){
          if(board[i][0] == 'O')
              pos.push({i, 0});
          if(board[i][cols-1] == 'O')
              pos.push({i, cols-1});
      }
  
      for(int i = 1; i < cols - 1; ++i){
          if(board[0][i] == 'O')
              pos.push({0, i});
          if(board[rows-1][i] == 'O')
              pos.push({rows-1, i});
      }
  
      vector<pair<int, int> > directions {{1,0}, {-1, 0}, {0, 1}, {0, -1}};
      while(!pos.empty()) {
          auto cur = pos.front();
          pos.pop();
          int x = cur.first;
          int y = cur.second;
          board[x][y] = 'Y';
          for(auto &[h, v] : directions) {
              // 
              if(check(board, rows, cols, x+h, y+v)) {
               	pos.push({x+h, y+v});   
              }
          }
      }
  
      for(int i = 0; i<rows; ++i){
          for(int j = 0; j<cols; ++j) {
              if(board[i][j] == 'O')
                  board[i][j] = 'X';
              else if(board[i][j] == 'Y')
                  board[i][j] = 'O';
          }
      }
  }
  
  bool check(vector<vector<char> > &board,  int rows, int cols, int x, int y){
      if(x >= rows || y >= cols || x < 0 || y < 0 || board[x][y] != 'O')
          return false;
      return true;
  }
  ```

## [网格中的最短路径](https://leetcode-cn.com/problems/shortest-path-in-a-grid-with-obstacles-elimination/)(重点)

* 题目描述

  <img src="imgs/algorithm/problem1293.png" alt="problem1293" style="zoom:80%;" />

* 解题思路

  <img src="imgs/algorithm/solution1293.png" alt="solution1293" style="zoom:80%;" />

  * 优化：这里是直接看visited\[i]\[j][rest] 但是假如重复访问i,j点的时候，如果已访问过的rest值更大，就不需要再访问了。因为rest大的情况肯定比rest小的情况走的远

* 解题代码

  ```c++
  struct Nagato {
      int x, y;
      int rest;
      Nagato(int _x, int _y, int _r): x(_x), y(_y), rest(_r) {}
  };
  
  class Solution {
  private:
      static constexpr int dirs[4][2] = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
  
  public:
      int shortestPath(vector<vector<int>>& grid, int k) {
          int m = grid.size(), n = grid[0].size();
          if (m == 1 && n == 1) {
              return 0;
          }
  		// 缩小搜索空间
          k = min(k, m + n - 3);
          
          // 记录状态，第三维很重要，记录x,y位置，还能消除多少个障碍，
          // 由于i,j节点可能被多次遍历，每次剩余数不同代表不同状态
          bool visited[m][n][k + 1];
          memset(visited, false, sizeof(visited));
          queue<Nagato> q;
          q.emplace(0, 0, k);
          visited[0][0][k] = true;
  
          for (int step = 1; q.size() > 0; ++step) {
              int cnt = q.size();
              for (int _ = 0; _ < cnt; ++_) {
                  Nagato cur = q.front();
                  q.pop();
                  for (int i = 0; i < 4; ++i) {
                      int nx = cur.x + dirs[i][0];
                      int ny = cur.y + dirs[i][1];
                      if (nx >= 0 && nx < m && ny >= 0 && ny < n) {
                          if (grid[nx][ny] == 0 && !visited[nx][ny][cur.rest]) {
                              if (nx == m - 1 && ny == n - 1) {
                                  return step;
                              }
                              q.emplace(nx, ny, cur.rest);
                              visited[nx][ny][cur.rest] = true;
                          }
                          else if (grid[nx][ny] == 1 && cur.rest > 0 
                                   && !visited[nx][ny][cur.rest - 1]) {
                              q.emplace(nx, ny, cur.rest - 1);
                              visited[nx][ny][cur.rest - 1] = true;
                          }
                      }
                  }
              }
          }
          return -1;
      }
  };
  ```

# 图

## [网络延迟时间(Dijkstra)](https://leetcode-cn.com/problems/network-delay-time/)

* 题目描述

  <img src="imgs/algorithm/problem743.png" alt="problem743" style="zoom:80%;" />

* 思路：根据题意，从节点 k 发出的信号，到达节点 x 的时间就是节点 k 到节点 x 的最短路的长度。因此我们需要求出节点 k 到其余所有点的最短路，其中的最大值就是答案。若存在从 k 出发无法到达的点，则返回 -1

  * 使用Dijkstra算法求从k到x的单源最短路径：
    * Dijkstra算法采用的是一种**贪心**的策略，声明一个数组dis来保存源点到各个顶点的最短距离和一个**保存已经找到了最短路径的顶点的集合**：T，初始时，源点 s 的路径权重被赋为 0 （dist[s] = 0）。若对于顶点 s 存在能**直接到达的边**（s,m），则把dist[m]设为w(s, m)，同时把所有其他（s**不能直接到达的**）顶点的**路径长度设为无穷大**。初始时，集合T只有顶点s
    * 然后，**从dist数组选择最小值**，则该值就是源点s到该值对应的顶点的最短路径，并且把该点加入到T中，OK，此时完成一个顶点，然后，**需要看看新加入的顶点是否可以到达其他顶点并且看看通过该顶点到达其他点的路径长度是否比源点直接到达短**，如果是，那么就**替换这些顶点在dist中的值**
    * 再从dis中找出最小值，重复上述动作，直到T中包含了图的所有顶点

  <img src="imgs/algorithm/solution743.png" alt="solution743" style="zoom:80%;" />

* 解题代码

  ```c++
  // 朴素dijkstra
  class Solution {
  public:
      int networkDelayTime(vector<vector<int>> &times, int n, int k) {
          const int inf = INT_MAX / 2;	// 防止上溢
          // 邻接矩阵建图
          vector<vector<int>> g(n, vector<int>(n, inf));
          for (auto &t : times) {
              int x = t[0] - 1, y = t[1] - 1;	// 编号-1存储
              g[x][y] = t[2];		// x-->y
          }
  		// 记录源点(k)到其他各节点的最短距离
          vector<int> dist(n, inf);
          dist[k - 1] = 0;
          // 记录已经确定最短路径的节点
          vector<int> used(n);
          for (int i = 0; i < n; ++i) {
              int x = -1;
              // 遍历未确定节点
              for (int y = 0; y < n; ++y) {
                  // 从未确定节点中选出距离源点最近的节点
                  if (!used[y] && (x == -1 || dist[y] < dist[x])) {
                      x = y;
                  }
              }
              used[x] = true;
              // 用x更新所有x能到达的y，dist[y] = min(dist[x]+dist[x->y], dist[y])
              for (int y = 0; y < n; ++y) {
                  dist[y] = min(dist[y], dist[x] + g[x][y]);
              }
          }
  
          int ans = *max_element(dist.begin(), dist.end());
          return ans == inf ? -1 : ans;
      }
  };
  
  // 优化：邻接表+优先队列
  class Solution {
  public:
  	using pii = pair<int, int>;
      int networkDelayTime(vector<vector<int>>& times, int n, int k) {
          int inf = 0x3f3f3f3f;
          vector<vector<pii> > eg(n+1);
          /* 预分配空间，减少稠密图时多次插入造成空间重新分配的消耗 */
          for(auto& E: eg) 
              E.reserve(n+1);
          
          // 前驱节点-<后继节点，时间>
          for(auto& x: times){
              eg[x[0]].emplace_back(x[1], x[2]);
          }
          
          int ans = 0;
          vector<int> vis(n+1, inf); // 记录源点到其他所有节点的最短距离
          
          // 小根堆，寻找「未确定节点」中与源点距离最近的节点
          priority_queue<pii, vector<pii>, greater<pii> > q;
          q.emplace(0, k);
          vis[k] = 0;
          while(!q.empty()) {
              // time为源点到当前cur节点的最小距离
              auto [time, cur] = q.top();
              q.pop();
              // 源点到cur的当前路径非最短路径 ******
              if(vis[cur] < time) 
                  continue;
              ans = max(ans, time);
              // 从cur出发更新cur能到达的所有节点的最小距离
              for(auto [next, nextTime]: eg[cur]) {
                  // 更新结果存到vis中，同时加入队列
                  int tmpTime = nextTime + time;
                  if(vis[next] > tmpTime) {
                      q.emplace(tmpTime, next);
                      vis[next] = tmpTime;
                  }
              }
          }
          // 有节点未被遍历，vis的值为inf
          return find(begin(vis)+1, end(vis), inf) == end(vis) ? ans : -1;
      }
  };
  ```

## [访问所有节点的最短路径](https://leetcode-cn.com/problems/shortest-path-visiting-all-nodes/)

* 题目描述

  <img src="imgs/algorithm/problem847.png" alt="problem847" style="zoom: 80%;" />

* 解题思路

  <img src="imgs/algorithm/solution847.png" alt="solution847" style="zoom:80%;" />

* 解题代码

  ```c++
  class Solution {
  public:
      int shortestPathLength(vector<vector<int>>& graph) {
          int n = graph.size();
          // 三元组表示每个节点，包含
          // 1. 节点编号
          // 2. 已遍历节点(mask表示，已遍历的位置为1)
          // 3. 到目前节点的路径长度
          queue<tuple<int, int, int>> q;
          // 记录已遍历的路径，防止重复遍历
          vector<vector<int>> seen(n, vector<int>(1 << n));
          for (int i = 0; i < n; ++i) {
              q.emplace(i, 1 << i, 0);
              seen[i][1 << i] = true;
          }
  
          int ans = 0;
          while (!q.empty()) {
              auto [u, mask, dist] = q.front();
              q.pop();
              // mask = (1 << n) - 1表示全部节点都已被遍历
              if (mask == (1 << n) - 1) {
                  ans = dist;
                  break;
              }
              // 搜索相邻的节点
              for (int v: graph[u]) {
                  // 将 mask 的第 v 位置为 1
                  int mask_v = mask | (1 << v);
                  // 当前路径没被搜索过，入队
                  if (!seen[v][mask_v]) {
                      q.emplace(v, mask_v, dist + 1);
                      seen[v][mask_v] = true;
                  }
              }
          }
          return ans;
      }
  };
  ```

  

# 拓扑排序

## [课程表I](https://leetcode-cn.com/problems/course-schedule/)

* 题目描述

  <img src="imgs/algorithm/problem207.png" alt="problem207" style="zoom:80%;" />

* 解题思路

  <img src="imgs/algorithm/solution207.png" alt="solution207" style="zoom:80%;" />

* 代码实现

  ```c++
  // dfs
  bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
      // 0：未搜索， 1：搜索中， 2：已完成
      vector<vector<int> > graph(numCourses, vector<int>());
      // vector<int> stk; // 需要输出排序顺序才需要记录过程
      vector<int> visited(numCourses, 0);
      bool valid = true; 
  	// 构建有向图
      for(auto &vec : prerequisites){
          graph[vec[1]].push_back(vec[0]);
      }
      for(int i = 0; i < numCourses && valid; i++){
          if(!visited[i]){
              // dfs(graph, stk, prerequisites,  visited, i);
              auto ret = dfs(graph, prerequisites,  visited, i);
              if(!ret) {
                  return false;
              }
          }
      }
      return true;
  }
  
  
  bool dfs(vector<vector<int> >& graph, vector<vector<int> >& prerequisites, 
               vector<int> & visited, int idx){
      // 标记为搜索中
      visited[idx] = 1;
      for(auto & next : graph[idx]) {
          if(visited[next] == 0) {
              // dfs(graph, stk, prerequisites, visited, next);
              auto ret = dfs(graph, prerequisites, visited, next);
              if(!ret) {
                  return false;
              }
          }
          else if(visited[next] == 1) { // 遇到标记为搜索中的节点，表示有环，不存在拓扑排序
              return false;
          }
      }
      visited[idx] = 2;
      // stk.push_back(idx);
      return true;
  }
  
  /*
  考虑拓扑排序中最前面的节点，该节点一定不会有任何入边，也就是它没有任何的先修课程要求。当我们将一个节点加入答案中后，我们就可以移除它的所有出边，代表着它的相邻节点少了一门先修课程的要求。如果某个相邻节点变成了「没有任何入边的节点」，那么就代表着这门课可以开始学习了。按照这样的流程，我们不断地将没有入边的节点加入答案，直到答案中包含所有的节点（得到了一种拓扑排序）或者不存在没有入边的节点（图中包含环）。
  */
  
  // bfs
  bool canFinishBFS(int numCourses, vector<vector<int>>& prerequisites) {
      // 0：未搜索， 1：搜索中， 2：已完成
      vector<vector<int> > graph(numCourses, vector<int>());
      // 记录节点的入度
      vector<int> indeg(numCourses, 0);
      queue<int> q;
      for(auto &vec : prerequisites){
          graph[vec[1]].push_back(vec[0]);
          ++indeg[vec[0]];
      }
      // 从入度为0的节点开始遍历
      for(int i = 0; i<numCourses; i++) {
          if(indeg[i] == 0)
              q.push(i);
      }
      // 记录遍历节点数，如果全部刚好遍历一次，表明不存在环，存在拓扑排序
      int visited = 0;
      while(!q.empty()){
          int u = q.front();
          q.pop();
          // 将u标记为已遍历，如果要记录过程，则需存入数组中，这里只记录状态
          ++visited;
          for(auto &v : graph[u]) {
              // v的入度-1，表明一门先决课程已满足
              --indeg[v];
              // 若v的入度为0，表明所有先决课程都已满足，可以进行v课程的学习，加入q中进行搜索
              if(indeg[v] == 0) {
                  q.push(v);
              }
          }
      }
  
      return visited == numCourses;
  }
  
  ```

## [课程表 II](https://leetcode-cn.com/problems/course-schedule-ii/)

## [找到最终的安全状态](https://leetcode-cn.com/problems/find-eventual-safe-states/)

* 题目描述

  <img src="imgs/algorithm/problem807.png" alt="problem807" style="zoom:80%;" />

* 解题思路

  <img src="imgs/algorithm/solution807_1.png" alt="solution807_1" style="zoom:80%;" />

  <img src="imgs/algorithm/solution807_2.png" alt="solution807_2" style="zoom:80%;" />

* 解题代码

  ```c++
  class Solution {
  public:
      // dfs
      vector<int> eventualSafeNodes(vector<vector<int>>& graph) {
          int n = graph.size();
          vector<int> ret;
          vector<int>color(n, 0); 
          function<bool(int)> dfs = [&](int idx)->bool{
              if(color[idx] > 0) {
                  return color[idx]==2;
              }
  
              color[idx] = 1;
              for(auto next : graph[idx]) {
                  if(!dfs(next)) {
                      return false;
                  }    
              }
              color[idx] = 2;
              return true;
          };
          for(int i = 0; i < n; ++i) {
              if(dfs(i)) {
                  ret.push_back(i);
              }
          }
          return ret;
      }
      
      // 拓扑排序
      vector<int> eventualSafeNodes(vector<vector<int>> &graph) {
          int n = graph.size();
          vector<vector<int>> rg(n);
          vector<int> inDeg(n);
          for (int x = 0; x < n; ++x) {
              for (int y : graph[x]) {
                  rg[y].push_back(x);
              }
              inDeg[x] = graph[x].size();
          }
  
          queue<int> q;
          for (int i = 0; i < n; ++i) {
              if (inDeg[i] == 0) {
                  q.push(i);
              }
          }
          while (!q.empty()) {
              int y = q.front();
              q.pop();
              for (int x : rg[y]) {
                  if (--inDeg[x] == 0) {
                      q.push(x);
                  }
              }
          }
  
          vector<int> ans;
          for (int i = 0; i < n; ++i) {
              if (inDeg[i] == 0) {
                  ans.push_back(i);
              }
          }
          return ans;
      }
  
  };
  ```

  

# 二分查找

## [寻找两个正序数组的中位数](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/)

* 题目描述：给定两个大小分别为 `m` 和 `n` 的正序（从小到大）数组 `nums1` 和 `nums2`。请你找出并返回这两个正序数组的 **中位数**

* 代码实现

  ```c++
  int getKthElement(const vector<int>& nums1, const vector<int>& nums2, int k) {
      /* 主要思路：要找到第 k (k>1) 小的元素，那么就取 pivot1 = nums1[k/2-1] 和 pivot2 = nums2[k/2-1] 进行比较
       * 这里的 "/" 表示整除
       * nums1 中小于等于 pivot1 的元素有 nums1[0 .. k/2-2] 共计 k/2-1 个
       * nums2 中小于等于 pivot2 的元素有 nums2[0 .. k/2-2] 共计 k/2-1 个
       * 取 pivot = min(pivot1, pivot2)，两个数组中小于等于 pivot 的元素共计不会超过 (k/2-1) + (k/2-1) <= k-2 个
       * 这样 pivot 本身最大也只能是第 k-1 小的元素
       * 如果 pivot = pivot1，那么 nums1[0 .. k/2-1] 都不可能是第 k 小的元素。把这些元素全部 "删除"，剩下的作为新的 nums1 数组
       * 如果 pivot = pivot2，那么 nums2[0 .. k/2-1] 都不可能是第 k 小的元素。把这些元素全部 "删除"，剩下的作为新的 nums2 数组
       * 由于我们 "删除" 了一些元素（这些元素都比第 k 小的元素要小），因此需要修改 k 的值，减去删除的数的个数
       */
  
      int m = nums1.size();
      int n = nums2.size();
      int index1 = 0, index2 = 0;
  
      while (true) {
          // 边界情况
          if (index1 == m) {
              return nums2[index2 + k - 1];
          }
          if (index2 == n) {
              return nums1[index1 + k - 1];
          }
          if (k == 1) {
              return min(nums1[index1], nums2[index2]);
          }
  
          // 正常情况
          int newIndex1 = min(index1 + k / 2 - 1, m - 1);
          int newIndex2 = min(index2 + k / 2 - 1, n - 1);
          int pivot1 = nums1[newIndex1];
          int pivot2 = nums2[newIndex2];
          if (pivot1 <= pivot2) {
              k -= newIndex1 - index1 + 1;	// k必须这样更新，因为newIdx可能会越界
              index1 = newIndex1 + 1;
          }
          else {
              k -= newIndex2 - index2 + 1;
              index2 = newIndex2 + 1;
          }
      }
  }
  
  double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
      int totalLength = nums1.size() + nums2.size();
      if (totalLength % 2 == 1) {
          return getKthElement(nums1, nums2, (totalLength + 1) / 2);
      }
      else {
          return (getKthElement(nums1, nums2, totalLength / 2) \
                  + getKthElement(nums1, nums2, totalLength / 2 + 1)) / 2.0;
      }
  }
  ```

## [搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)

* 二分查找，分类讨论比较复杂

## [绝对差值和](https://leetcode-cn.com/problems/minimum-absolute-sum-difference/)

* 题目描述

  <img src="imgs/algorithm/problem1818.png" alt="problem1818" style="zoom: 80%;" />

* 思路

  <img src="imgs/algorithm/solution7.png" alt="solution7" style="zoom: 80%;" />

* 代码实现

  ```c++ 
  int mod = 1e9 + 7;
  int minAbsoluteSumDiff(vector<int>& nums1, vector<int>& nums2) {    
      int n = nums1.size();    
      vector<int> record(nums1);    
      sort(record.begin(), record.end());    
      int maxVal = INT_MIN, ret = 0;    
      for(int i = 0; i < n; ++i) {        
          int diff = abs(nums1[i] - nums2[i]);        
          int idx = lower_bound(record.begin(), record.end(), nums2[i]) - record.begin();        
          if(idx < n) {            
              maxVal = max(maxVal, diff - abs(record[idx] - nums2[i]));        
          }        
          if(idx > 0) {            
              maxVal = max(maxVal, diff - abs(record[idx - 1] - nums2[i]));        
          }        
          ret = (ret + diff) % mod;    
      }    
      ret = (ret - maxVal + mod) % mod;    
      return ret;
  }
  ```

# 滑动窗口

## 最小覆盖子串

* 给出两个字符串$S$和$T$，要求在$\textbf{O}(n)$的时间复杂度内在中$S$找出最短的包含 $T$中所有字符的子串。
  例如：

  $S="ADOBECODEBANC"$
  $T="ABC"$
  找出的最短子串为$BANC$

  注意：
  如果$S$中没有包含 $T$中所有字符的子串，返回空字符串 “”；
  满足条件的子串可能有很多，但是题目保证满足条件的最短的子串唯一

* 思路：可以用滑动窗口的思想解决这个问题。在滑动窗口类型的问题中都会有两个指针，一个用于「延伸」现有窗口的`right`指针，和一个用于「收缩」窗口的`left`指针。在任意时刻，只有一个指针运动，而另一个保持静止。我们在$S$上滑动窗口，通过移动`right`指针不断扩张窗口。当窗口包含`T`全部所需的字符后，如果能收缩，我们就收缩窗口直到得到最小窗口

* 优化：其实在$S$ 中，有的字符我们是不关心的，我们只关心$T$中出现的字符，我们可以先预处理 $S$，扔掉那些$T$中没有出现的字符，然后再做滑动窗口

* 代码

  ```c++
  string minWindow(string S, string T) {
      unordered_map<char, int> window, target;
      int left = 0, right =0, n=S.size();
      int start = 0, count = 0;
      int minLen = INT_MAX;
      for(auto & ch : T)
          ++target[ch];
      while(right<n){
          char ch = S[right];
          ++right;
          if(target.count(ch)>0) {  // 优化，仅统计T中出现的字符
              ++window[ch];
              if(window[ch] == target[ch])
                  ++count;
              while(count == int(target.size())) {
                  if(right-left < minLen) {
                      start = left;
                      minLen = right-left;
                  }
                  ch = S[left];
                  ++left;		// 右移缩小窗口
                  if(target.count(ch) > 0) {	// 当前left所指元素在T中，减1
                      if(window[ch] == target[ch]) 
                          --count;
                      --window[ch];
                  }
              }
          }
      }
      return minLen == INT_MAX?"":S.substr(start, minLen);
  }
  ```

## [长度最小的子数组](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)

* 代码实现

  ```c++
  int minSubArrayLen(int target, vector<int>& nums) {
      int n = nums.size();
      int len = INT_MAX;
      int sum = 0;
      int left = 0, right = 0;
      while(right < n) {
          sum += nums[right];	// 前缀和
          while(sum >= target) {	// 缩小窗口，左指针右移
              len = min(len, right - left + 1);
              sum -= nums[left];
              ++left;
          }
          ++right;
      }
      return len == INT_MAX ? 0 : len;
  }
  ```

## [无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

* 实现代码

  ```c++
  int lengthOfLongestSubstring(string s) {
      if(s.empty()) {
          return 0;
      }
      // 记录前面出现的字符，值为对应下标
      unordered_map<char, int> record;
      int maxCount = 1;
  
      int left = 0, right = 0;
      int n = s.size();
      while(right < n) {
          auto ch = s[right];
          // 出现重复，统计前面left到right的长度，并更新left为重复字符前面坐标+1
          if(record.count(ch) > 0 && record[ch] >= left) {
              maxCount = max(maxCount, right - left);
              left = record[ch] + 1;	// 跳过重复字符位置前面所有字符
          }
          record[ch] = right;	// 插入/更新ch的位置
          ++right;
      }
      // 最后可能全部都不重复，统计最后一次
      maxCount = max(maxCount, right - left);
      return maxCount;
  }
  ```

## [串联所有单词的子串](https://leetcode-cn.com/problems/substring-with-concatenation-of-all-words/)

```c++
vector<int> findSubstring(string s, vector<string>& words) {
    int n = s.size();
    int wordLen = words[0].size();
    int wordNum = words.size();
    unordered_map<string, int> wordRec;
    for(auto &str:words) {
        ++wordRec[str];
    }
    vector<int> ret;
    int seqLen = wordLen * wordNum;
    // 多起点滑动窗口
    for(int k = 0; k < wordLen; ++k) {
        for(int i = k; i <= n - seqLen; i += wordLen) {
            unordered_map<string, int> rec;
            int j = i;
            int count = 0;
            while(j < i + seqLen) {
                string word = s.substr(j, wordLen);
                if(wordRec.find(word) == wordRec.end()) {
                    break;
                }
                ++rec[word];
                if(rec[word] == wordRec[word]) {
                    ++count;
                }
                else if(rec[word] > wordRec[word]) {
                    --count;
                    break;
                }
                j += wordLen;
            }
            if(count == wordRec.size()) {
                ret.push_back(i);
            }
        }
    }
    return ret;
}
```

## [可获得的最大点数](https://leetcode-cn.com/problems/maximum-points-you-can-obtain-from-cards/)

* 题目描述

  <img src="imgs/algorithm/problem1423.png" alt="problem1423" style="zoom:80%;" />

* 解题思路：

  * 滑动窗口：每次从开头或末尾拿一张卡牌，取k次，使点数和最大，相当于使中间剩余连续的n-k张卡牌点数之和最小，通过固定窗口大小，滑动过程统计最小和
  * 前缀和：问题转换为从开头取i(i<=k)张卡牌，末尾取k-i张卡牌，使sum[i] + sum[n] - sum[n-k-i]最大

* 代码实现

  ```c++
  // 滑动窗口
  int maxScore(vector<int>& cardPoints, int k) {
      int n = cardPoints.size();
      // 滑动窗口大小为 n-k
      int windowSize = n - k;
      // 选前 n-k 个作为初始值
      int sum = accumulate(cardPoints.begin(), cardPoints.begin() + windowSize, 0);
      int minSum = sum;
      for (int i = windowSize; i < n; ++i) {
          // 滑动窗口每向右移动一格，增加从右侧进入窗口的元素值，并减少从左侧离开窗口的元素值
          sum += cardPoints[i] - cardPoints[i - windowSize];
          minSum = min(minSum, sum);
      }
      return accumulate(cardPoints.begin(), cardPoints.end(), 0) - minSum;
  }
  
  // 前缀和
  int maxScore(vector<int>& cardPoints, int k) {
      int n = cardPoints.size();
      for(int i = 1; i < n; ++i) {
          cardPoints[i] += cardPoints[i-1];
      }
      if(n == k) {
          return cardPoints[n-1];
      }
      // 将maxSum初始化为i=0的值
      int maxSum = cardPoints[n-1] - cardPoints[n-1-k];
      // i 为 1...k
      for(int i = 1; i <= k; ++i) {
          int sum = cardPoints[i-1] + cardPoints[n-1] - cardPoints[n-1+i-k];
          maxSum = max(maxSum, sum);
      }
      return maxSum;
  }
  ```


# 动态规划

## 分苹果

* 题目描述：把m个同样的苹果放在n个同样的盘子里，允许有的盘子空着不放，问共有多少种不同的分法？（用K表示）5，1，1和1，5，1 是同一种分法

* 思路：$dp[n][m]$为$n$个盘子$m$个苹果总摆放数，$dp[i][j]$为$i$个盘子$j$个苹果的摆放数，则有

  * 如果$i>j$则，必有一个以上是空盘，其摆放方式相当于将$m$个苹果放入$i-1$个盘子，所以$dp[i][j] = dp[i-1][j]$
  * 如果$i\le j$则，有两种情况：
    * 如果有一个以上空盘，$dp[i][j] = dp[i-1][j]$；
    * 若每个盘子都摆有一个苹果，摆放数为相当于将剩余$j-i$个苹果摆放到$i$个盘子中
    * 摆放总数为上面两种情况的和：$dp[i][j] = dp[i-1][j] + dp[i][j-i]$

* 边界条件，只有一个苹果或0个苹果时，摆放数为1

* 代码

  ```c++
  int putApples(int n, int m){
      if(n < 0||m < 0)
          return 0;
      vector<vector<int> > dp(n+1, vector<int> (m+1, 0));
      
      for(int i = 0; i <= n; i++)
          dp[i][0] = dp[i][1] = 1;
      
      for(int i = 1; i <= n; i++){
          for(int j = 2; j <= m; j++){
              dp[i][j] = dp[i-1][j];
              if(j >= i)
                  dp[i][j] += dp[i][j-i];
          }
      }
      return dp[n][m];
  }
  ```

## [最长公共子序列(LCS)](https://leetcode-cn.com/problems/longest-common-subsequence/)

* 题目描述

  * 给定两个字符串 text1 和 text2，返回这两个字符串的最长 公共子序列 的长度。如果不存在 公共子序列 ，返回 0 。一个字符串的 子序列 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。

    例如，"ace" 是 "abcde" 的子序列，但 "aec" 不是 "abcde" 的子序列。
    两个字符串的 公共子序列 是这两个字符串所共同拥有的子序列。

* 思路

  <img src="imgs/algorithm/solution11.png" alt="solution11" style="zoom:80%;" />

* 代码实现

  ```c++
  int longestCommonSubsequence(string text1, string text2) {
      int n = text1.size();
      int m = text2.size();
      vector<vector<int> > dp(n+1, vector<int> (m+1, 0));
  
      for(int i = 1; i <= n; ++i) {
          for(int j = 1; j <= m; ++j) {
              if(text1[i-1] == text2[j-1]) {
                  dp[i][j] = dp[i-1][j-1] + 1;
              }
              else {
                  dp[i][j] = max(dp[i][j-1], dp[i-1][j]);
              }
          }
      }
      return dp[n][m];
  }
  ```

## 最长递增子序列(LIS)

* 最长递增子序列不唯一，但是最长大小唯一，可以使用动态规划

* 贪心+二分解法：如果我们要使上升子序列尽可能的长，则我们需要让序列上升得尽可能慢，因此我们希望每次在上升子序列最后加上的那个数尽可能的小

  ![solution2](imgs/algorithm/solution2.png)

* 代码

  ```c++
  // n2 解法
  int LIS(vector<int> & arr){
      if(arr.empty())
          return 0;
      int n = arr.size();
      vector<int> dp(n, 1);
      int maxLen = 0;
      for(int i = 1; i < n; i++) {
          for(int j = 0; j < i; j++){
              if(arr[j] < arr[i]){
                  dp[i] = max(dp[i], dp[j] + 1);
              }
          }
          maxLen = max(maxLen, dp[i]);
      }
      return maxLen;
  }
  
  // nlogn 解法，贪心+二分
  int lengthOfLIS(vector<int>& nums) {
      int n = (int)nums.size();
      if (n == 0) {
          return 0;
      }
      vector<int> d;
      d.push_back(nums[0]);
      for (int i = 1; i < n; ++i) {
          if (nums[i] > d.back()) {
              d.push_back(nums[i]);
          } else {
              auto pos = lower_bound(d.begin(), d.end(), nums[i]) - d.begin();
              d[pos] = nums[i];
          }
      }
      return int(d.size());
  }
  ```

## LIS进阶

* 给定数组arr，设长度为n，输出arr的最长递增子序列。**如果有多个答案，请输出其中字典序最小的**。此时需要输出字典序中最小的子序列

* 解法：

  * 两步走：1. 求最大递增子序列长度；2. 求字典靠前子序列

* 第一步思路：

  * 贪心+二分：假设数组arr为`[2, 3, 1, 2, 3]`，**vec数组里面存放递增子序列(存放长度为i+1的LIS的最后一个数为vec[i])，maxLen数组里存放以元素i结尾的最大递增子序列长度**，那么遍历数组arr并执行如下更新规则:

    1. 初始情况下，vec为[2]，maxLen[1]
    2. 接下来遇到3，由于vec最后一个元素小于3，直接更新，vec为[2,3]，maxLen[1,2]
    3. 接下来遇到1，由于vec**最后的元素大于1**, 我们在vec中**查找大于等于1的第一个元素的下标，并用1替换之**，此时vec为[1,3], maxLen[1,2,1]
    4. 接下来遇到2，由于vec最后的元素大于2，我们在vec中查找大于等于2的第一个元素的下标，并用2替换之，此时vec为[1,2], maxLen[1,2,1,2]
    5. 接下来遇到3，由于vec最后一个元素小于3，直接更新，vec为[1,2,3]，maxLen为[1,2,1,2,3]
    6. 此时**vec的大小就是整个序列中最长递增子序列的长度**（但是vec不一定是本题的最终解）

* 第二步思路：

  * 假设我们原始数组是arr，得到的maxLen为`[1,2,3,1,3]`，最终输出结果为res（字典序最小的最长递增子序列），res的最后一个元素在arr中位置无庸置疑是`maxLen[i]==3`对应的下标，那么到底是`arr1[2]`还是`arr[4]`呢？如果是`arr[2]`，那么`arr[2]<arr[4]`，则`maxLen[4]=4`，与已知条件相悖。因此我们应该取`arr[4]`放在res的最后一个位置，**同长度，后面的一定比前面小，所以从后往前面填充res**

* 代码实现

  ```c++
  vector<int> LIS(vector<int> & arr){
      if(arr.empty())
          return {};
      vector<int> ret;	// 记录结果
      vector<int> maxLen;	// 记录以arr[i]为最后元素的LIS长度
      int n = arr.size();
      ret.emplace_back(arr[0]);
      maxLen.emplace_back(1);
      for(int i = 1; i < n; ++i){
          if(arr[i] > ret.back()){
              ret.emplace_back(arr[i]);
              maxLen.emplace_back(ret.size());
          }
          else{
              // 第一个大于等于arr[i]的位置
              int pos = lower_bound(ret.begin(), ret.end(), arr[i]) - ret.begin();	            				arr[pos] = arr[i];
              maxLen.emplace_back(pos + 1);
          }   
      }
      for(int i = n - 1, j = ret.size(); i>=0; --i) {	// 从后往前填充
          if(maxLen[i] == j)
              ret[--j] = arr[i];
      }
      return ret;
  }
  ```

## [俄罗斯套娃信封问题](https://leetcode-cn.com/problems/russian-doll-envelopes/)

* 二维矩阵，与上面LIS有异曲同工之妙，只需把第一维排好序，然后在第二维求LIS即可。同样有两种复杂度解法

  ```c++
  // n2 解法
  int maxEnvelopes(vector<vector<int>>& envelopes) {
  	int n = envelopes.size();
      // 排序函数，在第一维相同情况下，第二维降序排列，能防止同时出现第一维度相同的互套的情况，很妙！
      auto comp = [](vector<int> &vec1, vector<int> &vec2) {
          return (vec1[0] < vec2[0]) || (vec1[0] == vec2[0] && vec1[1] > vec2[1]);
      };
      sort(envelopes.begin(), envelopes.end(), comp);
     	vector<int> dp(n, 1);
      int maxNum = 1;
      for(int i = 1; i < n; ++i) {
          for(int j = i-1; j >= 0; --j) {
              if(envelops[j][1] < envelopes[i][1]) {
                  dp[i] = max(dp[i], dp[j] + 1);
              }
          }
          maxNum = max(maxNum, dp[i]);
      }
      return maxNum;
  }
  
  // nlogn解法
  int maxEnvelopes(vector<vector<int>>& envelopes) {
  	int n = envelopes.size();
      auto comp = [](vector<int> &vec1, vector<int> &vec2) {
          return (vec1[0] < vec2[0]) || (vec1[0] == vec2[0] && vec1[1] > vec2[1]);
      };
      sort(envelopes.begin(), envelopes.end(), comp);
     	vector<int> vec;
      vec.emplace_back(envelopes[0][1]);
      for(int i = 1; i < n; ++i) {
          if(envelopes[i][1] > vec.back()) {
              vec.emplace_back(envelopes[i][1]);
          }
          else {
              auto pos = lower_bound(vec.begin(), vec.end(), envelopes[i][1]) - vec.begin();
              vec[pos] = envelopes[i][1];
          }
      }
      return int(vec.size());
  }
  ```
  

## 正则表达式匹配

* 给你一个字符串 s 和一个字符规律 p，请你来实现一个支持 '.' 和 '*' 的正则表达式匹配

  '.' 匹配任意单个字符
  '*' 匹配零个或多个前面的那一个元素
  所谓匹配，是要涵盖 整个 字符串 s的，而不是部分字符串

* 思路一：用$f[i][j]$表示 *s* 的前 *i* 个字符与 *p* 中的前 *j* 个字符是否能够匹配。考虑 *p* 的第 *j* 个字符的匹配情况

  * 如果*p*的第 *j* 个字符是一个小写字母，那么我们必须在 *s* 中匹配一个相同的小写字母，即

    $$f[i][j] = \begin{cases}&f[i-1][j-1], &s[i]=p[j];\\ &false, &s[i] \ne p[j]. \end{cases}$$

  * 字母 + 星号的组合在匹配的过程中，本质上只会有两种情况

    * 匹配 *s* 末尾的一个字符，将该字符扔掉，而该组合还可以继续进行匹配；
    * 不匹配字符，将该组合扔掉，不再进行匹配

  * 可以得到字母 + 星号的组合的状态转移方程为

    $$f[i][j] = \begin{cases}&f[i-1][j] \; or\; f[i][j-2], &s[i]=p[j-1];\\ &f[i][j-2], &s[i] \ne p[j-1]. \end{cases}$$

  * 在任意情况下，只要*p*[*j*] 是 `.`，那么*p*[*j*] 一定成功匹配 *s* 中的任意一个小写字母

  * 边界条件：两个空字符串是可以匹配的，$f[0][0] = $true

* 思路二：递归实现

  <img src="https://gitee.com/canqchen/cloudimg/raw/master/img/regex_recursive.png" alt="regex_recursive" style="zoom:80%;" />

* 代码实现

  ```c++
  // 动态规划
  bool isMatch(string s, string p) {
      int m = s.size();
      int n = p.size();
  
      auto matches = [&](int i, int j) {
          if (i == 0) {
              return false;
          }
          if (p[j - 1] == '.') {
              return true;
          }
          return s[i - 1] == p[j - 1];
      };
  
      vector<vector<bool> > f(m + 1, vector<bool>(n + 1, false));
      f[0][0] = true;
      for (int i = 0; i <= m; ++i) {
          for (int j = 1; j <= n; ++j) {
              if (p[j - 1] == '*') {
                  f[i][j] = f[i][j] | f[i][j - 2];	// 不匹配"字母+星号"组合
                  if (matches(i, j - 1)) {		// 星号前字母匹配，匹配"字母+星号"组合
                      f[i][j] = f[i][j] | f[i - 1][j];  
                  }
              }
              else {
                  if (matches(i, j)) {
                      f[i][j] = f[i][j] | f[i - 1][j - 1];
                  }
              }
          }
      }
      return f[m][n];
  }
  
  // 递归实现
  bool isMatch(string s, string p) {
      if(s.empty() && p.empty())
          return true;
      if(p.empty())
          return false;
      int m = s.size(), n = p.size();
      if(n > 1 && p[1]=='*') {		// alpha + *
          if(m > 0 && (s[0] == p[0] || p[0]=='.')){	// 星号前字母匹配
              return isMatch(s.substr(1), p) 			 // 匹配大于等于一个alpha
                  || isMatch(s, p.substr(2));			// 不匹配alpha，丢弃 alpha + *
          }
          else{
              return isMatch(s, p.substr(2));  // 星号前字母不匹配，丢弃 alpha + *
          }
      }
      if(m > 0 && (s[0] == p[0] || p[0] == '.'))
          return isMatch(s.substr(1), p.substr(1));  
      return false;
  }
  ```

## [完全平方数](https://leetcode-cn.com/problems/perfect-squares/)

* 题目描述

  <img src="imgs/algorithm/problem279.png" alt="problem279" style="zoom:80%;" />

* 解题代码

  ```c++
  int numSquares(int n) {
      if(n<=0)
          return 0;
      vector<int> dp(n + 1, INT_MAX);
      int max_idx = sqrt(n) + 1;
      vector<int> square_nums(max_idx);
      dp[0] = 0;
      for(int i = 1; i < max_idx; i++) {
          square_nums[i] = i*i;
      }
  
      for(int i = 1; i <= n; i++){
          for(int j = 1; j < max_idx; j++){
              if(i < square_nums[j])
                  break;
              dp[i] = min(dp[i], dp[i-square_nums[j]] + 1);
          }
      }
      return dp[n];
  }
  ```

## [目标和](https://leetcode-cn.com/problems/target-sum/)

* 题目描述

  <img src="imgs/algorithm/problem494.png" alt="problem494" style="zoom:80%;" />

* 解题思路

  * 回溯，暴力搜索
  * dp

  <img src="imgs/algorithm/solution494.png" alt="solution494" style="zoom:80%;" />

* 解题代码

  ```c++
  int findTargetSumWays(vector<int>& nums, int target) {
      int n = nums.size();
      int totalSum = 0;
      for(auto & num : nums){
          totalSum += num;
      }
  
      if(abs(target) > abs(totalSum)){
          return 0;
      }
      int len = totalSum * 2 + 1;
      vector<vector<int> > dp(n, vector<int> (len, 0));
  
      dp[0][nums[0] + totalSum] = 1;
      dp[0][-nums[0] + totalSum] += 1;
      for(int i = 1; i < n; i++) {
          for(int j = -totalSum; j <= totalSum; j++) {
              if(dp[i-1][j+totalSum] > 0){
                  dp[i][j - nums[i] + totalSum] += dp[i - 1][j + totalSum];
                  dp[i][j + nums[i] + totalSum] += dp[i - 1][j + totalSum];
              }
          }
      }
      return dp[n-1][target + totalSum];
  }
  
  // 根据思路优化
  int findTargetSumWays(vector<int>& nums, int target) {
      int sum = 0;
      for (int& num : nums) {
          sum += num;
      }
      int diff = sum - target;
      if (diff < 0 || diff % 2 != 0) {
          return 0;
      }
      int n = nums.size(), neg = diff / 2;
      vector<vector<int>> dp(n + 1, vector<int>(neg + 1));
      dp[0][0] = 1;
      for (int i = 1; i <= n; i++) {
          int num = nums[i - 1];
          for (int j = 0; j <= neg; j++) {
              dp[i][j] = dp[i - 1][j];
              if (j >= num) {
                  dp[i][j] += dp[i - 1][j - num];
              }
          }
      }
      return dp[n][neg];
  }
  ```

  

## [打家劫舍 II](https://leetcode-cn.com/problems/house-robber-ii/)

* 代码实现(不能同时抢劫第一间房子和最后一间，分两种情况)

  ```c++
  // 利用滚动数组，降低空间复杂度
  int rob(vector<int>& nums) {
      if(nums.empty())
          return 0;
      if(nums.size() == 1)
          return nums[0];
      if(nums.size() == 2)
          return max(nums[0], nums[1]);
  
      int n = nums.size();
  	// [0, n-2]
      int preMax = nums[0];
      int curMax1 = max(nums[0], nums[1]);
      for(int i = 2; i < n-1; ++i) {
          int tmp = max1;
          curMax1 = max(preMax + nums[i], curMax1);
          preMax = tmp;
      }
  	// [1, n-1]
      preMax = nums[1];
      int curMax2 = max(nums[1], nums[2]);
      for(int i = 3; i < n; ++i) {
          int tmp = curMax2;
          curMax2 = max(preMax + nums[i], curMax2);
          preMax = tmp;
      }
      return max(curMax1, curMax2);
  }
  ```

## [不同的二叉搜索树](https://leetcode-cn.com/problems/unique-binary-search-trees/)

## [最大整除子集](https://leetcode-cn.com/problems/largest-divisible-subset/)

* 重要思路：整除关系传递性

  * 若a整除b，且b整除c，则a整除c，据此，有

    * 如果整数 a是整除子集 S1的最小整数 b 的约数（即 a整除b），那么可以将 a添加到 S1 中得到一个更大的整除子集；
    * 如果整数 c 是整除子集 S2的最大整数 d 的倍数（即 d整除c），那么可以将 c 添加到 S2

    <img src="imgs/algorithm/solution1.png" alt="solution1" style="zoom:80%;" />

  * 代码实现

    ```c++
    vector<int> largestDivisibleSubset(vector<int>& nums) {
        if(nums.empty()) 
            return {};
        sort(nums.begin(), nums.end());
        int n = nums.size();
        // dp定义为以dp[i]为最大数的集合的元素数目
        // 初始化，每个集合至少包含自身，设为1
        vector<int> dp(n, 1);
        // 统计dp[i]为最大数的集合的元素数目
        for(int i = 1; i < n; ++i) {
            for(int j = 0; j < i; ++j) {
                if(nums[i] % nums[j] == 0) {	// 可将dp[i]纳入dp[j]所在集合
                    dp[i] = max(dp[i], dp[j] + 1);
                }
            }
        }
        // 找到最大集合，并记录其下标
        int maxCount = dp[0], maxIdx = 0;
        for(int i = 1; i < n; ++i) {
            if(dp[i] > maxCount) {
                maxCount = dp[i];
                maxIdx = i;
            }
        }
        // 找到最大集合元素
        vector<int> ret;
        // 从后往前找，需要记录较大数用来判断是否该集合
        int larger = nums[maxIdx];
        for(int i = maxIdx; i >= 0; --i) {
            if(dp[i] == maxCount && larger % nums[i] == 0) {
                ret.push_back(nums[i]);
                // 用于寻找下一个
                larger = nums[i];
                --maxCount;
            }
        }
        return ret;
    }
    ```

## [最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)

* 思路：

  * 动态规划

    <img src="imgs/algorithm/solution5.png" alt="solution5" style="zoom:67%;" />

  * 中心扩展

    <img src="imgs/algorithm/solution6.png" alt="solution6" style="zoom:67%;" />

* 代码实现

  ```c++
  // dp
  string longestPalindrome(string s) {
      if(s.empty())
          return "";
      int n = s.size();
      int maxLen = 1, start = 0;
      vector<vector<bool> > dp(n, vector<bool> (n, false));
      for(int i = 0; i<n;i++)
          dp[i][i] = true;
      for(int i = n - 1; i >= 0; i--){
          for(int j = i + 1; j < n; j++){
              if(s[i] == s[j]){
                  if(j == i+1)
                      dp[i][j] = true;
                  else
                      dp[i][j] = dp[i+1][j-1];
              }
              else {
                  dp[i][j] = false;
              }
              
              if(dp[i][j] && j - i + 1 > maxLen) {
                  maxLen = j - i + 1;
                  start = i;
              }
          }
      }
      return s.substr(start, maxLen);
  }
  
  // 中心扩展
  string longestPalindrome(string s) {
      if(s.empty())
          return "";
      int n = s.size();
      
      auto expand = [s = s, n = n](int left, int right)->pair<int, int> {
          while(left >= 0 && right < n && s[left] == s[right]) {
              ++right;
              --left;
          }
          // 回退到回文串首尾
          return {left + 1, right - 1};
      };
      
      int maxLen = 1, start = 0;
      for(int i = 0; i < n; ++i) {
          auto [left1, right1] = expand(i, i);
          auto [left2, right2] = expand(i, i + 1);
          if(right1 - left1 + 1 > maxLen) {
              start = left1;
              maxLen = right1 - left1 + 1;
          }
          if(right2 - left2 + 1 > maxLen) {
              start = left2;
              maxLen = right2 - left2 + 1;
          }
      }
      return s.substr(start, maxLen);
  }
  ```

## [最长回文子序列](https://leetcode-cn.com/problems/longest-palindromic-subsequence/)

* 思路

  <img src="imgs/algorithm/solution516.png" alt="solution516" style="zoom:80%;" />

  * 其他思路：将字符串反转，然后求两个字符串的最长公共子序列

* 解题代码

  ```c++
  int longestPalindromeSubseq(string s) {
      int n = s.size();
      vector<vector<int> > dp(n, vector<int> (n, 0));
      for(int i = n-1; i >= 0; --i) {
          dp[i][i] = 1;
          for(int j = i + 1; j < n; ++j) {
              if(s[i] == s[j]) {
                  dp[i][j] = dp[i+1][j-1] + 2;
              }
              else {
                  dp[i][j] = max(dp[i+1][j], dp[i][j-1]);
              }
          }
      }
      return dp[0][n-1];
  }
  ```

  

## [分割回文串 II](https://leetcode-cn.com/problems/palindrome-partitioning-ii/)

* 思路  

  <img src="imgs/algorithm/solution8.png" alt="solution8" style="zoom:67%;" />

* 代码实现  
	```c++  
	int minCut(string s) {
      if(s.empty()) {
          return 0;
      }
      int n = s.size();
      // 先统计回文串
      vector<vector<bool> > isPam(n, vector<bool>(n, false));
      for(int i = 0; i < n; ++i) {
          isPam[i][i] = true;
      }
      for(int i = n - 2; i >= 0; --i) {
          for(int j = i + 1; j < n; ++j) {
              if(s[j] == s[i]) {
                  if(j == i + 1){
                      isPam[i][j] = true;
                  }
                  else {
                      isPam[i][j] = isPam[i+1][j-1];
                  }
              }
          }
      }
  
      // 简单写法
      /*
        vector<vector<bool> > isPam(n, vector<bool>(n, true));
        for(int i = n - 2; i >= 0; --i) {
            for(int j = i + 1; j < n; ++j) {
              isPam[i][j] = (s[i] == s[j]) && isPam[i+1][j-1];
            }
        }
        */
  
      vector<int> dp(n, INT_MAX);
      for(int i = 0; i < n; ++i) {
          // 注意，容易漏掉整个串为回文串情况
          if(isPam[0][i]) {
              dp[i] = 0;
          }
          else {
              for(int j = 0; j < i; ++j) {
                  if(isPam[j + 1][i]) {
                      dp[i] = min(dp[i], dp[j] + 1);
                  }
              }
          }
      }
      return dp[n-1];
  }
  ```

## [编辑距离](https://leetcode-cn.com/problems/edit-distance/)

* 题目描述

  <img src="imgs/algorithm/problem72.png" alt="problem72" style="zoom:80%;" />

* 代码及思路

  ```c++
  int minDistance(string word1, string word2) {
      int m = word1.size(), n = word2.size();
      // dp[i][j]表示word1[0...i-1]转换成word2[0...j-1]所使用的最少操作
      vector<vector<int> > dp(m+1, vector<int> (n+1, 0));
      // 初始化边界条件
      // word1为空(0个字符)，转换成word2[0...i-1]只需插入i次
      for(int i = 1; i <= n; i++)
          dp[0][i] = i;
      // word2为空(0个字符)，转换成word1[0...i-1]只需插入i次
      for(int i = 1; i <= m; i++)
          dp[i][0] = i;
  
      for(int i = 1; i <= m; i++) {
          for(int j = 1; j <= n; j++) {
              // 两个字符相同，转换次数与之前字串相同
              if(word1[i-1] == word2[j-1]) {
                  dp[i][j] = dp[i-1][j-1];
              }
              else {
                  // dp[i][j]可以由
                  //     dp[i][j-1] 插入一个字符
                  //     dp[i-1][j] 删除一个字符
                  //     dp[i-1][j-1] 替换一个字符
                  // 转化得到。最少操作数取最小值
                  dp[i][j] = min(min(dp[i][j-1], dp[i-1][j]), dp[i-1][j-1]) + 1;
              }
          }
      }
      return dp[m][n];
  }
  ```

## [戳气球](https://leetcode-cn.com/problems/burst-balloons/)

## [恢复空格](https://leetcode-cn.com/problems/re-space-lcci/)(Trie+dp)

* 题目描述

  <img src="imgs/algorithm/problem171-3.png" alt="problem171-3" style="zoom: 80%;" />

* 解题思路，Trie+dp

  <img src="imgs/algorithm/solution17-13.png" alt="solution17-13" style="zoom:80%;" />

* 解题代码

  ```c++
  // 暴力解法：hash
  int respace(vector<string>& dictionary, string sentence) {
      unordered_set<string> dict(dictionary.begin(), dictionary.end());
      int n = sentence.size();
      vector<int> dp(n+1, INT_MAX);
      dp[0] = 0;
      for(int i = 1; i <= n; ++i) {
          dp[i] = dp[i-1] + 1;
          for(int j = 0; j < i; ++j) {
              string tmp = move(sentence.substr(j, i - j));
              if(dict.count(tmp)) {
                  dp[i] = min(dp[i], dp[j]);
              }
          }
      }
      return dp[n];
  }
  
  // 在hash内层循环中，判断j...i的字符串是否存在时，后缀都是相同的，
  // 所以使用前缀树，将字符串倒序插入前缀树，进行优化，内层遍历从后往前
  class Trie {
  public:
      Trie* next[26] = {nullptr};
      bool isEnd;
      
      Trie() {
          isEnd = false;
      }
  
      void insert(string s) {
          Trie* curPos = this;
  
          for (int i = s.length() - 1; i >= 0; --i) {
              int t = s[i] - 'a';
              if (curPos->next[t] == nullptr) {
                  curPos->next[t] = new Trie();
              }
              curPos = curPos->next[t];
          }
          curPos->isEnd = true;
      }
  };
  
  class Solution {
  public:
      int respace(vector<string>& dictionary, string sentence) {
          int n = sentence.length(), inf = 0x3f3f3f3f;
  
          Trie* root = new Trie();
          for (auto& word: dictionary) {
              root->insert(word);
          }
  
          vector<int> dp(n + 1, inf);
          dp[0] = 0;
          for (int i = 1; i <= n; ++i) {
              dp[i] = dp[i - 1] + 1;
  
              Trie* curPos = root;
              // 从后往前进行遍历，当不存在后缀时，尽早跳出进行优化
              for (int j = i; j >= 1; --j) {
                  int t = sentence[j - 1] - 'a';
                  if (curPos->next[t] == nullptr) {
                      break;
                  } else if (curPos->next[t]->isEnd) {
                      dp[i] = min(dp[i], dp[j - 1]);
                  }
                  if (dp[i] == 0) {
                      break;
                  }
                  curPos = curPos->next[t];
              }
          }
          return dp[n];
      }
  };
  ```

## [丑数 II](https://leetcode-cn.com/problems/ugly-number-ii/)

* 解题代码

  ```c++
  int nthUglyNumber(int n) {
      vector<int> dp(n + 1);
      dp[1] = 1;
      int p2 = 1, p3 = 1, p5 = 1;
      for (int i = 2; i <= n; i++) {
          int num2 = dp[p2] * 2, num3 = dp[p3] * 3, num5 = dp[p5] * 5;
          dp[i] = min(min(num2, num3), num5);
          if (dp[i] == num2) {
              p2++;
          }
          if (dp[i] == num3) {
              p3++;
          }
          if (dp[i] == num5) {
              p5++;
          }
      }
      return dp[n];
  }
  ```

## [超级丑数](https://leetcode-cn.com/problems/super-ugly-number/)

* 解题代码：

  ```c++
  int nthSuperUglyNumber(int n, vector<int>& primes) {
      vector<int> dp(n + 1);
      dp[1] = 1;
      int m = primes.size();
      vector<int> pointers(m, 1);
      for (int i = 2; i <= n; i++) {
          vector<int> nums(m);
          int minNum = INT_MAX;
          for (int j = 0; j < m; j++) {
              nums[j] = dp[pointers[j]] * primes[j];
              minNum = min(minNum, nums[j]);
          }
          dp[i] = minNum;
          for (int j = 0; j < m; j++) {
              if (minNum == nums[j]) {
                  pointers[j]++;
              }
          }
      }
      return dp[n];
  }
  ```

## [等差数列划分 II - 子序列](https://leetcode-cn.com/problems/arithmetic-slices-ii-subsequence/)

* 题目描述

  ![problem446](imgs/algorithm/problem446.png)

* 思路：dp+哈希

  ![solution446](imgs/algorithm/solution446.png)

* 解题代码

  ```c++
  int numberOfArithmeticSlices(vector<int>& nums) {
      int n = nums.size();
      int ret = 0;
      vector<unordered_map<long long, int> > dp(n);
      for(int i = 0; i < n; ++i) {
          for(int j = 0; j < i; ++j) {
              long long diff = 0LL + nums[i] - nums[j];
              auto it = dp[j].find(diff);
              auto count = it == dp[j].end() ? 0 : it->second;
              ret += count;
              dp[i][diff] += count + 1;
          }
      }
      return ret;
  }
  ```

## [交错字符串](https://leetcode-cn.com/problems/interleaving-string/)

* 题目描述

  <img src="imgs/algorithm/problem97.png" alt="problem97" style="zoom:80%;" />

* 解题思路

  <img src="imgs/algorithm/solution97.png" alt="solution97" style="zoom:80%;" />

* 代码实现

  ```c++
  
  bool isInterleave(string s1, string s2, string s3) {
      auto f = vector < vector <int> > (s1.size() + 1, vector <int> (s2.size() + 1, false));
  
      int n = s1.size(), m = s2.size(), t = s3.size();
  
      if (n + m != t) {
          return false;
      }
  
      f[0][0] = true;
      for (int i = 0; i <= n; ++i) {
          for (int j = 0; j <= m; ++j) {
              int p = i + j - 1;
              if (i > 0) {
                  f[i][j] |= (f[i - 1][j] && s1[i - 1] == s3[p]);
              }
              if (j > 0) {
                  f[i][j] |= (f[i][j - 1] && s2[j - 1] == s3[p]);
              }
          }
      }
  
      return f[n][m];
  }
  
  // 滚动数组优化
  bool isInterleave(string s1, string s2, string s3) {
      auto f = vector <int> (s2.size() + 1, false);
  
      int n = s1.size(), m = s2.size(), t = s3.size();
  
      if (n + m != t) {
          return false;
      }
  
      f[0] = true;
      for (int i = 0; i <= n; ++i) {
          for (int j = 0; j <= m; ++j) {
              int p = i + j - 1;
              if (i > 0) {
                  f[j] &= (s1[i - 1] == s3[p]);
              }
              if (j > 0) {
                  f[j] |= (f[j - 1] && s2[j - 1] == s3[p]);
              }
          }
      }
  
      return f[m];
  }
  ```

## [出界的路径数](https://leetcode-cn.com/problems/out-of-boundary-paths/)

* 题目描述

  <img src="imgs/algorithm/problem576.png" alt="problem576" style="zoom:80%;" />

* 解题思路

  * 记忆化搜索：自上而下

  * 动态规划：自下而上

    <img src="imgs/algorithm/solution576.png" alt="solution576" style="zoom:80%;" />

* 解题代码

  ```c++
  int findPaths(int m, int n, int maxMove, int startRow, int startColumn) {
      if(maxMove == 0) {
          return 0;
      }
  
      vector<pair<int, int> > directions{{1,0}, {-1, 0}, {0, 1}, {0, -1}};
  	// 记录状态
      vector<vector<vector<int>> > rec(maxMove + 1, vector<vector<int> > (m, vector<int>(n, -1)));
      function<int(int,int,int)> dfs = [&](int row, int col, int move) {
          if(row < 0 || row >= m || col < 0 || col >= n) {
              return 1;
          }
          // move为0，还没到达界外，返回0
          if(move == 0) {
              return 0;
          }
          // 已搜索过此状态
          if(rec[move][row][col] != -1) {
              return rec[move][row][col];
          }
          int count = 0;
          for(auto [x, y] : directions) {
              count += dfs(row + x, col + y, move - 1);
              count %= 1000000007;
          }
          // 记录状态
          rec[move][row][col] = count;
          return count;
      };
      return dfs(startRow, startColumn, maxMove);
  }
  
  // dp
  int findPaths(int m, int n, int maxMove, int startRow, int startColumn) {
      constexpr int MOD = 1'000'000'007;
      vector<vector<int>> directions = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
      int outCounts = 0;
      vector<vector<vector<int>>> dp(maxMove + 1, vector<vector<int>>(m, vector<int>(n)));
      dp[0][startRow][startColumn] = 1;
      for (int i = 0; i < maxMove; i++) {
          for (int j = 0; j < m; j++) {
              for (int k = 0; k < n; k++) {
                  int count = dp[i][j][k];
                  if (count > 0) {
                      for (auto &direction : directions) {
                          int j1 = j + direction[0], k1 = k + direction[1];
                          if (j1 >= 0 && j1 < m && k1 >= 0 && k1 < n) {
                              dp[i + 1][j1][k1] = (dp[i + 1][j1][k1] + count) % MOD;
                          } else {
                              outCounts = (outCounts + count) % MOD;
                          }
                      }
                  }
              }
          }
      }
      return outCounts;
  }
  ```

## [不同的子序列](https://leetcode-cn.com/problems/distinct-subsequences/)

* 题目描述

  <img src="imgs/algorithm/problem115.png" alt="problem115" style="zoom:80%;" />

* 解题思路：[「手画图解」详解两种解法：记忆化递归、动态规划 | 115.不同的子序列 - 不同的子序列 - 力扣（LeetCode） (leetcode-cn.com)](https://leetcode-cn.com/problems/distinct-subsequences/solution/shou-hua-tu-jie-xiang-jie-liang-chong-ji-4r2y/)

* 解题代码

  ```c++
  int numDistinct1(string s, string t) {
      int sLen = s.size(), tLen = t.size();
      if(sLen < tLen) {
          return 0;
      }
  	// 判断极端case越界
      if(sLen == tLen) {
          if(s == t)
              return 1;
          return 0;
      }
  
      vector<vector<long> > dp(sLen + 1, vector<long>(tLen + 1, 0));
      for(int i = 0; i <= sLen; ++i) {
          dp[i][0] = 1;
      }
      for(int j = 1; j <= tLen; ++j) {
          dp[0][j] = 0;
      }
      for(int i = 1; i <= sLen; ++i) {
          for(int j = 1; j <= tLen; ++j) {
              dp[i][j] = dp[i-1][j];
              if(s[i-1] == t[j-1]) {
                  dp[i][j] += dp[i-1][j-1];
              }
          }
      }
      return dp[sLen][tLen];
  }
  ```

## [学生出勤记录 II](https://leetcode-cn.com/problems/student-attendance-record-ii/)

* 题目描述

  <img src="imgs/algorithm/problem552.png" alt="problem552" style="zoom:80%;" />

* 解题思路

  <img src="imgs/algorithm/solution552.png" alt="solution552" style="zoom:80%;" />

  * 状态机

    <img src="imgs/algorithm/solution552.png" alt="solution552" style="zoom:80%;" />

* 解题代码

  ```c++
  int checkRecord(int n) {
      int mod = 1000000007;
      vector<vector<vector<int> > > dp(n + 1, vector<vector<int>>(2, vector<int>(3)));
      dp[0][0][0] = 1;
      for(int i = 1; i <= n; ++i) {
         // 第i天以p结尾
          for(int j = 0; j <=1; ++j) {
              for(int k = 0; k <= 2; ++k) {
                  dp[i][j][0] = (dp[i][j][0] + dp[i-1][j][k]) % mod;
              }
          }
          // 第i天以A结尾
          for(int k = 0; k <= 2; ++k) {
             dp[i][1][0] = (dp[i][1][0] + dp[i-1][0][k]) % mod;
          }
          // 第i天以L结尾
          for(int j = 0; j <= 1; ++j) {
              for(int k = 0; k <= 1; ++k) {
                  dp[i][j][k + 1] = (dp[i][j][k + 1] + dp[i-1][j][k]) % mod;
              }
          }
      }
      int ret = 0;
      for(int j = 0; j <= 1; ++j) {
          for(int k = 0; k <= 2; ++k) {
              ret = (ret + dp[n][j][k]) % mod;
          }
      }
      return ret;
  }
  
  // 空间优化
  int checkRecord(int n) {
      constexpr int MOD = 1'000'000'007;
      int dp[2][3];  // A 的数量，结尾连续 L 的数量
      memset(dp, 0, sizeof(dp));
      dp[0][0] = 1;
      for (int i = 1; i <= n; i++) {
          int dpNew[2][3];  // A 的数量，结尾连续 L 的数量
          memset(dpNew, 0, sizeof(dpNew));
          // 以 P 结尾的数量
          for (int j = 0; j <= 1; j++) {
              for (int k = 0; k <= 2; k++) {
                  dpNew[j][0] = (dpNew[j][0] + dp[j][k]) % MOD;
              }
          }
          // 以 A 结尾的数量
          for (int k = 0; k <= 2; k++) {
              dpNew[1][0] = (dpNew[1][0] + dp[0][k]) % MOD;
          }
          // 以 L 结尾的数量
          for (int j = 0; j <= 1; j++) {
              for (int k = 1; k <= 2; k++) {
                  dpNew[j][k] = (dpNew[j][k] + dp[j][k - 1]) % MOD;
              }
          }
          memcpy(dp, dpNew, sizeof(dp));
      }
      int sum = 0;
      for (int j = 0; j <= 1; j++) {
          for (int k = 0; k <= 2; k++) {
              sum = (sum + dp[j][k]) % MOD;
          }
      }
      return sum;
  }
  ```

## [摆动序列](https://leetcode-cn.com/problems/wiggle-subsequence/)

# 贪心算法

## [跳跃游戏 II](https://leetcode-cn.com/problems/jump-game-ii/)

* 反向查找出发位置

  * 我们的目标是到达数组的最后一个位置，因此我们可以考虑最后一步跳跃前所在的位置，该位置通过跳跃能够到达最后一个位置

  * 如果有多个位置通过跳跃都能够到达最后一个位置，那么我们应该如何进行选择呢？直观上来看，我们可以「贪心」地选择距离最后一个位置最远的那个位置，也就是对应下标最小的那个位置。因此，我们可以从左到右遍历数组，选择第一个满足要求的位置

  * 找到最后一步跳跃前所在的位置之后，我们继续贪心地寻找倒数第二步跳跃前所在的位置，以此类推，直到找到数组的开始位置

    ```c++
    // On2解法
    int jump(vector<int>& nums) {
        int n = nums.size();
        int count = 0;
        int pos = n - 1;
        while(pos > 0) {
            // 从左到右找到第一个能到达pos的位置(离pos最远)
            for(int i = 0; i < pos; ++i) {
                if(i + nums[i] >= pos) {
                    ++count;  
                    pos = i;	// 更新pos到离pos最远的位置
                    break;
                }
            }
        }
        return count;
    }
    ```

* 正向查找可到达的最大位置

  * 如果我们「贪心」地进行正向查找，每次找到可到达的最远位置，就可以在线性时间内得到最少的跳跃次数

  * 每次在上次能跳到的范围（end）内选择一个能跳的最远的位置（也就是能跳到maxFar位置的点）作为下次的起跳点 

    ```c++
    // On解法
    int jump(vector<int> & nums) {
        int maxFar = 0; // 目前能跳到的最远位置
        int step = 0;   // 跳跃次数
        int end = 0;    // 上次跳跃可达范围右边界（下次的最右起跳点）
        for (int i = 0; i < nums.size() - 1; i++) {
            // 统计到达end前，能跳到最远的位置
            max_far = max(maxFar, i + nums[i]);
            // 到达上次跳跃能到达的右边界了
            if (i == end) {
                end = maxFar;  // 目前能跳到的最远位置变成了下次起跳位置的有边界
                step++;         // 进入下一次跳跃
            }
        }
        return step;
    }
    ```

## [得到子序列的最少操作次数](https://leetcode-cn.com/problems/minimum-operations-to-make-a-subsequence/)

* 题目描述

  <img src="imgs/algorithm/problem1713.png" alt="problem1713" style="zoom:80%;" />

* 解题思路

  <img src="imgs/algorithm/solution10.png" alt="solution10" style="zoom:80%;" />

* 代码实现

  ```c++
  int minOperations(vector<int>& target, vector<int>& arr) {
      unordered_map<int, int> rec;
      int nt = target.size();
  
      for(int i = 0; i < nt; ++i) {
          rec[target[i]] = i;
      }
      
      vector<int> vec;
      for(auto &val : arr) {
          if(rec.find(val) != rec.end()) {
              int idx = rec[val];
              auto it = lower_bound(vec.begin(), vec.end(), idx);
              // vec前面出现大于等于idx的，用idx替换，使上升速度最缓
              if(it != vec.end()) {
                  *it = idx;
              }
              else {
                  // idx最大，直接加入
                  vec.push_back(idx);
              }
          }
      }
  
      return nt - vec.size();
  }
  ```

## [ 任务调度器](https://leetcode-cn.com/problems/task-scheduler/)

# 扫描线

## [天际线问题](https://leetcode-cn.com/problems/the-skyline-problem/)

# [匈牙利算法](https://zhuanlan.zhihu.com/p/96229700)

* 匈牙利算法主要用于解决一些与**二分图匹配**有关的问题

* **二分图**（**Bipartite graph**）是一类特殊的**图**，它可以被划分为两个部分，每个部分内的点互不相连。下图是典型的二分图

  <img src="https://gitee.com/canqchen/cloudimg/raw/master/img/binary_graph.png" alt="binary_graph" style="zoom:50%;" />

* 可以看到，在上面的二分图中，每条边的端点都分别处于点集X和Y中。匈牙利算法主要用来解决两个问题：求二分图的**最大匹配数**和**最小点覆盖数**

* **例题**：若两个正整数的和为素数，则这两个正整数称之为“素数伴侣”，如2和5、6和13，它们能应用于通信加密。现在密码学会请你设计一个程序，从已有的N（N为偶数）个正整数中挑选出若干对组成“素数伴侣”，挑选方案多种多样，例如有4个正整数：2，5，6，13，如果将5和6分为一组中只能得到一组“素数伴侣”，而将2和5、6和13编组将得到两组“素数伴侣”，能组成“素数伴侣”最多的方案称为“最佳方案”，当然密码学会希望你寻找出“最佳方案”。

* 素数(质数)：指只能被1和本身整除的数，判断方法：

  ```c++
  bool isPrime(int num){
      if(num<=1)
          return false;
      for(int i = 2; i*i <= num; i++)
          if(num%i==0)
              return false;
      return true;
  }
  ```

* 解题思路：**把输入的数分为奇数和偶数两个集合，因为两个奇数或两个偶数的和为偶数，必能被2整除，不为素数。所以问题转变成，求奇数集合和偶数集合构成二分图，通过判断奇数和偶数两两相加是否为素数构建邻接矩阵，然后利用匈牙利算法求最大匹配数**

* 代码：

  ```c++
  // 寻找oddIdx下标指向的奇数的匹配
  bool match(vector<vector<bool> > & map, vector<int> & record, vector<bool> & visited, int oddIdx){
      for(int i = 0; i<record.size(); i++){
          // odds[oddIdx]和evens[i]之间有路径，并且evens[i]未检查
          if(map[oddIdx][i] && !visited[i]){		
              visited[i] = true;			// 记录evens[i]已检查
              // evens[i]未匹配或者已批配，并递归寻找能否将evens[i]已匹配的奇数，
              if(record[i]==-1|| match(map, record, visited, record[i])){		
                  record[i] = oddIdx;		// 找到另一个偶数匹配，让出evens[i]给odds[oddIdx]
                  return true;												
              }
          }
      }
      return false;
  }
  
  int main(){
      int n;
      while(cin>>n){
          vector<int> odds;
          vector<int> evens;
          int tmp;
          for(int i = 0; i<n;i++){
              cin>>tmp;
              if(tmp&1)
                  odds.push_back(tmp);
              else
                  evens.push_back(tmp);
          }
          vector<vector<bool> > map(odds.size(), vector<bool> (evens.size(), false));	// 邻接矩阵
          for(int i = 0; i < odds.size(); i++){
              for(int j = 0; j<evens.size(); j++){
                  if(isPrime(odds[i] + evens[j]))		// 构建邻接矩阵
                      map[i][j] = true;
              }
          }
          int count = 0;
          vector<int> record(evens.size(), -1);	// 记录evens[i]与odds中哪个数匹配
          for(int i = 0; i < odds.size();i++){
              vector<bool> visited(n, false);		// 记录i下表所指奇数对应偶数匹配情况
              if(match(map, record, visited, i))	// 能找到当前奇数的匹配，count+1
                  count++;
          }
          
          cout<<count<<endl;
      }
      return 0;
  }
  ```

# 前缀树

## [实现 Trie (前缀树)](https://leetcode-cn.com/problems/implement-trie-prefix-tree/)

* 代码实现

  ```c++
  struct TrieTreeNode {
      unordered_map<char, TrieTreeNode *> children;
      string word;
      
      void add(string str) {
          TrieTreeNode * node = this;
          for(auto & ch : str) {
              if(node->children[ch] == nullptr) {
                  node->children[ch] = new TrieTreeNode();
              }
              node = node->children[ch];
          }
          node->word = str;
      }
      
      bool find(string str) {
          TrieTreeNode node = this;
          for(auto & ch : str) {
              if(node->children.count(ch) > 0) {
                  node = children[ch];
              }
              else {
                  return false;
              }
          }
          return node->word == str;
      }
  };
  ```

## [最小高度树](https://leetcode-cn.com/problems/minimum-height-trees/)

* 题目描述

  <img src="imgs/algorithm/problem310.png" alt="problem310" style="zoom:80%;" />

* 解题思路：

  * 暴力枚举每个节点为根的树的高度，取最小值时的根节点
  * 看图（题目介绍里面的图）分析一下，发现，越是靠里面的节点越有可能是最小高度树。**度为1的节点为叶节点**，从边缘开始，先找到所有度为1的节点，然后**把所有度为1的节点进队列**，然后不断地bfs，最后找到的就是两边同时向中间靠近的节点，那么这个中间节点就相当于把整个距离二分了，那么它当然就是到两边距离最小的点，也就是**到其他叶子节点最近的节点**

* 代码实现

  ```c++
  // 暴力法，超时
  vector<int> findMinHeightTrees(int n, vector<vector<int>>& edges) {
      vector<vector<int> > graph(n, vector<int>());
      for(auto &vec : edges) {
          graph[vec[0]].push_back(vec[1]);
          graph[vec[1]].push_back(vec[0]);
      }
      vector<int> ret;
      int minHeight = INT_MAX;
      for(int root = 0; root < n; ++root) {
          queue<int> q;
          q.push(root);
          unordered_set<int> visited;
          int height = 0;
          while(!q.empty()) {
              int size = q.size();
              ++height;
              while(size--) {
                  int cur = q.front();
                  q.pop();
                  visited.insert(cur);
                  for(const int & next : graph[cur]) {
                      if(!visited.count(next)) {
                          q.push(next);
                      }
                  }
              }
          }
          if(height < minHeight) {
              ret.clear();
              ret.push_back(root);
              minHeight = height;
          }
          else if(height == minHeight) {
              ret.push_back(root);
          }
      }
      return ret;
  }
  
  // 拓扑+bfs
  vector<int> findMinHeightTrees(int n, vector<vector<int>>& edges) {
      if(n == 1) {
          return {0};
      }
      vector<vector<int> > graph(n, vector<int>());
      vector<int> degree(n, 0);	// 记录节点的度
      for(auto &vec : edges) {
          graph[vec[0]].push_back(vec[1]);
          graph[vec[1]].push_back(vec[0]);
          ++degree[vec[0]];
          ++degree[vec[1]];
      }
      queue<int> q;
      for(int i = 0; i < n; ++i) {
          if(degree[i] == 1) {
              q.push(i);
          }
      }
      vector<int> ret;  // 记录最后一次遍历的结果，即最内层的节点集合
      while(!q.empty()) {
          ret.clear();
          int size = q.size();
          while(size--) {
              int cur = q.front();
              q.pop();
              ret.push_back(cur); // 记录遍历结果
              for(auto &next : graph[cur]) {
                  --degree[next];	
                  if(degree[next] == 1) {	// 度为1，可以加入队列，进行下一层遍历
                      q.push(next);
                  }
              }
          }
      }
      return ret;
  }
  ```

## [多次搜索](https://leetcode-cn.com/problems/multi-search-lcci/)

* 题目描述

  <img src="imgs/algorithm/problem17-17.png" alt="problem17-17" style="zoom:80%;" />

* 解题思路

  * 对smalls建trie树，其中每个树节点的smId记录对应的smalls数组中的下标，用于查找时，记录对应smalls[smIdx]出现位置
  * 遍历big的所有后缀，并在trie树中查找后缀。对于查找路径上经过的所有有效smId（smId有效值为大于等于0的数），将后缀的起始id加入到smId对应的ret中

* 解题代码

  ```c++
  class Solution {
  public:
      vector<vector<int>> multiSearch(string big, vector<string>& smalls) {
          int n = smalls.size();
          vector<vector<int> > ret(n);
          Trie t;
          for(int i = 0; i < n; ++i) {
              if(!smalls[i].empty()) {
                  t.insert(smalls[i], i);
              }
          }
          for(int i = 0; i < big.size(); ++i) {
              string subWord(std::move(big.substr(i)));
              t.find(subWord, ret, i);
          }
          return ret;
      }
  private:
      struct Trie {
          vector<Trie *> children;
          int smIdx;
          Trie() : children(26, nullptr), smIdx(-1){}
  
          void insert(string & word, int smallIdx) {
              auto cur = this;
              for(auto ch : word) {
                  auto next = ch - 'a';
                  if(cur->children[next] == nullptr) {
                      cur->children[next] = new Trie();
                  }
                  cur = cur->children[next];
              }
              cur->smIdx = smallIdx;
          }
  
          void find(string & word, vector<vector<int> > &ret, int startIdx) {
              auto cur = this;
              for(auto& ch : word) {
                  auto next = ch - 'a';
                  if(cur->children[next] == nullptr) {
                      return;
                  }
                  if(cur->children[next]->smIdx != -1) {
                      ret[cur->children[next]->smIdx].push_back(startIdx);
                  }
                  cur = cur->children[next];
              }
          }
      };
  };
  ```

  

# [并查集](https://zhuanlan.zhihu.com/p/93647900)

## 朴素版本实现

```c++
class UnionFind {
public:
    UnionFind(int n) : parent(std::move(vector<int>(n))) {
        // 将每个节点的父节点设为自身
        for(int i = 0; i < n; ++i) {
            parent[i] = i;
        }
    }
    
    // 递归寻找某个节点的根节点(代表节点)
    int find(int node) {
        if(node < 0 || node > parent.size()) {
            return -1;
        }
        // 父节点等于自身，为根节点
        if(parent[node] == node) {
            return node;
        }
        // 往上寻找根节点
        return find(parent[node]);
    }
    
    // 合并两个节点，找到他们各自的根节点，然后令一个根节点为另一个根节点的父节点
    void merge(int node1, int node2) {
        int root1 = find(node1);
        int root2 = find(node2);
        // 令root1为root2的父节点
        parent[root2] = root1;
    }
private:
    vector<int> parent;
};
```

* 存在问题：可能会形成一条长长的**链**，随着链越来越长，想要从底部找到根节点会变得越来越难

## 路径压缩实现

* 既然我们只关心一个元素对应的**根节点**，那我们希望**每个元素到根节点的路径尽可能短，最好只需要一步**，形成以下效果

  <img src="imgs/algorithm/unionfind1.jpg" alt="unionfind1" style="zoom:50%;" />

* 只要我们在查询的过程中，**把沿途的每个节点的父节点都设为根节点**即可

  ```c++
  int find(int node) {
  	if(node == parent[node]) {
  		return node;
  	}
      parent[node] = find(parent[node]); // 将当前节点的父节点设为根节点
      return parent[node];	// 返回父节点(根节点)
  }
  ```

* 存在问题：有些人可能有一个误解，以为路径压缩优化后，并查集始终都是一个**菊花图**（只有两层的树的俗称）。但其实，由于路径压缩只在查询时进行，也**只压缩一条路径**，所以并查集最终的结构仍然可能是比较复杂的

## 按秩合并实现

* 增加一个统计节点秩的数组rank[]记录**每个根节点对应的树的深度**（如果不是根节点，其rank相当于以它作为根节点的**子树**的深度）

* 一开始，把所有元素的rank（**秩**）设为1。合并时比较两个根节点，**把rank较小者往较大者上合并**。**如果两个根节点的深度相同，新的根节点深度+1**

* 路径压缩和按秩合并如果一起使用，时间复杂度接近$\textbf{O}(n)$，但可**能会破坏rank的准确性**

  ```c++
  class UnionFind {
  public:
      UnionFind(int n) : parent(std::move(vector<int>(n))),
      rank(std::move(vector<int>(n))) {
          // 将每个节点的父节点设为自身
          for(int i = 0; i < n; ++i) {
              parent[i] = i;
              rank[i] = 1;
          }
      }
      
      // 递归寻找某个节点的根节点(代表节点)
      int find(int node) {
          // 父节点等于自身，为根节点
          if(parent[node] == node) {
              return node;
          }
          // 往上寻找根节点
          return find(parent[node]);
      }
      
      // 合并两个节点，找到他们各自的根节点，然后令rank大的根节点成为rank小的根节点的父节点
      // 如果两个根节点的rank相同，新的根节点rank+1
      void merge(int node1, int node2) {
          int root1 = find(node1);
          int root2 = find(node2);
          if(root1 != root2){
              if(rank[root1] < rank[root2]) {
                  swap(root1, root2) // rank大的根节点成为rank小的根节点的父节点
              }
  			parent[root1] = root2;
              if(rank[root1] == rank[root2]) {
                  ++rank[root1];	// root1的rank+1
              }
          }
      }
  private:
      vector<int> parent;
      vector<int> rank;
  };
  ```

## [账户合并](https://leetcode-cn.com/problems/accounts-merge/)
```c++
/*
    思路：
    根据题意可知：
    	1. 存在相同邮箱的账号一定属于同一个人
    	2. 名称相同的账户不一定属于同一个人

    由于名称相同无法判断为同1人，所以只能使用邮箱是否相同来判断是否为同一人。
    这样建立并查集就比较简单了：
    	1. 先初始化每个账户为1个连通分量
    	2. 遍历每个账户下的邮箱，判断该邮箱是否在其他账户下出现
    	3. 如果未出现，继续
    	4. 如果账户A、B下出现了相同的邮箱email，那么将账户A和账户B两个连通分量进行合并
    	5. 最后遍历并查集中每个连通分量，将所有连通分量内部账户的邮箱全部合并(相同的去重，不同的合并)
    	6. 结束
    */
class UnionFind {
    public:
    vector<int> parent;  // 记录节点的根
    vector<int> rank;  // 记录根节点的深度（用于优化）
    UnionFind(int n): parent(vector<int>(n)), rank(vector<int>(n)) {
        for (int i = 0; i < n; i++) {
            parent[i] = i;
        }
    }

    int find(int x) {
        // 压缩方式：直接指向根节点
        if (x != parent[x]) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }
    // 按秩合并
    void merge(int x, int y) {
        int rootx = find(x);
        int rooty = find(y);
        if (rootx != rooty) {
            if (rank[rootx] < rank[rooty]) {
                swap(rootx, rooty);
            }
            parent[rooty] = rootx;
            if (rank[rootx] == rank[rooty]) 
                rank[rootx] += 1;
        }
    }
};

class Solution {
    public:
    vector<vector<string>> accountsMerge(vector<vector<string>>& accounts) {
        vector<vector<string> > res;

        // 作用：存储每个邮箱属于哪个账户，同时在遍历邮箱时，判断邮箱是否出现过
        // 格式：<邮箱，账户下标>
        unordered_map<string, int> mailToIdx;
        int n = accounts.size();

        UnionFind uf(n);
        for (int i = 0; i < n; i++) {
            int m = accounts[i].size();
            for (int j = 1; j < m; j++) {
                string mail = accounts[i][j];
                if (mailToIdx.find(mail) == mailToIdx.end()) {
                    mailToIdx[mail] = i;	// 记录邮箱首次出现的idx
                } else {
                    uf.merge(i, mailToIdx[mail]);	// 邮箱之前出现过，合并连通分量
                }
            }
        }

        // 作用：存储每个账户下的邮箱
        // 格式：<账户id, 邮箱列表>
        // 注意：这里的key必须是账户id，不能是账户名称，名称可能相同，会造成覆盖
        unordered_map<int, vector<string> > idxToMails;
        // 遍历所有连通分量，将同一连通分量(同一账户)内的邮箱合并
        for (auto& [mail, idx] : mailToIdx) {
            idxToMails[uf.find(idx)].emplace_back(mail);
        }
        // 将合并的邮箱排序，加上名称
        for (auto& [idx, mailVec] : idxToMails){
            sort(mailVec.begin(), mailVec.end());
            vector<string> tmp(1, accounts[idx][0]);
            tmp.insert(tmp.end(), mailVec.begin(), mailVec.end());
            res.emplace_back(tmp);
        } 
        return res;
    }
};
```

  

# 洗牌算法

## [打乱数组](https://leetcode-cn.com/problems/shuffle-an-array/)

```c++
class Solution {
public:
    Solution(vector<int>& nums) {
        origin = nums;
    }
    
    /** Resets the array to its original configuration and return it. */
    vector<int> reset() {
        return origin;
    }
    
    /** Returns a random shuffling of the array. */
    vector<int> shuffle() {
        int n = origin.size();
        vector<int> ret(origin);
        for(int i = 0; i < n; i++){
            int randNum = rand() % (n - i) + i; // 生成 [i, n) 之间的随机数
            swap(ret[i], ret[randNum]);
        }
        return ret;
    }
private:
    vector<int> origin;
};
```

# 堆/哈希

## [前 K 个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements/)

# 二叉树

## Morris遍历

* 由于在遍历的时候，我们需要记住某种遍历次序的的后驱或者前驱节点，常见的递归和非递归都是采用栈的方式完成这个过程，需要O(n)的空间复杂度
* 有没有内部空间来记录这些后驱或者前驱节点，使额外空间复杂度为O(1)呢？有，那就是叶节点的左，右孩子节点，因为叶节点的两个孩子节点都是空指针，如果利用好这些空间，我们就可以在O(1) 的空间完成遍历
* 利用叶节点的左、右孩子指向遍历的前驱或者后驱结点，这些指针叫做线索，对应的二叉树叫做线索二叉树。

### Morris先序遍历

* Morris 前序遍历的流程如下：

  1. 当前结点的左孩子是否为空，若**是则输出当前结点**，**并更新当前结点为当前结点的右孩子**；否则进入2

  2. **在当前结点的左子树中寻找中序遍历下的前驱结点**（**左子树中最右结点**）

     a. **若前驱结点的右孩子为空，则将前驱结点的右孩子指向当前结点，输出当前结点**（在这里输出，和中序遍历不同的地方），当前结点更新为当前结点的左孩子，进入3

     b. **若前驱结点的右孩子为当前结点**（不为空），将前驱结点的右孩子置NULL，当前结点更新为当前结点的右孩子，进入3

  3. 若当前结点不为空，进入1；否则程序结束

<img src="imgs/algorithm/morris_pre.png" alt="morris_pre" style="zoom:80%;" />

* 代码实现

  ```c++
  ![morris_in](imgs/algorithm/morris_in.png)void morrisPre(TreeNode * root){
      TreeNode * cur = root;
      TreeNode * rightMost = nullptr;
      vector<int> vec;
      while(cur != nullptr) {
          // cur的左子节点为空，到达最左边，直接输出，并开始遍历其右子树
          if(cur->left == nullptr) {
              vec.push_back(cur->val);
              cur = cur->right;
          }
          else {	// 找到cur节点在中序遍历中的前驱节点，也就是左子树最右边的节点
              rightMost = cur->left;
              while(rightMost->right != nullptr && rightMost->right != cur) {
                  rightMost = rightMost->right;
              }
              // 前驱节点的右子节点为空，表示cur节点被首次遍历，输出cur
              if(rightMost->right == nullptr) {
                  vec.push_back(cur->val);
                  rightMost->right = cur;
                  cur = cur->left;
              }
              else {
                  // 前驱节点的右子节点不为空，cur节点已经被遍历，
                  // 还原前驱节点为nullptr，然后去遍历cur的右子树
                  rightMost->right = nullptr;
                  cur = cur->right;
              }
          }
      }
  }
  ```

### Morris中序遍历

* Morris 中序遍历过程如下：

  1. 当前结点的左孩子是否为空，若是则输出当前结点，更当前结点为当前结点的右孩子；否则进入2

  2. 在当前结点的左子树中寻找中序遍历下的前驱结点（左子树中最右结点）

     a. 若前驱结点的右孩子为空，则将前驱结点的右孩子指向当前结点，当前结点更新为当前结点的左孩子；进入3

     b. 若前驱结点的右孩子为当前结点（不为空），将前驱结点的右孩子置NULL，输出当前结点，当前结点更新为当前结点的右孩子，进入3

  3. 若当前结点不为空，进入1；否则程序结束

  <img src="imgs/algorithm/morris_in.png" alt="morris_in" style="zoom:80%;" />

* 代码实现

  ```c++
  void morrisIn(TreeNode * root, vector<vector<int> > & ret) {
      TreeNode * cur = root;
      TreeNode * rightMost = nullptr;
      vector<int> vec;
      while(cur != nullptr){
          if(cur->left == nullptr) {
              vec.push_back(cur->val);
              cur = cur->right;
          }
          else{
              rightMost = cur->left;
              while(rightMost->right != nullptr && rightMost->right != cur) {
                  rightMost = rightMost->right;
              }
              if(rightMost->right == nullptr) {
                  rightMost->right = cur;
                  cur = cur->left;
              }
              else { 
                  // 中序遍历与先序遍历不同点，在返回时输出cur节点
                  vec.push_back(cur->val);
                  rightMost->right = nullptr;
                  cur = cur->right;
              }
          }
      }
  }
  ```

### Morris后序遍历

* 后序遍历规则：

  1. 第二次访问到某一节点时，逆序打印其左子树的右边界。最后单独逆序打印整棵树的右边界

  <img src="imgs/algorithm/morris_post.png" alt="morris_post" style="zoom:80%;" />

* 代码实现

  ```c++
  void morrisPost(TreeNode * root, vector<vector<int> >& ret){
      TreeNode * cur = root;
      TreeNode * rightMost = nullptr;
      vector<int> vec;
      while(cur != nullptr) {
          if(cur->left == nullptr) {
              cur = cur->right;
          }
          else{
              rightMost = cur->left;
              while(rightMost->right != nullptr && rightMost->right != cur){
                  rightMost = rightMost->right;
              }
              // 第一次访问cur节点
              if(rightMost->right == nullptr) {
                  rightMost->right = cur;
                  cur = cur->left;
              }
              else {
                  // 第二次访问cur节点，逆序打印cur的左子树的右边界
                  rightMost->right = nullptr;
                  reverseOutput(cur->left, vec);
                  cur = cur->right;
              }
          }
      }
      // 最后逆序打印整棵树的右边界
      reverseOutput(root, vec);
  }
  
  void reverseOutput(TreeNode* node, vector<int>&vec){
      stack<int>stk;
      while(node != nullptr){
          stk.push(node->val);
          node = node->right;
      }
      while(!stk.empty()){
          vec.push_back(stk.top());
          stk.pop();
      }
  }
  ```

  

## [路径总和 III](https://leetcode-cn.com/problems/path-sum-iii/)

* 代码实现：利用树的前缀和，巧妙

  ```c++
  int pathSum(TreeNode * root, int targetSum) {
      if(root == nullptr)
          return 0;
      unordered_map<int, int> map;
      // 关键，前缀和-target=0，节点本身为一条路径
      map[0] = 1;
      return calSum(root, map, 0, targetSum);
  }
  
  int calSum(TreeNode *root, unordered_map<int, int> &map, int curSum, int target) {
      if(root == nullptr)
          return 0;
  
      curSum += root->val;
      int ret = 0;
      // 统计前面前缀和为curSum - target有多少条路径
      if(map.count(curSum - target))
          ret = map[curSum - target];
      ++map[curSum];	// 当前节点为末端的前缀和
      // 左右子树前缀和
      int left = calSum(root->left, map, curSum, target);
      int right = calSum(root->right, map, curSum, target);
      // 关键！统计完，返回时需要消除当前节点前缀和，否则会重复。因为题目要求是从上往下
      --map[curSum];
      return ret + left + right;
  }
  ```

## [二叉树中的最大路径和](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/)

* 代码实现

  ```c++
  int maxPathSum(TreeNode* root) {
      if(root == nullptr)
          return 0;
      int maxVal = INT_MIN;
      calMax(root, maxVal);
      return maxVal;
  }
  
  int calMax(TreeNode * node, int& maxVal) {
      if(node == nullptr) {
          return 0;
      }
      // 返回左右子树最大路径
      int leftMax = calMax(node->left, maxVal);
      int rightMax = calMax(node->right, maxVal);
      // 记录当前节点最大，可能左+根，右+根，左+右+根，根
      int curMax = node->val + max(leftMax, 0) + max(rightMax, 0);
      // 维护全局最大
      maxVal = max(maxVal, curMax);
      // 返回当前层最大路径，可能左+根，右+根，根。因为路径定义，不存在左+右+根
      return max(node->val, max(leftMax, rightMax) + node->val);
  }
  ```

## [检查平衡性](https://leetcode-cn.com/problems/check-balance-lcci/)(自底向上递归树)

* 代码实现

  ```c++
  bool isBalanced(TreeNode * root) {
      return height(root) >= 0;
  }
  
  // 小于0表示不平衡
  int height(TreeNode * node) {
      if(node == nullptr) {
          return 0;
      }
      int leftHeight = height(node->left);	
      int rightHeight = height(node->right);
      
      if(leftHeight < 0 || rightHeight < 0 || abs(leftHeight-rightHeight) > 1) {
          return -1;  // 向上报告不平衡，退出递归
      }
      // 平衡
      return max(leftHeight, rightHeight) + 1;
  }
  ```

## [二叉树的右视图](https://leetcode-cn.com/problems/binary-tree-right-side-view/)

* 深度优先遍历，先遍历右子树，每个深度遍历的第一个节点即是这一深度右视图能看到的节点，可以存储在每个深度访问的第一个结点，一旦我们知道了树的层数，就可以得到最终的结果数组

  ```c++
  // 迭代版本
  class Solution {
  public:
      vector<int> rightSideView(TreeNode* root) {
          unordered_map<int, int> rightmostValueAtDepth;
          int max_depth = -1;
  
          stack<TreeNode*> nodeStack;
          stack<int> depthStack;
          nodeStack.push(root);
          depthStack.push(0);
  
          while (!nodeStack.empty()) {
              TreeNode* node = nodeStack.top();
              nodeStack.pop();
              int depth = depthStack.top();
              depthStack.pop();
              
              if (node != NULL) {
              	// 维护二叉树的最大深度
                  max_depth = max(max_depth, depth);
                  
                  // 如果不存在对应深度的节点我们才插入
                  if (rightmostValueAtDepth.find(depth) == rightmostValueAtDepth.end()) {
                      rightmostValueAtDepth[depth] =  node -> val;
                  }
                  
                  nodeStack.push(node -> left);
                  nodeStack.push(node -> right);
                  depthStack.push(depth + 1);
                  depthStack.push(depth + 1);
              }
          }
  
          vector<int> rightView;
          for (int depth = 0; depth <= max_depth; ++depth) {
              rightView.push_back(rightmostValueAtDepth[depth]);
          }
  
          return rightView;
      }
  };
  
  // 递归版本
  class Solution {
  public:
      vector<int> rightSideView(TreeNode* root) {
  		if(root == nullptr) {
              return {};
          }
          vector<int> rightView;
          dfs(root, 0, rightView);
          return rightView;
      }
      // 根-右-左
      void dfs(TreeNode * node, int depth, vector<int> &rightView) {
          if(node == nullptr) {
              return;
          }
          // 该深度第一个节点
          if(depth == rightView.size()) {
              rightView.push_back(node->val);
          }
          ++depth;
          dfs(node->right, depth, rightView);
          dfs(node->left, depth, rightView);
      }
  };
  ```

* 广度优先遍历，可以对二叉树进行层次遍历，那么对于每层来说，最右边的结点一定是最后被遍历到的。二叉树的层次遍历可以用广度优先搜索实现

  ```c++
  class Solution {
  public:
      vector<int> rightSideView(TreeNode* root) {
          unordered_map<int, int> rightmostValueAtDepth;
          int max_depth = -1;
  
          queue<TreeNode*> nodeQueue;
          queue<int> depthQueue;
          nodeQueue.push(root);
          depthQueue.push(0);
  
          while (!nodeQueue.empty()) {
              TreeNode* node = nodeQueue.front();
              nodeQueue.pop();
              int depth = depthQueue.front();
              depthQueue.pop();
  
              if (node != NULL) {
              	// 维护二叉树的最大深度
                  max_depth = max(max_depth, depth);
  
                  // 由于每一层最后一个访问到的节点才是我们要的答案，因此不断更新对应深度的信息即可
                  rightmostValueAtDepth[depth] =  node -> val;
  
                  nodeQueue.push(node -> left);
                  nodeQueue.push(node -> right);
                  depthQueue.push(depth + 1);
                  depthQueue.push(depth + 1);
              }
          }
  
          vector<int> rightView;
          for (int depth = 0; depth <= max_depth; ++depth) {
              rightView.push_back(rightmostValueAtDepth[depth]);
          }
  
          return rightView;
      }
  };
  ```

##  判断一棵二叉树是否为二叉搜索树和完全二叉树

* 代码实现

  ```c++
  vector<bool> judgeIt(TreeNode* root) {
      vector<bool> ret(2, false);
      if(isBST(root, INT_MIN, INT_MAX))
          ret[0] = true;
      if(isCompleted(root))
          ret[1] = true;
      return ret;
  }
  
  bool isBST(TreeNode * node, int minVal, int maxVal){
      if(node == nullptr)
          return true;
      // 当前节点值违反上层最大最小约束，则不平衡
      if(node->val >= maxVal || node->val <= minVal)
          return false;
      // 左子树所有节点都小于当前根节点(node)，所以最大约束为node->val
      // 同理，右子树所有节点都大于当前根节点(node)，所以最小约束为node->val
      return isBST(node->left, minVal, node->val) && isBST(node->right, node->val, maxVal);
  }
  
  bool isCompleted(TreeNode * node){
      if(node == nullptr)
          return true;
      // 左子树非空，右子树为空，需要递归判断左子树
      if(node->left != nullptr && node->right == nullptr)
          return isCompleted(node->left);
      // 左子树为空，右子树非空，必非完全二叉树
      if(node->left == nullptr && node->right != nullptr)
          return false;
      // 左右非空，递归判断
      return isCompleted(node->left) && isCompleted(node->right);
  }
  
  // 完全二叉树其他判断思路：
  // 	  1. 层次遍历，遇到空节点后，后面出现非空节点则为不完全树
  //    2. dfs或者bfs遍历的同时，给节点编号，完全树i节点的子节点为(2*i + 1)和(2*i + 2)，
  //       最后判断最后一个编号是否等于节点总数可判断
  
  // #1
  class Solution {
  public:
      bool isCompleteTree(TreeNode* root) {
          queue<TreeNode*>q;
          q.push(root);
          bool flag=false;
          while(!q.empty()) {
              auto node =q.front();
              q.pop();
              if(node==nullptr) {
                  flag=true;
                  continue;
              }
              // 非空节点且之前出现空节点
              if(flag) return false;
              q.push(node->left);
              q.push(node->right);
          }
          return true;
      }
  };
  
  // #2
  class Solution {
      int size = 0;
      int maxCode = 0;
      public boolean isCompleteTree(TreeNode root) {
          if(root == null){
              return true;
          }
          recursive(root, 1);
          return size == maxCode;
      }
  
      public void recursive(TreeNode root, int index){
          if(root == null){
              return;
          }
          ++size;
          maxCode = Math.max(maxCode, index);
          recursive(root.left,index * 2);
          recursive(root.right,index * 2 + 1);
      }
  }
  ```

## 二叉搜索树与双向链表转换

* 代码实现

  ```c++
  class BSTToSortedList {
  public:
      // 递归版本
      TreeNode * Convert(TreeNode * pRootOfTree)
      {
          if(pRootOfTree==nullptr)
              return nullptr;
        
          Convert(pRootOfTree->left);
          
          if(first==nullptr) {
              first = pRootOfTree;
          }
          
          if(last!=nullptr) {
              last->right = pRootOfTree;
              pRootOfTree->left = last;
          }
          last = pRootOfTree;
          Convert(pRootOfTree->right);
          return first;
      }
      // 迭代版本
      TreeNode* Convert(TreeNode* pRootOfTree)
      {
          if(pRootOfTree==nullptr)
              return nullptr;
          stack<TreeNode*> stk;
          TreeNode * last = nullptr, *first=nullptr, * cur = pRootOfTree;
          while(cur! = nullptr || !stk.empty()) {
              while(cur != nullptr) {
                  stk.push(cur);
                  cur = cur->left;
              }
               
              if(last != nullptr) {
                  stk.top()->left = last;
                  last->right = stk.top();
              }
              
              if(first==nullptr) {
                  first = stk.top();
              }
  
              last = stk.top();
              cur = stk.top()->right;
              stk.pop();
          }
          return first;
      }
  private:
      TreeNode * first = nullptr, * last = nullptr;
  };
  ```

  ```c++
  class sortedListToBST {
  public:
      TreeNode* convert(ListNode *head) {
          return BST(head, nullptr);
      }
      
      TreeNode *BST(ListNode *head, ListNode *tail)
      {
          if(head == tail)
              return NULL;
          ListNode *slow = head;
          ListNode *fast = head;
          // 找到中点
          while(fast != tail && fast->next != tail) {
              slow = slow->next;
              fast = fast->next->next;
          }
          TreeNode *root = new TreeNode(slow->val);
          root->left = BST(head, slow);
          root->right = BST(slow->next, tail);
          
          return root;
      }
  };
  ```
  
  

## 二叉树的下一个结点

* 题目描述

  * 给定一个二叉树其中的一个结点，请找出中序遍历顺序的下一个结点并且返回。注意，树中的结点不仅包含左右子结点，同时包含**指向父结点的next指针**

* 解题思路

  <img src="imgs/algorithm/solution9.png" alt="solution9" style="zoom:67%;" />

  结合图，我们可发现分成两大类：

   	1. **有右子树的，那么下个结点就是右子树最左边的点**；（eg：D，B，E，A，C，G） 
   	2. **没有右子树的，也可以分成两类**
   	  	1. 是父节点左孩子（eg：N，I，L） ，那么父节点就是下一个节点 
   	  	2. 是父节点的右孩子（eg：H，J，K，M）找他的父节点的父节点的父节点...**直到当前结点是其父节点的左孩子位置，则返回其父节点**。如果没有eg：M，那么他就是尾节点

* 代码实现

  ```c++
  TreeLinkNode* GetNext(TreeLinkNode* pNode)
  {
      if(pNode == nullptr)
          return nullptr;
      TreeLinkNode*ret = nullptr;
      // 第一类，有右孩子
      if(pNode->right != nullptr) {
          // 找右子树最左边的叶子节点
          ret = pNode->right;
          while(ret->left != nullptr)
              ret = ret->left;
          return ret;
      }
      // 第二类，没有右孩子，往上找父节点判定
      // 
      while(pNode->next != nullptr){
          TreeLinkNode *root = pNode->next;
          // 当前节点是父节点的左子节点，返回
          if(root->left == pNode) {
              return root;
          }
          // 继续向上找父节点的父节点...
          pNode = pNode->next;
      }
      // 尾节点
      return nullptr;
  }
  ```

## [ 二叉树中所有距离为 K 的结点](https://leetcode-cn.com/problems/all-nodes-distance-k-in-binary-tree/)

* 题目描述

  <img src="imgs/algorithm/problem863.png" alt="problem863" style="zoom: 80%;" />

* 解题思路

  <img src="imgs/algorithm/solution863.png" alt="solution863" style="zoom:80%;" />
  * 记录父节点的过程，相当于记录建图，然后以target节点为起始节点，遍历全图，找到距离target为k的节点
  * 遍历的时候，记录来源节点，防止遍历父节点时，回到子节点，造成结果重复

* 代码实现

  ```c++
  using TreeNodePtr = TreeNode*;
  vector<int> distanceK(TreeNodePtr root, TreeNodePtr target, int k) {
      int no = -1;
      unordered_map<TreeNodePtr, TreeNodePtr> parent;
      vector<int> ret;
      findParent(root, parent);
      findAns(target, nullptr, 0,ret, parent, k);
      return ret;
  }
  
  void findParent(TreeNodePtr node, unordered_map<TreeNodePtr, TreeNodePtr> &parent) {
      if(node->left != nullptr) {
          parent[node->left] = node;
          findParent(node->left, parent);
      }
      if(node->right != nullptr) {
          parent[node->right] = node;
          findParent(node->right, parent);
      }
  }
  // from记录从哪个节点来，防止回头，因为parent节点的左右节点是没有变化的，在遍历左右节点处，可能会造成重复
  void findAns(TreeNodePtr node, TreeNodePtr from, int no, vector<int> &ret, 
               unordered_map<TreeNodePtr, TreeNodePtr> &parent, int k) {
      if(node == nullptr) {
          return;
      }
      if(no == k) {
          ret.push_back(node->val);
          return;        
      }
      if(node->left != from) {
          findAns(node->left, node, no + 1, ret, parent, k);
      }
      if(node->right != from) {
          findAns(node->right, node, no + 1, ret, parent, k);
      }
      if(parent[node] != from) {
          findAns(parent[node], node, no + 1, ret, parent, k);
      }
  }
  
  // 纯dfs递归实现
  class Solution {
  private:
    vector<int> ans;
  public:
    vector<int> distanceK(TreeNode* root, TreeNode* target, int k) {
      dfs(root, target, k);
      return ans;
    }
  
    int dfs(TreeNode* root, TreeNode* target, int k) {
      if (!root) 
          return -1;
      if (root == target) {
      	find(root, k, true, true); // target为根向下搜索距离为k
          return 0;
      }
      int leftDis = dfs(root->left, target, k);
      int rightDis = dfs(root->right, target, k);
      if (leftDis != -1) {
          // 回溯后target在左子树，向当前根和其右子树搜索，距离为k - leftDis - 1
          find(root, k - leftDis - 1, false, true);
          return leftDis + 1;
      }
      if (rightDis != -1) {
          // 回溯后target在右子树，向当前根和其左子树搜索，距离为k - rightDis - 1
          find(root, k - rightDis - 1, true, false);
          return rightDis + 1;
      }
      return -1;
    }
  
    void find(TreeNode* root, int dis, bool searchLeft, bool searchRight) {
      if (!root || dis < 0) return;
      if (dis == 0) {
        ans.push_back(root->val);
        return;
      }
      if (searchLeft) find(root->left, dis - 1, true, true);
      if (searchRight) find(root->right, dis - 1, true, true);
    }
  };
  ```

## [不同的二叉搜索树 II](https://leetcode-cn.com/problems/unique-binary-search-trees-ii/)

* 题目描述

  <img src="imgs/algorithm/problem95.png" alt="problem95" style="zoom:80%;" />

* 解题思路

  <img src="imgs/algorithm/solution95.png" alt="solution95" style="zoom:80%;" />

* 解题代码

  ```c++
  vector<TreeNode*> generateTrees(int n) {
      if(n < 1) {
          return {nullptr};
      }
      function<vector<TreeNode*>(int,int)> buildTree = [&](int start, int end)->vector<TreeNode*> {
          if(start > end) {
              return {nullptr};
          }
          vector<TreeNode*> tmpRet;
          for(int i = start; i <= end; ++i) {
              vector<TreeNode*> leftTrees = buildTree(start, i - 1);
              vector<TreeNode*> rightTrees = buildTree(i + 1, end);
              // 从左右子树中，取左右子树与i组合成子树，返回上层
              for(auto leftTree: leftTrees) {
                  for(auto rightTree : rightTrees) {
                      TreeNode * root = new TreeNode(i);
                      root->left = leftTree;
                      root->right = rightTree;
                      tmpRet.push_back(root);
                  }
              }
          }
          return tmpRet;
      };
  
      return buildTree(1, n);
  }
  ```

## [恢复二叉搜索树](https://leetcode-cn.com/problems/recover-binary-search-tree/)

* 题目描述

  <img src="imgs/algorithm/problem99.png" alt="problem99" style="zoom:80%;" />

* 解题思路

  <img src="imgs/algorithm/solution99_1.png" alt="solution99_1" style="zoom:80%;" />

  <img src="imgs/algorithm/solution99_2.png" alt="solution99_2" style="zoom:80%;" />

  <img src="imgs/algorithm/solution99_3.png" alt="solution99_3" style="zoom:80%;" />

* 解题代码

  ```c++
  // 空间复杂度O(n)解法
  void recoverTree(TreeNode* root) {
      TreeNode *x = nullptr, *y = nullptr, *prev = nullptr;
      stack<TreeNode*> stk;
      while(!stk.empty() || root != nullptr) {
          while(root != nullptr) {
              stk.push(root);
              root = root->left;
          }
          root = stk.top();
          stk.pop();
          if(prev != nullptr && prev->val > root->val) {
              y = root;
              if(x == nullptr) {
                  x = prev;
              }
          }
          prev = root;
          root = root->right;
      }
      swap(x->val, y->val);
  }
  
  // 空间复杂度O(1)解法：morris中序遍历
  void recoverTree(TreeNode* root) {
      TreeNode *x = nullptr, *y = nullptr, *prev = nullptr;
      TreeNode * cur = root;
      while(cur != nullptr) {
          // 没有左子节点，直接访问
          if(cur->left == nullptr) {
              if(prev != nullptr && prev->val > cur->val) {
                  y = cur;
                  if(x == nullptr) {
                      x = prev;
                  }
              }
              prev = cur;
              cur = cur->right;
          }
          else {
              TreeNode * rightMost = cur->left;
              while(rightMost->right != nullptr && rightMost->right != cur) {
                  rightMost = rightMost->right;
              }
              // 利用叶子节点空子节点
              if(rightMost->right == nullptr) {
                  rightMost->right = cur;     
                  cur = cur->left;
              }
              // rightMost->right = cur，第二次访问到cur节点，左子树已经访问完
              else {
                  if(prev != nullptr && prev->val > cur->val) {
                      y = cur;
                      if(x == nullptr) {
                          x = prev;
                      }
                  }
                  rightMost->right = nullptr; // 复原树结构
                  prev = cur;
                  cur = cur->right;
              }
          }
  
      }
      swap(x->val, y->val);
  }
  ```

## 最近公共祖先

* 解题代码

  ```c++
  TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
      if(root == nullptr)
          return nullptr;
  	// 提前终止搜索，如果都在同一边，返回root
      if(root == p || root == q)
          return root;
  
      TreeNode * left = lowestCommonAncestor(root->left, p, q);
      TreeNode * right = lowestCommonAncestor(root->right, p, q);
  
      if(left != nullptr && right != nullptr)
          return root;
      
      return left == nullptr ? right : left;
  }
  
  // bst的lca
  TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
      TreeNode* ancestor = root;
      while (true) {
          if (p->val < ancestor->val && q->val < ancestor->val) {
              ancestor = ancestor->left;
          }
          else if (p->val > ancestor->val && q->val > ancestor->val) {
              ancestor = ancestor->right;
          }
          else {
              break;
          }
      }
      return ancestor;
  }
  ```

## [填充每个节点的下一个右侧节点指针I&II](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node/)

```c++
// I: 完全二叉树
Node* connect(Node* root) {
    if(root == nullptr) {
        return root;
    }
    Node * leftMost = root;
    while(leftMost->left != nullptr) {
        Node*head = leftMost;
        while(head != nullptr) {
            if(head->right!=nullptr) {
                head->left->next = head->right;
            }
            if(head->next != nullptr) {
                head->right->next = head->next->left;
            }
            head = head->next;
        }
        leftMost = leftMost->left;
    }
    return root;
}
// I递归解法
void connect(Node*left, Node*right) {
    if(left == nullptr) {
        return;
    }
    left->next = right;
    connect(left->left, left->right);
    if(right != nullptr) {
        connect(right->left, right->right);
        connect(left->right, right->left);
    }
}

// II：非完全二叉树，思路1：层次遍历
// 思路2：很妙
Node* connect(Node* root) {
    if(root == nullptr) {
        return nullptr;
    }
    Node * cur = root;
    while(cur != nullptr) {
        Node dum;
        Node * prev = &dum;
        Node * node = cur;
        while(node != nullptr) {
            if(node->left != nullptr) {
                prev->next = node->left;
                prev = prev->next;
            }
            if(node->right != nullptr) {
                prev->next = node->right;
                prev = prev->next;
            }
            node = node->next;
        }
        cur = dum.next;
    }
    return root;
}
```

