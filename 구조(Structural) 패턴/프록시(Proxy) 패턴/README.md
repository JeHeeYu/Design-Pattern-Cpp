# 프록시 패턴 정리 내용

## 프록시(Proxy) 패턴
프록시 패턴은 다른 객체에 대한 대체 또는 자리표시자를 제공할 수 있는 구조 디자인 패턴이다.
<br>
프록시는 원래 객체에 대한 접근을 제어하므로, 요청이 원래 객체에 전달되기 전 또는 후에 무언가를 수행할 수 있도록 한다.
<br>
<br>
프록시 디자인 패턴은 어떤 객체의 기능을 수정/확장한다는 목적이 있다.
<br>
<br>
프록시는 API를 일관되게 유지하기 위한 것은 아니다.

<br>

## 스마트 포인터
가장 단순하면서도 직접적인 프록시 패턴의 예제는 스마트 포인터이다.
<br>
스마트 포인터는 포인터의 참조 횟수를 관리하고, 몇명 연산자를 오버라이딩하는 래퍼이다.
<br>
<br>
스마트 포인터는 일반적인 포인터를 사용할 때와 완전히 도일한 방식으로 사용할 수 있다.
<br>
즉, 보통의 포인터가 가진 인터페이스를 유지한다.

```
struct BankAccount
{
    void deposit(int amount) { ... }

BankAcount *ba = new BankAccount;
ba->deposit(123);
auto b2 = make_shared<BankAccount>();
ba2->deposit(123); // same API
};
```

스마트 포인터는 일반 포인터가 사용될 자리에 대신 들어갈 수 있다.
<br>
예를 들어 if(ba) { ... } 와 같은 구문에서 ba가 포인터일 때 의미 있는 동작을 한다면 스마트 포인터여도 의미 있는 동작을 한다.
<br>
<br>
*ba와 같은 역참조에서도 마찬가지로 일반 포인터, 스마트 포인터 둘 다 포인터가 가리키는 실 객체를 얻는다.
<br>
이런 식으로 스마트 포인터가 사용될 자리에 스마트 포인터가 대신 사용될 수 있다.
<br>
<br>
물론 다른 점도 있는데, 예를 들어 스마트 포인터는 delete를 호출할 필요가 없다.
<br>
하지만 그런 부분을 제외하면 일반 포인터와 최대한 가깝게 동작하도록 구현되어 있다.

<br>

## 속성 프록시

다른 프로그래밍 언어에서 속성(Property)을 get/set 메서드가 지원되는 필드로서 특별하게 취급하여 언어 차원에서 지원한다.
<br>
하지만 C++ 언어에서는 그러한 기능이 없다.
<br>
하지만 어떤 필드에 특별히 지정된 접근자/변경자를 부여하고 싶다면 속성 프록시를 만들면 된다.
<br>
<br>
속성 프록시는 기본적으로 속성을 가장한 어떤 클래스이다.
<br>
속성 프록시는 아래와 같이 정의할 수 있다.

```
template <typename T> struct Property
{
    T value;
    Property(const T initial_value)
    {
        *this = inital_value;
    }

    operator T()
    {
        // get 작업 수행
        return value;
    }

    T operator(T new value)
    {
        // set 작업 수행
        return value = new_valuue;
    }
};
```

위 코드에는 get/set 함수를 대신해서 get 작업, set 작업에 해당하는 동작이 구현되어야 할 위치가 각각 주석으로 표시되어 있다.
<br>
보통 이 부분들은 상황에 맞게 커스터마이징 되어야 한다.
<br>
이렇게 연산자 오버라이딩을 하는 대신 그냥 get/set 함수로 대체할 수도 있다.

<br>
<br>

Property<T> 클래스는 단순하게 T가 차지할 자리를 대체한다.
<br>
필드의 값이 사용될 때 Property에서 T로 또는 T에서 Property로 타입 변환됨으로써 동작한다.

<br>
<br>
다음과 같이 Property 프록시로 감싸는 필드를 정의한다.

```
struct Creature
{
    Property<int> strength{ 10 };
    Property<int> agility{ 5 };
};
```

이 프록시 변수들을 아래와 같ㅇ티 일반 필드 변수처럼 사용할 수 있다.

```
Creature creature;
creature.agility = 20;
auto x = creature.strength;
```

<br>

