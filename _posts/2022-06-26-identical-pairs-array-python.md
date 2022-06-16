---
layout: post
title: Python - find number of identical pairs in an array
author: Sukanya M
categories: [Programming]
tags: [Python, programming]
date: 2022-06-16 20:20:00 +0800
math: true
mermaid: true

---

Let's see how to find number of pairs of elements of the array that are equal but that occuppy different positions in the array. That means, a pair of indices (P,Q) is called identical if 0 <= P < Q and array element A[P] = A[Q]. The goal is to calculate the number of identical pairs of indices.

For example, consider array A such that:

```
A[0] = 3
A[1] = 5
a[2] = 6
A[3] = 3
A[4] = 3
A[5] = 5
```
There are four pairs of identical indices: (0,3), (0,4), (1,5) and (3,4). Note that pairs (2,2) and (5,1) are not counted since their first indices are not smaller than the second.

So our function will return the number of identical indices pair which is 4 in this example. Let's convert this to Python program.

```
def matchFinder(num):
    count = 0
    for i in range(len(num)):
        for j in range(i+1, len(num)):
            # print("this is j", j)
            if i == j:
                continue
            else:
                if num[i] == num[j]:
                    print(("matched index is "), i, "and",
                          j, (" and matched number is "), num[i])
                    count = count + 1
    return count


num = input("Enter the Array of numbers separated by space: \n")
num = num.split()
print(matchFinder(num))
```

That's it!
