# Pointers
- OSPF routing protocol(similar to BGP) uses Djkistra algorithm
- Heaps are used as storage for garbage collection in java
- heap is a nealry complete binary tree
- maxheap => value of i <= value of parent. Positional order of siblings doesnt matter
- minheap => value of i >= value of parent
- maxheap = heapsort, minheap = priorityqueue
- maxheap = left(i) = 2*i + 1, right = 2*1 + 2, parent = Math.floor((i-1)/2)

# Remove Outer paranthesis - leetcode

````
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
````
# Reverse words in string - leetcode
````
class Solution:
   def reverseWords(self, s: str) -> str:
       s = s.strip()
       ls = re.split("\s+",s)
       ls = ls[::-1]
       return " ".join(ls)

````
# Largest odd number
````
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
````
# Longest Common Prefix
````
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
````
# Isomosrphic strings
````
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
````

# Rotate string
````
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
````
# Anagram
````
class Solution:
    def isAnagram(self, s: str, t: str) -> bool:
        s = sorted(s)
        t = sorted(t)

        if s == t:
            return True
        return False
````

# Sort chars by frequency
````
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
````
# Nesting depth paranthesis
````
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
````
# Roman to int

// i++ doesnt work in i in range
// use while and increment

````
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
````

# String to integer (Atoi)
````
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
````

# Longest palindromic substring
````
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
            
````
# Sum of beauty of all strings
````
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
````

# Add a node
````
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
````
# Delete a node
````
class Solution:
    def deleteNode(self, node):
        """
        :type node: ListNode
        :rtype: void Do not return anything, modify node in-place instead.
        """
        node.val = node.next.val
        node.next = node.next.next
````
# Length of linked list
````
class Solution:
    def getCount(self, head):
        # code here
        ans = 0
        while head != None:
            head = head.next
            ans += 1
        return ans

````
# Search in linkedlist
- O(n) - Linear
````
class Solution:
    def searchKey(self, head, key):
        #Code here
        while head != None:
            if head.data == key:
                return True
            head = head.next
        return False
````
# Insert in doubly linked list (DLL)
````
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
````
# Delete a node in DLLL
````
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
````
# Reverse a DLL (Tricky - Check)
````
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
````
# Middle of linked list - (Tortoise Hare)
````
class Solution:
    def middleNode(self, head: Optional[ListNode]) -> Optional[ListNode]:
        tor = head
        hare = head

        while hare != None and hare.next != None:
            hare = hare.next.next
            tor = tor.next
        return tor
````
# Reverse a LL
````
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
````
# Loop in LL
````
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
````
# Linked List Cycle 2
````
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
````
# Length of loop
- Find loop starting point, traverse again
````
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
````
# LL is a palindrome
- reverse first part of linkedlist till the middle and traverse again
````
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
````
     
# Odd Even Linked List
````
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

````
 
# Remove Nth Node from end of list
````
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
````
# Remove middle element
````
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
````
# Find intersection of two Linked lists
- Approach 1
- Find lengths of two lists, move largest list by the difference and compare
````
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
````
- Approach 2 (Trick)
- connect end of l1 to l2 and l2 to l1

````
class Solution:
    def getIntersectionNode(self, headA: ListNode, headB: ListNode) -> Optional[ListNode]:
        l1, l2 = headA, headB
        while l1 != l2:
            l1 = l1.next if l1 else headB
            l2 = l2.next if l2 else headA
        return l1
````
# Add two numbers in LL

# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
````
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
````
# Add 1 to a linked list
- Edge cases: 456, 459, 9, 999
````

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
````
# Delete all occurences of key
````
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
````
# Fund pairs with given sum in DLL
- Use set() for fast lookups
````
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
````
# Remove duplicates is sorted DLL
````
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
````

# Rotate Linked List
- Use mod to cancel unnecersary rotations
````
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
````

# Flatten a linked list
- links are sorted
- Use merge() in merge sort
````
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

````
     
# Copy list with Random pointer
- Use dict with none checks
````
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
````
# Pow(x,n)
- Power function
- O(n) - simple while on n, time limit exceeded
- O(logn)
- Recusrion
- 2^10 = 2^5 * 2^5 - No need to compute 2 ^ 10 fully
- 2^5 = 2 * 2^2 * 2^2
````
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
````

