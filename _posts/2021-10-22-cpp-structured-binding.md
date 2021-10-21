---
published: true
title: C++ 구조적 바인딩(structured bindings)
---
## structured bindings란
Structured bindings는 C++17에 도입된 기능이다. 이걸 한글로 바꾸면 '구조적 바인딩' 정도가 될 것 같다.

> Binds the specified names to subobjects or elements of the initializer.
>
> -- [Structured binding declaration - cppreference](https://en.cppreference.com/w/cpp/language/structured_binding/)

structured binding은 여러 변수에 subobject 또는 elements of 

```javascript
// in javascript
let a, b, rest;
[a, b, ...rest] = [10, 20, 30, 40, 50];

console.log(a);	 // 10
console.log(b);	 // 20
console.log(c);	 // Array [30, 40, 50]
```
이 기능은 javascript의 `destructuring assignment`와 비슷하다.


## Before C++17 (ex. C++11)
C++17 이전에 structured bindings가 도입되기 전에는 `std::tuple`에서 값을 뽑아내는 것이 좀 귀찮았다. 

```cpp
#include <tuple>

auto t = std::make_tuple(1, "abc", 2.0f);

auto _a = std::get<0>(t);  // 1
auto _b = std::get<1>(t);  // "abc"
auto _c = std::get<2>(t);  // 2.0f
```

[`std::get`](https://en.cppreference.com/w/cpp/utility/tuple/get)을 사용해야 했기 때문인데 매우 귀찮다.

```cpp
int _a;
std::string _b;
float _c;

std::tie(_a, _b, _c) = t;
```

그래서 대안이 [`std::tie`](https://en.cppreference.com/w/cpp/utility/tuple/tie)이다. lvalue 레퍼런스들의 튜플을 만드는 함수이다.

`std::tie`는 `std::ignore`랑 연계해서 쓸 때 상당히 유용한데, 그 외엔 솔직히 잘 모르겠다. 안에 들어가는 값을 미리 선언해놔야하는 점도 조금 번거롭다.


## After C++17


## structured bindings + std::ignore?

>
>
>

거두절미하면 structured binding과 std::ignore
