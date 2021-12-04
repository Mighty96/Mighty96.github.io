---
title: "큐로 스택, 스택으로 큐 만들기"

categories:
  - TIL
tags:
  - JAVA
  - Stack
  - Queue
toc: true
toc_sticky: true
---

# 개요

평소 두개의 큐로 스택을 구현한다던가, 두개의 스택으로 큐를 구현하는 문제를 종종 접했지만, 그냥 적당히 머릿속으로만 생각하고 지나갔다.  
다른 공부를 하다가 잠깐 머리를 식힐 겸 한번 직접 구현해보면 어떨까 했다.

아직 자바의 컬렉션에는 익숙하지 않아서 실제 반환타입 등에는 차이가 있을 수 있다.

# Queue

하나는 인풋용, 하나는 아웃풋 용으로 사용해서 새로운 원소가 들어올땐 모두 `inputStack`으로 옮긴 뒤에 집어넣고, 원소를 꺼내거나 읽을 때에는 `outputStack`으로 옮긴 뒤에 꺼내거나 읽었다.


```java
class myQueue<E> {
    Stack<E> inputStack = new Stack<>();
    Stack<E> outputStack = new Stack<>();

    public void add(E element) {
        outputToInput();
        inputStack.add(element);
    }

    void remove() {
        inputToOutput();
        outputStack.pop();
    }

    E peek() {
        inputToOutput();
        return outputStack.peek();
    }

    E poll() {
        E num = peek();
        remove();
        return num;
    }

    void inputToOutput() {
        while (!inputStack.isEmpty()) {
            outputStack.add(inputStack.peek());
            inputStack.pop();
        }
    }

    void outputToInput() {
        while (!outputStack.isEmpty()) {
            inputStack.add(outputStack.peek());
            outputStack.pop();
        }
    }

    int size() {
        return inputStack.size() + outputStack.size();
    }
    boolean isEmpty() {
        return inputStack.isEmpty() && outputStack.isEmpty();
    }
}
```

# Stack

두개의 큐를 같은 조건으로 사용했다. 원소를 집어 넣을 때에는 이미 원소가 존재하는 큐에 집어넣고, 빼거나 읽을 때에는 하나의 원소만 제외하고 나머지를 다른 큐로 옮긴 뒤에 남은 하나의 원소를 뺴거나 읽었다.


```java
class myStack<E> {
    private final Queue<E> que1 = new LinkedList<>();
    private final Queue<E> que2 = new LinkedList<>();

    void add(E element) {
        if (que1.isEmpty()) {
            que2.add(element);
        } else {
            que1.add(element);
        }
    }

    E peek() {
        E element;

        if (!que1.isEmpty()) {
            while (que1.size() != 1) {
                que2.add(que1.poll());
            }
            element = que1.peek();
            que2.add(que1.poll());
        } else {
            while (que2.size() != 1) {
                que1.add(que2.poll());
            }
            element = que2.peek();
            que1.add(que2.poll());
        }

        return element;
    }

    void pop() {
        if (!que1.isEmpty()) {
            while (que1.size() != 1) {
                que2.add(que1.poll());
            }
            que1.remove();
        } else {
            while (que2.size() != 1) {
                que1.add(que2.poll());
            }
            que2.remove();
        }
    }

    int size() {
        return que1.size() + que2.size();
    }

    boolean isEmpty() {
        return que1.isEmpty() && que2.isEmpty();
    }
}
```