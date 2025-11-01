# Remove Outer paranthesis - leetcode

class Solution:
    def removeOuterParentheses(self, s: str) -> str:
        stack = []
        res = []
        pops=0

        for idx,i in enumerate(s):
            if i == '(':
                stack.append(i)
            if i == ')':
                stack.pop()
                pops+=1
                if len(stack)==0:
                    res.append(idx)
                    res.append(idx - (2*pops-1))
                    pops=0
        print(res)
        
        ans = ""
        for idx, i in enumerate(s):
            if idx not in res:
                ans += i
        return ans

# Reverse words in string - leetcode

class Solution:
   def reverseWords(self, s: str) -> str:
       s = s.strip()
       ls = re.split("\s+",s)
       ls = ls[::-1]
       return " ".join(ls)


# Largest odd number

class Solution:
    def largestOddNumber(self, num: str) -> str:
        l = len(num)
        ans = -1

        for i in range(l-1,-1,-1):
            if int(num[i])%2 != 0:
                ans = i
                break

        if ans == -1:
            return ""
        return num[:ans+1]

# Longest Common Prefix
class Solution:
   def longestCommonPrefix(self, strs: List[str]) -> str:
       res = ""

       strs = sorted(strs, key=len)

       for i in range(len(strs[0])):
           for j in strs:
               if j[i] != strs[0][i]:
                   return res
           res += strs[0][i]
       return res       

# Isomosrphic strings
class Solution:
    def isIsomorphic(self, s: str, t: str) -> bool:
        dd = {}

        if len(s) != len(t):
            return False

        for i in range(len(s)):
            if s[i] not in dd and t[i] in dd.values():
                return False
            elif s[i] not in dd:
                dd[s[i]] = t[i]
            else:
                if dd[s[i]] != t[i]:
                    return False
        
        return True


# Rotate string
 class Solution:
    def rotateString(self, s: str, goal: str) -> bool:
        res = s
        if res == goal:
            return True
        for i in range(len(s)):
            res = res[1:] + res[0]
            if res == goal:
                return True
        
        return False
               
# Anagram
class Solution:
    def isAnagram(self, s: str, t: str) -> bool:
        s = sorted(s)
        t = sorted(t)

        if s == t:
            return True
        return False


# Sort chars by frequency
 class Solution:
    def frequencySort(self, s: str) -> str:
        fre = {}
        for i in s:
            if i not in fre:
                fre[i] = 1
            else:
                fre[i] += 1
        fre = dict(sorted(fre.items(), key=lambda item: item[1], reverse=True))        
        res = ""
        for key, val in fre.items():
            s = ("" + key ) * val
            res += s
        return res

# Nesting depth paranthesis

class Solution:
    def maxDepth(self, s: str) -> int:
        ss = []
        ans = 0
        for i in s:
            if i == '(':
                ss.append(i)
            elif i == ')':
                ans = max(ans,len(ss))
                ss.pop()
        return ans

# Roman to int

// i++ doesnt work in i in range
// use while and increment

 class Solution:
    def romanToInt(self, s: str) -> int:
        dd = {'I': 1, 'V': 5, 'X': 10, 'L': 50, 'C': 100, 'D': 500, 'M': 1000}

        ans = 0
        i = 0
        
        while i < len(s):
            if i+1 < len(s) and dd[s[i+1]] > dd[s[i]]:
                ans += dd[s[i+1]]-dd[s[i]]
                i+=2
            else:
                ans += dd[s[i]]
                i += 1
        return ans
               
# String to integer (Atoi)
class Solution:
    def myAtoi(self, s: str) -> int:
        sign = '+'
        num = "0123456789"

        ans = "0"

        s = s.strip()
        print(s)
        for i in range(len(s)):
            if i == 0:
                if s[i] == '+' or s[i]=='-':
                    sign = s[i]
                    continue
            if s[i] not in num:
                if int(ans) > 2**31-1:
                    if sign == '+':
                        return 2**31-1
                    else:
                        return -1 * 2**31
                return int(ans) if sign == '+' else -1 * int(ans)
            ans += s[i]

        if sign == '-':
            return max(-1 * 2**31, -1 * int(ans))
        return min(2**31-1, int(ans))


# Longest palindromic substring
 class Solution:
    def longestPalindrome(self, s: str) -> str:
        l = len(s)

        for i in range(l,-1,-1):
            n = l-i
            if n == 0:
                if s == s[::-1]:
                    return s
            for j in range(n+1):
                ss = s[j:j+i]
                if ss == ss[::-1]:
                    return ss
        return ""
            

# Sum of beauty of all strings
class Solution:
    def beauty(self,s):
        fre = {}

        for i in s:
            fre[i] = fre.get(i,0) + 1
        maxx = max(fre.values())
        minn = min(fre.values())

        return maxx - minn      

    def beautySum(self, s: str) -> int:
        ans = 0
        for i in range(len(s)):
            for j in range(i+1, len(s)+1):
                ans += self.beauty(s[i:j])
        return ans


# Add a node
class Solution:
    def insertAtHead(self, head, X):
        node = ListNode(X)
        node.next = head
        head = node

        ans = []
        while head!=None:
            ans.append(head.val)
            head = head.next
        print(" ".join(ans))

# Delete a node
class Solution:
    def deleteNode(self, node):
        """
        :type node: ListNode
        :rtype: void Do not return anything, modify node in-place instead.
        """
        node.val = node.next.val
        node.next = node.next.next

# 
