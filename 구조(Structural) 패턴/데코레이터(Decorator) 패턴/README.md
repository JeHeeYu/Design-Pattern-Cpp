# 데코레이터 패턴 정리 내용

## 데코레이터(Decorator) 패턴 정리 내용
데코레이터 패턴은 객체들을 새로운 행동들을 포함하는 특수 래퍼 객체들 내에 넣어서 위 행동들들 객체들이 연결시키는 구조적 디자인 패턴이다.
<br>
예를 들어 동료가 작성한 클래스르 기반으로 어떤 기능을 확장해야 하는 상황이다.
<br>
원본 코드를 수정하지 않고 사용해야 하는 방법을 찾아야 한다.
<br>
<br>
가장 쉽게 생각나는 방법은 상속을 이용하는 것이다.
<br>
동료의 클래스를 부모로하는 자식 클래스를 만들어 거기에 새로운 기능을 추가하는 것이다.
<br>
몇몇 메서드들을 오버라이딩 한다면 원본 코드에 수정을 가하지 않고 작업을 진행할 수 있다.
<br>
<br>
하지만 항상 이렇게 사용할 수 있는 것은 아니다.
<br>
예를 들어 상속을 사용할 수 없는 상황이 있는데, std::vector의 경우 버추얼 소멸자가 없다는 제한이 있다.
<br>
<br>
무엇보다도 상속을 활용하기 어려운 가장 큰 이유는, 수정하는 이유가 여러 가지인 경우 단일 책임 원칙(SRP)에 따라 그 수정 사항들을 각각 완전히 분리하는 것이 바람직하기 때문이다.
<br>
<br>
즉, 데코레이터 패턴은 이미 존재하는 타입에 새로운 기능을 추가하면서도 원래 타입의 코드에 수정을 피할 수 있게 해준다.(열림-닫힘 원칙(OCP)이 준수됨)
<br>
여기에 더해서 파생해야 할 타입의 개수가 과도하게 늘어나는 것도 막을 수 있다.

<br>

## 시나리오
예를 들어 도형을 나타내는 클래스 Shape가 기존에 존재하고 있었고 이를 상속받아 색상이 있는 도형(ColoredShape)과 투명한 도형(TransparentShape)을 추가했다고 가정한다.
<br>
그리고 나중에 두 가지 특성을 모두 필요로 하는 경우가 발생하여 추가로 ColoredTransparentShape를 만들었다고 가정한다.
<br>
<br>
결과적으로 두 가지 기능을 추가하기 위해 클래스를 3개 만들었다.
<br>
이런 식이라면 기능이 하나 더 추가될 경우 7개의 클래스를 만들어야 할 수도 있다.
<br>
<br>
설상가상으로 Squre, Circle 등과 같은 도형의 파생 클래스들까지 적용해야 할 수도 있다.
<br>
<br>
세 가지의 추가 기능을 두 가지의 도형에 적용하려면 상속받아 만들어야 할 클래스가 14개까지 늘어난다.
<br>
이런 상황은 비효율적이며 감당할 수 없다.
<br>
<br>
이러한 코드를 다음과 같이 작성할 수 있다.
<br>
예를 들어 추상 클래스 Shape이 아래와 같이 정의되어 있다.

```
struct Shape
{
    virtual string str() const = 0;
};
```

이 클래스에서 str()은 버추얼 함수이고 특정 도형의 상태를 텍스트로 나타낸다.
<br>
이 클래스를 베이스로 하여 원, 사각형 같은 도형을 정의하고 str() 인터페이스를 구현할 수 있다.

```
struct Circle : Shape
{
    float radius;

    explicit Circle(const float radius)
        : radius{radius} {}

    void resize(float factor) { radius *= factor; }

    string str() const override
    {
        ostringstream oss;
        oss << "A Circle of radius " << radius;
        return oss.str();
    }
};

/ Sqaure 구현 생략
```

평범한 상속으로는 효율적으로 새로운 기능을 도형에 추가할 수가 없다는 것을 알 수 있다.
<br>
따라서 접근 방법을 달리해서 컴포지션을 활용한다.
<br>
<br>
컴포지션은 데코레이터 패턴에서 객체들에 새로운 기능을 확장할 때 활용되는 메커니즘이다.
<br>
이 접근 방식은 다시 두 가지 서로 다른 방식으로 나누어진다.

