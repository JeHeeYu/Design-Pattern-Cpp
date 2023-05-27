# 프로토타입 패턴 정리 내용

## 프로토타입(Prototype) 패턴
프로토타입은 코드를 그들의 클래스에 의존시키지 않고 기존 객체들을 복사할 수 있도록 하는 생성 디자인 패턴이다.
<br>
<br>
프로토타입 패턴은 소프트웨어 디자인 패턴 용어로, 생성할 객체들의 타입이 프로토타입인 인스턴스로부터 결정되도록 하며, 인스턴스는 새 객체를 만들기 위해 자신을 복제하게 된다
<br>
<br>
프로토타입 패턴을 이용하면 어떤 모델 객체를 복제할 수 있고, 복제본을 커스터마이즈할 수도 있고, 사용할 수도 있다.
<br>
프로토타입 패턴에서 가장 까다로운 부분은 복제를 구현하는 부분이다.

## 객체 생성
대부분 객체를 생성할 때 생성자를 이용한다.
<br>
하지만 이미 잘 설정된 객체가 있다면 같은 것을 복제하는 것이 가장 쉽다.
<br>
<br>
빌더 패턴을 이용해 복잡한 생성 과정이 단순화되어 있다면 복제를 하는 것이 더욱 의미가 있다.
<br>
<br>
아래는 중복된 객체가 있는 단순 예제 코드이다.

```
Contact john{ "John Doe", Address{"123 East Dr", "London", 10 } };
Contact jane{ "Jane Doe", Address{"123 East Dr", "London", 11 } };
```

john과 jane은 사무실 방만 다르고 같은 빌딩에서 일 하고 있다.
<br>
이 두 사람뿐만 아니라 다른 많은 직원도 직장 주소가 "123 East Dr in London"일 것이다.
<br>
따라서 수많은 객체가 같은 값으로 중복되게 초기화되는 작업이 발생한다.
<br>
<br>
여기서 프로토타입 패턴을 사용할 수 있다.
<br>
<br>
사실 프로토타입 패턴은 객체의 복제가 주요 기능이다.
<br>
당연하게도 객체를 복제하는 하나의 일관된 방법은 없다.

<br>

## 평범한 중복 처리
복제의 목적이 값을 사용하는 것에 있고, 복제 대상 객체의 모든 항목이 값으로만 되어 있다면 복제하는데 문제가 될 것이 없다.
<br>
예를 들어 연락처(Contact)와 주소(Address)가 아래와 같이 정의되어 있다.

```
struct Address
{
    string street, city;
    int suite;
}

struct Contact
{
    string name;
    Address address;
}
```

이 구조는 아래와 같이 코드를 사용하는데 아무런 문제가 없다

```
// 프로토타입 객체
Contact worker{"", Address{"123 East Dr", "London", 0}};

// 프로토타입을 복제하여 일부 수정
Contact john = worker;
john.name = "John Doe";
john.address.suite = 10;
```

하지만 실제 이렇게 쉬운 경우는 드물며 아래 코드와 같이 내부 객체가 포인터로 된 경우가 많다.

```
struct Contact
{
    string name;
    Address *address; 
}
```

이러한 코드는 Contanct john = prototype 코드가 수행될 때 포인터가 복제되기 때문에 큰 문제가 발생한다.
<br>
포인터가 복제되어 prototype과 john 둘 다 같은 address 객체를 가지게 된다.
<br>
<br>
즉, john의 address를 수정했을 뿐인데 prototype의 address도 바뀌게 된다.

<br>

## 복제 생성자를 통한 중복 처리

중복을 피하는 가장 단순한 방법은 객체의 복제 생성자에서 내부 구성 요소들(연락처와 주소) 모두를 적합하게 다루도록 정의하는 것이다.
<br>
예를 들어 아래와 같이 주소를 포인터로 저장되어 있다.

```
struct Contact
{
    string name;
    Address* address;
}
```


이 경우 복제 생성자를 정의해야 하는데, 간단한 방법으로는 아래와 같이 정의할 수 있다.

