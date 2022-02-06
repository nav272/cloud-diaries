---
layout: single
title: "Single element in a sorted array"
---


# Single element in a Sorted Array



### Problem Statement

You are given a sorted array consisting of only integers where every element appears exactly twice, except for one element which appears exactly once.

Return *the single element that appears only once*.

Your solution must run in `O(log n)` time and `O(1)` space.

**Example 1:**

```
Input: nums = [1,1,2,3,3,4,4,8,8]
Output: 2
```

**Example 2:**

```
Input: nums = [3,3,7,7,10,11,11]
Output: 10
```

**Constraints:**

- `1 <= nums.length <= 105`
- `0 <= nums[i] <= 105`

### Initial thoughts

- The numbers are duplicated, I think if you do a binary search, you are finding an `m` such that it has no value equal to `m` on the right side or the left side
- The question is how to set the greater than and less than parameter, who are you comparing it to? like if m is greater than what then start will go here if m is less than what then end will go here
- in python you can use count, but all the solutions you can think of are O(n)
- We want O(log(n))
- I did not find the solution of the binary search, but another student was able to find one which was great.

So, what is he doing? 

Firstly, he is checking if the mid is even or odd, but why is the question. 

if mid is even and he checks if the simultaneous numbers are equal and if they are, then start = mid +1 

```java
class Solution {
    public int singleNonDuplicate(int[] nums) {
        
        int start = 0;
        int end = nums.length-1;
    
        while (start < end) {
            int mid = start + (end - start) / 2;
        
            if (mid % 2 == 0) {
                if (nums[mid] == nums[mid+1]) {
                    start = mid + 1;
                } 
                else if ((mid - 1 >= 0) && nums[mid] == nums[mid-1]) {
                    end = mid - 1;
                } 
                else {
                start = mid;
                end = mid;
                }
            } 
            else if (mid % 2 != 0) {
                if (nums[mid] == nums[mid + 1]) {
                    end = mid - 1;
                } 
                else if (nums[mid] == nums[mid-1]) {
                    start = mid + 1;
                }
            } 
            else return nums[mid];
        }
        return nums[start]; 
    }
}
```

A better explained solution:

```java
class Solution {
    public int singleNonDuplicate(int[] nums) {
        int low = 0;
        int high = nums.length - 1;
        
        if (nums.length == 1) {
            return nums[0];
        }
  
        // The array that has the single element will always be odd in size
        // The input array will always be odd in size
        while (low <= high) {
            int mid = low + (high - low) / 2;
            
            if (mid - 1 > -1 && nums[mid] == nums[mid - 1]) {
                // if we take out the mid and mid-1 elements, 
                // check which section has the odd number of elements, 
				// and move in the direction of odd section 
                int numElementsInLowerSection = (mid - 1) - low;
                int numElementsInUpperSection = high - mid;
                
                if (numElementsInLowerSection % 2 != 0) {
                    high = mid - 2;
                } else if (numElementsInUpperSection  % 2 != 0) {
                    low = mid + 1;
                }
            } else if (mid + 1 < nums.length && nums[mid] == nums[mid + 1]) {
                 // if we take out the mid and mid+1 elements, 
                // check which section has the odd number of elements,
				// and move in the direction of odd section
                int numElementsInLowerSection = mid - low;
                int numElementsInUpperSection = high - (mid + 1);
                
                if (numElementsInLowerSection % 2 != 0) {
                    high = mid - 1;
                } else if (numElementsInUpperSection  % 2 != 0) {
                    low = mid + 2;
                }
            } else {
                return nums[mid];
            }
        }
        
        return -1;
    }
}
```
