# 방문자 패턴 정리 내용

## 방문자(Visitor) 패턴
방문자 패턴은 알고리즘들을 그들이 작동하는 객체들로부터 분리할 수 있도록 하는 행동 디자인 패턴이다.
<br>
<br>
예를 들어, 어떤 계층을 이루는 클래스들을 사용할 때, 그 소스 코드를 수정할 수 없다면 각 계층마다 메서드를 추가하는 것이 불가능하다.
<br>
이 문제를 해결하기 위해서는 선제적으로 어떤 준비가 되어야 있어야만 한다.
<br>
<br>
방문자 패턴은 바로 이러한 목적으로 만들어졌다.
<br>
<br>
예를 들어 아래와 같이 어떤 수학 수식을 파싱해야 한다고 가정한다.
<br>
이 수식은 double 타입 숫자와 덧셈 연산자로 구성된다.

```
(1.0 + (2.0 + 3.0))
```

위와 같은 수식은 아래와 같이 계층적인 형태로 표현될 수 있다.

```
struct Expression
{
    // 아직까지 아무것도 없음
};


struct DoubleExpression : Expression
{
    double value;
    explicit DoubleExpression(const double value)
        : value{value} {}
};

struct AdditionExpression : Expression
{
    Expression *left, *right;
    
    AdditionExpression(Expression* const left, Expression* const right)
        : left{left}, right{right} {}
   
    ~AdditionExpression()
    {
        delete left;
        delete right;
    }
};
```

이러한 객체의 계층이 주어져 있을 때, 만약 여러 종류의 수식을 지원할 수 있도록 상속 계층마다 새로운 동작을 추가해야 한다고 가정한다.

<br>

## 침습적 방문자

먼저 가장 직접적인 접근 방법인데, 이미 작성된 코드의 Expression 인터페이스를 직접적으로 수정하는 방법이다.
<br>
이 방법은 열림-닫힘 원칙(OCP)에 위배된다.
<br>
상속 관계에 따라 모든 하위 클래수도 수정된다.

```
struct Expression
{
    virtual void print(ostringstream& oss) = 0;
};
```

이러한 변경을 위해 AdditionExpression과 DoubleExpression에 print() 메서드를 구현한다.

```
struct AdditionExpression : Expression
{
    Expression *left, *right;
    ...
    void print(ostringstream& oss) override
    {
        oss << "(";
        left->print(oss);
        oss << "+";
        right->print(oss);
        oss << ")";
    }
};
```

하위 수식에 대한 print()의 호출은 재귀적이면서 다형적이다.
<br>
아래와 같이 테스트해볼 수 있다.

```
auto e = new AdditionExpression{
    new DoubleExpression{1},
    new AdditionExpression{
        new DoubleExpression{2},
        new DoubleExpression{3}
        }
};

ostringstream oss;
e->print(oss);
cout << oss.str() << endl;
// 출력 결과: (1+(2+3))
```

여기까지는 간단한데, 만약 클래스 계층이 10개에 이른다면 범위가 너무 커져버린다.
<br>
여기에 또 만약 eval() 연산을 추가해야 한다면, 10개의 서로 다른 클래스 각가겡 10가지의 수정이 가해져야 한다.
<br>
이 방식은 너무나도 비효율적이다.
<br>
<br>
실질적인 문제는 단일 책임 원칙(SRP)에 있다.
<br>
위 코드에 있듯 출력 가능은 특별한 주의를 필요로 한다.
<br>
<br>
모든 Expression마다 스스로 자신을 print() 하게 하는 대신 출력 방법을 알고 있는 ExpressionPrinter를 별도로 도입하는 방법을 생각해 볼 수 있다.
<br>
이렇게 하면 Expression의 전체 상속 계층에 영향을 주지 않고도 목적하는 것을 할 수 있다.

<br>

## 반추적(Reflective) 출력

이제 별도의 출력 컴포넌트를 만드는 접근 방법을 사용해 본다.
<br>
각 클래스에서 메서드 print()를 제거한다.(베이스 클래스에는 남겨둠)
<br>
<br>
그런데 한 가지 까다로운 부분이 있다.
<br>
Expression 클래스를 공백으로 남겨둘 수가 없다.
<br>
왜냐하면 다형성을 활용하려면 베이스 클래스에 무엇이 되었든 버추얼 속성을 가진 메서드가 있어야 한다.
<br>
<br>
따라서 버추얼 소멸자를 두어 다형성을 가질 수 있도록 한다.

