# 팩토리 패턴 정리 내용

## 시나리오

예를 들어 직교 좌표계의 좌표 정보를 지정하려고 한다.
<br>
아래와 같이 간단하게 구현할 수 있다.

```
struct Point
{
    Point(const float x, const float y)
      : x{x}, y{y} {}
    float x, y; // 직교 좌표계 좌푯값
};
```
그리고 만약 극좌표계로 좌푯값을 저장하려면 아래와 같이 할 수 있다.

```
Point(const float r, const float theta)
{
    x = r * cos(theta);
    y = r * sin(theta);
}
```

위의 코드는 문제가 있는데, 생성자도 두 개의 float 값을 파라미터로 하기 때문에 극좌표계의 생성자와 구분할 수 없다.
<br>
한 가지 단순한 구분 방법은 좌표계 종류를 enum 타입 값으로 파라미터에 추가하는 것이다.

```
enum class PointType
{
    cartesian,
    polar
};

Point(float a, float b, PointType type = PointType::cartesian)
{
    if(type == PointType::cartesian)
    {
        x = a;
        y = b;
    }
    else
    {
        y = a * cos(b);
        y = a * sin(b);
    }
}
```

위와 같은 코드는 기능적으로는 문제가 없지만, 좋은 코드라고 할 수 없다.

<br>

## 팩토리 메서드(Factory Method)

시나리오의 예제에서 생성자의 문제는 항상 타입과 같은 이름을 가진다는 것이다.
<br>
즉, 일반적인 함수 이름과 달리 생성자의 이름에는 추가적인 정보를 표시할 수가 없다.
<br>
그리고 생성자 오버로딩으로는 같은 float 타입인 x, y와 theta를 구분할 수 없다.
<br>
<br>
이러한 문제를 해결해야 하는데, 아래 코드는 생성자를 protected로 하여 숨기고, 대신 Point 객체를 만들어 리턴하는 static 함수로 제공하는 예제이다.

```
struct Point
{
protected:
    Point(const float x, const float y)
      : x{x}, y{y} {}
public:
    static Point NewCartesian(float x, float y)
    {
        return { x, y };
    }
    static Point NewPolar(float r, float theta)
    {
        return { r * cos(theta), r * sin(theta) };
    }
};
```

여기서 각각의 static 함수들을 팩토리 메서드라고 부른다.
<br>
이러한 메서드가 하는 일은 Point 객체를 생성하여 리턴하는 것뿐이다.
<br>
<br>
함수의 이름과 좌표 파라미터의 이름 모두 그 의미가 무엇인지, 어떤 값이 인자로 주어져야 하는지 명확하게 표현하고 있다.
<br>
<br>
이제 좌표점을 생성할 때 다음과 같이 명료하게 할 수 있다.

```
auto p = Point::NewPolar(5, M_PI_4);
```

위 코드는 가독성이 매우 좋은 코드이다.
<br>
r = 5이고, theta = 𝝅/4 인 극좌표를 만든다는 것을 바로 알 수 있다.


<br>


## 팩토리(Factory)
빌더와 마찬가지로 Point를 생성하는 함수들을 별도의 클래스에 몰아넣을 수 있다.
<br>
그러한 클래스를 팩토리라고 부른다.
<br>
<br>
팩토리 클래스를 만들기 위해 아래와 같이 Point 예제 클래스가 있다고 가정한다.

```
struct Point
{
    float x, y;
    friend class PointFactory;
private:
    Point(float x, float y) : x(x), y(y) {}
};
```

위 코드에서 두 가지의 확인해 둘 사항이 있다.
- Point의 생성자는 private으로 선언되어 사용자가 직접 생성자를 호출하는 경우가 없게 함<br>이 부분은 필수 사항은 아니지만 같은 일을 하는데 두 가지 방벙블 제공하여 사용자를 혼란스럽게 할 필요는 없음
- Point는 PointFactory를 Friend 클래스로 선언하였으며 이 의도는 팩토리가 Point의 생성자의 접근할 수 있게 하려는 의도임<br>이 선언이 없으면 팩토리에서 Point의 인스턴스를 생성할 수 없는데, 이 부분은 생성할 클래스와 그 팩토리 클래스가 동시에 만들어져야 한다는 것을 암시함<br>팩토리 클래스를 훨씬 나중에 만들 수가 없음

이제 별도의 클래스 PointFactory에 NewXxx() 함수들을 정의하기만 하면 된다.