# Count Good Numbers
- Leetcode
- Time Limit Exceeded for below
````
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
`````

- Formula
- Combinations
- 5 possiblilites in even places (0,2,4,6,8) , 4 possibilities in odd places (2,3,5,7)
- n = 4, _ _ _ _ = (5 * 4 * 5 * 4)
- Still time limit exceeded for pow complexity (Inbuilt ** doesnt work)
````
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
````
# sort stack
- Using another stack
````
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
````
# Subsequences

# Generate all binary strings
- Using bin() method
- bin() output is a string
````
class Solution:
    def binstr(self, n):
        ans = []
        for i in range(2**n):
            a = bin(i)[2:]
            a = '0' * (n-len(a)) + a
            ans.append(a)
        return ans

- Using recursion
class Solution:
    def binstr(self, n):
        ans = []
        def gen(n,s):
            if n == 0:
                ans.append(s)
                return
            gen(n-1, s+'0')
            gen(n-1, s+'1')
            return
        gen(n,'')
        return ans
````
# generate paranthesis
- Brute
- Find all occurences and pick only well formed
````
class Solution:
    def generateParenthesis(self, n: int) -> List[str]:
        ans = []
        def gen(n,s):
            if n == 0:
                ans.append(s)
                return
            gen(n-1, s+'(')
            gen(n-1, s+')')
            return
        gen(2*n-1,'(')
        # print('ans',ans)
        def well(s):
            st = []
            for i in s:
                if i == '(':
                    st.append('(')
                else:
                    if len(st)==0:
                        return False
                    st.pop()
            if len(st) == 0:
                return True
            return False
        ans2 = []
        for i in ans:
            if well(i):
                ans2.append(i)
        return ans2
````
- Optimal (recursion)
````
class Solution:
    def generateParenthesis(self, n: int) -> List[str]:
        ans = []
        def gen(s,l,r):
            if len(s) == 2 *n:
                ans.append(s)
                return
            if l < n:
                gen(s+'(', l+1, r)
            if r < l:
                gen(s+')',l,r+1)
        gen('',0,0)
        return ans

````

# Subsets (powerset)
- Not so good
````
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        ans = []

        def gen(nums):
            if nums.copy() not in ans:
              ans.append(nums.copy())

            for i in range(len(nums)):
                if nums == []:
                    return
                tmp = nums.copy()
                nums.pop(i)
                gen(nums)
                nums = tmp
        gen(nums)
        return ans
````

- Optimal
````
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        res = []
        subset = []

        def dfs(i):
            if i >= len(nums):
                res.append(subset.copy())
                return
            
            # to include nums[i]
            subset.append(nums[i])
            dfs(i+1)

            # to skip nums[i]
            subset.pop()
            dfs(i+1)
        dfs(0)
        return res
````

# First occurence of needle in haystack
````
class Solution:
    def strStr(self, haystack: str, needle: str) -> int:
        for i in range(len(haystack)):
            if needle in haystack[i:i+(len(needle))]:
                return i
        return -1

````
# Dijkistra algroithm
- minheap priority queue
- greedy BFS, picks closest unvisited vertex
- cant handle negative edges
- neetocde : https://neetcode.io/problems/dijkstra/question
- Used by OSPF protocol. Every router runs dijkstra with src vertex as itself

````
class Solution:
    def shortestPath(self, n: int, edges: List[List[int]], src: int) -> Dict[int, int]:
        adj = {}

        for i in range(n):
            adj[i] = []
        
        for s, d, w in edges:
            adj[s].append([w,d])

        shortest = {}

        minheap = [[0,src]]

        while minheap:
            w1, n1 = heapq.heappop(minheap)
            if n1 not in shortest:
                shortest[n1] = w1
            
            for w2, n2 in adj[n1]:
                if n2 not in shortest:
                    heapq.heappush(minheap, [w2+w1, n2])
        
        for i in range(n):
            if i not in shortest:
                shortest[i] = -1
        return shortest
````

# Bellman Ford algorithm
- relaxes all edges repeatedly, gradually improves distance estimates
- can handle negative edges
- slower than dijkistra

- N vertices, Iterate N-1 times sequentially
- For N-1 logic, take 4 vertices with 1 as distance between them and use decresing order vertes edges
- Atmost it takes N-1. If there is a negative cycle, doing another iteration will still decresae the distance which would be wrong, hence do it once more and compare

````

class Solution:
    def bellmanFord(self, V, edges, src):
        #code here
        
        shortest = {}
        
        for i in range(V):
            shortest[i] = float('inf')
        
        shortest[src] = 0
        
        for i in range(V-1):
            for s,d,w in edges:
                if shortest[s]+w < shortest[d]:
                    shortest[d] = shortest[s]+w
                    
        nt = shortest.copy()
        for s,d,w in edges:
            if nt[s]+w < nt[d]:
                nt[d] = nt[s]+w
        
        if list(nt.values()) != list(shortest.values()):
            return [-1]

        # Needed in GFG problem statement
        for i in shortest:
            if shortest[i] == float('inf'):
                shortest[i] = 10**8
        return list(shortest.values())    
````

# Floyd-Warshal algorithm

- Best paths between all vertices
- Can handles negative weights
- Can detect negative cycles. If the dist between itself is less than 0, its a negative cycle. Path sum is less than 0, its negative cycle
- O(V**3) complexity
- Create 2d matrix, calculate distance between two vertices through each of the vertex (k)

````

class Solution:
	def floydWarshall(self, dist):
		#Code here
		n = len(dist)
# 		print(n)
		
# 		O(V**3)
		for k in range(n):
		    for j in range(n):
		        for i in range(n):
		            if dist[i][k] != 10**8 and dist[k][j] != 10**8 and dist[i][k] + dist[k][j] < dist[i][j]:
		                dist[i][j] = dist[i][k] + dist[k][j]
		return dist

````

# BFS

- Uses queue
- Goes by level

````

class Solution:
    def bfs(self, adj):
        # code here
        
        queue = []
        
        vis = [0] * len(adj)
        
        ans = []
        
        queue.append(0)
        
        vis[0] = 1
        
        while queue:
            ele = queue.pop(0)
            
            ans.append(ele)
            
            for i in adj[ele]:
                if not vis[i]:
                    queue.append(i)
                    vis[i] = 1
        
        return ans
````
# DFS

- Recusrions
- recursive dfs() function

````

class Solution:
    def rec_dfs(self,node, vis,ans,adj):
        vis[node] =1
        ans.append(node)
        
        for i in adj[node]:
            if not vis[i]:
                self.rec_dfs(i,vis,ans,adj)
    def dfs(self, adj):
        n = len(adj)
        vis = [0]*n
        vis[0] = 1
        ans = []
        self.rec_dfs(0,vis,ans,adj)
        
        return ans

````

# Number of provinces
- DFS on connected nodes

````
class Solution:
    def dfs(self,node, vis, myadj):
        vis[node] = 1
        
        for x in myadj[node]:
            if vis[x] != 1:
                self.dfs(x,vis,myadj)
        
        
    def numProvinces(self, adj, V):
        # code here 
        
        myadj = {}
        
        for i in range(V):
            myadj[i] = []
            
        for idx,x in enumerate(adj):
            for idx2,i in enumerate(x):
                if i == 1:
                    myadj[idx].append(idx2)
                    
        # print(myadj)
        
        vis = [0]*V
        
        ans = 0
        
        for i in range(V):
            if vis[i] != 1:
                self.dfs(i,vis,myadj)
                ans += 1
        
        return ans
````

# Number of islands | connected componenets
- Simple DFS of matrix

````

class Solution:
    def dfs(self, i,j, m, n, vis):
        vis[i][j] = -1
        
        for x in range(i-1,i+2):
            for y in range(j-1,j+2):
                if x>=0 and x<n and y>=0 and y<m:
                    # print("h1",x,y,vis[x][y])
                    if vis[x][y]==0:
                        # print("h2",x,y,vis[x][y])
                        self.dfs(x,y,m,n,vis)
        
    def numIslands(self, grid):
        # code here
        n = len(grid)
        m = len(grid[0])
        
        vis = [[-1 for _ in range(m)] for _ in range(n)]
        # print(vis)
        
        for i in range(n):
            for j in range(m):
                if grid[i][j]=='L':
                    vis[i][j]=0
        
        # print(vis)
        
        ans = 0
        
        for i in range(n):
            for j in range(m):
                if vis[i][j] == 0:
                    self.dfs(i,j,m,n,vis)
                    # print(i,j,vis)
                    ans += 1
        
        return ans
````

# Flood fill algorithm
- Little tricky
- DFS with edge case (infinite loop)
- If val is same as newColor, just return the image

When Does Infinite Loop Actually Occur?
The infinite loop happens when multiple connected cells have the same value:
Image:        sr=0, sc=0, newColor=1, val=1
[1, 1, 1]
[1, 1, 0]     All these 1s are connected!
[1, 0, 1]
Trace:

DFS at (0,0): set to 1, check neighbors
Neighbor (0,1) has value 1 → recurse to (0,1)
DFS at (0,1): set to 1, check neighbors
Neighbor (0,0) still has value 1 → recurse back to (0,0) ❌
Infinite loop between (0,0) ↔ (0,1)

````
class Solution:
    def dfs(self, newColor, val, i, j, n, m, image):
        image[i][j] = newColor
        
        # 4-directional neighbors
        directions = [(-1,0), (1,0), (0,-1), (0,1)]
        for dx, dy in directions:
            x, y = i + dx, j + dy
            if 0 <= x < n and 0 <= y < m and image[x][y] == val:
                self.dfs(newColor, val, x, y, n, m, image)
    
    def floodFill(self, image, sr, sc, newColor):
        n = len(image)
        m = len(image[0])
        val = image[sr][sc]
        
        # Avoid infinite recursion if colors are the same
        if val == newColor:
            return image
        
        self.dfs(newColor, val, sr, sc, n, m, image)
        return image
````


# Rotten Oranges
- Tricky BFS
- Add time as well in queue along with pos

````
	def orangesRot(self, mat):
		# code here
		
		queue = []
		
		n = len(mat)
		m = len(mat[0])
		
		for i in range(n):
		    for j in range(m):
		        if mat[i][j] == 2:
		            queue.append([i,j,0])

		ad = [[-1,0],[0,-1],[1,0],[0,1]]
		
		ans = 0
		            
		while queue:
		    i,j, t = queue.pop(0)
		    
		    ans = max(ans,t)
		    
		    for dx,dy in ad:
		        if i+dx >=0 and i+dx<n and j+dy>=0 and j+dy<m:
		            if mat[i+dx][j+dy] == 1:
		                queue.append([i+dx,j+dy,t+1])
		                mat[i+dx][j+dy]=2
		                
	    for i in range(n):
		     for j in range(m):
		         if mat[i][j] == 1:
		             return -1
	    return ans 
		                
````
# ZigZag Conversion (leetcode)
- Easy
- Create 2d matrix as same in below like for loop

````
class Solution:
    def convert(self, s: str, numRows: int) -> str:
        m = len(s)
        n = numRows

        if n == 1:
            return s

        mat = [["" for _ in range(m)] for _ in range(n)]

        i,j = 0,0

        dir = 0

        for let in s:
            mat[i][j]=let
            # print(i,j,mat[i][j])

            if i>=n-1:
                dir = -1
            if dir == 0:
                i+=1
            if i == 0:
                dir = 0
                i+=1
            if dir == -1:
                j+=1
                i-=1

        ans = ""
        for i in mat:
            for j in i:
                if j != "":
                    ans += j

        return ans
````

# Letter Combinatioon of phone number(leetcode)
- Backtrack
- Looks like complex combination probelm
- Small edge case at the end

````

class Solution:
    def letterCombinations(self, digits: str) -> List[str]:
        n = len(digits)

        m = {1:"", 2:"abc", 3:"def",4:"ghi",5:"jkl",6:"mno",7:"pqrs",8:"tuv",9:"wxyz"}

        res = []

        def backtrack(i, curStr):
            if len(curStr) == n:
                res.append(curStr)
                return
            
            for c in m[int(digits[i])]:
                backtrack(i+1, curStr + c)
        
        if digits:
            backtrack(0,"")
        
        return res

````
