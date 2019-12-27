---
title: "Bubble Sort"
date: 2019-04-22T13:09:29+09:00
archives: "2019"
tags: ["algorithm", "bubblesort", "hackerrank"]
author: Joohan Lee
---

## Bubble Sort

```java
public class Solution {
	public static void bubblesort(int[] array) {
		boolean isSorted = false;
		int lastUnsorted = array.length - 1;

		while (!isSorted) {
			isSorted = true;
			for(int i = 0; i < lastUnsorted; i++) {
				if(array[i] > array[i+1]) {
					int tmp = array[i];
					array[i] = array[i+1];
					array[i+1] = tmp;
					lastSorted = false;
				}
			}
			lastUnsorted--;
		}
	}
}
```
