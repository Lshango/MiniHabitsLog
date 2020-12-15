# 2020 [MiniHabits](./miniHabit.md)

## 08/12/2020

+ Leetcode-->[Problem #1](https://leetcode.com/problems/two-sum/)

```Python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        num_to_index = {}
            for i, num in enumerate(nums):
                if target-num in num_to_index:
                    return [num_to_index[target-num], i]
                num_to_index[num] = i
        return []
```

## 09/12/2020

+ Leetcode-->[Problem #1](https://leetcode.com/problems/two-sum/)

```C++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> mapping;
        vector<int> result;
        for (int i = 0; i < nums.size(); ++i) {
            mapping[nums[i]] = i;
        }
        for (int i = 0; i < nums.size(); ++i){
            int secNum = target - nums[i];
            if (mapping.find(secNum) != mapping.end() && mapping[secNum] > i){
                result.push_back(i);
                result.push_back(mapping[secNum]);
                break;                  
            }
        }
        return result;
    }
};
```

## 10/12/2020

+ Leetcode-->[Problem #2](https://leetcode.com/problems/add-two-numbers/)

```Python
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None
class Solution:
    def addTwoNumbers(self, l1: ListNode, l2: ListNode) -> ListNode:
        prev = r_val = ListNode(None)
        carry = 0
        while l1 or l2 or carry:
            if l1:
                carry += l1.val
                l1 = l1.next
            if l2:
                carry += l2.val
                l2 = l2.next
            prev.next = ListNode(carry % 10)
            prev = prev.next
            carry //= 10
        return r_val.next
```

## 11/12/2020

+ Leetcode-->[Problem #2](https://leetcode.com/problems/add-two-numbers/)

```C++
struct ListNode {
    int val;
    ListNode *next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode *next) : val(x), next(next) {}
 };
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        int carry(0);
        ListNode* result = new ListNode(0);
        ListNode* prev = result;
        while(l1 || l2 || carry){
            if(l1){
                carry += l1->val;
                l1 = l1->next;
            }
            if(l2){
                carry += l2->val;
                l2 = l2->next;
            }
            prev->next = new ListNode(carry % 10);
            prev = prev->next;
            carry = carry / 10;
        }
        return result->next;
    }
};
```

## 12/12/2020

+ Leetcode-->[Problem #3](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

```Python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        subString = ""
        retLongth = 0
        for i in range(0, len(s)):
            if s[i] in subString:
                retLongth = max(retLongth, len(subString))
                pos = subString.find(s[i])
                subString = subString[(pos+1):] + s[i]
            else:
                subString += s[i]
        
        return max(retLongth, len(subString))
```

```C++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        string subString{""};
        int retLength{0};
        for (const auto& c : s){
            if(subString.find(c) != string::npos){
                retLength = (retLength > subString.size() ? retLength : subString.size());
                subString = subString.substr(subString.find(c) + 1);
            }
            subString += c;
        }
        retLength = (retLength > subString.size() ? retLength : subString.size());
        return retLength;
    }
};
```

## 13/12/2020

+ Leetcode-->[Problem #4](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/)

```Python
class Pro004Solution:
    @classmethod
    def find_median_sorted_arrays(cls, nums1, nums2):
        """
        :param nums1: list[int]
        :param nums2: list[int]
        :return: float
        """
        def get_kth_smallest(a_start, b_start, k):
            if k <= 0 or k > len(nums1) - a_start + len(nums2) - b_start:
                raise ValueError("The input value error!")
            if len(nums1) == a_start:
                return nums2[b_start + k - 1]
            if len(nums2) == b_start:
                return nums1[a_start + k - 1]
            if k == 1:
                return min(nums1[a_start], nums2[b_start])

            mid_a, mid_b = float('inf'), float('inf')
            if k//2 <= len(nums1) - a_start:
                mid_a = nums1[a_start + k // 2 - 1]
            if k//2 <= len(nums2) - b_start:
                mid_b = nums2[b_start + k // 2 - 1]

            if mid_a < mid_b:
                return get_kth_smallest(a_start + k // 2, b_start, k - k // 2)
            else:
                return get_kth_smallest(a_start, b_start + k // 2, k - k // 2)

        right = get_kth_smallest(0, 0, 1 + (len(nums1) + len(nums2))//2)
        if (len(nums1) + len(nums2)) % 2 == 1:
            return right
        else:
            left = get_kth_smallest(0, 0, (len(nums1) + len(nums2))//2)
            return (left + right)/2.0

```

## 14/12/2020

+ Leetcode-->[Problem #4](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/)
  
  ```C++
    class Solution {
    public:
        double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
            const int m = nums1.size();
            const int n = nums2.size();
            int total = m + n;
            if (total & 0x1)
                return find_kth(nums1.begin(), m, nums2.begin(), n, total / 2 + 1);
            else
                return (find_kth(nums1.begin(), m, nums2.begin(), n, total / 2) + find_kth(nums1.begin(), m, nums2.begin(), n, total / 2 + 1)) / 2.0;
        }
    private:
        int find_kth(std::vector<int>::const_iterator A, int m, std::vector<int>::const_iterator B, int n, int k) {
            if(m > n) return find_kth(B, n, A, m, k);
            if(m == 0) return *(B + k - 1);
            if (k == 1) return min(*A, *B);
            
            int ia = min(k / 2, m), ib = k - ia;
            if (*(A + ia - 1) < *(B + ib - 1))
                return find_kth(A + ia, m - ia, B, n, k - ia);
            else if (*(A + ia - 1) > *(B + ib - 1))
                return find_kth(A, m, B + ib, n - ib, k - ib);
            else
                return A[ia - 1];
        }
    };
  ```
  