## 가상 프록시
nullptr 이나 초기화되지 않은 포인터를 역참조하면 Crash가 발생한다.
<br>
그런데, 어떤 경우에는 객체를 생성하되 불필요하게 일찍 자원이 할당되는 것을 원하지 않을 수도 있다.
<br>
<br>
이러한 접근 방법을 '느긋한 인스턴스화(Lazy Instantiation)' 이라고 한다.
<br>
정확히 어느 시점에 이러한 느긋한 동작 방식이 필요한지 알고 있다면 사전에 계획해서 특별히 준비해둘 수 있다.
<br>
<br>
만약 정확한 시점을 모른다면 이미 존재하는 것으로 간주되는 객체를 대리하는 프록시를 만들어 '느긋한' 동작을 하게 만들 수 있다.
<br>
<br>
실제 존재하지 않는 객체를 나타낼 수 있기 때문에 이러한 프록시를 버추얼 프록시(Virtual Proxy)라고 부른다.
<br>
버추얼 프록시를 이용하면 실제 인스턴스에 접근하는 대신 무언가 가상의 것에 접근하게 된다.
<br>
<br>
예를 들어, 아래와 같이 어떤 이미지에 대한 전형적인 인터페이스가 있다고 가정한다.

```
struct Image
{
    virtual void draw() = 0;
};
```

반대로 bitmap을 '성급한' 동작 방식으로 구현한다면 당장 이미지를 출력할 필요가 없어도 생성자에서 바로 파일을 로딩해 버릴 것이다.
<br>
아래의 코드는 '성급한' 코드의 구현이다.

```
struct Bitmap : Image
{
    Bitmap(const string& filename)
    {
        cout << "Loading image from " << filename << endl;
    }

    void draw() override
    {
        cout << "Drawing image " << filename << endl;
    }
};
```

이와 같이 구현된 Bitmap은 아래와 같이 객체를 생성하는 시점에 그림 파일을 로딩한다.

```
Bitmap img{ "Image.png" }; // Image 파일 로딩
```

이러한 동작은 일반적으로 원하는 동작이 아니다.
<br>
보통은 실제 그림을 그리는 draw() 메서드가 호출될 때 그림 파일이 로딩되길 원한다.
<br>
이렇게 '느긋한' 동작 방식으로 바꾸고 싶지만 Bitmap이 외부 라이브러리여서 코드를 수정할 수 없다고 가정한다.
<br>
그리고 다른 여러 가지 가상의 이유로 상속을 이용할 수도 없다.
<br>
<br>
바로 이런 상황에서 가상 프록시를 활용할 수 있다.
<br>
Bitmap과 동일한 인터페이스를 제공하고 Bitmap의 기능을 활용하되 동작 방식만 바꾼다.

```
struct LazyBitmap : Image
{
    LazyBitmap(const string& filename)
        : filename(filename) {}

    ~LazyBitmap() { delete bmp; }

    void draw() override
    {
        if(!bmp)
            bmp = new Bitmap(filename);
        bmp->draw();
    }

private:
    Bitmap *bmp{nullptr}
    string filename;
};
```

위 콛,에서 볼 수 있듯이 프록시 LazyBitmap의 생성자는 원래 Bitmap의 생성자보다 훨씬 더 가볍다.
<br>
단지 그림 파일의 이름만 저장하고 실제 파일 로딩 작업은 수행하지 않는다.
<br>
<br>
'느긋한' 동작의 실행은 draw()에서 일어난다.
<br>
draw() 에서는 포인터 bmp를 검사하여 객체가 생성되었는지 확인하고, 만약 생성되지 않았다면 생성한 후 실제 그림을 bmp의 draw()를 호출한다.

<br>
<br>

아래와 같이 Image 타입을 사용하는 함수가 있다.

```
void draw_image(Image& img)
{
    cout << "About to draw the image" << endl;
    img.draw();
    cout << "Done drawing the image" << endl;
}
```

위 함수의 파라미터 img에 Bitmap 대신 LazyBitmap을 사용하여 '느긋한' 동작 방식으로 그림을 로딩하고 그릴 수 있다.

```
LazyBitmap img{ "Image.png" };
draw_image(img); // 그림 파일 로딩은 여기서 일어남
```

<br>

