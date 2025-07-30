## This document gives some key points about algorithms using Python

When we slice a list in python, it creates a new list and copies elements one by one from the original list into the new one. This results in O(n) time complexity !  

We can avoid this by using numpy arrays if we work with arrays and not lists since slicing numpy arrays will only create views to point on the range of the array !  