- <b>동적 컴포지션</b> : 참조를 주고받으면서 런타임에 동적으로 무언가를 합성할 수 있게 한다.<br>이 방식은 최대한의 유연성을 제공한다.<br>예를 들어 사용자 입력에 따라 런타임에 반응하여 컴포지션을 만들 수 있다.
- <b>정적 컴포지션</b> : 템플릿을 이용하여 컴파일 시점에 추가기능이 합성되게 한다.<br>이것은 코드 작성 시점에 객체에 대한 정확한 추가 기능 조합이 결정되어야만 한다는 것을 암시한다.<br>즉, 나중에 수정될 수 없다.

<br>

## 동적 데코레이터(Dynamic Decorator)
예를 들어 도형에 색을 입히려고 하는 상황이다.
<br>
상속 대신 컴포지션으로 ColoredShape를 만들 수 있다.
<br>
<br>
이미 생성된 Shape 객체의 참조를 가지고 새로운 기능을 추가한다.

```
struct ColoredShape : Shape
{
    Shape& shape;
    string color;

    ColoredShape(Shape& shape, const string& color)
        : shape{shape}, color{color} {}

    string str() const override
    {
        ostringstream oss;
        oss << shape.str() << " has the color " << color;
        return oss.str();
    }
};
```

위 코드에서 볼 수 있듯이 ColoredShape은 그 자체로서도 Shape이다.
<br>
ColoredShape는 다음과 같이 이용될 수 있다.

```
Circle circle{0.5f};
ColoredShape redCircle{circle, "red"};
cout << redCircle.str();
// 출력 결과 "A circle of radius 0.5 has the color red"
```

만약 여기에 더하여 도형이 투명도를 가지게 하고 싶다면 마찬가지 방법으로 다음과 같이 쉽게 구현할 수 있다.

```
struct TransparentShape : Shape
{
    Shape& shape;
    uint8_t transparency;
    
    TransparentShape(Shape& shape, const uint8_t transparency)
        : shape{shape}, transaprentcy{transparency} {}
        
    string str() const override
    {
        ostringstream oss;
        oss << shape.str() << " has " << static_cast<float>(transparency) / 255.f * 100.f << "% transparency";
        
        return oss.str();
    }
};
```

이렇게 하면 0 ~ 255 범위의 투명도를 지정하면 그것을 퍼센티지로 출력해주는 새로운 기능이 추가되었다.
<br>이러한 추가 기능은 그 자체만으로는 사용할 수 없고 적용할 도형 인스턴스가 있어야만 한다.

```
Square sqaure{3};
TransparentShape demiSquare{sqaure, 85};
cout << demiSquaure.str();
// 출력 결과 "A square with side 3 has 33.333% transparency"
```

하지만 편리하게도 ColoredShape와 TransparentShape를 합성하여 색상과 투명도 두 기능 모두 도형에 적용되도록 할 수 있다.

```
TransparentShape myCircle {
    ColoredShape { 
        Circle{23}, "green"
    }, 64
};

cout << myCircle.str();

// 출력 결과 A circle of radius 23 has the color green has 25.098% transparency"
```

위 코드에서 볼 수 잇듯이 모두 즉석에서 생성하고 있다.
<br>
하지만 이 코드는 문제도 있다. 
<br>
비상식적인 합성도 가능하다.
<br>
<br>
같은 데코레이터를 중복해서 적용해버릴 수도 있다.
<br>
<br>
예를 들어 ColoredShape{ColoredShape{...}}과 같은 합성은 비상식적이지만 동작한다.
<br>
이렇게 중복된 합성은 "빨간색이면서 노란색"이라는 모순된 상황을 야기한다.
<br.
<br>
OOP의 테크닉들을 활용하면 예로든 중복 합성이 방지되도록 할 수 있다.
<br>
하지만 또 아래와 같은 경우에서 문제가 발생한다.

```
ColoredShape{TransparentShape{ColoredShape{...}}}
```



이런 경우는 탐지해내가기 훨씬 어려우며, 만약 가능하다고 해도 투자 대비 효과를 따져봐야 한다.

<br>

## 정적 데코레이터(Static Decorator)

예제 시나리오에서 Circle에는 resize() 멤버 함수가 있으나 이 함수는 Shape 인터페이스와 관계가 없다.
<br>
그래서 resize() 함수는 Shape 인터페이스에 없기 때문에 데코레이터에서 호출할 수가 없다.
<br>
따라서 아래 코드는 컴파일이 안되는 오류가 발생한다.

```
Circle circle{3};
ColoredShae redCircle{circle, "red"};
redCircle.resize(2); // 컴파일 오류
```

