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