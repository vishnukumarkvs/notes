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