```
struct Expression
{
    virtual ~Expression() = default;
};
```

이제 ExpressionPrinter를 구현해야 하는데, 먼저 아래와 같은 아이디어가 먼저 떠오를 것이다.

```
struct ExpressionPrinter
{
    void print(DoubleExpression *de, ostringstream& oss) const
    {
        oss << de->value;
    }
    
    void print(AdditionExpression *ae, ostringstream& oss) const
    {
        oss << "(";
        print(ae->left, oss);
        oss << "+";
        print(ae->right, oss);
        oss << ")";
    }
};
```

이 코드의 가장 큰 문제는 컴파일이 안된다.
<br>
컴파일 시점에는 ae->left의 타입이 Expression이라는 사실만 알고 있다.
<br>
<br>
다른 동적 프로그래밍 언어들과는 달리 C++는 런타임에 타입 체크를 하여 오버로딩 하는 방식이 아니라 컴파일 시점에 오버로딩이 결정되기 때문에 두 개의 print() 중 어느 것이 선택되어야 하는지 알지 못한다.
<br>
<br>
유일한 해결책은 오버로딩을 버리고 런타임 타입 체크를 명시적으로 구현해 넣는 것이다.
<br>
타입이 무엇인지 되돌아보기 때문에 반추적 방법이라고 한다.

```
struct ExpressionPrinter
{
    void print(Expression* e)
    {
        if(auto de = dynamic_cast<DoubleExpression*>(e))
        {
            oss << de->value;
        }
        else if(auto ae = dynamic_cast<AdditionExpression*>(e))
        {
            oss << "(";
            print(ae->left, oss);
            oss << "+";
            print(ae->right, oss);
            oss << ")";
        }
    }
    
    string str() const { return oss.str(); }
    
private:
    ostringstream oss;
};
```


코드가 지저분하더라도 꽤 실용적인 구현이다.

```
auto e = new AdditionExpression{
    new DoubleExpression{ 1 },
    new AdditionExpression{
        new DoubleExpression{ 2 },
        new DoubleExpression{ 3 }
    }
};

ExpressionPrinter ep;
ep.print(e);
cout << ep.str() << endl; // prints "(1+(2+3))"
```

이러한 접근 방법은 큰 단점이 있다.
<br>
모든 클래스 계층마다 구현해 넣은 print() 메서드에 대한 타입 체크를 포기해야 한다.
<br>
<br>
이로 인해 새로운 항목을 추가헀을 때 ExpressionPrinter에 필요한 수정을 하지 않더라도 그냥 컴파일이 되어 버린다.
<br>
그리고 런타임 타입을 체크하는 if 체인에 새로운 타입이 맻이되지 않아 그냥 무시되어 버린다.
<br>
<br>
하지만 이 방식은 분명 쓸만한 방문자 패턴의 구현 방법이다.

<br>

## 디스패치(Dispatch)
방문자 패턴에서 항상 '디스패치' 라는 용어가 언급된다.
<br>
디스패치란 주어진 정보 하에서 호출할 함수를 어떻게 특정하느냐의 문제이다.
<br>
<br>
즉, 일을 처리할 담당자를 찾아서 전달하는 작업이다.
<br>
<br>
아래는 예제이다.

```
struct Stuff {}
struct Foo : Stuff {}
struct Bar : Stuff {}

void func(Foo* foo) {}
void func(Bar* bar) {}
```

위와 같은 코드에서 Foo 객체를 만들어서 func() 함수의 인자로 넘긴다면 func() 함수의 호출에 아무런 문제가 없다.
<br>
타입 정보를 기반으로 컴파일러가 오버로딩을 해준다.

```
Foo *foo = new Foo;
func(foo); // ok
```

하지만 Foo의 포인터를 그 상위 클래스인 Stuff으로 업캐스팅하여 func()의 인자로 사용한다면 컴파일러가 오버로딩할 함수를 찾지 못한다.

