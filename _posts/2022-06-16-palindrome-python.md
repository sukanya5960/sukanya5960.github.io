---
layout: post
title: Python - make palindrome by replacing '?' in a string
author: Sukanya M
categories: [Programming]
tags: [Python, programming]
date: 2022-06-16 12:20:00 +0800
math: true
mermaid: true

---

This is a program in Python which returns palindrome which can be obtained by replacing all of the question marks in the given string str with 'Z'. If no palindrome can be obtained, the function will return the string "Not palindrome".

A palindrome is a string that reads the same both forwards and backwards. Some examples of palindromes are: "malayalam", "radar", "47874"

Sample output:

1. Given str = "?ab??a", the function should print "aabbaa"
2. Given str = "bab??a", the function should print "Not palindrome"
3. Given str = "?a?", the function should print "ZaZ"

Let's check how we can achieve this using python:

```
def printPalindrome(str):
    n = len(str)
    for i in range(n//2):
        if str[i] != str[n-i-1] and str[i] != '?' and str[n-i-1] != '?':
            print("Not palindrome")
            return False
    str = list(str)
    for i in range(n):
        if str[i] == '?':
            if str[n-i-1] == '?':
                str[i] = str[n-i-1] = 'Z'
            else:
                str[i] = str[n-i-1]
    str = "".join(str)
    print(str)

str = input("Enter the string: ")
printPalindrome(str)
```

That's it!
