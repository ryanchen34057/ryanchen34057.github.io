---
layout:     post
title: Java LinkedList實作 
categories: [Data Structure]
description: some word here
keywords: Data Structure, Java, LinkedList
---
```java
public class LinkedList<T> {
    private int size = 0;
    private Node<T> head = null;
    private Node<T> tail = null;

    private class Node<T> {
        T data;
        Node<T> prev, next;
        public Node(T data, Node<T> prev, Node<T> next) {
            this.data = data;
            this.prev = prev;
            this.next = next;
        }

        @Override
        public String toString() {
            return data.toString();
        }
    }

    public void clear() {
        Node<T> trav = head;
        while(trav != null) {
            Node<T> next = trav.next;
            trav.prev = trav.next = null;
            trav.data = null;
            trav = next;
        }
        head = tail = trav = null;
        size = 0;
    }

    public int size() {
        return size;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    public void addLast(T elem) {
        if(isEmpty()) {
            head = tail = new Node<>(elem, null, null);
            size++;
            return;
        }
        Node<T> newNode = new Node<>(elem, tail, null);
        tail.next = newNode;
        tail = newNode;
        size++;
    }

    public void add(T elem) {
        addLast(elem);
    }

    public void addFirst(T elem) {
        Node<T> newNode = new Node<>(elem, null, head);
        head.prev = newNode;
        head = newNode;
    }

    public T peekFirst() {
        return head.data;
    }

    public void removeFirst() {
        head = head.next;
        head.prev = null;
        size--;
    }

    public void removeLast() {
        tail = tail.prev;
        tail.next = null;
        size--;
    }

    public void removeAt(int index) {
        if(index < 0 || index >= size) {
            throw new RuntimeException();
        }
        if(index == 0) {
            removeFirst();
            return;
        }
        if(index == size - 1) {
            removeLast();
            return;
        }
        Node<T> trav = head;
        for(int i=0;i<index-1;i++) {
            trav = trav.next;
        }
        trav.next = trav.next.next;
        trav.next.prev = trav;
        size--;
    }

    public int indexOf(T elem) {
        Node<T> trav = head;
        int index = 0;
        while(trav != null) {
            if(trav.data == elem) {
                return index;
            }
            index++;
            trav = trav.next;
        }
        return -1;
    }

    public boolean remove(T elem) {
        int index = indexOf(elem);
        if(index == -1) {
            return false;
        }
        removeAt(index);
        return true;
    }

    public boolean contains(T elem) {
        if(indexOf(elem) == -1) return false;
        return true;
    }

    public java.util.Iterator<T> iterator() {
        return new java.util.Iterator<T>() {
            Node<T> trav = head;
            @Override
            public boolean hasNext() {
                return trav != null;
            }

            @Override
            public T next() {
                T data = trav.data;
                trav = trav.next;
                return data;
            }
        };
    }

    @Override
    public String toString() {
        StringBuilder builder = new StringBuilder();
        Node<T> trav = head;
        while(trav != null) {
            if(trav.next != null) {
                builder.append(trav.data.toString()).append(" -> ");
            }
            else {
                builder.append(trav.data.toString());
            }
            trav = trav.next;
        }
        return builder.toString();
    }
}
```