```
struct PointFactory
{
    static Point NewCartesian(float x, float y)
    {
        return Point {x, y};
    }
    static Point NewPolar(float r, float theta)
    {
        return Point{ r * cos(theta), r * sin(theta) };
    }
};
```

이제 Point의 인스턴스 생성을 전담하는 별도의 클래스가 만들어졌다.
<br>
이 클래스를 이용해 사용자는 다음과 같이 인스턴스를 생성할 수 있다.

```
auto my_point = PointFactory::NewCartesian(3, 4);
```

내부 팩토리는 **생성할 타입의 내부 클래스로서 존재하는 간단한 팩토리**를 말한다.
<br>
C#, 자바 등 friend 키워드에 해당하는 문법이 없는 프로그래밍 언어들에서는 내부 팩토리를 흔하게 사용한다.
<br>
<br>
내부 팩토리의 장점은 생성할 타입의 내부 클래스이기 때문에 **private 멤버들에 자동적으로 자유로운 접근 권한을 가진다는 점**이다.
<br>
<br>
반대로 내부 클래스를 보유한 외부 클래스도 내부 클래스의 private 멤버들에 접근할 수 있다.
<br>
내부 팩토리를 이용하면 Point 클래스를 다음과 같이 정의할 수 있다.

```
struct Point
{
private:
    Point(float x, float y) : x(x), y(y) {}
    
    struct PointFactory
    {
    private:
        PointFactory() {}
    public:
        static Point NewCartesian(float x, float y)
        {
            return { x, y };
        }
        static Point NewPolar(float r, float theta)
        {
            return { r * cos(theta), r * sin(theta) };
        }
    };
    
public:
    float x, y;
    static PointFactory Factory;
};
```

위 코드를 보면, 팩토리가 생성할 바로 그 클래스 안에 팩토리 클래스가 들어가 있다.
<br>
이러한 방법은 팩토리가 생성해야 할 클래스가 단 한 종류일 때 유용하게 사용할 수 있다.
<br>
<br>
팩토리가여러 타입을 활용하여 객체를 생성해야 한다면 내부 팩토리 방식은 적합하지 않다.(객체 생성에 필요한 다른 타입들의 private 멤버에 접근하는 것이 불가능함)
<br>
<br>
그리고 팩토리 클래스 전체가 private 블록 안에 있고 그 생성자 마저도 private으로 선언되어 있다.
<br>
원래 private이라면 접근할 수 없지만, Factory 멤버 변수를 static 멤버 변수로 선언하여 다음과 같이 접근할 수 있게 된다.

```
auto pp = Point::Factory.NewCartesian(2, 3);
```

또한 여러 가지 방법으로도 수정할 수 있다.

- 팩토리를 public으로 선언하였을 때

```
Point::PointFactory::NewXxx(...)
```

- Point가 중복해서 등장하는 것을 수정할 때

```
typedef PointFactory Factory

Point::Factory::NewXxx(...)
```

두 번째 방법은 가장 자연스러운 표현을 가능하게 한다.
<br>
또는 가장 원천적인 방법으로 내부 Factory를 바로 호출할 수도 있다.
<br>
<br>
단 중요한 점은, 나중에 확장할 일이 없다는 전제하에서 가능하다.

<br>
<br>
이렇게 내부 팩토리를 사용할지 말지는 코드를 어떻게 구성하여 관리할 것이냐에 따라 달라진다.
<br>
원본 객체 자체에서 팩토리를 얻을 수 있게 하면 API의 사용성에 크게 도움이 된다.


<br>

## 추상 팩토리(Abstract Factory)
추상 팩토리리는 **여러 종류의 연관된 객체들을 생성해야 할 때 사용하는 경우를 위한 패턴**이다.
<br>
추상 팩토리는 팩토리 메서더나 단순 팩토리 패턴의 경우와 달리 복잡한 시스템에서만 필요성이 나타난다.
<br>
<br>
예를 들어 뜨거운 차와 커피를 판매하는 장비에 대한 코드가 있다. (음료를 차갑게 제공할 수도 있음)
<br>
이 두 음료는 전혀 다른 장비를 이용해 만들어 진다.
<br>
<br>
먼저 뜨거운 음료를 추상화하는 HotDrink 코드를 정의한다.

```
struct HotDrink
{
    virtual void prepare(int volume) = 0;
};
```

prepare() 함수는 지정된 용량의 뜨거운 음료를 준비할 때 호출한다.
<br>
HotDrink를 상속받아 아래와 같이 차 타입을 구현할 수 있다.

