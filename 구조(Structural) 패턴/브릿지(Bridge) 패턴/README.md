# 브릿지 패턴 정리 내용

## 브릿지(Bridge) 패턴
브릿지 패턴은 큰 클래스 또는 밀접하게 관련된 클래스들의 집합을 두 개의 개별 계층구조(추상화 및 구현)로 나눈 후 각각 독립적으로 개발할 수 있도록 하는 구조 디자인 패턴이다.
<br>

## Pimpl(Pointer to Implementation)
Pimpl은 구현부를 포인터로 참조하는 관례를 뜻한다.
<br>
예를 들어, 개인 정보를 담는 Person 클래스를 만들기로 했다고 가정한다.
<br>
이 클래스는 사람의 이름을 저장하고, 인사말을 출력하는 메서드를 가진다.
<br>
<br>
통상적인 방식으로 아래와 같은 형태로 클래스를 정의한다.

```
struct Person
{
    std::string name;
    void greet();
    
    Person();
    ~Person();
    
    class PersomImpl;
    
    PersomImpl *impl;
};
```

이 코드는 간단하지만 뭔가 이상한 코드이다.
<br>
단순한 클래스임에도 난해한 정의들이 들어간다.
<br>
<br>
필드 변수 name, greet() 함수 까지는 자연스럽지만, 생성자와 소멸자를 필요로 하지는 않을 수 있다.
<br>
또한 PersonImpl의 의도도 잘 모른다.
<br>
<br>
이 클래스의 이상함은 클래스의 구현부를 다른 클래스(PersonImpl)에 숨기고자 하는 의도에서 발생한다.
<br>
구현 클래스 PersonImpl을 헤더 팡리이 아닌 cpp 파일에 정의하는 것이 핵심적이다.
<br>
<br>
예를 들어 Person과 PersonImpl이 모두 종속하는 Person.cpp 안에 정의한다.
<br>
정의는 아래와 같이 단순하다.

```
struct Person::PersonImpl
{
    void greet(Person* p);
}
```

원본 Person 클래스는 PersonImpl를 전방 선언(Foward-Declare)하고 포인터로 관리한다.
<br>
Person의 생성자와 소멸자에서는 PersonImpl의 포인터를 초기화하고 삭제하낟.
<br>
이 경우는 스마트 포인터를 사용해도 무방하다.

```
Person::Person() 
    : impl(new PersonImpl) {}
    
Person::~Person() { delete impl; }
```

이제 Person::greet() 함수를 구현할 차례이다.
<br>
이 모든 제어를 PersonImpl::greet()에 위임한다.

```
void Person::greet()
{
    impl->greet(this);
}

void Person::PersonImpl::greet(Person* p)
{
    printf("hello %s", p->name.c_str());
}
```

여기까지가 Pimpl 관례의 기본적인 내용이라고 할 수 있다.
<br>
이 코드처럼 왜 클래스를 이중으로 정의해서 포인터를 왔다 갔다 하고 있는지에 대한 의문이 생긴다.
<br>
이러한 방식은 3가지의 장점이 있다.

- 클래스 구현부의 상당 부분이 실제로 감춰질 수 있다.<br>만약 Person 클래스가 많은 수의 private/protected 멤버를 가진다면 헤더를 통해 모두 클라이언트에 노출된다.<br>private/protected 한정자로 인해 실제 사용은 할 수 없지만 불필요하게 많은 정보가 노출되는 것은 피할 수가 없다.<br>하지만 Pimpl 관례를 이용하면 꼭 필요한 public 인터페이스만 노출할 수 있다.
- 바이너리 호환성을 보증하기 쉬워진다.<br>숨겨진 구현 클래스에 대한 수정은 바이너리 호환성에 영향을 미치지 않는다.
- 헤더 파일이 멤버 선언에 필요한 헤더들만 include 할 수 있다.<br>즉, 구현부 때문에 클라이언트에서는 필요 없는 헤더까지 include 해야하는 상황을 피할 수 있다.<br>예를 들어 Person이 private 멤버로 vector string를 가진다고 할 때, 어쩔 수 없이 Person.h 파일 안에서 vector와 string을 include 해야 한다.<br>그리고 Person.h를 사용하는 모든 사용자는 강제로 vector와 string을 include 해야 한다.<br>하지만 Pimpl 관례를 이용하면 Person.cpp 에서만 해당 헤더들을 include 하면 된다.

이러한 장점들은 깨끗하고 안정적인 헤더를 유지할 수 있게 해준다.
<br>
그리고 덤으로 컴파일 소요 시간을 줄여준다.
<br>
<br>
디자인 패턴 관점에서는 Pimpl을 브릿지 패턴의 좋은 예로 볼 수 있다.
<br>
위 예에서는 멤버 변수 pimpl이 불투명 포인터이다.
<br>
<br>
즉, 포인터의 상세 사항이 숨겨져 있다.
<br>
이 포인터는 공개용 인터페이스와 숨겨야 할 .cpp 파일의 세부 구현을 연결하는 다리(브릿지) 역할을 한다.

<br>

