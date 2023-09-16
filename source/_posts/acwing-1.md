---
title: 算法题常用模板
date: 2023-07-24 16:45:49
tags: [算法, acwing, 模板]
---

> 模板来自acwing
>
>

快速排序

``` c++

#include<iostream>
using namespace std;

const int N = 1e6 + 10;
int n;
int q[N];

void quick_sort(int *q, int l, int r){
    if(l >= r) return;
    //确定分界
    int x = q[l], i = l - 1, j = r + 1;
    //调整区间
    while(i < j){
        do i++; while(q[i] < x);
        do j--; while(q[j] > x);
        if(i < j) swap(q[i], q[j]);
    }
    //递归处理左右两段
    quick_sort(q, l, j);
    quick_sort(q, j + 1, r);
}

int main(){
    scanf("%d", &n);
    //遇到输入输出字符比较多的时候，尽量使用scanf和printf而非cin和cout
    for(int i = 0;i < n;i++) scanf("%d", &q[i]); 
    quick_sort(q, 0, n-1);
    for(int i = 0;i < n;i++) printf("%d ", q[i]);
    return 0;
}

```


归并排序

``` c++
#include<iostream>
using namespace std;

const int N = 1e6+10;
int n;
int q[N], tmp[N];

void merge_sort(int *q, int l, int r){
    if(l >= r) return;

    //确定边界
    int mid = l + r >> 1;
    
    //递归划分
    merge_sort(q, l, mid);
    merge_sort(q, mid+1, r);

    //归并
    int k = 0, i = l, j = mid + 1;
    while(i <= mid && j <= r){
        if(q[i] <= q[j]) tmp[k++] = q[i++];
        else tmp[k++] = q[j++];
    }
    while(i <= mid) tmp[k++] = q[i++];
    while(j <= r) tmp[k++] = q[j++];

    //更新结果
    for(i = l, j = 0;i <= r;i++, j++) q[i] = tmp[j];
}

int main(){
    scanf("%d", &n);
    for(int i = 0;i < n; ++i) scanf("%d", &q[i]);
    merge_sort(q, 0, n-1);
    for(int i = 0;i < n; ++i) printf("%d ", q[i]);
    return 0;
}

```


二分查找

``` c++

int bsearch_1(int l ,int r){
    while(l < r){
        //必须+1否则会陷入死循环
        int mid = l + r + 1 >> 1;
        if(check(mid)) l = mid;
        else r = mid - 1;
    }
}

int bsearch_2(int l, int r){
    while(l < r){
        int mid = l + r >> 1;
        if(check(mid)) r = mid;
        else l = mid + 1;
    }
}

```

高精度

