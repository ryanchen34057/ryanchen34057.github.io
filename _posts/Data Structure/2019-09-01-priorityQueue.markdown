---
layout:     post
title: 用Java介紹Priority Queue
categories: [Data Structure]
description: some word here
keywords: Data Structure, Java, Heap, Priority Queue
---
## Priority Queue是啥?
Priority Queue(好長，以下簡稱PQ)是一個抽象的資料型態(Abstract Data Type)。
會有個Queue就是有點像Queue，但是跟Queue不同的地方在於，**Queue是FIFO，也就是說先進去的資料一定會先出來**，但是PQ則是**每個資料都有自己的優先順序!**

而PQ裡的資料的優先順序就決定了從PQ中移除的順序。

### PQ概念
>假設我們現在有一堆數字，而我們想要以由小到大的順序把資料取出，要怎麼取呢?

nums:
```java
14, 8, 4, 22, 1, 3
```

首先，我們會先所有數字去比大小，然後把最小的數找出來:

*Queue的取出叫做poll
```java
poll() -> 1 // nums: 14, 8, 4, 22, 3
```
如果再加入2呢?
```java
add(2) // nums: 14, 8, 4, 22, 3, 2
```
然後再poll一次的話就會是2:
```java
poll() -> 2 // nums: 14, 8, 4, 22, 3
```
```java
add(4) // nums: 14, 8, 4, 22, 3, 4
poll() -> 3 // nums: 14, 8, 4, 22, 4
add(5) // nums: 14, 8, 4, 22, 4, 5
add(9) // nums: 14, 8, 4, 22, 4, 5, 9
poll() -> 4 // nums: 14, 8, 22, 4, 5, 9
poll() -> 4 // nums: 14, 8, 22, 5, 9
poll() -> 5 // nums: 14, 8, 22, 9
poll() -> 8 // nums: 14, 22, 9
poll() -> 9 // nums: 14, 22
poll() -> 14 // nums: 8, 22
poll() -> 22 // nums: 
```
將取出的數字排出
```
1, 2, 3, 4, 4, 5, 8, 9, 14, 22
```
排出來剛好就是由小到大排序好的陣列了! 但是其實這只是巧合，取出來其實不一定會是按照順序的，不過最重要的原則就是**每次都是取出優先度最高的元素!**(我們這個例子來說優先度最高就是最小的數)

BUT，**要怎麼用最有效率的方式找出優先度最高(最大或最小)的元素就是一個問題了!!**