```
struct Tea : HotDrink
{
    void prepare(int volume) override
    {
        cout << "Take tea bag, boil water, pour " << volume << "ml add some lemon" << endl;
    }
};
```

커피 타입도 비슷하게 만들 수 있다.
<br>
이렇게 차와 커피 타입이 준비되어 있으면 make_drink() 함수를 만들 수 있다.
<br>
<br>
이 함수는 음료의 이름을 받아 그 해당하는 음료를 생성하여 리턴한다.

```
unique_ptr<HotDrink> make_drink(string type)
{
    unique_ptr<HotDrink> drink;
    
    if(type == "tea") 
    {
        drink = make_unique<Tea>();
        drink->prepare(200);
    }
    else
    {
        drink = make_unique<Coffee>();
        drink->prepare(50);
    }
    return drink;
}
```

이렇게 뜨거운 음료를 만드는 장비를 만드는 코드를 알아 보았다.
<br>
이 코드를 이용해 뜨거운 커피를 만드는 장비 팩토리를 만들어야 하는데, HotDrinkFactory를 기반으로 모델링 한다.

```
struct HotDrinkFactory
{
    virtual unique_ptr<HotDrink> make() const = 0;
};
```

이 팩토리가 바로 추상 팩토리다.
<br>
어떤 특정 인터페이스를 규정하고 있지만 구현 클래스가 아니라 추상 클래스이다.
<br>
<br>
즉, 이 타입의 함수 인자로서 사용될 수는ㄴ 있지만, 실제 음료 객체를 만들어 내려면 구체화된 클래스가 있어야 한다.
<br>
<br>
예를 들어 아래와 같은 자식 클래스를 구현 클래스로서 만들 수 있다.

```
struct CoffeeFactory : HotDrinkFactory
{
    unique_ptr<HotDrink> make() const override
    {
        return make_unique<Coffee>();
    }
};
```

이제 TeaFactory도 만들어야 한다.
<br>
이제 뜨거운 음료와 차가운 음료도 만들어야 한다고 가정한다.
<br>
<br>
이를 위해 DrinkFactory를 두어 사용 가능한 다양한 팩토리들에 대한 참조를 내부에 가지도록 한다.

```
class DrinkFactory
{
    map<string, unique_ptr<HotDrinkFactory>> hot_factories;
public:
    DrinkFactory()
    {
        hot_factories["coffee"] = make_unique<CoffeeFactory>();
        hot_factories["tea"] = make_unique<TeaFactory>();
    }
    
    unique_ptr<HotDrink> make_drink(const string& name)
    {
        auto drink = hot_factories[name]->make();
        drink->prepare(200);
        
        return drink;
    }
};
```

음료를 선택할 때 어떤 숫자나 enum 항목이 아닌 이름으로 한다고 가정한다.
<br>
그러면 문자열을 팩토리에 연관시킨 map을 만들어 각 음료를 생성할 팩토리를 지정할 수 있다.
<br>
<br>
map에 저장할 팩토리 타입은 추상 팩토리인 HotDrinkFactory로 하고 객체 자체가 아니라 스마트 포인터로 저장한다.
<br>
(포인터 대신 객체 값을 직접 저장하면 저장소의 타입에 따라 객체 슬라이싱 문제가 발생할 수 있음)
<br>
<br>
이제 음료 주문을 받으면 그에 맞는 팩토리를 찾아서 음료를 만들고 리턴한다.

<br>

## 함수형 팩토리(Function Factory)
보통 팩토리라고 말할 때 다음 두 가지 중 하나를 의미한다.

- 객체를 어떻게 생성하는지 알고 있는 어떠한 클래스
- 호출했을 때 객체를 생성하는 함수

두 번째의 경우를 팩토리 메서드의 하나처럼 볼 수 있을 것 같지만 사실 다르다.
<br>
어떤 타입 T를 리턴하는 std::function을 어떤 함수의 인자로 넘겨져 객체를 생성하는 것이 두 번째 경우에 해당하고 그냥 팩토리라고 부른다.
<br>
<br>
여기서 팩토리 메서드라고 하지 않고 그냥 팩토리라고 하는 이유는 메서드는 어떠한 클래스의 멤버 함수를 의미하는 것이기 때문이다.
<br>
<br>
함수도 변수에 저장될 수 있다.
<br>
즉, 팩토리에 포인터를 저장하는 방법 대신에 200ml의 음료를 생성하는 절차 자체를 팩토리가 내장하게 할 수 있다.
<br>
아래와 같이 함수 블록을 이용하면 쉽게 수정할 수 있다.

