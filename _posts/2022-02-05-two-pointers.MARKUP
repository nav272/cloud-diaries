---
layout: single
title: "Squares of a sorted array version 2"
---

# Squares of a sorted array

Given an integer array `nums` sorted in **non-decreasing** order, return *an array of **the squares of each number** sorted in non-decreasing order*.

**Example 1:**

```
Input: nums = [-4,-1,0,3,10]
Output: [0,1,9,16,100]
Explanation: After squaring, the array becomes [16,1,0,9,100].
After sorting, it becomes [0,1,9,16,100].

```

**Example 2:**

```
Input: nums = [-7,-3,2,3,11]
Output: [4,9,9,49,121]

```

**Constraints:**

- `1 <= nums.length <= 104`
- `104 <= nums[i] <= 104`
- `nums` is sorted in **non-decreasing** order.

**Follow up:**

Squaring each element and sorting the new array is very trivial, could you find an

```
O(n)
```

solution using a different approach?

The brute force approach looks like this: 

```java

//This approach takes O(logn^2) approach. It takes n approaches to square and n approaches to sort. Java has an inbuilt function to sort it. 
class Solution {
    public int[] sortedSquares(int[] nums) {
        int[] squares = new int[nums.length];
        for(int i=0; i<nums.length; i++){
            squares[i]=nums[i]*nums[i];
        }
        Arrays.sort(squares);
        return squares;
        
        
    }
}
```

Another approach which is the two-pointer approach looks like this: 

```java
//this approach uses 2 pointers to sort the array, which is interesting. The interesting thing is it will have no more iterations than the length of the array which means the approach will be O(log n) no matter what. This will only work in this sorted array where we know the last element will be the largest and we assign elements to the array just like that. 

class Solution {
    public int[] sortedSquares(int[] nums) {
        
        int l =0;
        int r = nums.length -1;
        int k = r;
        int[] ans = new int[r+1];
        
        while(l<=r)
        {
            if(Math.abs(nums[l]) > Math.abs(nums[r]))
            {
                ans[k--] = nums[l]*nums[l];
                l++;
            }else{
                ans[k--] = nums[r]*nums[r];
                r--;
            }
            //System.out.println(r);
        }
        return ans;
        
    }
}
```
