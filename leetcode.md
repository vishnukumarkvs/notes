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

# Length of linked list

class Solution:
    def getCount(self, head):
        # code here
        ans = 0
        while head != None:
            head = head.next
            ans += 1
        return ans


# Search in linkedlist
- O(n) - Linear

class Solution:
    def searchKey(self, head, key):
        #Code here
        while head != None:
            if head.data == key:
                return True
            head = head.next
        return False

# Insert in doubly linked list (DLL)
class Solution:
    def insertAtPos(self, head, p, x):
        # Code Here
        node = Node(x)
        
        pos = 0
        
        hhead = head
        
        while head!=None:
            if pos == p:
                node.next = head.next
                node.prev = head
                ll = head.next
                if ll != None:
                    ll.prev = node
                head.next = node
            pos += 1
            head = head.next
        return hhead

# Delete a node in DLLL
class Solution:
    def delPos(self, head, x):
        # code here
        if x == 1:
            if head.next:
                head.next.prev = None # Important
            return head.next
        pos = 1
        hhead = head
        
        while head!=None:
            if pos == x:
                if head.next == None:
                    ll = head.prev
                    ll.next = None
                else:
                    pprev = head.prev
                    nnext = head.next
                    pprev.next = nnext
                    nnext.prev = pprev
            head = head.next
            pos += 1
        return hhead

# Reverse a DLL (Tricky - Check)
class Solution:
    def reverse(self, head):
        # code here
        cur = head
        temp = None
        while cur != None:
            temp = cur.prev
            cur.prev = cur.next
            cur.next = temp
            cur = cur.prev
        
        if temp:
            return temp.prev
        return head

# Middle of linked list - (Tortoise Hare)
class Solution:
    def middleNode(self, head: Optional[ListNode]) -> Optional[ListNode]:
        tor = head
        hare = head

        while hare != None and hare.next != None:
            hare = hare.next.next
            tor = tor.next
        return tor

# Reverse a LL
class Solution:
   def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
       prev = None
       cur = head
       while cur != None:
           nnext = cur.next # Save next node
           cur.next = prev # Reverse link
           prev = cur # Love prev
           cur = nnext # Move cur
       return prev       

# Loop in LL
class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        tor = head
        hare = head
        while head != None and head.next != None:
            tor = tor.next
            head = head.next.next
            if tor == head:
                return True
        return False

# Linked List Cycle 2
class Solution:
    def detectCycle(self, head: Optional[ListNode]) -> Optional[ListNode]:
        tor = head
        hare = head

        pos = 0

        while hare != None and hare.next != None:
            tor = tor.next
            hare = hare.next.next

            if hare == tor:
                tor = head
                while tor != hare:
                    tor = tor.next
                    hare = hare.next
                return tor
        return None

# Length of loop
- Find loop starting point, traverse again
 class Solution:
    def lengthOfLoop(self, head):
        #code here
        tor = head
        hare = head
        
        while hare and hare.next:
            tor = tor.next
            hare = hare.next.next
            
            if tor == hare:
                tor = head
                while tor != hare:
                    tor = tor.next
                    hare=hare.next
                ans = 1
                n = tor.next
                while n!=tor:
                    n= n.next
                    ans +=1
                return ans
        return 0

# LL is a palindrome
- reverse first part of linkedlist till the middle and traverse again
class Solution:
    def isPalindrome(self, head: Optional[ListNode]) -> bool:
        tor = head
        hare = head

        while hare and hare.next:
            tor = tor.next
            hare = hare.next.next
        
        # Reverse first part
        prev = None
        while head != tor:
            nnext = head.next
            head.next = prev
            prev = head
            head = nnext

        # For odd lengths
        if hare:
            tor = tor.next
            
        while prev!=None:
            if prev.val != tor.val:
                return False
            prev = prev.next
            tor = tor.next
        return True
        
# Odd Even Linked List
class Solution:
    def oddEvenList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        if head == None:
            return head
        odd = head
        even = head.next
        e2 = even

        hare = head
        
        while hare and hare.next:
            hare = hare.next.next
            if not hare:
                break
            odd.next = hare
            even.next = hare.next
            odd = odd.next
            even = even.next
        odd.next = e2
        return head

 
# Remove Nth Node from end of list
class Solution:
    def removeNthFromEnd(self, head: Optional[ListNode], n: int) -> Optional[ListNode]:
        l = 0
        cur = head
        while cur:
            cur = cur.next
            l+=1

        if l ==1:
            return None
        p = l-n

        if p == 0:
            return head.next

        cur2 = head
        while p-1 > 0:
            cur2 = cur2.next
            p-=1
        cur2.next = cur2.next.next
        return head

# Remove middle element
class Solution:
    def deleteMiddle(self, head: Optional[ListNode]) -> Optional[ListNode]:
        prev = None
        tor = head
        hare = head
        while hare and hare.next:
            prev = tor
            tor =tor.next
            hare = hare.next.next

        if not prev:
            return None
        prev.next = prev.next.next
        return head

# Find intersection of two Linked lists
- Approach 1
- Find lengths of two lists, move largest list by the difference and compare
class Solution:
    def getIntersectionNode(self, headA: ListNode, headB: ListNode) -> Optional[ListNode]:
        l1 = 0
        l2 = 0
        a = headA
        b = headB
        while headA:
            headA = headA.next
            l1 += 1
        while headB:
            headB = headB.next
            l2 += 1

        if l1 > l2:
            ll = l1-l2
            while ll:
                a = a.next
                ll -= 1
        else:
            ll = l2-l1
            while ll:
                b = b.next
                ll -= 1
        while a:
            if a == b:
                return a
            a = a.next
            b = b.next
        return None

- Approach 2 (Trick)
- connect end of l1 to l2 and l2 to l1
class Solution:
    def getIntersectionNode(self, headA: ListNode, headB: ListNode) -> Optional[ListNode]:
        l1, l2 = headA, headB
        while l1 != l2:
            l1 = l1.next if l1 else headB
            l2 = l2.next if l2 else headA
        return l1