```
class DrinkWithVolumeFactory
{
    map<string, function<unique_ptr<HotDrinkFactory>()>> hot_factories;
public:
    DrinkWithVolumeFactory()
    {
        hot_factories["tea"] = [] {
            auto tea = make_unique<Tea>();
            tea->prepare(200);
            return tea;
        }; // Coffe와 유사함
    }
};
```


이러한 접근 방법을 이용하면 저장된 팩토리를 직접 호출하는 과정을 다음과 같이 생략할 수 있다.

```
inline unique_ptr<HotDrink>
DrinkWithVolumeFactory::make_drink(const sring& name)
{
    return factories[name]();
}
```

이러한 코드는 기존과 동일하게 사용자가 이용할 수 있다.


<br>

## 장단점

### 장점
- 생성자와 객체들이 단단하게 결합되지 않도록 할 수 있음
- 생성자의 생성 코드를 프로그램의 한 위치로 이동하여 코드를 더 쉽게 유지보수 할 수 있음 (단일 책임 원칙)
- 기존 클라이언트의 코드를 훼손하지 않고 새로운 유형들의 생성자를 프로그램에 도입할 수 있음(개방 폐쇄 원칙)

### 단점
- 패턴을 구현하기 위해 많은 새로운 자식 클래스들을 도입해야 하므로 코드가 복잡해질 수 있음<br>이것을 해결하기 위해 생성 클래스들의 기존 계층 구조에 패턴을 도입하는 것임

<br>

## 다른 패턴과의 관계

- 많은 디자인은 복잡성이 낮고 자식 클래스들을 통해 더 많은 커스터마이징이 가능한 팩토리 메서드로 시작해 더 유연하면서도 더 복잡한 추상 팩토리, 프로토타입 또는 빌더 패턴으로 발전해 나감
- 추상 팩토리 클래스들은 팩토리 메서드들의 집합을 기반으로 하는 경우가 많음
- 프로토 타입을 사용하여 추상 팩토리의 구상 클래스들의 생성 메서드들을 구현할 수 있음
- 팩토리 메서드는 템플릿 메서드의 특수화라고 볼 수 있으며 동시에 대규모 템플릿 메서드의 한 단계의 역할을 팩토리 메서드가 할 수 있음

<br>

## 요약

- 팩토리 메서드는 생성할 타입의 멤버 함수로 객체를 생성하려 리턴하고 이 메서드는 생성자를 대신함
- 팩토리는 별도의 클래스로서 복적하는 객체의 생성 방법을 알고 있으며 **클래스 대신 함수의 형태로 존재하여 인자로서 사용될 수 있는 경우도 팩토리라고 부름**
- 추상 팩토리는 그 이름이 암시하는 바와 같이 구현 클래스에서 상속받는 추상 클래스로 추상 팩토리는 **어떤 타입 하나가 아니라 여러 타입의 패밀리를 사용할 때 사용됨**<br>실제 프로그래밍을 할 때 추상 팩토리를 사용하는 경우는 극히 드물음


팩토리를 생성자 호출에 대비해 몇 가지 중요한 장점이 있다.

- **팩토리는 객체의 생성 자체를 거부할 수 있음**<br> 생성자는 Exception을 발생시키는 방법밖에 없지만 팩토리는 nullptr를 리턴하는 등의 방법으로 문제가 있는 경우 객체 생성에 자연스럽게 실패할 수 있음
- **팩토리는 가독성 높은 명명이 가능함**<br>생성자는 타입 이름과 같은 이름을 가질 수밖에 없지만 팩토리는 용도를 잘 설명하는 이름을 부여할 수 있음
- **단일 팩토리가 서로 다른 여러 타입의 객체를 생성할 수 있음**
- **팩토리에 다형성을 부여할 수 있음**<br>서브 클래스에서 인스턴스를 만들고 베이스 클래스에서 인스턴스의 참조나 포인터를 리턴할 수 있음
- **팩토리는 캐싱과 같은 메모리 최적화 구현이 가능함**<br>풀링(Pooling)이나 싱글턴 패턴을 적용하기에 매우 자연스러운 코드 구조를 제공함
- 
팩토리와 빌더는 비슷하지만 다른 패턴이다.
<br>
일반적으로 팩토리는 객체를 한 번에 생성하지만, 빌더는 각 구성요소마다 필요한 정보를 제공하며 여러 단계를 거쳐 객체를 생성하낟.











