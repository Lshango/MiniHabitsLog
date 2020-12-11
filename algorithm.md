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