而找出最大或最小最好的方式就是用`heap`!!!([maxheap、minheap介紹](https://ryanchen34057.github.io/2019/09/01/heapify/))

藉由`maxheap`及`minheap`來快速找出最大或最小的值，然後讓優先度最高的一直保持在最上面(`index=0`)，要將資料取出時只要從最上面(`index=0`)取出就好囉!

以下為實作的原始碼:
```java
public class PriorityQueue<T extends Comparable<T>> {
    // heap裡目前的資料數量
    private int heapSize = 0;

    // 目前heap的容量
    private int heapCapacity = 0;

    // 一個動態的陣列，紀錄heap裡的資料
    private List<T> heap = null;

    // 建構一個空的heap
    public PriorityQueue() {
        this(1);
    }

    // 建構一個priority queue
    public PriorityQueue(int heapCapacity) {
        heap = new ArrayList<>(heapCapacity);
    }

    // 從陣列利用heapify建構一個priority queue
    public PriorityQueue(T[] elem) {
        heapSize = heapCapacity = elem.length;
        heap = new ArrayList<T>(heapCapacity);

        // 將所有元素放入heap裡
        for(int i=0;i<heapSize;i++) {
            heap.add(elem[i]);
        }

        // Heapify, O(n) (為什麼是n而不是nlog(n)的原因請參照:http://www.cs.umd.edu/~meesh/351/mount/lectures/lect14-heapsort-analysis-part.pdf)
        for(int i=Math.max(0, (heapSize/2))-1;i>=0;i--) {
            sink(i);
        }
    }

    // 建構Priority queue, O(nlog(n))
    public PriorityQueue(Collection<T> elems) {
        this(elems.size());
        for(T elem: elems) add(elem);
    }

    // 如果priority queue是空的回傳true，不是就false
    public boolean isEmpty() {
        return heapSize == 0;
    }

    // 將priority queue 清空
    public void clear() {
        for(int i=0;i<heapCapacity;i++) heap.set(i, null);
        heapSize = 0;
    }

    // 回傳heap的大小
    public int size() {
        return heapSize;
    }

    // 回傳priority queue裡有最低priority的元素(排在最前面的元素)
    public T peek() {
        if(isEmpty()) return null;
        return heap.get(0);
    }

    // 移除heap的root, O(log(n))
    public T poll() {
        return removeAt(0);
    }

    // 測試資料是否在heap裡，O(n)
    public boolean contains(T elem) {
        for(int i=0;i<heapSize;i++) {
            if(heap.get(i).equals(elem)) return true;
        }
        return false;
    }

    public void add(T elem) {
        if(elem == null) throw new IllegalArgumentException();

        if(heapSize < heapCapacity) {
            heap.set(heapSize, elem);
        }
        else {
            heap.add(elem);
            heapCapacity++;
        }

        //swim(heapSize);
        heapSize++;
        buildMinheap();
    }

    // 測試節點i的值是否小於節點j的值
    // 假設i,j都是合理的索引(0<=index<length), O(1)
    private boolean less(int i, int j) {
        T node1 = heap.get(i);
        T node2 = heap.get(j);
        return node1.compareTo(node2) <= 0;
    }

    private void buildMinheap() {
        for(int i=heapSize/2-1;i>=0;i--) {
            minheapify(i);
        }
    }

    private void minheapify(int root) {
        int left = 2 * root + 1;
        int right = 2 * root + 2;
        int minNode;
        if(left < heapSize && less(left, root)) minNode = left;
        else {
            minNode = root;
        }
        if(right < heapSize && less(right, minNode)) minNode = right;

        if(minNode != root) {
            swap(root, minNode);
            minheapify(minNode);
        }
    }

    // 執行由下到上的節點swim，O(log(n))
    private void swim(int k) {
        // 找出父節點的索引, *從0開始要減1!
        int parent = (k - 1) / 2;
        while(k > 0 && less(k, parent)) {
            swap(parent, k);

            k = parent;
            parent = (k - 1) / 2;
        }
    }

    // 執行由上到下的節點sink, O(log(n))
    private void sink(int k) {
        while(true) {
            int left = 2 * k + 1;
            int right = 2 * k + 2;
            int smallest = left; // 先假設最小的是left

            // 比較左邊小還是右邊小
            if(right < heapSize && less(right, smallest)) smallest = right;

            // 如果超出heap的範圍或是根(k)比較小，就跳出迴圈
            if(left >= heapSize || less(k, smallest)) break;

            // 跟著最小的節點往下走
            swap(smallest, k);
            k = smallest;
        }
    }

    // 從一個特定的索引移除node
    private T removeAt(int index) {
        if(isEmpty()) return null;

        heapSize--;
        T removedData = heap.get(index);
        swap(index, heapSize);

        // 將欲刪除的值設為null
        heap.set(heapSize, null);

        // 確認是否只剩一個element i = heapSize
        if(index == heapSize) return removedData;
        T elem = heap.get(index);

        // 開始將heap調整為minheap
        sink(index);

        // 如果sink沒有起效用就再swim
        if(heap.get(index).equals(elem)) {
            swim(index);
        }
        return removedData;
    }

    // 移除heap裡某個特定的元素, O(n)
    public boolean remove(T elem) {
        if(elem == null) return false;
        // 找出elem
        for(int i=0;i<heapSize;i++) {
            if(elem.equals(heap.get(i))) {
                removeAt(i);
                return true;
            }
        }
        return false;
    }

    private void swap(int index1, int index2) {
        T elem1 = heap.get(index1);
        T elem2 = heap.get(index2);

        heap.set(index1, elem2);
        heap.set(index2, elem1);
    }

    // 用遞迴確認是否是minheap
    // 這個方法只是用來測試用，看看heap在add或remove之後有沒有維持minheap的invariant
    // 用k=0呼叫此方法(等於從root開始)
    public boolean isMinheap(int k) {
        // 如果k超出範圍回傳true
        if(k >= heapSize) return true;

        int leftChild = 2 * k + 1;
        int rightChild = 2 * k + 2;
        if(leftChild < heapSize && !less(k, leftChild)) return false;
        if(rightChild < heapSize && !less(k, rightChild)) return false;
        return isMinheap(leftChild) && isMinheap(rightChild);
    }

    @Override
    public String toString() {
        return heap.toString();
    }
}
```

