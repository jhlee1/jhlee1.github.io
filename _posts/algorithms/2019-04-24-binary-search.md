---
layout: post
title: "Binary Search"
date: 2019-04-24 13:09:29 +0900
tags: ["algorithm", "binary_search"]
categories: [algorithms]
author: Joohan Lee
---

- Recursive

```java
public class Solution {
    public static boolean binarySearchRecursive(int[] array, int x, int left, int right) {
        if (left > right) {
            return false;
        }

        int mid = left + ((right - left) / 2); // To avoid integer overflow used instead of (left + right) / 2

        if(array[mid] == x) {
	        return true;
    	} else if (x < array[mid]) {
	        return binarySearchRecursive(array, x, left, mid - 1);
    	} else {
        	return binarySearchRecursive(array, x, mid + 1, right);
    	}
    }

    public static boolean binarySaerchRecursive(int[] array, int x) {
        return binarySaerchRecursive(array, x, 0, array.length - 1);
    }
}
```

- Iterative

```java
public class Solution {
    public static boolean binarySearchIterative(int[] array, int x) {
        if (left > right) {
            return false;
        }

        int left = 0;
        int right = array.length - 1;

        while (left <= right) {
        	int mid = left + ((right - left) / 2); // To avoid integer overflow used instead of (left + right) / 2

        	if(array[mid] == x) {
		        return true;
	    	} else if (x < array[mid]) {
		        right = mid - 1;
    		} else {
        		left = mid + 1;
    		}            
        }

        return false;
    }

    public static boolean binarySaerchRecursive(int[] array, int x) {
        return binarySaerchRecursive(array, x, 0, array.length - 1);
    }
}
```
