# 컴포지트 패턴 정리 내용


## 컴포지트(Composite) 패턴
컴포지트 패턴은 복합체 패턴이라고도 불린다.
<br>
컴포지트 패턴은 객체들의 집합에 대해 개별 인터페이스를 동일하게 가져갈 수 있게 하는 구조 패턴이다.
<br>
물론 각 객체에서 동일한 인터페이스를 구현하면 같은 목적을 달성할 수 있다.

<br>

## 배열에 기반한 속성

컴포지트 디자인 패턴은 보통 전체 클래스를 대상으로 적용된다.
<br>
먼저 클래스의 속성 수준에서 어떻게 사용될 수 있는지 확인해야 한다.
<br>
<br>
'속성' 이라는 용어는 클래스의 필드 멤버는 물론이고 API를 통해서 사용자에게 그 필드가 노출되는 방식을 포괄하여 지칭한다.
<br>
<br>
예를 들어, 컴퓨터 게임에서 크리처 등을 만들려고 한다고 가정한다.
<br>
이 크리처들은 숫자로 표현되는 여러 속성들을 가지고 있다.
<br>
<br>
크리처들마다 힘, 빠르기 등이 서로 다르며, 이 구조를 클래스로 정의하면 아래와 같다.

```
class Creature
{
    int strength, agility, intelligence;
public:
    int get_strength() const
    {
        return strength;
    }

    void set_strength(int strength)
    {
       Creature::strength = strength;
    }
};
```

여기까지는 코드에 문제가 없다.
<br>
하지만 집합적으로 무언가를 하려고 할 때 문제가 발생한다.
<br>
<br>
예를 들어, 크리처들의 특성 값들에 대한 통계를 내야한다고 가정해보면 알 수 있다.
<br>
전체 능력치의 총합 또는 평균은 어떻게 되는지, 가장 수치가 높은 능력치의 값은 무엇인지 등등을 알아야 하는 상황이다.
<br>
<br>
이러한 통계를 내려면 객체의 필드별로 흩어져 있는 값들을 모아 계산해야 한다.
<br>
<br>
단순하게 생각하면 아래와 같은 구현이 나올 수 있다.

```
class Creature
{
    // 다른 멤버들...

    int sum const 
    {
        return strength + agility + intelligence;
    }

    double average() const
    {
        return sum() / 3.0;
    }

    int max() const
    {
        return ::max(::max(strength, agility), intelligence);
    }
};
```

위 코드는 아래와 같이 몇 가지의 이유로 바람직하지 않은 코드이다.

- 전체 합계를 계싼할 때 필드 중 하나를 빠뜨리기 쉬움
- 평균을 계산할 때 상숫값 3.0이 사용되고 있는데, 이 값은 필드의 개수가 바뀔 때 함께 바뀌어야 해서 의도치 않은 종속성을 가짐
- 최대 값을 구할 때 모든 필드 값 쌍마다 std::max()을 반복 호출해야 함

여기에 또다른 속성이 추가될 때마다 sum(), average(), max() 등 모든 집합적 계산 코드가 리팩토링되어야 한다.
<br>
그러므로 이 코든느 최악의 코드이다.
<br>
<br>
이러한 문제를 해결하기 위해 배열 기반 속성을 사용하는 방법이 있다.
<br>
<br>
배열 기반 속성은 다음과 같은 접근 방법을 말한다.
<br>
먼저, 필요한 속성 종류에 대한 enum 타입 멤버를 정의하고 적절한 크기의 배열을 생성한다.

```
class Creature
{
    enum Abilities { str, agl, intl, count };
    array<int, count> abilities;
};
```

위의 enum 정의는 enum 항목의 개수를 나타내기 위해 count라는 항목을 추가로 가지고 있다.
<br>
여기서 enum 클래스가 아니라 enum 타입을 이용한다는 점을 눈여겨 봐야 한다.
<br>
enum 타입을 이용하면 enum 항목을 다루기가 조금 더 수월해 진다.
<br>
<br>
이제 힘, 빠르기 등등에 대해 get/set 메서드를 정의할 수 있다.
<br>
배열의 각 항목에 속성을 맵핑하여 다음과 같이 구현할 수 있다.

```
int get_strength() const { return abilities[str]; }
void set_strength(int value) { abilities[str] = value };

// 다른 속성들도 같은 방식으로 구현
```

이렇게 되면 sum(), average(), max()의 구현이 쉬워진다.
<br>
단지 배열을 순회하기만 하면 모든 경우에 대응할 수 있다.

```
int sum() const
{
    return accumulate(abilities.begin(), abilities.end(), 0);
}

double averagae() const
{
    return sum() / (double)count;
}

int max() const
{
    return *max_element(abilities.begin(), abilities.end());
}
```

