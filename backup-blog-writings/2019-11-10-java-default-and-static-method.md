---
layout: post
title:  "[Java] Java 인터페이스의 Default 메서드"
date:   2019-09-03
tags: [Dev]
---

### Objectives

Java 인터페이스의 Default 메서드에 대해 알아봅니다.

### Default 메서드

Java 8이 등장하면서 인터페이스에 대한 정의가 몇 가지 변경되었습니다.

```java
public interface Calculator {
  public int plus(int i, int j);
  public int multiple(int i, int j);
  default int exec(int i, int j) { //default로 선언함으로써 메서드를 구현할 수 있다.
    return i + j;
  }
}

//Calculator인터페이스를 구현한 MyCalculator클래스
public class MyCalculator implements Calculator {

  @Override
  public int plus(int i, int j) {
    return i + j;
  }

  @Override
  public int multiple(int i, int j) {
    return i * j;
  }
}

public class MyCalculatorExam {
  public static void main(String[] args){
    Calculator cal = new MyCalculator();
    int value = cal.exec(5, 10);
    System.out.println(value);
  }
}
```

### 궁금한 점
1. abstract class vs. interface...
2. interface with default method vs. normal class --> 이건 아마 다중 상속을 염두에 둔 것일 듯.


### Reference

1. [인터페이스의 default method][1]

[1]:https://programmers.co.kr/learn/courses/5/lessons/241