```
Contact(const Contact& other)
    : name{other.name}
    address = new Address(
        other.address->street,
        other.address->city,
        other.address->suite
    );
```

위 코드는 구현은 간단하지만 범용적이지 않은 코드이다.
<br>
이 예제에서는 정상적으로 동작하지만 만약 주소의 항목이 바뀐다면 문제가 생긴다.
<br>
<br>
예를 들어 거리명에서 건물번호 항목이 분리되고 부가적인 정부가 추가된다면 또 복제 문제가 생긴다.
<br>
<br>
여기에서 상식적인 대응은 Address에 복제 생성자를 정의하는 것이다.

```
Address(const string& street, const string& city, const int suite)
    : street{street}, city{city}, suite{suite} {}
```

그러면 Contact의 생성자를 재활용하여 아래와 같이 복제 생성자를 이용하게 할 수 있다.


```
Contact(const Contact& other)
    : name{other.name}
    , address{ new Address{*other.address} } {}
```

이제 복제와 이동 연산 코드를 추가해야 한다.

```
Contact& operator=(const Contact& other)
{
    if(this == &other)
        retutn this;
    name = other.name;
    address = other.address;
    
    return *this;
}
```

이제 사용이 훨씬 좋아졌다.
<br>
이전과 같이 프로토타입을 생성하면서도 안전하게 재사용할 수 있다.


```
// 프로토타입 객체
Contact worker{"", new Address{"123 East Dr", "London", 0}};

// 프로토타입을 복제하여 일부 수정
Contact john = worker;
john.name = "John Doe";
john.address.suite = 10;
```

이러한 방법은 잘 동작한다.
<br>
이 방법의 유일한 문제는 온갖 복제 생성자를 하나하나 구현하는 데 많은 노력이 든다는 점이다.
<br>
<br>
예를 들어 아래와 같은 코드가 작성되었다.

```
Contact john = worker;
```

그리고 Address에 복제 생성자와 대입 연산자의 구현이 누락되었다고 가정한다.
<br>
이 코드는 컴파일되는데는 문제가 없다.
<br>
하지만 대입 연산자는 모든 상황에서 Default 동작이 정해져 있으므로 적절한 대입 연산자를 정의하지 않더라도 컴파일되고 실행되어 버린다.
<br>


## 직렬화(Serialization)

객체복제를 위해그 구성요소 그래프전체에 걸쳐 각요소마다 명시적인 복제연산을 일일이 정의해야 한다.
<br>
이러한 불편함을 해결하기 위해 어떤 클래스든 쉽게 직렬화될 수 있어야 한다.
<br>
<br>
직렬화란 **어떤 객체 데이터를 비트의 나열로 만들어 파일에 저장하거나 네트워크로 전송할 수 있게 하는 것**을 말한다.
<br>
별도의 특별한 변환 작업 없이 직렬화될 수 있으면 객체를 복제하기가 쉬워진다.
<br>
<br>
직렬화가 객체 복제를 쉽게 만들 수 있는 이유는, 객체를 비트열로 나타내어 온전한 상태로 파일이나 메모리에 쓸 수 있으면, 거꾸로 읽어 들여서(역직렬화, Deserialize) 모든 정보와 내부 구성 요소들을 복구할 수 있다.
<br>
그렇게 된다면 복제 작업과 동일한 작업이 된다.
<br>
<br>
하지만 C++에서는 다른 언어들과 다르게 직렬화를 언어 차원에서 지원하지 않는다.
<br>
C++언어는 컴파일된 바이너리 안에 메타 데이터(Meta Data)도 없고 리플렉션(Reflection) 기능도 존재하지 않는다.
<br>
<br>
C++에서 직렬화를 하려면 복제 연산을 명시적으로 일일이 정의했듯이, 직렬화 연산도 일일이 정의해야 한다.
<br>
다행히도 비트 단위의 데이터 포맷과 std::string에 저장할 방법을 Boost 라이브러리에서 Boost.Serialization의 메서드로 직렬화 연산을 정의할 수 있다.
<br>
이것을 이용하면 아래와 같이 Address 타입에 직렬화 기능을 추가할 수 있다.

