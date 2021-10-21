---
published: true
title: C++ 구조적 바인딩(structured bindings)
---
## structured bindings란
Structured bindings는 C++17에 도입된 기능이다. 이걸 한글로 바꾸면 '구조적 바인딩' 정도가 될 것 같다. 비슷한 말로는 destructuring, unpack, decomposition 등이 있을 것 같다. 나는 언팩이란 단어가 제일 짧아 자주 사용한다.

> Binds the specified names to subobjects or elements of the initializer.
>
> -- [Structured binding declaration - cppreference](https://en.cppreference.com/w/cpp/language/structured_binding/)

structured binding은 여러 변수에 subobject 또는 initializer의 elements를 할당할 수 있게 해주는 기능이다.

```javascript
// in javascript
let a, b, rest;
[a, b, ...rest] = [10, 20, 30, 40, 50];

console.log(a);	 // 10
console.log(b);	 // 20
console.log(c);	 // Array [30, 40, 50]
```
javascript의 destructuring assignment(구조분해 할당)와 어느정도 비슷하다.

> attr(optional) cv-auto ref-qualifier(optional) [ identifier-list ] = expression ; (1)
>
> attr(optional) cv-auto ref-qualifier(optional) [ identifier-list ] { expression } ; (2)
>
> attr(optional) cv-auto ref-qualifier(optional) [ identifier-list ] ( expression ) ; (3)

선언 방법이 3가지인데, ref-qualifier가 없는 경우, 즉 `auto`인 경우 (1)은 복사, (2, 3)은 direct initialization이다.

`const` 등의 attribute도 옵셔널하게 붙일 수 있다.

`cv-auto`라고 되어 있는 점이 중요한데, [cv(const and volatile) type qualifiers](https://en.cppreference.com/w/cpp/language/cv) 페이지를 참고하자.

```cpp
int a[2] = {1,2};
 
auto [x,y] = a; // creates e[2], copies a into e, then x refers to e[0], y refers to e[1]
auto& [xr, yr] = a; // xr refers to a[0], yr refers to a[1]
```
(1)의 예시이다. 레퍼런스 언팩도 가능한 점을 알아두자.


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
C++17부터 structured bindings를 사용할 수 있다.

```cpp
std::tuple<int, std::string, float> f();

auto [a, b, c] = f();
```
auto와 대괄호를 이용해 tuple을 분해할 수 있다.

```cpp
#include <iostream>
struct S {
    mutable int x1;
    volatile double y1;
};
S f() { return S{1, 2.3}; }
 
int main() {
    const auto [x, y] = f();  // x is an int lvalue
                              // y is a const volatile double lvalue
    std::cout << x << ' ' << y << '\n';  // 1 2.3
    x = -2;   // OK
//  y = -2.;  // Error: y is const-qualified
    std::cout << x << ' ' << y << '\n';  // -2 2.3
}
```
레퍼런스의 예제. struct의 멤버를 이런식으로 분해할 수 있다. x는 int lvalue, y는 const volatile double lvalue로 선언되었다.


## 아쉬운 점들

### structured bindings + std::ignore?

```cpp
tuple<T1, T2, T3> f();

auto [x, std::ignore, z] = f(); // NOT proposed: ignore second element
```

거두절미하면 structured binding과 `std::ignore`는 함께 쓸 수 없다. [structured bindings proposal에도 명시가 되어 있다고 한다.](https://stackoverflow.com/a/40714311/4295499)

```cpp
// method (1)
auto [x, dummydummy, z] = f();

// method (2)
[[maybe_unused]] auto [x, dummydummy, z] = f();
```

(1)번 방식처럼 더미 변수를 만드는 게 최선이겠는데, 이 경우 `-Wunused-variable` warning이 나온다.

그래서 (2)처럼 `[[maybe_unused]]`를 붙이자니, 더미가 아닌 다른 변수들이 사용되지 않더라도 `unused-variable` 경고가 무시되는 부작용이 생긴다.

결국 `std::ignore`는 `std::tie`밖에 쓸 수 없다. :(


### 여러 attribute, reference 함께 쓸 수 없을까?

```cpp
auto [& x, const y, const& z] = f(); // NOT proposed
```

이렇게 x는 `auto`, y는 `const auto`, z는 `const auto& z`로 쓰고싶다 라고 했을 때, 위처럼 쓰고 싶겠지만 못한다. 역시나 proposal에 명시되어 있다고 한다.

```cpp
auto val = f(); // or auto&&

T1& x = get<0>(val);
T2 const y = get<1>(val);
T3 const& z = get<2>(val);
```

답은 `std::get`을 쓰는건데, `val`이라는 변수가 남아있는 단점이 있다.

아니면 애초에 f()가 `tuple<int&, const int, const int&>`를 반환하게 하는 방법도 있을 것 같지만, 만족스럽지는 않다.