```
Stuff* stuff = new Foo;
func(stuff); // 호출할 함수를 특정할 수 없음
```

그래도 런타임에 명시적으로 타입체크를 하지 않고서도 올바르게 오버로딩할 방법이 있다.
<br>
<br>
Stuff에 대해서 어떤 함수를 호출할 때 그 함수는 다형성을 가질 수 있고 필요한 컴포넌트로 바로 디스패치될 수 있다.
<br>
즉, 필요한 부분이 오버로딩이 일어나게 할 수 있다.
<br>
<br>
이러한 방식을 '이중 디스패치' 라고 한다.
<br>
그 이유는 두 가지로 다음과 같다.
1. 먼저 실제 객체에 다형성 호출
2. 다형성 호출이 수행되는 객체 안에서는 오버로딩을 일으킬 수 있는데, 왜냐하면 this가 특정 타입을 가지기 때문임

다음 코드가 있다.

```
struct Stuff
{
    virtual void call() = 0;
};

struct Foo : Stuff
{
    void call() override { func(this); }
};

struct Bar : Stuff
{
    void call() override { func(this); }
};

void func(Foo* foo) {}
void func(Bar* bar) {}
```

Stuff에 정의된 범용 인터페이스 call()에 하나에만 의존할 수 없다.
<br>
서로 다른 구현이 해당 클래스 안에 존재하여 this 포인터로 적합하게 타입이 지정되어야 한다.
<br>
<br>
이러한 구현으로 다음과 같은 코드가 가능해진다.

```
Stuff *stuff = new Foo;
stuff->call();  // func(stuff)이 올바르게 호출됨
```

<br>

## 전통적인 방문자
방문자 디자인 패턴의 전통적인 구현 방식은 이중 디스패치를 이용한다.
<br>
이중 디스패치를 구현할 때는 아래와 같이 호출될 방문자 메서드에 대한 네이밍 관례를 따른다.
- 방문자의 메서드는 visit() 라는 이름을 가짐
- 클래스 계층마다 구현될 메서드는 accept() 라는 이름을 가짐

이제는 베이스 클래스 Expression에 버추얼 소멸자를 일부러 두지 않아도 된다.
<br>
왜냐하면 실질적으로 필요한 버추얼 메서드가 생겼기 때문이다.
<br>
<br>
아래와 같이 베이스 클래스에 버추얼 멤버 함수 accept()를 둔다.

```
struct Expression
{
    virtual void accept(ExpressionVisitor *visitor) = 0;
};
```

위 코드에서 볼 수 있듯이 추상 클래스 ExpressionVisitor를 참조하고 있다.
<br>
ExpressionVisitor는 여러 가지 방문자들의 베이스 역할을 한다.
<br>
<br>
이제 Expression을 상속받는 모든 클래스는 accept() 메서드를 동일한 방식으로 구현해야 한다.
<br>
즉, 아래와 같다.

```
void accept(ExpressionVisitor* visitor) override
{
    this->visit(this);
}
```

그리고 ExpressionVisitor는 아래와 같이 정의할 수 있다.

```
struct ExpressionVisitor
{
    virtual void visit(DoubleExpression* de) = 0;
    virtual void visit(AdditionExpression* ae) = 0;
};
```

퓨어 버추얼 메서드기 때문에 이를 상속받는 모든 객체에서 오버로딩 구현을 하도록 강제된다.
<br>
구현하지 않을 경우 그에 연관된 accept()에서 컴파일 에러가 발생한다.
<br>
<br>
이제 아래와 같이 이 클래스를 상속받아 ExpressionPrinter를 정의할 수 있다.

```
struct ExpressionPrinter : ExpressionVisitor
{
    ostringstream oss;
    string str() const { return oss.str(); }
    void visit(DoubleExpression* de) override;
    void visit(AdditionExpression* ae) override;
};
```


### 방문자 추가하기

이러한 접근 방법의 장점은 전체 클래스 계층에 걸쳐서 accept() 메서드를 단 한 번만 구현하면 된다는 것이 장점이다.
<br>
해당 클래스 계층에서 accept()를 두 번 다시 손댈 일이 없다.
<br>
<br>
예를 들어 수식의 계산 결과를 구하는 기능을 추가하기 위해 다음과 같이 새로운 방문자를 정의했다고 가정한다.

