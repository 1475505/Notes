# Hot100

## 1. 两数之和@HashMap

使用`Unordered_map`实现哈希表。

```c++
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> hashMap;
        int len = nums.size();
        for (int i = 0; i < len; ++i){
            auto it = hashMap.find(target - nums[i]);//find的是键
            if (it != hashMap.end())
                return vector<int>{i,it->second};
            hashMap.insert(pair<int, int>{nums[i], i});//插入pair类型
        }
        return vector<int>{-1, -1};
    }
```

使用`map`也可以实现哈希表。

## 2. 两数相加

```c++
ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode* head = new ListNode();//建议这样创建结构体
        ListNode* tmp = head;
/* 省去新空间需要：ListNode* res = l1; ListNode* pre = l2（异常初始化问题）;*/
        bool bias = 0;
        while ( l1 || l2 || bias ){/*不建新链表：while ( l2 || bias ) */
            int val1 = 0, val2 = 0;
            if (l1) { val1 = l1->val; l1 = l1->next;}
    /* if (!l1->next) { pre = l1;   l1->next = new ListNode(0);} */
            if (l2) { val2 = l2->val; l2 = l2->next;}
            tmp->next = new ListNode( (val1 + val2 + bias) % 10 );
            if (val1 + val2 + bias > 9)    bias = 1; //注意处理顺序
            else bias = 0;
            tmp = tmp->next;
        }
    //if (l1->val == 0)   pre->next = nullptr;
        return head->next;
    }
```

## 3.无重复字符的最长子串

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int len = s.size();
        if (len == 0)   return 0;
        int i = 0, j = 0, ans = 0;
        unordered_map<char, int> hshmp;
        while( j < len){
            if ( hshmp.find(s[j]) == hshmp.end() ){
                hshmp.insert(pair<char, int>{s[j], j});
            } else {
                ans = max(ans, j - i);
                i = max(i, hshmp[s[j]] + 1);
                hshmp[s[j]] = j;
            }
            cout << i << j << endl;
            ++j;
        }
        return max(ans, j - i);
    }
};
```

## 归并向量的中位数

```c++
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        return medium(nums1, nums2, 0, 0, nums1.size() - 1, nums2.size() - 1);
    }

    double medium(vector<int>& nums1, vector<int>& nums2, int l1, int l2, int r1, int r2){
        // cout << l1 << r1 << l2 << r2 << endl;
        int len1 = r1 - l1 + 1, len2 = r2 - l2 + 1;
        if (len1 > len2) return medium(nums2, nums1, l2, l1, r2, r1);
        if (len1 == 0 || len2 < 5){
            return trivalmedium(nums1, nums2, l1, l2, r1, r2);
        }
        if (2 * len1 < len2)
            return medium(nums1, nums2, l1, l2 + (len2 - len1 - 1)/2, r1, r2 - (len2 - len1 - 1)/2);
        int mid1 = l1 + len1/2, mid2a = l2 + (len1 - 1)/2, mid2b = r2 - len1/2;
        if (nums1[mid1] > nums2[mid2b])
            return medium(nums1, nums2, l1, mid2a, mid1, r2);
        else if (nums1[mid1] < nums2[mid2a])
            return medium(nums1, nums2, mid1, l2, r1, mid2b);
        else
            return medium(nums1, nums2, l1, l2 + len1/2, r1, r2 - len1/2);
    }

    double trivalmedium(vector<int>& nums1, vector<int>& nums2, int l1, int l2, int r1, int r2){
        vector<int> tmp;
        while (l1 <= r1 && l2 <= r2){
            while (l1 <= r1 && nums1[l1] <= nums2[l2] ) {tmp.push_back(nums1[l1]); l1++;}
            if (l1 > r1) break;
            while (l2 <= r2 && nums1[l1] >= nums2[l2] ) {tmp.push_back(nums2[l2]); l2++;}
        }
        while (l1 <= r1) {tmp.push_back(nums1[l1]); l1++;}
        while (l2 <= r2) {tmp.push_back(nums2[l2]); l2++;}
        int len = tmp.size();
        // for(int i : tmp) cout << i;
        if (len % 2 == 0){
            return (tmp[len/2] + tmp[len/2 - 1] ) / 2.0;
        } else 
            return tmp[len/2];
        return -1;
    }
};
```

## 5.最长回文子串@dp

```c++
string longestPalindrome(string s) {
        string p = s;
        reverse(p.begin(), p.end());
        return LCS(s,p);
    }

    string LCS(string a, string b){
        int len = a.size();
        int maxi = 0, maxlen = 0;
        int dp[len][len];
        for (int i = 0; i < len; ++i){
            for(int j = 0; j < len; ++j){
                if (a[i] == b[j]){
                    if (i == 0 || j == 0)   dp[i][j] = 1;
                    else dp[i][j] = dp[i - 1][j - 1] + 1;
                } else  dp[i][j] = 0;
                if (dp[i][j] > maxlen && i + j + 2 == dp[i][j] + len){
                    maxlen = dp[i][j];
                    maxi = i;
                }
            }
        }
        return a.substr(maxi - maxlen + 1,maxlen);
    } 