런타임에 객체를 합성할 수 있는지는 별로 상관하지 않는다.
<br>
하지만 데코레이션된 객체의 멤버 함수와 필드에는 모두 접근할 수 있어야 한다.
<br>
<br>
그러기 위해서 템플릿과 상속을 활용하되 여기에서의 상속은 가능한 조합의 수가 폭발적으로 증가하지 않는다.
<br>
보통의 상속 대신 믹스인(MixIn) 상속이라 불리는 방식을 이용한다.
<br>
<br>
믹스인 상속은 템플릿 인자로 받은 클래스를 부모 클래스로 지정하는 방식을 말한다.
<br>
<br>
기본 아이디어는 다음과 같다.
<br>
새로운 클래스 ColoredShape를 만들고 템플릿 인자로 받은 클래스를 상속받게 한다.
<br>
이때 템플릿 파라미터를 제약할 방법은 없다.
<br>
즉, 어떤 타입이든 올 수 있다.
<br>
<br>
따라서 static_assert를 이용해 Shape 이외의 타입이 지정되는 것을 막는다.

```
template <typename T> struct ColoredShape : T
{
    static_assert(is_base_of<Shape, T>::value,
        "template argument must be a Shape");
        
    string color;
    
    string str() const override
    {
        ostringstream oss;
        oss << T::str() << " has the color " << color;
        
        return oss.str();
    }
}; // TransparentShape<T>의 구현 생략
```

ColoredShape와 TransparentShape의 구현을 기반으로 하여 색상이 있는 투명한 도형을 아래와 같이 합성할 수 있다.

```
ColoredShape<TransparentShape<Square>> square{"blue"};
square.size = 2;
square.transparentcy = 0.5;
cout << square.str();

// 이제 sqaure의 어떤 멤버든 접근 가능
sqaure.resize(3);
```

위 코드는 정상적으로 동작하지만 완벽한 코드는 아니다.
<br>
이유는 모든 생성자를 한 번에 편리하게 호출하던 부분을 잃어버렸기 때문이다.
<br>
<br>
기존 바깥의 클래스는 생성자로 초기화할 수 있지만 도형의 크기, 색상, 투명도까지 한 번에 설정할 수는 없다.

<br>
<br>

데코레이션을 완성하기 위해 ColoredShape와 Transparent에 생성자를 전달한다.
<br>
이 생성자들은 두 종류의 인자를 받는다.
<br>
<br>
첫 번째 인자들은 현재의 템플릿 클래스에 적용되는 것들이다.
<br>
두 번째 인자들은 부모 클래스에 전달될 제너릭 파라미터 팩이다.

```
template <typename T> struct TransparentShape : T
{
    uint8_t transparency;
    
    template <typename...Args>
    TransparentShape(const uint8_t transparency, Args ...args)
        T(std::forward<Args>(args)...)
        , transparency{ transparency } {}
    ...
}; // ColoredShape에서도 동일하게 구현
```

위의 생성자는 임의의 개수를 인자로 받을 수 있다.
<br>
앞쪽 인자는 투명도 값을 초기화하는 데 이용되고 나머지 인자들은 그 인자가 어떻게 구성되었느냐와 관계없이 단순히 상위 클래스에 전달된다.

<br>
<br>

생성자들에 전달되는 인자의 타입과 개수, 순서가 맞지 않으면 컴파일 에러가 발생하기 때문에 올바르게 맞춰질 수밖에 없다.
<br>
<br>
클래스 하나에 디폴트 생성자를 추가하면 파라미터의 설정에 훨씬 더 융통성이 생긴다.
<br>
하지만 인자 배분에 혼란과 모호성이 발생할 수도 있다.
<br>
<br>
이러한 생성자들에 explicit 지정자를 부여하지 않도록 주의해야 한다.
<br>
만약 그렇게 할 경우 복수의 데코레이터를 합성할 때 C++의 복제 리스트 초기화 규칙 위반 오류가 발생한다.
<br>
<br>
이러한 내용들을 아래와 같이 구현할 수 있다.

```
ColoredShape2<TransparentShape2<Square>> sq = { "red", 51, 5 };
cout << sq.str() << endl;
// 출력 결과 "A square with side 5 has 20% transparency has the color red"
```

이것으로 정적 데코레이터의 구현을 모두 다루었다.


<br>


## 함수형 데코레이터(Fuction Decorator)

