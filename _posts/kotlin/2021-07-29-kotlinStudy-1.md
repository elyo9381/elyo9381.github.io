---
layout: post
title: kotlinStudy-1
subtitle: "study, java"
categories: book
tags: java
comments: true
---
> kotlin 스터디 



## 변수 및 자료형

  - 주석은 자바와 동일하다.
  - 세미콜론은 붙이지 않아도 된다. 
  - 클래스는 파스칼표기법
  - 변수, 함수는 카멜표기법을 지켜야한다.

  - var : 일반적인 변수이다.
  - val : 불변 변수이다.

  - var a : int 등의 방법으로 자료형을 표현해야한다.
  - println()함수 로 CLI로 출력 가능하다.


  - 코틀린은 무조건 변수에 초기값을 선언해야한다. 
  - var a? = null을 허용하는 변수가 선언된다.

  - 자료형은 자바와 거의 동일하다. 
  - 다른점은 String타입인데 """ 블라블라 """
  - """으로 감싸진 String은 특수문자까지 String으로 표현가능하다.

  - 코틀린에서는 프리미티브 타입이 Int 이런 형식으로 대문자로 시작한다. 



## 형변환과 배열

  - toLong(), toInteger() 등 명시적 형변환을 제공한다. 
  - 형변환을 위해서는 사용자가 직접 변경하여야한다. 
  

  - var intArr = arrOf(1,2,3,4,5) 
  - 위와같은 형식으로 배열을 선언할수있다. 

  - var nullArr = arrayOfNulls<Int>(5)
  - 위와같은 형식으로 배열을 길이를 제한하고 null로 채울수있다. 


## 타입추론과 함수 

  - 변수 var는 다양한 함수추론을 제공한다. 
  - 기본적인 int,string,double,char등 타입추론이 가능하다.


  - 함수작성은 자바와 비슷하다.
  - 인자를 받고 리턴값을 설정한다.
  ```
  fun add(a:Int, b:Int, c:Int): Int{
      return a+b+c;
  }
  ``` 

  위의 코드처럼 리턴값의 자료형은 함수선언부에 같이 작성 해야한다.

  그리고 코틀린에서 함수를 단일 표현식 함수를 허용한다. 
  ```
  fun add(a:Int,b:Int,c:Int) = a+b+c;
  ```

  위의코드는 단일표현식을 허용하기 때문에 선언부에서 함수의 리턴값의 정의를 표현할수있으며, 타입추론으로 인해 리턴타입이 생략된 코드이다. 

  




  
  