## 커뮤니케이션 프록시
Bar라는 타입의 객체에서 멤버 함수 foo()를 호출한다고 가정한다.
<br>
보통은 Bar의 객체가 그 객체를 이용하는 코드가 구동되는 컴퓨터와 같은 컴퓨터 안에 존재한다고 가정할 수 있다.
<br>
그리고 Bar::foo()의 구동도 같은 프로세스 안에서 된다고 가정한다.
<br>
<br>
이제 Bar 객체와 그 멤버들을 네트워크로 연결된 원격의 다른 컴퓨터에 옮기는 것으로 설계 차원이 결정이 되었다고 가정한다.
<br>
하지만 기존의 코드가 예전처럼 문제가 없이 잘 동작할 수 있어야 한다.
<br>
그렇게 하기 위해 커뮤니케이션 프록시가 필요하다.
<br>
<br>
커뮤니케이션 프록시는 원격에서 작업을 수행하고 결과를 모아 로컬에 중계해주는 컴포넌트이다.
<br>
<br>
예를 들기 위해 단순한 핑퐁 서비스가 있다.
<br>
먼저 아래와 같이 간단한 인터페이스를 정의한다.


```
struct Pingable
{
    virtual wstring ping(const wstring& message) = 0;
};
```

만약 핑퐁이 하나의 프로세스 안에서 일어난다면 다음과 같이 Pong 클래스를 구현할 수 있다.

```
struct Pong : Pingable
{
    wstring ping(const wstring& message) override
    {
        return message + L" pong";
    }
};
```

기본적으로, Pong에 핑을 날리면 Pong은 문자열 " pong"을 덧붙여 응답 메시지를 리턴한다.
<br>
이 코드에서  ostringstream&을 사용하지 않고 매 응답마다 문자열을 생성하고 있다.
<br>
이러한 API는 웹서비스의 동작 방식을 따라 한 것이다.

<br>
<br>

이러한 서비스는 다음과 같이 이용될 수 있으며, 하나의 프로세스에서 동작할 수 있다.

```
void tryit(Pingable& pp)
{
    wcout << pp.ping(L"ping") << "\n";
}

Pong pp;

for(int i = 0; i < 3; ++i)
{
tryit(pp);
}
```

위 코드는 의도대로 "ping pong"이 세 번 출력되는 것이다.
<br>
<br>
이제 핑 서비스를 멀리 떨어진 웹 서버로 옮기기로 했다.

```
[Route("api/[controller]")]
public class PingPongController : Controller
{
    [HttpGet("{msg}")]

    public string Get(string msg)
    {
        return msg + " pong";
    }
}
}
```

<br>

## 장단점

### 장점
- 클라이언트들이 알지 못하는 상태에서 서비스 객체를 제어할 수 있음
- 클라이언트들이 신경 쓰지 않을 때 서비스 객체의 수명 주기를 관리할 수 있음
- 서비스 객체가 준비ㅚ지 않았거나 사용할 수 없는 경우에도 작동함
- 서비스나 클라이언트들을 변경하지 않고도 새 프록시들을 도입할 수 있음

### 단점
- 새로운 클래스들을 많이 도입해야 하므로 코드가 복잡해 질 수 있음
- 서비스의 응답이 늦어질 수 있음

<br>

## 요약
프록시는 어떤 객체에 새로운 멤버를 추가하여 기능을 확장하지 않는다.
<br>
단지 이미 존재하는 멤버들의 동작을 목적에 맞게 변경한다.

<br>
<br>

프록시에는 매우 다양한 종류가 있다.

- <b>속성 프록시</b> : 필드를 대신하여 클래스에 내장되는 객체로 접근/대입 연산자가 구동될 때 추가적인 작업을 수행할 수 있게 함
- <b>버추얼 프록시</b> : 감싸고 있는 객체에 가상으로 접근할 수 있게 하고 이를 통해 느긋한 객체의 로딩을 구현할 수 있음<br>버추얼 프록시를 이용하면 객체가 아직 생성되지 않았음에도 객체를 이용한 듯한 효과를 볼 수 있음
- <b>커뮤니케이션 프록시</b> : 객체의 물리적 위치를 바꾸면서도 API를 기존과 거의 동일하게 유지할 수 있게 해줌
- <b>로깅 프록시</b> : 감싸진 함수가 호출될 때 로깅과 같은 추가적인 작업 수행 가능

이 밖에도 다양한 프록시가 있다.
<br>
실제 상황에서 프록시를 개발할 때는 이미 존재하는 분류상에 속하지 않을 가능성이 높다.
