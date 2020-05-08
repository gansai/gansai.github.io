---
layout: post
title: LeetCode Problem - First Bad Version
published: true
disqus_identifier: LeetCode Problem - First Bad Version
comments: true
---

You are a product manager and currently leading a team to develop a new product. Unfortunately, the latest version of your product fails the quality check. Since each version is developed based on the previous version, all the versions after a bad version are also bad.

Suppose you have `n` versions `[1, 2, ..., n]` and you want to find out the first bad one, which causes all the following ones to be bad.

You are given an API `bool isBadVersion(version)` which will return whether `version` is bad. Implement a function to find the first bad version. You should minimize the number of calls to the API.

**Example:**

```
Given n = 5, and version = 4 is the first bad version.

call isBadVersion(3) -> false
call isBadVersion(5) -> true
call isBadVersion(4) -> true

Then 4 is the first bad version. 
```



### Brute-Force Approach

One basic approach which I tried, is by iterating over elements one-by-one and return the index, where first occurrence of bad version is reached. For example, following is the code:



​    
```java
/* The isBadVersion API is defined in the parent class VersionControl.
   boolean isBadVersion(int version); */

public class Solution extends VersionControl {
    public int firstBadVersion(int n) {
        int i = 1;
 		while(i <= n){
            if(isBadVersion(i)){
                return i;
            }
        }   
        return n;
    }
}
```



The problem with above code is that, we are checking each element. And this will be slow for handling large number of elements. 

### Binary Search Approach

If you actually see the brute force approach, we are iterating over some elements unnecessarily.

The fact of the matter is: beyond some element, all elements are bad. before that element, all elements are good. So, why should you iterate over all good elements?

Is there a way to remove iteration of good elements or atleast reduce visits to good elements?

If we see, from one perspective, it is similar to Binary Search, where we are searching for a particular element. And at each iteration, we can discard half of the remaining problem space.

Binary Search helps us to reduce problem space by 1/2.




        
```java
/* The isBadVersion API is defined in the parent class VersionControl.
   boolean isBadVersion(int version); */

public class Solution extends VersionControl {
    public int firstBadVersion(int n) {
    int left = 0;
    int right = n;
    int mid = 0;
    int ans = -1;
    
    while(left <= right){
        mid = left + (right-left)/2;
        
        if(isBadVersion(mid)){
            ans = mid;
            
            right = mid - 1;
        }
        else{
            left = mid + 1;
        }
    }
    
    return ans;
    
}
}
```
