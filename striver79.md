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


