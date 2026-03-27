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
