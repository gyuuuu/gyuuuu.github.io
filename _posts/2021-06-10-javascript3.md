---
title:  "JS-Immutable"
excerpt: "자바스크립트에서 불변객체를 다루는 방법"

categories:
  - javascript
tags:
  - javascript
  - study
last_modified_at: 2021-06-20T22:10:00-00:00
---

## Introduction

이전 [포스팅](https://gyuuuu.github.io/javascript/javascript2/)에서 자바스크립트에서 데이터를 어떻게 다루는 지 살펴보았다. 이어서 자바스크립트에서 불변 객체를 다루는 방법에 대해 알아보자.

## 불변값

불변값이라는 것은 무엇일까? 흔히 상수와 헷갈리기도 하는데 둘은 엄연히 다르므로 구분해야한다.
상수는 변수와 반대되는 개념으로 변수 영역의 메모리가 변경될 가능성이 없어야한다. 한 번 데이터 할당이 이뤄진 변수 공간에 다른 데이터를 재할당 할수 없어야 한다는 말과 동일하다.

불변하다는 것은 데이터 값 자체가 변하지 않는다는 말이다.

```javascript
var a = 'abc';
a = a + 'def';
```

위에서 변수 a에 문자열 'abc'를 할당했다가 뒤에 'def'를 추가하면 기존의 'abc가 'abcdef'로 바뀌는 것이 아니라 새로운 문자열 'abcdef'가 만들어져 변수 a에 저장된다. 이것이 바로 불변하다는 성질이다. 'abc'라는 문자열을 다른 값으로 변경할 수 없고, 'abc'와 'abcdef'는 완전히 별개의 데이터가 된다. 

## 변수 복사 비교

앞선 [포스팅](https://gyuuuu.github.io/javascript/javascript2/)에서 Primitive type은 불변하고 Reference type은 가변하다고 말했다. 이런 차이가 어떤 식으로 나타나고, 나중에 어떤 문제를 발생시킬 수 있을지 살펴보기 위해 변수를 복사할 때 어떤 일이 일어나는 지 알아보도록 하겠다.

```javascript
var a = 10;
var b = a;

var obj1 = { c : 10, d : 'ddd' };
var obj2 = obj1;
```

![immutable](/assets/images/immutable1.PNG)

- 기본형(1~2라인)
1. 변수 영역에 빈 공간 @1001을 확보하고 식별자 a로 지정한다.
2. 숫자 10을 데이터 영역에서 검색하고 없으므로 빈 공간 @5001에 저장하고, 이 주소를 @1001에 넣는다.
3.  변수 영역의 빈 공간 @1002를 확보하고 식별자 b로 지정한다.
4.  식별자 a를 검색해 @1001에 저장된 값인@5001을 @1002에 대입한다.

- 참조형(3~4라인)
1.  변수 영역의 빈 공간 @1003을 확보해 obj1로 지정한다.
2. 데이터 영역의 빈 공간 @5002를 확보하고, 데이터 그룹이 담겨야 하기 때문에 별도의 영역 @7103~ 을확보해 주소를 지정한다.
3. @7103에는 c라고, @7104에는 d라고 식별자를 지정한다.
4.  c에 대입할 10을 데이터 영역에서 검색하고, 이미 @5001에 저장되어 있으므로 이 주소를 @7103에 저장한다.
5. 문자열 'ddd'는 데이터 영역의 빈 공간에 새로 만들어서 @7104에 저장한다.
6.  변수 영역의 빈 공간 @1004를 확보하고, 식별자를 obj2라고 지정한다.
7. 식별자 obj1을 검색해(@1003) 그 값인 @5002를 @1004에 대입한다.

변수를 복사하는 과정은 기본형 데이터와 참조형 데이터 모두 같은 주소를 바라보게 되는 점에서 동일하다고 볼 수 있다. 하지만 데이터 할당 과정에서 이미 차이가 있기 때문에 변수 복사 이후의 동작에도 큰 차이가 발생한다.

## 변수 복사 이후 값 변경

```javascript
var a = 10;
var b = a;
var obj1 = { c : 10, d : 'ddd' };
var obj2 = obj1;

b = 15;
obj2.c = 20;
```

4번째 라인까지는 위에서 확인한 내용이고, 이후에 변수의 값을 변경해 보았다. 기본형, 참조형 각각에서 어떤 일이 발생할까?

![immutable2](/assets/images/immutable2.PNG)

- 기본형
  1. 데이터 영역에 15가 없으므로 새로운 공간 @5004에 저장한다.
  2. @5004의 값을 @1002에 저장한다.

- 참조형
  1. 데이터 영역에 20이 없으므로 새로운 공간 @5005에 저장한다.
  2. @5005를 든 채로 변수 영역에서 obj2를 찾고(@1004), obj2의 값인 @5002가 가리키는 변수 영역에서 다시 c를 찾아(@7103) 그곳에 @5005를 대입한다.

기본형 데이터를 복사한 변수 b의 값을 바꿨더니 @1002의 값이 달라진 반면, 참조형 데이터를 복사한 변수 obj2의 프로퍼티의 값을 바꾸었더디 @1004의 값은 달라지지 않았다. 즉, 변수 a와 b는 서로 다른 주소를 바라보게 되었으나, 변수 obj1과 obj2는 여전히 같은 객체를 바라보고 있게 된다.

## 불변 객체

앞서서 참조형 변수는 복사되는 과정에서 객체의 값이 변할 수 있다는 점을 살펴보았다. 이제는 변하지 않는 불변 객체를 만들어 볼탠데 우선 왜 불변 객체를 만들어야하는 것일까? 어떤 상황에서 불변객체가 필요한 것일까?

```javascript
var user = {
  name: 'Gyu',
  gneder: 'male'
};

var changeName = function(user, newName){
  var newUser = user;
  newUser.name = newName;
  return newUser;
};

var user2 = changeName(user, 'Lee');

console.log(user.name, user2.name); 
console.log(user === user2);
```

맨 아래 console.log의 값으로 어떤게 찍힐까?

```
Gyu, Lee
false
 ```

가 찍히는 것이 아마 changeName이라는 함수를 작성한 의도일 것이다. 하지만 결과는

```
Lee, Lee
true 
```
이다.

왜냐하면 changeName 내부에서 user의 값이 복사되었고, user는 참조 변수이기 때문에 복사된 newUser도 결과적으로 user와 같은 값을 가리키게 된다. 
그렇기 때문에 newUser의 값을 바꾸더라고 user의 값도 같이 바뀌게 되는 것이다. 
이는 분명 의도한 changeName이 의도한 결과는 아닐 것이고, 특정 상황에서는 심각한 문제를 발생시킬 수도 있다. 

```javascript
var user = {
  name: 'Gyu',
  gneder: 'male'
};

var changeName = function(user, newName){
  return {
    name: newName,
    gender: user.gender
  }
};

var user2 = changeName(user, 'Lee');

console.log(user.name, user2.name);   // Gyu, Lee
console.log(user === user2);            // false
```

간단한 해결 방법으로 위처럼 새로운 객체를 리턴할 수 있다. user와 user2는 이제 서로 다른 객체이므로 안전하게 비교가능하다. 하지만 위 방법은 그닥 좋은 방법이라고 생각되지는 않는다.
왜냐하면 객체의 기존 프로퍼티를 하드코딩했기 떄문에 객체의 프로퍼티가 많아 질수록 함수는 복잡하고, 매번 고생을 해야하기 떄문이다.

이런 방식보다는 객체를 복사할 때 모든 프로퍼티를 복사해주는 함수를 만드는 것이 좋을 것 같다.
```javascript
var copyObject = function(target){
  var result = {};
  for(var prop in target){
    result[prop] = target[prop];
  }
  return result;
};
```

이런 식으로 객체의 모든 프로퍼티를 복사해 준다면 안전하게 비교 가능하고, user를 불변 객체라고 볼 수 있다. 하지만 여기에도 아직 부족한 점이 있다. 만약 복사하려는 객체가 중첩된 객체라면 어떻게 될까?
즉, 

```javascript
var user = {
  name: 'Gyu',
  urls: {
    github: 'https://github.com/gyuuuu',
    blog: 'https://gyuuuu.github.io/'
  }
}
```

이런식으로 중첩된 객체라면 아까 발생한 문제가 다시 발생할 수 있다. 이는 위의 copyObject가 얕은 복사를 하기 때문이다. 다시 말해 user에서 urls를 복사할 때 urls의 프로퍼티들까지 복사하는 것이 아니라 그 **주소**만 복사하기 때문이다. 따라서 urls의 사본을 바꾸면 원본도 바뀌게 되는 것이고, 원본을 바꾸면 사본도 바뀌게 되는 것이다.

이를 해결하기 위해서는 깊은 복사를 수행하는 copyObjectDeep를 만들어 주면된다.

```javascript
var copyObjectDeep = function(target){
  var result = {};
  if(typeof target === 'object' && target !== null){
    for(var prop in target){
      result[prop] = copyObjectDeep(target[prop]);
    }
  } else {
    result = target;
  }
  return result;
};
```

3번째 줄에서 target이 객체인 경우에는 내부 프로퍼티들을 순회하며 copyObjectDeep 함수를 재귀적으로 호출하고, 객체가 아닌 경우에는 8번째 줄에서 target을구대로 지정하도록 하였다.
이 함수를 이용하여 객체를 복사한다면, 원본과 사본이 완전히 다른 객체를 참조하게 되어 어느 쪽의 프로퍼티를 변경하더라도 다른 쪽에 영향을 주지 않게된다.

## END

저번 포스팅과 이번 포스팅을 통해서 자바스크립트가 데이터를 어떻게 처리하는지 메모리 중심으로 살펴보고, 불변 객체를 만들기 위해선 어떻게 해야하는 지 살펴보았다.
평소에 막연하게만 생각하던 것들을 깊이 파고들어 로우레벨에서 살펴보니 자바스크립트에 대해 좀 더 이해할 수 있었고, 예전에 가지고 있던 모호함들이 해결되었다.
다음에는 **실행 컨텍스트**에 관련된 내용을 공부해보고 공유해보려고 한다.

## Reference

정재남(2020), 코어 자바스크립트, 위키북스, pp.15-29