```
struct Address
{
    string street;
    string city;
    int suite;

private:
    friend class boost::serialization::access;
    template<class Ar> void serialize(Ar& ar, const unsigned int version)
    {
        ar & street;
        ar & city;
        ar & suite;
    }    
}
```

이렇게 하면 겨과는 & 연산자로 지정한 대로 Address를 구성하는 모든 항목이 객체를 저장하는 위치에 비트열로 기록된다.
<br>
위의 코드는 저장과 복구 양쪽 모두에서 사용되는 메서드이다.
<br>
<br>
Boost는 저장과 복구 시점에 서로 다른 작업을 지정하는 기능도 제공하나, 프로토타입 예제에서는 굳이 사용할 필요는 없다.
<br>
<br>
Conact 타입도 같은 방식으로 직렬화 기능을 추가할 수 있다.



```
struct Contact
{
    string name;
    Address* address = nullptr;

private:
    friend class boost::serialization::access;
    template<class Ar> void serialize(Ar& ar, const unsigned int version)
    {
        ar & name;
        ar & address; // 포인터(*)가 없음
    }    
}
```

위의 serialize() 함수는 address를 직렬화할 때 ar & *address가 아니라 포인터 역참조 없이 ar & address를 사용하고 있다.
<br>
Boost는 이러한 코드에 대해 자동으로 대응한다.
<br>
심지어 address에 nullptr가 대입되어 있더라도 적절히 동작한다.
<br>
<br>
직렬화를 이용해 프로토타입 패턴을 구현하고 싶다면 이러한 방식으로 객체를 구성하는 모든 타입에 serialize()를 구현해야 한다.
<br>
그렇게 구현하고 나면 아래와 같이 직렬화/역직렬화를 이용해 객체 복제 함수를 구현할 수 있다.

```
auto clone = [](const Contact& c)
{
    // 1. 연락처 직렬화
    ostringstream oss;
    boost::archive::text_oarchive oa(oss);
    oa << c;
    
    string s = oss.str();
    
    // 2. 역직렬화로 연락처 복구
    isstringstream iss(oss.str());
    boost::archive::text_iarchive ia(iss);
    
    Contact result;
    ia >> result;
    
    return result;
}
```

이렇게 만들어진 복제 함수가 있으면 john의 연락처를 아래와 같이 쉽게 복제하여 jane의 연락처를 초기화할 수 있다.
<br>
즉, 원본 객체에 영향을 미치지 않는다.

```
Contact jane = clone(john);
jane.name = "Jane";
```

<br>

## 프로토타입 팩토리(Prototype Factory)

자주 복제해서 사용할 기본 객체들은 어떠한 곳에 저장해두고 사용해야 한다.
<br>
예를 들어 회사 주소가 본점과 지점 두 곳이 있고 임직원 정보를 생성할 때마다 복제해서 쓴다고 가정한다.
<br>
<br>
본점(main) 주소와 지점(aux) 주소를 관리해야 하는데, 간단한 방법으로 전역 변수를 사용할 수 있다.

```
Contact main{ "", new Address{ "123 East Dr", "London", 0 } };
Contact aux{ "", new Address{ "123B East Dr", "London", 0 } };
```

이렇게 하면 Contact 클래스를 사용하는 쪽에서 가져다 복제해 사용할 수 있다.
<br>
하지만 좀 더 직관적인 방법은 프로토타입을 저장할 별도의 클래스를 두고 프로토타입의 사용자가 원할 때, 목적에 맞는 복제본을 요구받는 시점에 만들어 제공하는 것이다.
<br>
이렇게 하면 전역 변수보다 더 융통성 있는 일들이 가능해진다.
<br>
<br>
예를 들어 객체를 unique_ptr로 적절히 초기화하여 리턴해주는 편의 함수를 만들 수 있다.


