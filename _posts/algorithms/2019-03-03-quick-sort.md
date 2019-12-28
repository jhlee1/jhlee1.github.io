---
layout: post
title: "Quick Sort"
date: 2019-03-03 13:09:29 +0900
tags: ["algorithm", "quicksort"]
categories: [algorithms]
author: Joohan Lee
---

# Quick Sort

```java
public static void main(String[] args) {
    int[] arr = new int[];
    
    quickSort(arr, 0, arr.length - 1)
    System.out.println(arr);
}

public static void quickSort(int[] arr, int left, int right) {
    if(left >= right) {
        return;
    }
    
    int pivot = (left + right) / 2;
    int index = partition(arr, left, right, pivot)

    quickSort(arr, left, index - 1);
    quickSort(arr, index, right);
}

public static int partition(int[] arr, int left, int right, int pivot) {
    while(left <= right) {
        while(arr[left] < pivot) {
            left++;
        }
        
        while(arr[right] > pivot) {
            right--;
        }

        if(left <= right) {
            swap(arr, left, right);
            left++;
            right--;
        }
    }

    return left
}

```
