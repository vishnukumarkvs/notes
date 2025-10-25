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
        
