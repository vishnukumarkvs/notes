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

# Add two numbers in LL

# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def addTwoNumbers(self, l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
        n = 0

        a = l1
        b = l2
        ans = l2

        while l1 or l2:
            if l1.next and not l2.next:
                l2.next = ListNode(0)
            if l2.next and not l1.next:
                l1.next = ListNode(0)
            l1 = l1.next
            l2 = l2.next
        prev = None
        while a:
            v = a.val + b.val + n
            b.val = v %10
            if v >= 10:
                n = 1
            else:
                n = 0
            prev = b
            a = a.next
            b = b.next
        if n:
            prev.next = ListNode(1)
        return ans

# Add 1 to a linked list
- Edge cases: 456, 459, 9, 999


class Node:
    def __init__(self, data):   # data -> value stored in node
        self.data = data
        self.next = None
'''

class Solution:
    def reverse(self, head):
        prev = None
        cur = head
        while cur:
            nnext = cur.next
            cur.next = prev
            prev = cur
            cur = nnext
        return prev
    def addOne(self,head):
        #Returns new head of linked List.
        h = head
        l = self.reverse(head)
        ans = l

        n = 0
        ll = l.data + 1
        if ll < 10:
            l.data = l.data + 1
            return self.reverse(l)
        else:
            l.data = (l.data + 1) % 10
            n = 1
            if l.next == None:
                l.next = Node(1)
                return self.reverse(l)
            l = l.next
        pp = None

        while l:
            v = l.data + n
            l.data = (l.data + n) % 10
            if v >= 10:
                n = 1
            else:
                n = 0
            pp = l
            l = l.next
        if n:
            pp.next = Node(1)
            
        return self.reverse(ans)        

# Delete all occurences of key
class Solution:
    #Function to delete all the occurances of a key from the linked list.
    def deleteAllOccurOfX(self, head, x):
        # code here
        # edit the linked list
        h = head
        while h:
            if h.data == x and h.next != None:
                if h.prev == None:
                    head = head.next
                    h = h.next
                    head.prev = None
                else:
                    pp = h.prev
                    nn = h.next
                    pp.next = nn
                    nn.prev = pp
                    h = h.next
            elif h.data == x and h.next == None:
                h.prev.next = None
                return head
            else:
                h = h.next
        return head

# Fund pairs with given sum in DLL
- Use set() for fast lookups
class Solution:
    def findPairsWithGivenSum(self, target : int, head : Optional['Node']) -> List[List[int]]:
        # code here
        ll = set()
        ans = []
        h = head
        while h:
            ll.add(target-h.data)
            if h.data in ll and h.data != target-h.data:
                ans.append([target-h.data, h.data])
            h = h.next
        return ans[::-1]

# Remove duplicates is sorted DLL
class Solution:
    #Function to remove duplicates from unsorted linked list.
    def removeDuplicates(self, head):
        # code here
        # return head after editing list
        h = head
        while h and h.next:
            if h.data == h.next.data:
                h.next = h.next.next
            else:
                h = h.next
        return head


# Rotate Linked List
- Use mod to cancel unnecersary rotations
class Solution:
    def rotateRight(self, head: Optional[ListNode], k: int) -> Optional[ListNode]:
        if head == None:
            return head
        ll = 1
        cur = head
        while cur.next!=None:
            cur = cur.next
            ll += 1
        cur.next = head

        r = k % ll

        print(r)

        lp = ll - r
        prev = None

        for i in range(lp):
            prev = head
            head = head.next

        prev.next = None

        return head

# Flatten a linked list
- links are sorted
- Use merge() in merge sort

class Solution:
    def merge(self, a ,b):
        if not b:
            return a 
        if not a:
            return b
        result = None
        if a.data < b.data:
            result = a
            result.bottom = self.merge(a.bottom,b)
        else:
            result = b
            result.bottom = self.merge(a, b.bottom)
        result.next = None
        return result
    
    def flatten(self, root):
        if not root:
            return root
        
        return self.merge(root, self.flatten(root.next))
        
# Copy list with Random pointer
- Use dict with none checks
class Solution:
    def copyRandomList(self, head: 'Optional[Node]') -> 'Optional[Node]':
        dic = {}
        h = head
        if h is None:
            return h

        while head:
            p = Node(head.val)
            dic[head]=p
            head = head.next
        for k,v in dic.items():
            if k.next == None:
                v.next = None
            else:
                v.next = dic[k.next]
            if k.random == None:
                v.random = None
            else:
                v.random = dic[k.random]
        return dic[h]

# Pow(x,n)
- Power function
- O(n) - simple while on n, time limit exceeded
- O(logn)
- Recusrion
- 2^10 = 2^5 * 2^5 - No need to compute 2 ^ 10 fully
- 2^5 = 2 * 2^2 * 2^2

class Solution:
    def myPow(self, x: float, n: int) -> float:
        def helper(x,n):
            if x == 0: return 0
            if n == 0: return 1

            res = helper(x,n//2)
            res = res * res
            return x*res if n%2 else res
        res = helper(x, abs(n))
        return res if n >= 0 else 1/res

# Count Good Numbers
- Leetcode

- Time Limit Exceeded for below
class Solution:
    def countGoodNumbers(self, n: int) -> int:
        MOD = 10**9 + 7

        def isPrime(x):
            if x<=1:
                return False
            isprime = True
            for i in range(2,int(x**0.5)+1):
                if x%i==0:
                    isprime = False
                    break
            return isprime
        def good(x):
            for i,e in enumerate(x):
                if i%2 == 0:
                    if int(e) % 2 != 0:
                        return False
                if i%2 != 0:
                    return isPrime(int(e))
                return True
        ans = 0
        for i in range(0, 10**n):
            if good(str(i)):
                ans += 1
        return ans % MOD

- Formula
- Combinations
- 5 possiblilites in even places (0,2,4,6,8) , 4 possibilities in odd places (2,3,5,7)
- n = 4, _ _ _ _ = (5 * 4 * 5 * 4)
- Still time limit exceeded for pow complexity (Inbuilt ** doesnt work)

class Solution:
    def countGoodNumbers(self, n: int) -> int:
        MOD = 10**9 + 7
        if n == 1:
            return 5

        even = math.ceil(n/2)
        odd = math.floor(n/2)
        print(even, odd)
        ans = (5**even * 4**odd) % MOD
        return ans

- Optimal
class Solution:
    def countGoodNumbers(self, n: int) -> int:
        MOD = 10**9 + 7
        
        def pow(x,n):
            # 5^8
            # 25^4
            # 625^2
            # 3665^1

            res = 1
            while n > 0:
                if n%2:
                    res = (res * x) % MOD
                x = (x * x) % MOD
                n = n //2
            return res

        even = math.ceil(n/2)
        odd = math.floor(n/2)
        print(even, odd)
        ans = (pow(5,even)*pow(4,odd)) % MOD
        return ans

# sort stack
- Using another stack
class Solution:
   def sortStack(self, st):
       st2 = []
       while len(st):
           a = st.pop()
           
           # Move elements from st2 back to st that are smaller than a
           while len(st2) and st2[-1] < a:
               st.append(st2.pop())
           
           # Now insert a into st2
           st2.append(a)
       while len(st2):
           st.append(st2.pop())
