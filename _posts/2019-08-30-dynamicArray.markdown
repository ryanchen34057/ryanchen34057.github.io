---
layout:     post
title:      "Java Dynamic List實作原始碼"
subtitle:   ""
date:       2019-08-30
author:     "Ryan"
header-style: text
tags:
    - 資料結構
    - Java
---
```java
public class Array<T>{
    private T[] arr;
    private int len = 0;
    private int capacity = 0;

    public Array() { this(16);}
    public Array(int capacity) {
        this.capacity = capacity;
        this.arr = (T[]) new Object[capacity];
    }

    public int size() {
        return len;
    }

    public boolean isEmpty() {
        if(len == 0) return true;
        return false;
    }

    public T get(int index) {
        if(index < 0 || index >= len) throw new IndexOutOfBoundsException();
        return arr[index];
    }

    public void set(int index, T elem) {
        if(index < 0 || index >= len) throw new IndexOutOfBoundsException();
        arr[index] = elem;
    }

    public void clear() {
        for(int i=0;i<capacity;i++) {
            arr[i] = null;
        }
        len = 0;
    }

    public void add(T elem) {
        if(len == capacity) {
            T[] newArr = (T[]) new Object[capacity * 2];
            for(int i=0;i<len;i++) {
                newArr[i] = arr[i];
            }
            arr = newArr;
            capacity *= 2;
        }
        arr[len++] = elem;
    }

    public T removeAt(int index) {
        if(index < 0 || index >= len) throw new IndexOutOfBoundsException();
        T data = arr[index];
        int numMoved = len - index - 1;
        System.arraycopy(arr, index + 1, arr, index, numMoved);
        arr[--len] = null;
        return data;
    }

    public int indexOf(T elem) {
        for(int i=0;i<len;i++) {
            if(get(i) == elem) {
                return i;
            }
        }
        return -1;
    }

    public boolean remove(T elem) {
        int index = indexOf(elem);
        if(index == -1) return false;
        removeAt(index);
        return true;
    }

    public boolean contains(T elem) {
        if(indexOf(elem) == -1) return false;
        return true;
    }

    public java.util.Iterator<T> iterator() {
        return new Iterator<T>() {
            int index = 0;
            @Override
            public boolean hasNext() {
                return index < len;
            }

            @Override
            public T next() {
                return arr[index++];
            }
        };
    }

    public String toString() {
        StringBuilder builder = new StringBuilder("[");
        for(int i=0;i<len;i++) {
            if(i != len - 1) {
                builder.append(get(i).toString()).append(",");
            }
            else {
                builder.append(get(i).toString());
            }
        }
        builder.append("]");
        return builder.toString();
    }
}
```