## 브릿지(Brdige)
Pimpl 관례는 브릿지 디자인 패턴의 특별한 예일 뿐이다.
<br>
일반적인 관점에서 브릿지 패턴을 확인하기 위해 두 종류의 클래스가 있다.
<br>
<br>
하나는 기하 도형 클래스이고 다른 하나는 화면에 그림을 그리는 렌더 클래스들이다.
<br>
<br>
먼저 랜더러의 베이스 클래스는 다음과 같이 정의할 수 있다.

```
struct Renderer
{
    virtual void render_circle(float x, float y, float radius) = 0;
};
```
벡터와 래스터(타자기처럼 한 줄씩 그리는) 구현은 쉽게 만들 수 있다.
<br>
실제 그림을 그리는 대신 그린다는 것을 나타내기 위해 콘솔에 메시지를 출력한다.

```
struct VectorRender : Renderer
{
    void render_circle(float x, float y, float radius) override
    {
        cout << "Rasterizing circle of radius " << radius << endl;
    }
};

struct RasterRenderer : Renderer
{
    void render_circle(float x, float y, float radius) override
    {
        cout << "Drawing a vector circle of radius " << radius << endl;
    }
};
```

기하 도형의 베이스 클래스 Shape은 렌더러에 대한 참조를 가진다.
<br>
기하 도형은 메서드 draw()를 이용해 자기 자신을 그릴 수 있다.
<br>
추가적으로 크기 변경을 할 수 있는 resize() 메서드도 가진다.

```
struct Shape
{
protected:
    Render& renderer;
    Shape(Renderer& renderer) : renderer{ renderer; }
    
public:
    virtual void draw() = 0;
    virtual void resize(float factor) = 0;
};
```

위 코드에서 renderer는 Renderer 클래스의 참조 변수로 선언되어 있다.
<br>
어쩌다 보니 이렇게 브릿지 패턴이 구현되었다.
<br>
<br>
이제 Share 클래스의 구현을 해야한다.
<br>
구현부에서는 원의 중심 좌표, 지름과 같은 추가 정보를 가진다.

```
struct Circle : Shape
{
    float x, y, radius;
    
    void draw() override
    {
        renderer.render_circle(x, y, radius);
    }
    
    void resize(float factor) override
    {
        radius *= factor;
    }
    
    Circle(Renderer* renderer, float x, float y, float radius)
        : Shape{renderer}, x{x}, y{y}, radius{radius} {}
};
```

브릿지를 이용해 Circle을 렌더링 절차에 연결하는 부분이 draw() 안에 구현된다.
<br>
여기서 브릿지는 Render인데, 예를 들어서

```
RasterRenderer rr;
Circle raster_circle{ rr, 5, 5, 5 };
raster_circle.draw();
raster_circle.resize(2);
raster_circle.draw();
```

위의 코드에서 RasterRenderer가 브릿지이다.
<br>
브릿지를 만든 다음 참조로서 Circle에 넘기면 draw()가 호출될 때 브릿지 RasterRenderer를 이용해 원을 그린다.
<br>
<br>
원의 크기 조절이 필요하다면 resize()를 이용할 수 있고 원을 다시 그릴 때 아무 문제 없이 반영된다.
<br>
렌더러는 자신이 다루는 것이 Circle인지 전혀 모르고 참조 변수로 접근되고 있다는 것도 알지 못한다.

<br>

## 장단점

### 장점
- 플랫폼 독립적인 클래스들과 앱들을 만들 수 있음
- 클라이언트 코드는 상위 수준의 추상화를 통해 작동하며, 플랫폼 세부 정보에 노출되지 않음
- 새로운 추상화들과 구현들을 상호 독립적으로 도입할 수 있음
- 추상화의 상위 수준 논리와 구현의 플랫폼 세부 정보에 집중할 수 있음

### 단점
- 결합도가 높은 클래스에 패턴을 적용하여 코드를 더 복잡하게 만들 수 있음


<br>

## 다른 패턴과의 관계


- 브릿지는 일반적으로 사전에 설계되며 앱의 다양한 부분을 독립적으로 개발할 수 있도록 함<br>반면에 어댑터는 일반적으로 기존 앱과 사용디어 원래 호환되지 않던 일부 클래스들이 서로 잘 작동하도록 함
- 추상팩토리를 브릿지와 조합해서 사용할 수 있음<br>이 조합은 브릿지에 의해 정의된 추상화들이 특정 구현들과만 작동할 수 있을 때 유용함
- 빌더와 브릿지를 조합해서 사용할 수 있음<br>디렉터 클래스는 추상화의 역할을 하고 다양한 빌더들은 구현의 역할을 함

<br>

## 요약
브릿지는 단순한 개념이다. 연결 끈이나 접착제처럼 두 가지를 함께 연결한다.
<br>
추상화(인터페이스)를 하면 어떤 컴포넌트가 다른 컴포넌트와 그 상세 구현을 알지 못하는 상태에서도 서로 연동할 수 있다.
<br>
<br>
브릿지 패턴에 연결되는 것들은 서로의 존재를 알아야만 한다.
<br>
예를 들어 Circle은 Renderer의 참조를 가져야 했고, 반대로 Renderer는 원을 어떻게 그리는지 알아야 했다.
<br>
이 부분은 매개자(Mediator) 패턴과 대비되는 부분이다.
<br>
매개자 패턴은 서로를 직접적으로 알지 못하더라도 서로 연동할 수 있게 해준다.