데코레이터 패턴은 클래스를 적용 대상으로 하는 것이 보통이지만 함수에도 동등하게 적용될 수 있다.
<br>
<br>
예를 들어 코드에 문제를 일으키는 특정 동작이 있다고 가정한다.
<br>
그 동작이 수행될 때마다 모든 상황을 로그로 남겨 엑셀에 옮겨다가 상태를 분석하고 싶은 상황이다.
<br>
<br>
로그를 남기기 위해 아래와 같이 로그를 남긴다.

```
cout << "Entering function\n";

// 작업 수행...
cout << "Exiting function\n";
```

이러한 방식은 익숙하고 정상적으로 동작한다.
<br>
하지만 이해 관계를 분리하는 디자인 철학(separation of concerns) 관점에서는 바람직하지 않다.
<br>
<br>
로깅 기능을 분리하여 다른 부분과 얽히지 않고 재사용과 개선을 마음 편하게 할 수 있다면 훨씬 더 효과적이다.
<br>
<br>
이렇게 하는 데에는 여러 접근 방법이 있을 수 있다.
<br>
<br>
한 가지 방법은 문제의 동작과 관련된 전체 코드 단위를 통째로 어떤 로깅 컴포넌트에 람다로서 넘기는 것이다.
<br>
아래는 이러한 방법을 위한 로깅 컴포넌트의 정의이다.

```
struct Logger
{
    function<void()> func;
    string name;
    
    Logger(const function<void()>& func, const sring& name)
        : func{func}, name{name} {}
        
    void operator()() const
    {
        cout << "Entering " << name << endl;
        func();
        cout << "Exiting " << name << endl;
    }
};
```

이 로깅 컴포넌트는 다음과 같이 활용할 수 있다.

```
Logger([]() {cout << "Hello" << endl; }, "HelloFunction")();
// 출력 결과
// Entering HelloFunction
// Hello
// Exiting HelloFunction
```

코드 블록을 std::function으로서 전달하지 않고 템플릿 인자로 전달할 수도 있다.
<br>
이 경우 로딩 클래스가 다음과 같이 조금 달라진다.

```
template <typename Func>
struct Logger2
{
    Func func;
    string name;
    
    Logger2(const Func& func, const string& name)
        : func{func}, name{name} {}
        
    void operator() const
    {
        cout << "Entering " << name << endl;
        func();
        cout << "Exiting " << name << endl;
    }
};
```

이 로깅 컴포넌트의 사용법은 완전히 같다.
<br>
로깅 인스턴스를 생성하기 위해 다음과 같이 편의 함수를 만든다.

```
template <typename Func> auto make_logger2(Func func, const string& name)
{
    return Logger2<Func>{ func, name }; // () = call now
}
```

그리고 아래와 같이 활용한다.

```
auto call = make_logger2([]() {cout << "Hello!" << endl; }, "HelloFunction");
call();
```

이렇게 하면 데코레이터를 생성할 수 있는 기반이 마련되었다는 것에 의미가 있다.
<br>
즉, 임의의 코드 블록을 데코레이션 할 수 있고 데코레이션된 코드 블록을 필요할 때 호출할 수도 있다.
<br>
<br>
이제 좀 더 어려운 경우가 있다.
<br>
만약 로그를 남기고 싶은 함수의 리턴 값을 넘겨야 한다는 경우의 상황이다.
<br>
<br>
예를 들어 아래와 강티 add() 함수가 있다.

```
double add(double a, double b)
{
    cout << a << "+" << b << "=" << (a + b) << endl;
    return a + b;
}
```

이 함수에 로그를 남기면서 동시에 리턴 값도 넘겨야 하는 상황이다.
<br>
그냥 LOgger에서 값을 리턴하면 실제로 그렇게 되지는 않는다.
<br>
그래도 불가능하지는 않다.
<br>
<br>
Logger를 아래와 같이 변경한다.

```
template <typename R, typename... Args>
struct Logger3<R(Args...)>
{
    Loger3(function<R(Args...)> func, const string& name)
        : func{func}, name{name} {}
        
    R operator() (Args ...args)
    {
        cout << "Entering " << name << endl;
        R result = func(args...);
        cout << "Exiting " << name << endl;
        
        return result;
    }
    
    function<R(Args ...)> func;
    
    string name;
}
```

위 코드에서 템플릿 인자 R은 리턴 값의 타입을 의미한다.
<br>
그리고 Args는 파라미터 팩이다.
<br>
<br>
이 데코레이터는 함수를 가지고 있다가 필요할 때 호출할 수 있게 해준다.
<br>
유일하게 다른 점은 operator()가 R 타입의 리턴 값을 가진다는 점이다.
<br>
<br>
즉, 리턴 값을 잃지 않고 받아낼 수 있다.