```
struct ExpressionEvaluator : ExpressionVisitor
{
    double result;
    void visit(DoubleExpression* de) override;
    vois visit(AdditionExpression* ae) override;
};
```

그런데 visit()의 리턴 값이 void로 선언되었기 때문에 어색한 구현이 된다.
<br>
즉, 다음과 같은 형태가 된다.


```
void ExpressionEvaluator::visit(DoubleExpression* de)
{
    result = de->value;
}

void ExpressionEvaluator::visit(AdditionExpression* ae)
{
    ae->left->accept(this);
    auto temp = result;
    ae->right->accept(this);
    result += temp;
}
```

이와 같은 다소 난해한 구현은 accept()로부터 리턴값을 받을 수 없기 때문에 발생하는 현상이다.
<br>
기본적으로 왼쪽 부분을 계산하여 결과를 저장해 둔 다음, 오른쪽 부분을 계산하고 저장된 값만큼 더한다.
<br>
그러면 전체 합계가 계산된다.
<br>
<br>
이러한 코드는 직관적인 것과는 거리가 멀다.
<br>
하지만 목적하는 동작을 하기는 한다.

```
auto e = new AdditionExpression{ // 앞서 동일 };
Expressionprinter printer;
ExpressionEvaluator evaluator;
printer.visit(e);
evaluator.visit(e);
cout << printer.str() << " = " << evaluator.result << endl;
// 출력 "(1+(2+3)) = 6"
```

이와 비슷한 방식으로 열림-닫힘 원칙(OCP)을 준수하면서 서로 다른 방문자들을 얼마든지 추가할 수 있다.

<br>

## 비순환 방문자
방문자 디자인 패턴은 다음과 같이 두 가지 유형으로 나눌 수 있다.

- 순환 방문자 : 함수 오버로딩에 기반하는 방문자로 클래스 계층과 방문자 간에 상호 참조하는 순환적인 종속성이 발생함<br>이 때문에 순환 방문자 패턴이라 <br>이러한 순환 종속성 때문에 클래스 계층의 안전성이 보장되는 경우에만 <br>계층이 너무 자주 업데이트 된다면 문제가 발생할 수 있음
- 비순환 방문자 : 런타임 정보 타입에 의존하는 방문자로 이 방법의 장점은 방문될 클래스 <br>하지만 성능적인 부분에서는 약간의 손해가 있음

비순환 방문자를 구현하는 첫 번째 단계는 방문자의 인터페이스를 정의하는 것이다.
<br>
방문한 클래스 계층의 각 타입마다 visit() 메서드를 오버로딩하는 대신 아래와 같이 최대한 범용적인 형태로 방문자 인터페이스를 정의한다.

```
template <typename Visitable>
struct Visitor
{
    virtual void visit(Visitable& obj) = 0;
};
```

도메인 모델의 모든 항목마다 이러한 방문자를 수용(Accept)할 수 있어야 한다.
<br>
모든 실 구현 타입이 고유하기 때문에 버추얼 속성이 적용되도록 아래와 같이 버추얼 소멸자만을 가진 공백 클래스를 베이스로 둔다.
<br>
<br>
이렇게 동작에 대한 정의는 없지만 해당 인터페이스임을 표시하는 클래스를 마커(Marker) 인터페이스라고 한다.

```
struct VisitorBase // 마커 인터페이스
{
    virtual ~VisitorBase() = default;
};
```

이 클래스는 아무런 동작을 정의하고 있지 않지만 실제 방문할 객체에 대해서 호출될 accept() 메서드의 인자로 사용된다.
<br>
Expression 클래스를 다음과 같이 재정의한다.

```
struct Expression
{
    virtual ~Expression() = default;
    
    vitual void acept(VisitorBase& obj)
    {
        using EV = Visitor<Expression>;
        if(auto ev = dynamic_cast<EV*>(&obj))
            ev->visit(this);
    }
};
```

