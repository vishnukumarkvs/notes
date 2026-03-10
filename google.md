GFG Google
https://www.geeksforgeeks.org/interview-experiences/google-interview-questions/


3 Sum - Find all triplets with zero sum
Brute force



class Solution:
    def triplets(self, arr ):
        # code here
        n = len(arr)
        ans = set()
        for i in range(n):
            for j in range(i+1,n):
                for k in range(j+1,n):
                    if arr[i]+arr[j]+arr[k]==0:
                        ans.add(tuple(sorted((arr[i],arr[j],arr[k]))))
        return sorted(list(ans))
        

Optimal
