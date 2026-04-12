# Valid Anagrams

class Solution:
    def isAnagram(self, s: str, t: str) -> bool:
        if len(s) != len(t):   # early exit
            return False

        a = {}

        for i in s:
            a[i] = a.get(i, 0) + 1  # safe default

        for i in t:
            a[i] = a.get(i, 0) - 1  # handles chars not in s

        for i in a.values():
            if i != 0:
                return False
        return True

# Anagrams

class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        d = defaultdict(list)

        for s in strs:
            key = tuple(sorted(s))
            d[key].append(s)
        return list(d.values())

# Top K frequent elements

class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        m = defaultdict(int)

        for i in nums:
            m[i]+=1
        ans = []
        for k,v in m.items():
            heapq.heappush(ans,(-v,k))
        f = []
        for _ in range(k):
            f.append(heapq.heappop(ans)[1])
        return f

# Longest Consecutive sequence
class Solution:
    def longestConsecutive(self, nums: List[int]) -> int:
        if not nums:
            return 0
        nums = sorted(set(nums))
        ans = 0
        k = 0
        print(nums)
        for i in range(1,len(nums)):
            if nums[i]-1==nums[i-1]:
                k+=1
                ans = max(ans,k)
            else:
                k = 0
        return ans+1

Better sol
- set lookup, check starting by neigbour, set

class Solution:
    def longestConsecutive(self, nums: List[int]) -> int:

        if not nums:
            return 0
        s = set(nums)

        ans = 0
        for i in nums:
            if i-1 not in s:
                k = 0
                start = i
                while start+1 in s:
                    k+=1
                    start+=1
                ans = max(ans,k)
        return ans+1

# Palindrome

class Solution:
    def isPalindrome(self, s: str) -> bool:
        s1 = ""
        for i in s:
            if i.isalnum():
                s1+=i
        s1 = s1.lower()

        if s1 == s1[::-1]:
            return True
        return False

- Not optimal. On2 because s1+=i bcreates a new string as its immutable

def isPalindrome(self, s: str) -> bool:
    s1 = "".join(c.lower() for c in s if c.isalnum())
    return s1 == s1[::-1]

# Two sum 2
- O(n2)
- O(1) space
class Solution:
    def twoSum(self, numbers: List[int], target: int) -> List[int]:

        idx1 = 0
        val2 = 0
        idx2 = 0
        for idx, val in enumerate(numbers):
            if target - val in numbers:
                idx1 = idx
                val2 = target - val
                break
        for idx, val in enumerate(numbers):
            if val == val2:
                idx2 = idx
        
        return [idx1+1,idx2+1] if idx1<idx2 else [idx2+1,idx1+1]

# Container with most water
- Brute force, see all possible combinations

class Solution:
    def maxArea(self, heights: List[int]) -> int:
        n = len(heights)

        ans = 0
        for i in range(n):
            for j in range(i+1,n):
                area = min(heights[i],heights[j]) * (j-i)
                ans = max(ans,area)
        return ans

# Two pointer solution
class Solution:
    def maxArea(self, heights: List[int]) -> int:
        n = len(heights)
        i = 0
        j = n - 1

        ans = 0
        while i<j:
            area = min(heights[i],heights[j]) * (j-i)
            ans = max(ans,area)

            if heights[i]> heights[j]:
                j-=1
            else:
                i+=1
        return ans

# Buy and Sell stocks
- Two pointer
- Shift r continuously
- Shift l to r only if l is greater than r
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        l,r = 0,1
        maxP = 0

        while r < len(prices):
            if prices[l] < prices[r]:
                profit = prices[r]-prices[l]
                maxP = max(maxP,profit)
            else:
                l = r
            r += 1
        return maxP

# Longest substring without repeating chars
- sliding window with set

class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        l, r = 0, 0
        maxl = 0
        se = set()

        while r < len(s):
            if s[r] in se:
                while s[l] != s[r]:  # remove chars until we find the duplicate
                    se.remove(s[l])
                    l += 1
                l += 1  # skip past the duplicate itself
            se.add(s[r])
            maxl = max(maxl, r - l + 1)  # +1 for inclusive length
            r += 1

        return maxl


