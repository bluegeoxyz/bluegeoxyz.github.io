---
title: Merge Sort
date: 2023-06-26
categories: [Programing]
tags: [algorithm, sort]
---

## 算法原理

通常，归并排序按照如下步骤工作：

1. 将需要排序的集合，拆分成 n 个子集合，直到每个子集合包含一个元素(一个元素的集合默认有序)
2. 重复合并子集合，直到只剩一个集合（即排序后的集合）

时间复杂度 *O(nlogn)*

![Merge Sort Example](/assets/img/posts/2023/06/merge-sort-example.gif)


## Top-down 实现


```java

    private static Comparable[] aux;

    public <T extends Comparable<T>> T[] sort(T[] unsorted) {
        aux = new Comparable[unsorted.length];
        doSort(unsorted, 0, unsorted.length - 1);
        return unsorted;
    }

    private static <T extends Comparable<T>> void doSort(T[] arr, int left, int right) {
        if (left < right) {
            int mid = (left + right) >>> 1;
            doSort(arr, left, mid);
            doSort(arr, mid + 1, right);
            merge(arr, left, mid, right);
        }
    }

    private static <T extends Comparable<T>> void merge(T[] arr, int left, int mid, int right) {
        // i: the first item in the left
        // j: the first item in the right
        int i = left, j = mid + 1;
        System.arraycopy(arr, left, aux, left, right + 1 - left);

        for (int k = left; k <= right; k++) {
            if (j > right) { // mean that the right has been sorted
                arr[k] = (T) aux[i++];
            } else if (i > mid) { // mean that the left has been sorted
                arr[k] = (T) aux[j++];
            } else if (less(aux[j], aux[i])) {
                arr[k] = (T) aux[j++];
            } else {
                arr[k] = (T) aux[i++];
            }
        }
    }

```

## Bottom-Top 实现


```java

    private void bottomTopMergeSort(int[] array) {
        int n = array.length;
        int[] temp = new int[array.length];
        for (int block = 1; block < n; block *= 2) {
            for (int start = 0; start < n - block; start += 2 * block) {
                int mid = start + block - 1;
                int end = Math.min(start + 2 * block - 1, n - 1);
                merge(array, start, mid, end, temp);
            }
        }
    }

    private void merge(int[] array, int start, int mid, int end, int[] temp) {
        int left = start;
        int right = mid + 1;
        int index = start;
        while (left <= mid && right <= end) {
            if (array[left] <= array[right]) {
                temp[index] = array[left];
                left++;
            } else {
                temp[index] = array[right];
                right++;
            }
            index++;
        }
        while (left <= mid) {
            temp[index] = array[left];
            left++;
            index++;
        }
        while (right <= end) {
            temp[index] = array[right];
            right++;
            index++;
        }
        if (end + 1 - start >= 0) {
            System.arraycopy(temp, start, array, start, end + 1 - start);
        }
    }

```