이제 새로운 속성이 추가되더라도 그 속성의 get/set 구현외에는 손댈 필요가 없다.
<br>
즉, 집합적인 계산 코드에 수정을 할 필요가 전혀 없다.

<br>

## 그래픽 객체의 그루핑
예를 들어 파워포인트 같은 응용 프로그램이 있다.
<br>
여기서 서로 다른 객체들을 그래그로 여러 개 선택해서 마치 하나의 객체처럼 다루는 작업이 있다고 가정한다.
<br>
객체 하나를 더 선택해서 그룹에 추가할 수도 있다.
<br>
<br>
객체를 화면에 그리는 렌더링 작업도 마찬가지이다.
<br>
개별 그래픽 객체를 렌더링할 수도 있고 여러 개의 도형을 하나의 그룹으로 렌더링할 수도 있다.
<br>
<br>
이러한 사용 방식은 쉽게 구현할 수 있다.
<br>
아래와 같이 인터페이스 하나면 충분하다.

```
struct GraphicObject
{
    virtual void draw() = 0;
};
```

이 클래스의 이름에서 GraphicObject가 한 개의 그래픽 객체만 나타내는 것은 아니다.
<br>
여러 개의 사각형과 원들이 모인 그래픽 객체들도 집합적으로 하나의 그래픽 객체를 나타낼 수 있다.
<br>
<br>
여기서 바로 컴포지트 디자인 패턴이 드러난다.
<br>
<br>
예를 들어 아래와 같이 원을 정의했다.

```
struct Circle : GraphicObject
{
    void draw() override
    {
        std::cout << "Circle" << std::endl;
    }
};
```

비슷한 방식으로 여러 개의 그래픽 객체를 가지는 GraphicObject를 정의할 수 있다.
<br>
이러한 집합적 관계는 재귀적으로 무한히 반복될 수 있다.

```
struct Group : GraphicObject
{
    std::string name;

    explicit Group(const std::string& name)
        : name{name} {}

    void draw() override
    {
        std::cout << "Group" << name.c_str() << " contains:" << std::endl;
        for(auto&& o : objects)
            o->draw();
    }

    std::vector<GraphicObject*> objects;
};
```

개별 원 객체든 그룹 그래픽 객체든 draw() 함수를 구현하고 그릴 수 있는 도형이다.
<br>
그룹 그래픽 객체는 그것을 구성하는 도형들을 vector에 보관하여 스스로를 렌더링할 때 이용할 수 있다.
<br>
그리고 그룹 그래픽 객체 자체도 그룹에 속할 수 있다.
<br>
<br>
이제 아래와 같이 API를 이용할 수 있다.

```
Group root("root");
Circle c1, c2;
root.objects.push_back(&c1);

Group subgroup("sub");
subgroup.objects.push_back(&c2);

root.objects.push_Back(&subgroup);

root.draw();
```

위 코드는 아래와 같은 결과를 출력한다.

```
Group root contins:
Circle
Group sub contains:
Circle
```

이 예제는 인위적인 커스텀 인터페이스 정의가 사용되기는 했지만 가장 간단한 형태의 컴포지트 디자인 패턴 구현으로 볼 수 있다.

<br>


## 장단점

### 장점
- 다형성과 재귀를 유리하게 사용해 복잡한 트리 구조들과 더 편리하게 작업할 수 있음
- 객체 트리와 작동하는 기존 코드를 훼손하지 않고 새로운 요소 유형들을 도입할 수 있음

### 단점
- 기능이 너무 다른 클래스들에는 공통 인터페이스를 제공하기 어려울 수 있음
- 어떤 경우에는 컴포넌트 인터페이스를 과도하게 일반화해야 하며 이해하기 어렵게 만들 수도 있음

<br>

## 다른 패턴과의 관계
- 컴포지트 패턴 트리를 생성할 때 빌더를 사용할 수 있음<br>이유는 빌더의 생성 단계들을 재귀적으로 작동하도록 프로그래밍할 수 있음
- 책임 연쇄 패턴은 종종 컴포지트 패턴과 함께 사용됨
- 반복자들을 이용하여 컴포지트 패턴 트리들을 순회할 수 있음
- 비지터 패턴을 사용하여 컴포지트 패턴 트리 전체를 대상으로 작업을 수행할 수 있음

<br>

## 요약

컴포지트 디자인 패턴은 개별 객체와 컬렉션 객체 모두에 동일한 인터페이스를 사용할 수 있게 해준다.
<br>
명시적으로 인터페이스를 멤버에 두어도 되고 덕-타이핑을 적용해도 된다.
<br>
또는 범위 기반 for 루프를 이용하여 객체의 타입이 가지는 계층적인 의미를 훼손하지 않고 begin()/end() 메서드만 의존성을 가지게 할 수도 있다.
