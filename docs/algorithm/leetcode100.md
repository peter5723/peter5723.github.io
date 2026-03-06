# lc100

## 哈希表

```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        unordered_set<int> st(nums.begin(), nums.end());
        int ans = 0;
        for(int x:st){
            if(st.contains(x-1)){
                continue;
            }

            int y =x+1;
            while(st.contains(y)) {
                y++;
            }

            ans = max(ans, y-x);
        }
        return ans;
    }
};
```
这个解法里面每个元素最多被访问两次（遍历一次结算一次），所以时间复杂度为 O(n)。

## 双指针
```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int n = nums.size(), left = 0, right = 0;
        while(right<n){
            if(nums[right]) {
                swap(nums[left], nums[right]);
                left++;// 保证left的结尾是一个0。
            }
            right++;
        }
    }
};
```

```cpp
class Solution {
public:
    int maxArea(vector<int>& height) {
        int ans = 0, left =0 ,right=height.size()-1;
        while(left<right){
            int area = (right-left)*min(height[left], height[right]);
            ans = max(ans, area);
            if(height[left]<height[right]){
                left++;
            }else{
                right--;            
            }
        }
        return ans;
    }
};
```

## 滑动窗口

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        unordered_set<char> occ;
        int n = s.size();
        int rk=-1,ans=0;
        for(int i=0;i<n;i++){
            if(i!=0) occ.erase(s[i-1]);
        }
        while(rk+1<n && !occ.count(s[rk+1])) {
            occ.insert(s[rk+1]);
            rk++;
        }
        ans = max(ans, rk-i+1);
    }
    return ans;
};
```
动长滑动窗口

注意，滑动窗口虽然是二重循环，但本质上只不过是左指针（i）和右边指针（rk）遍历两次，所以复杂度仍然是 
o(N)。

```cpp
class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
        int sLen = s.size(), pLen = p.size();
        if(sLen < pLen) {
            return {};
        }
        vector<int> ans;
        vector<int> sCount(26);
        vector<int> pCount(26);

        for(int i=0;i<pLen;i++) {
            sCount[s[i]-'a']++;
            pCount[p[i]-'a']++;
        }
        if(sCount==pCount){
            ans.push_back(0);
        }
        for(int i=0;i<sLen-pLen;i++) {
            sCount[s[i]-'a']--;
            sCount[s[i+pLen]-'a']++;
            //这两句话就模拟了滑动窗口向前移动
            if(sCount == pCount) {
                ans.push_back(i+1);
            }
        }
        return ans;
    }
};
```

定长滑动窗口


## 数组
最大子数组和
```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        //f(i) = max(f(i-1)+nums[i], nums[i])
        int pre = 0, maxAns = nums[0];
        for(auto &x:nums){
            pre = max(pre+x,x);
            maxAns = max(maxAns, pre);
        }
        return maxAns;
    }
};
```

轮转数组：三次翻转

```cpp
class Solution {
public:

    void reverse(vector<int>& nums, int start, int end) {
        while(start<end){
            swap(nums[start], nums[end]);
            start++;
            end--;
        }
    }

    void rotate(vector<int>& nums, int k) {
        k%=nums.size();
        reverse(nums, 0, nums.size()-1);
        reverse(nums, 0, k-1);
        reverse(nums, k, nums.size()-1);
    }
};
```


## 链表

翻转链表，请背诵

中间节点，快慢指针经典应用

```cpp
class Solution {
public:
    ListNode* middleNode(ListNode* head) {
        if(head==nullptr){
            return nullptr;
        }
        ListNode* slow = head;
        ListNode* fast = head;
        while(fast!=nullptr && fast->next!=nullptr){
            slow = slow->next;
            fast = fast->next->next;
        } 
        return slow;
    }

    ListNode* reverseList(ListNode* head) {
        ListNode* pre = nullptr, *cur = head;
        while(cur){
            ListNode* next = cur->next;
            cur->next = pre;
            pre = cur;
            cur = next;
        }
        return pre;
    }
    bool isPalindrome(ListNode* head) {
        ListNode* mid = middleNode(head);
        ListNode* head2 = reverseList(mid);
        while(head2){
            if(head->val != head2->val) {
                return false;
            }

            head = head->next;
            head2 = head2->next;
        }
        return true;
    }
};
```

注意，这种情况下解决问题最清楚，但是在翻转链表的时候会破坏链表结构

典型错误：
```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode* head = new ListNode();
        ListNode* ans = head;
        int  now_ten = 0;
        while(l1 || l2) {
            int n1 = l1 ? l1->val:0;
            int n2 = l2 ? l2->val:0;
            ans->val = (n1 + n2+now_ten)%10;
            now_ten = (n1+n2+now_ten >=10) ? 1 : 0;
            if(l1) l1 = l1->next;
            if(l2) l2 = l2->next;
            ListNode* temp = new ListNode();
            ans->next = temp;
            ans = ans->next;
        }
        if(now_ten>0){
            ans->val = now_ten;
            ans->next = nullptr;
        }else{delete(ans);}
        return head;;
        
    }
};
```
后创建链表后free，会导致前一个next成为野指针，从而导致错误。

推荐使用哨兵

删除倒数第 n 个节点，快指针先走 n 步即可。