<br>
<br>
데코레이터 생성 편의 함수를 이에 맞게 수정하면 아래와 같다.

```
template <typename R, typename... Args>
auto make_logger3(R (*func)(Args...), const string& name)
{
    return Logger3<R(Args...)>(
        std::function<R(Args...)>(func), name);
}
```

이 편의 함수는 한 가지 달라진 점이 있다.
<br>
std::function 대신에 일반 함수 포인터를 첫 번째 인자로 받고 있다.
<br>
<br>
이 편의 함수를 이용하면 다음과 같이 add()를 데코레이션하고 호출할 수 있다.

```
auto logger_add = make_logger3(add, "Add");
auto result = logged_add(2, 3);
```

그리고 make_logger3를 종속성 주입으로 대체할 수도 있다.
<br>
종속성 주입 방식을 이용하면 다음과 같은 장점이 생긴다.

- 실제 로깅 컴포넌트 대신 Null 객체를 전달하여 로깅 작업을 동적으로 끄거나 켤 수 있음
- 로깅할 코드의 실제 실행을 막을 수도 있음<br>이 부분도 로깅 컴포넌트를 다르게 제공함으로써 구현할 수 있음

<br>

## 장단점

### 장점
- 새 자식 클래스를 만들지 않고도 객체의 행동을 확장할 수 잉씀
- 런타임에 객체들에서부터 책임들을 추가하거나 제거할 수 있음
- 객체를 여러 데코레이터로 래핑하여 여러 행동들을 합성할 수 있음
- 다양한 행동들의 여러 변형들을 구현하는 모놀리식 클래스를 여러 개의 작은 클래스들로 나눌 수 있음


### 단점
- 래퍼들의 스택에서 특정 래퍼를 제거하기가 어려움
- 데코레이터의 행동이 데코레이터 스택 내의 순서에 의존하지 않는 방식으로 데코레이터를 구현하기가 어려움
- 계층들의 초기 설정 코드가 보기 흉할 수 있음


<br>


## 다른 패턴과의 관계

- 어댑터는 기존 객체의 인터페이스를 변경하는 반면 데코레이터는 객체를 해당 객체의 인터페이스를 변경하지 않고 향상함<br>또한 데코레이터는 어댑터를 사용할 때는 불가능한 재귀적 합성을 지원함
- 어댑터는 다른 인터페이스를, 프록시는 같은 인터페이스를, 데코레이터는 향상된 인터페이스를 래핑된 객체에 제공함
- 책임 연쇄 패턴과 데코레이터는 클래스 구조가 매우 유사함<br>두 패턴 모두 실행을 일련의 객체들을 통해 전달할 때 재귀적인 합성에 의존하나, 몇 가지 결정적인 차이점이 있음
- 데코레이터는 객체의 피부를 변경할 수 있고 전략 패턴은 객체의 내장을 변경할 수 있다고 비유할 수 있

<br>

## 요약
데코레이터 패턴은 열림-닫힘 원칙(OCP, Open-Close Principle)을 준수하면서도 클래스에 새로운 기능을 추가할 수 있게 해준다.
<br>
데코레이터의 핵심적인 틍징은 데코레이터들을 합성할 수 있다는 것이다.
<br>
<br>
객체 하나에 복수의 데코레이터들을 순서와 무관하게 적용할 수 있다.
<br>
<br>
데코레이터의 종류를 요약하면 다음과 같다.

- <b>동적 데코레이터</b> : 데코레이션할 객체의 참조를 저장하고 런타임에 동적으로 합성할 수 있음<br>대신 원본 객체가 가진 멤버들에 접근할 수 없음
- <b>정적 데코레이터</b> : 믹스인 상속(템플릿 파라미터를 통한 상속)을 이용해 컴파일 시점에 데코레이터를 합성함<br>이 방법은 런타임 융통성은 가질 수 없음<br>즉, 런타임에 객체를 다시 합성할 수 없으나 원본 객체의 멤버들에 접근할 수 있는 장점이 있음<br>그리고 생성자 포워딩을 통해 객체를 완전하게 초기화할 수 있음
- <b>함수형 데코레이터</b> : 코드 블록이나 특정 함수에 다른 동작을 덧씌워 합성할 수 있음

다중 상속이 지원되지 않는 프로그래밍 언어에서는 데코레이터 패턴을 이용해 여러 객체를 연관시켜 다형성을 흉내 낸다.
<br>
즉, 연관시킨 여러 객체의 인터페이스를 데코레이터로 합성하여 단일한 인터페이스 집합을 제공한다.