새로운 accept() 메서드의 동작 방식은 다음과 같다.
<br>
VisitorBase를 인자로 받지만 Visitor Template로 타입 캐스팅을 시도한다.
<br>
여기서 T는 accept()가 구현되고 있는 현재 클래스의 타입이다.
<br>
<br>
만약 타입 캐스팅이 성공하면 방문자가 해당 타입을 어떻게 방문해야 하는지 알 수 있게 되고 캐스팅된 타입의 visit() 메서드를 호출한다.
<br>
만약 타입 캐스팅에 실패하면 아무것도 하지 않는다.
<br>
<br>
객체 자체에 visit() 메서드가 정의되지 않는 이유를 이해해야 한다.
<br>
만약 객체가 직접 visit() 메서드를 가질 겨웅 호출할 모든 타입마다 visit() 메서드를 정의하여 오버로딩 할 수 있어야 한다.
<br>
그렇게 하면 순환 종속성이 발생한다.
<br>
<br>
모델의 다른 쪽에서 accept()를 구현한 후 다시 한번 ExpressionPrinter를 정의하여 전체 기능을 정합할 수 있다.
<br>
구현은 다음과 같다.

```
struct ExpressionPrinter : VisitorBase, Visitor<DoubleExpression>, Visitor<AdditionExpression>
{
    void visit(doubleExpression &obj) override;
    void visit(AdditionExpression &obj) override;
    
    string str() const { return oss.str(); }
    
private:
    ostringstream oss;
};
```

위 코드에서 볼 수 있듯이, 마커 인터페이스 VisitorBase와 함께 방문할 모든 타입 T에 대해서 Visitor Template를 구현한다.
<br>
만약 특정 타입 T에 대해서 구현을 누락할 경우 프로그램이 컴파일은 되지만 연관된 accept()의 호출이 있을 때 아무 동작도 하지 않게 된다.
<br>
<br>
위의 visit() 구현은 전통적인 방문자 패턴의 구현 방식과 거의 동일하다고 볼 수 있으며, 그 실행 결과도 동일하다.

<br>

## 장단점

### 장점
- 다른 클래스를 변경하지 않으면서 해당 클래스의 객체와 작동할 수 있는 새로운 행동을 도입할 수 있음
- 같은 행동의 여러 버전을 같은 클래스로 이동할 수 있음
- 방문자 객체는 다양한 객체들과 작업하면서 유용한 정보를 축적할 수 있음


### 단점
- 클래스가 요소 계층구조에 추가되거나 제거될 때마다 모든 방문자를 업데이트 해야 함
- 방문자들과 함께 작업해야 하는 요소들의 비공개 필드들 및 메서드들에 접근하기 위해 필요한 권한이 부족할 수 있음

<br>

## 요약
방문자 디자인 패턴은 어떤 객체의 게층 각각마다 서로 다른 작업을 수행해야 할 때 편리하게 적용할 수 있다.

- 침습적 방법 : 개별 객체마다 버추얼 멤버 함수를 추가하며 클래스 계층의 소스 코드를 수정할 수 있을 경우에만 적용 <br>이 방식은 열림-닫힘 원칙(OCP)을 위배하는 문제가 있음
- 반추적 방법 : 객체를 수정할 필요가 없도록 별도로 분리된 방문자를 만들며 런타임에 디스패치가 필요한 경우 dynamic_cast를 이용함
- 전통적 방법(이중 디스패치) : 클래스 계층 전체에 걸쳐 수정이 필요하며 단 한 번만 매우 범용적인 형태로 만들어 두면 다음부터는 필요한 부분만 수정해도 <br>이중 디스패치는 계층의 각 항목들이 방문자를 처리할 accept()<br>전체 계층에 새로운 기능을 추가할 때는 하위 클래스를 두어 새로운 방문자를 처리하면 됨

방문자 패턴은 인터프리터 패턴과 나란히 등장할 때가 많다.
<br>
어떤 텍스트 입력을 해석하여 객체 지향적인 구조로 변하게 하려면, 예를 들어 추상 구문 트리를 특정한 방법으로 렌더링해야만 한다.
<br>
<br>
방문자 패턴은 ostringstream을 전체 클래스 게층에 걸쳐 전파하고 정보를 수집/분석하는데 편리하게 이용될 수 있다.