```

## 10.正则表达式匹配@dp

```c++
    bool isMatch(string s, string p) {
        s = " " + s; p = " " + p;
        int slen = s.size(), plen = p.size();
        bool dp[slen][plen];
        memset(dp, 0, sizeof dp);
        dp[0][0] = true;
        for (int i = 0; i < slen; ++i) {
            for (int j = 1; j < plen; ++j){
                if (j < plen - 1 && p[j + 1] == '*') continue;
                if (p[j] == '*'){
                     dp[i][j] = (j >= 2 && dp[i][j - 2] )|| \
                                (i >= 1 && dp[i - 1][j] && (s[i] == p[j - 1] || p[j - 1] == '.'));
                } else if (i >= 1){
                    dp[i][j] = dp[i - 1][j - 1] && (s[i] == p[j] || p[j] == '.');
                }
            }
        }
        return dp[slen - 1][plen - 1];
    }
```

## 15.三数之和

```c++
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        int len = nums.size();
        vector<vector<int>> ans;
        for (int i = 0; i < len - 2; i++){
            if (i > 0 && nums[i] == nums[i - 1])    continue;
            int j = i + 1, k = len - 1;
            while (j < k){
                int sum = nums[i] + nums[j] + nums[k];
                if (sum < 0) 
                    ++j;
                else if (sum > 0)
                    --k;
                else if (sum == 0) {
                    ans.push_back(vector<int>{nums[i],nums[j],nums[k]});
                    while(j < k && nums[j + 1] == nums[j])  ++j;
                    while(j < k && nums[k - 1] == nums[k])  --k;
                    ++j; --k;
                }
            }
        }
        return ans;
    }
};
```

## 17.电话号码的字母组合

```c++
class Solution {
public:
    string tmp;
    vector<string> res;
    vector<string> alphaT={"","","abc","def","ghi","jkl","mno","pqrs","tuv","wxyz"};
    void DFS(int pos, string digits){
        if (pos == digits.size()){
            res.push_back(tmp);
            return;
        }
        int num= digits[pos] - '0';
        for (int i = 0; i < alphaT[num].size(); i++){
            tmp.push_back(alphaT[num][i]);
            DFS(pos + 1, digits);
            tmp.pop_back();
        }
    }
    vector<string> letterCombinations(string digits) {
        if (digits.size()==0 ) return res;
        DFS(0, digits);
        return res;
    }
};
```

## 19.删除链表倒数第K个节点

```c++
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode* fptr = head, *sptr = head;
        for(int i = 0; i < n; ++i){
            fptr = fptr->next;
        }
        if(!fptr)   return head->next;//也可以设置空头结点，以解决删除头结点
        while (fptr && fptr->next != nullptr){        
            fptr = fptr->next;
            sptr = sptr->next;
        }
        sptr->next = sptr->next->next;
        return head;
    }
};
```

## 21.合并两个有序链表

```C++
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        if (!list1)  return list2;
        if (!list2)  return list1;
        ListNode *tmp1 = list1, *tmp2 = list2, *head = list1;
        if (tmp1->val > tmp2->val){
            tmp2 = list1; tmp1 = list2;
            head = list2;
        }
        while (tmp2){
            int tmp = tmp2->val;
            while (tmp1->next && tmp1->next->val <= tmp)
                tmp1 = tmp1-> next;
            if (!tmp1->next){
                tmp1->next = tmp2;
                return head;
            } else {
                list2 = tmp2->next;
                tmp2->next = tmp1->next;
                tmp1->next = tmp2;
            }
            tmp2 = list2;
        }
        return head;
    }
};
```

## 22.括号生成

```c++
class Solution {
public:
    vector<string> ans;
    int n;
    vector<string> generateParenthesis(int n) {
        this->n = n;
        string tmp = "";
        dfs(tmp, 0, 0);
        return ans;
    }
    void dfs(string tmp, int l, int r){
        if ( l > n || r > n || l < r)   return;
        if ( l == r && l == n) ans.push_back(tmp);
        dfs(tmp + "(", l + 1, r);
        dfs(tmp + ")", l, r + 1);
    }
};
```

