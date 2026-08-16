# Next Permutation

from typing import List

class Solution:
    def nextPermutation(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        n = len(nums)
        
        # Step 1: Find the pivot
        # Scan from right to left to find the first element that is less than its right neighbor.
        pivot = -1
        for i in range(n - 2, -1, -1):
            if nums[i] < nums[i + 1]:
                pivot = i
                break
        
        # Step 2: If a pivot was found, find the element to swap it with
        if pivot != -1:
            # Scan from right to left to find the first element strictly greater than the pivot
            for i in range(n - 1, pivot, -1):
                if nums[i] > nums[pivot]:
                    # Swap them
                    nums[i], nums[pivot] = nums[pivot], nums[i]
                    break
        
        # Step 3: Reverse the suffix
        # Everything to the right of the pivot is currently in descending order. 
        # Reversing it makes it ascending order, which gives us the smallest possible next permutation.
        # (If pivot == -1, this reverses the whole array, handling the [3, 2, 1] -> [1, 2, 3] edge case).
        left = pivot + 1
        right = n - 1
        while left < right:
            nums[left], nums[right] = nums[right], nums[left]
            left += 1
            right -= 1

# 3 sum

- sort and two pointer approach
- o(n2)

class Solution:
    def threeSum(self, nums: list[int]) -> list[list[int]]:
        nums.sort()
        res = []
        l = len(nums)

        for i in range(len(nums)-2):
            if i> 0 and nums[i]==nums[i-1]:
                continue
            
            left = i+1
            right = l-1

            while left < right:
                total = nums[i]+nums[left]+nums[right]
                if total == 0:
                    res.append([nums[i],nums[left],nums[right]])
                
                    # Skip duplicates
                    while left < right and nums[left]==nums[left+1]:
                        left+=1
                    
                    while left < right and nums[right]==nums[right-1]:
                        right-=1
                    
                    left+=1
                    right-=1

                elif total < 0:
                    left+=1
                else:
                    right-=1
                
        return res
                


