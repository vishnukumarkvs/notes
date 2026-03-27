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