```
struct EmployeeFactory
{
    static Contact main;
    static Contact aux;
    
    static unique<Contact> NewMainOfficeEmployee(string name, int suite)
    {
        return NewEmployee(name, suite, main);
    }
    
    static unique_ptr<Contact> NewAuxOfficeEmployee(string name, int suite)
    {
        return NewEmployee(name, suite, aux);
    }
    
private:
    static unique_ptr<Contact> NewEmployee(string name, int suite, Contact& proto)
    {
        auto result = make_unique<Contact>(proto);
        result->name = name;
        result->address->suite = suite;
        
        return result;
    }
}
```

위 코드는 아래와 같이 사용될 수 있다.

```
auto john = EmployeeFactory::NewMainOfficeEmployee("John Doe", 123);
auto jane = EmployeeFactory::NewAuxOfficeEmployee("John Doe", 123);
```

이렇게 팩토리를 사용해야 하는 이유는, 사용자가 프로토타입을 복제한 다음 새로 설정해야 할 부분들을 누락할 수 있다.
<br>
즉, 올바른 데이터로 채워져 있어야 할 항목들이 null이거나 공백 문자인 채로 이용될 수 있다.

<br>
<br>

객체를 직접 복제한다면 이러한 누락을 피할 방법이 마땅치 않다.
<br>
하지만 모든 항목을 온전히 지정받는 생성자를 제외한 부분적인 생성자들을 모두 private으로 선언하고, EmployeeFacctory만 freind로 선언되어 부분적인 생성자를 사용할 수 있게 한다면, 미완성된 객체 복제본들이 돌아다닐 가능성을 원천적으로 막을 수 있다.

<br>

## 장단점

### 장점
- 객체들을 구상 크래스들에 결합하지 않고 복제할 수 있음
- 반복되는 초기화 코드를 제거한 후 미리 만들어진 프로토타입들을 복제하는 방법을 사용할 수 있음
- 복잡한 객체들을 더 쉽게 생성할 수 있음
- 복잡한 객체들에 대한 사전 설정들을 처리할 때 상속 대신 사용할 수 있는 방법임

### 단점
- 순환 참조가 있는 복잡한 객체들을 복제하는 것이 까다로울 수 있음


<br>


## 다른 패턴과의 관계
- 프로토타입을 사용하여 추상 팩토리의 구상 클래스들을 생성 메서드들을 구현할 수 있음
- 프로토타입은 커맨드 패턴의 복사본들을 기록에 저장해야 할 때 도움이 될 수 있음
- 데코레이터 및 복합체 패턴을 많이 사용하는 디자인들은 프로토타입을 사용하면 장점이 생김<br>그 이유는 프로토타입 패턴을 적용하면 복잡한 구조들을 처음부터 다시 작성하는 대신 복제할 수 있기 때문임
- 때로 프로토타입이 메멘토 패턴의 더 간단한 대안이 될 수 있으며, 이 패턴은 상태를 기록에 저장하려는 객체가 간단하고 외부 리소스에 대한 링크가 없거나 링크들이 있어도 이들을 재설정하기 쉬운 경우에 작동함
- 

## 요약

프로토타입 패턴은 객체의 깊은 복제를 수행하되 매번 전체 초기화를 하는 대신 미리 부분적으로 만들어진 객체를 복제하여 약간의 수정만으로 이용할 수 있게 한다.
<br>
이 과정에서 원본 객체에 대한 걱정은 하지 않아도 되게 해준다.
<br>
<br>
C++에서 프로토타입 패턴을 구현할 방법은 아래 두 가지 방법밖에 없다.
<br>
두 작업 모두 수작업을 필요로 한다.

- 객체의 깊은 복제를 올바르게 수행하는 코드를 작성한다.<br>복제 생성자나 복제 대입 연산자를 구현할 수도 있고 별도의 멤버 함수를 만들 수도 있다.
- 직렬화/역직렬화 기능을 구현하여, 직렬화 직후 역직렬화를 하는 방법으로 복제를 한다.<br>이 방법은 부가적인 연산 비용이 발생한다.<br>이 비용이 심각할지 무시할 만한지는 복제 잡얼 얼마나 자주 하느냐에 따라 달라진다.<br><br>복제 생성자와 비교해 이 방법의 유일한 장점은 직렬화 기능이 덤으로 생긴다는 것 하나뿐이다. 
