# 관찰자 패턴 정리 내용

## 관찰자(Observer) 패턴
옵저버 패턴은 여러 객체 중 관찰 중인 객체에 발생하는 모든 이벤트에 대하여 알리는 행동 디자인 패턴이다.
<br>
관찰자 패턴은 널리 사용되는 꼭 필요한 패턴이다.
<br>
C# 등 다른 언어는 자체 또는 표준 라이브로 지원하고 있는 패턴이다.

<br>


## 속성 관찰자
예를 들어 아래와 같이 사람을 정의했다.

```
struct Person
{
    int age;
    Person(int age) : age{age} {}
};
```

age가 변경되는 것을 알 수 있는 방법을 생각해야 한다.
<br>
먼저 Person의 age에 쓰기 작업이 될 때 정보를 받을 수 있다면 가장 효율적일 것이다.
<br>
그러한 방법을 구현하기 위한 유일한 방법은 아래와 같이 set 메서드를 두는 것이다.

```
struct Person
{
    int get_age() const { return age; }
    void set_age(const int value) { age = value; }
    
private:
    int age;
};
```

이제 set_age()가 호출되어 값이 바뀔 때 어떤 알림을 보낼 수 있으면 된다.
<br>
그 알림을 보내는 방법을 알아야 한다.

<br>

## Observer Template
먼저 알림을 보내는 방법으로 한 가지 접근 방법은 Person의 변화에 관심을 가지는 쪽에서 사용할 수 있도록, 어떤 변경이 일어날 때마다 호출되는 메서드를 정의하여 베이스 클래스에 두는 것이다.
<br>
<br>
클라이언트에서는 이 베이스를 상속받아 변경 시점에 수행할 작업을 구현한다.

```
struct PersonListener
{
    virtual void person_changed(Person& p, const string& property_name, const any new_value) = 0;
};
```

하지만 이러한 접근 방법은 Person 타입에 한정되어 있다.
<br>
객체의 속성 변경과 그 변경을 모니터링해야 하는 작업은 매우 흔한 일이다.
<br>
그때마다 해당 객체 타입에 종속된 알림 전용 베이스 클래스를 정의하는 것은 너무 번거롭다.
<br>
<br>
따라서 좀 더 일반화된 접근 방법이 필요하다.
<br>
먼저 아래와 같은 Observer 정의가 있다.

```
template<typename T> struct Observer
{
    virtual void field_changed(T& source, const string& field_name) = 0;
}
```

field_chnage()의 파라미터들은 그 이름에서 의미를 쉽게 알 수 있다.
<br>
source는 모니터링할 필드를 가진 객체의 참조이고 field_name은 그 필드의 이름이다.
<br>
<br>
이 구현은 Person 클래스의 변경 사항을 모니털이할 수 있게 해준다.
<br>
예를 들어 age에 쓰기 작업이 있을 때마다 콘솔에 메시지를 출력하고 싶다면 아래와 같이 할 수 있다.

```
struct ConsolePersonObserver : Observer<Person>
{
    void field_change(Person& source, const string& field_name) override
    {
        cout << "Person's << field_name << " has changed to " << source.get_age() << ".\n";
    }
};
```

이 구현은 유연하기 때문에 여러 다른 시나리오들도 지원할 수 있다.
<br>
예를 들어 복수의 클래스의 필드 값들을 모니터링할 수 있다.
<br>
<br>
아래는 Person과 Creature 두 클래스를 한꺼번에 모니터링하는 예제이다.

```
struct ConsolePersonObserver : Observer<Person>, Observer<Creature>
{
    void field_changed(Person& source, ...) { ... }
    void field_changed(Creature& source, ...) { ... }
}
```

<br>

## Observable Template
다시 Person으로 돌아와서, Person을 모니터링이 가능한 Observable 클래스로 만든다.
<br>
Observable은 다음과 같은 책임을 갖게 된다.
- Person의 변경을 모니터링 하려는 관찰자들을 private 리스트로 관리
- 관찰자가 Person의 변경 이벤트에 수신 등록 또는 해제(subscribe()/unsubscribe()) 할수 있게 함
- notify()를 통해 변경 이벤트가 발생했을 때 모든 관찰자에게 정보가 전달 되도록 함

이 모든 기능은 별도의 베이스 클래스로 쉽게 옮길 수 있다.
<br>
그렇게 함으로써 잠재적으로 일어날 수 있는 중복 구현을 예방한다.

```
template <typename T> struct Observable
{
    void notify(T& source, const string& name) { ... }
    void subscribe(Observer<T>* f) { observers.push_back(f); }
    void unsubscribe(Observer<T>* f) { ... }
    
private:
    vector<Observer<T>*> observers;
};
```

subscribe()는 간단하게 구현되었다.
<br>
관찰자 목록에 새로운 관찰자를 추가하는 작업밖에 없다.
<br>
<br>
관찰자 목록은 private으로 내부에서만 접근할 수 있게 통제된다.
<br>
이 목록은 상속받는 클래스에서도 접근할 수 없다.
<br>
이렇게 함으로써 관찰자 목록이 임의로 수정되는 것을 막는다.
<br>
<br>
다음으로 notify()를 구현한다.
<br>
기본 아이디어는 단순한데, 모든 관찰자를 순회하며 관찰자의 field_changed()를 차례로 호출한다.

```
void notify(T& source, const string& name)
{
    for(auto obs : observers)
        obs->field_changed(source, name);
}
```

Observable Template 를 상속받는 것만으로는 부족하다.
<br>
관찰 받는 클래스에서도 자신의 필드가 변경될 때마다 notify()를 호출해 주어야 한다.
<br>
<br>
예를 들어 set_age() 메서드를 보면 다음과 강튼 세 가지 책임이 지워진다.
- 필드의 값이 실제로 바뀌었는지 검사함<br>만약 age의 값이 20살인 상태에서 동일한 20살로 값이 중복 설정될 때 대입 작업과 알림을 수행하는 것은 의미 없는 동작임
- 필드에 적절한 값이 설정되어야 함<br>예를 들어 age의 값으로서 -1은 의미가 없음
- 필드의 값이 바뀌었을 때 올바른 인자로 notify()를 호출

이에 따라 set_age()는 다음과 같은 새로운 구현을 가진다.

```
struct Person : Observable<Person>
{
    void set_age(const int age)
    {
        if(this->age == age) return;
        this->age = age;
        notify(*this, "age");
    }
    
private:
    int age;
};
```

<br> 

## 관찰자(Observer)와 관찰 대상(Observable)의 연결

이렇게 준비된 코드를 기반으로 하여 Person의 필드 값 변화에 대한 알림을 받을 수 있다.
<br>
알림을 받기 위해 관찰자는 다음과 같이 정의된다.

```
struct ConsolePersonObserver : Observer<Person>
{
    void filed_changed(Person& source, const string& field_name) override
    {
        cout << "Person's " << field_name << " has changed to " << source.get_age() << ".\n";
    }
};
```

그리고 아래와 같이 사용할 수 있다.

```
Person p{ 20 };
ConsolePersonObserver cpo;
p.subscribe(&cpo);
p.set_age(21); // Person의 age가 21로 변경
p.set_age(22); // Person의 age가 22로 변경
```

하지만 이 코드들은 속성의 종속성, 스레드 안전성, 재진입 안전성과 같은 문제들이 고려되지 않았다.

<br>

## 종속성 문제

예를 들어 16세가 되는 순간 투표 권한이 생긴다.
<br>
Person의 age 필드 변화에 따라 투표 권한이 생겼음을 알리는 기능을 추가한다.
<br>
<br>
먼저 Person에 아래와 같은 속성 읽기 메서드가 있다고 가정한다.

```
bool get_can_vote() const { return age >= 16 };
```

이 메서드는 전용 필드 멤버는 없고 대응되는 set 메서드도 없다.
<br>
그럼에도 불구하고 notify()를 호출해 주어야 한다.
<br>
<br>
그러기 위해서 set_age() 메서드를 이용해 간접적으로 알 수 있다.
<br>
따라서 can_vote의 변경 여부에 대한 알림은 set_age() 안에서 수행될 필요가 있다.

```
void set_age(int value) const
{
    if(age == value) return;
    
    auto old_can_vote = can_vote(); // 이전 값 저장
    age = value;
    notify(*this, "age"0;
    
    if(old_can_vote != can_vote()) // 값이 바뀌었는지 검사
        notify(*this, "can_vote");
}
```

위 코드는 원래 목적을 벗어나서 과도한 일을 하고 있다.
<br>
age의 변화뿐만 아니라 그에 영향받는 can_vote의 변화까지 찾아서 알림을 생성한다.
<br>
이런 방식은 확장성이 없다.

<br>

## 수신 해제와 스레드 안전성

전체 시나리오에 걸쳐서 스레드가 하나만 사용된다면 아래와 같은 단순한 구현으로 충분하다.

```
void unsubscribe(Observer<T>* observer)
{
    observers.erase(remove(observers.begin(), observers.end(), observer), observers.end());
}
```

이러한 삭제 이동은 관례가 틀린 것은 아니지만 단일 스레드 상황에서만 정상적인 동작이 보증된다.
<br>
std::vector는 스레드 안전성이 없다.
<br>
<br>
따라서 subscribe()와 unsubscribe()가 동시에 호출된다면 의도하지 않은 결과가 발생할 수 있다.
<br>
왜냐하면 두 함수 모두 vector를 수정하려 들기 때문이다.
<br>
<br>
이 문제는 쉽게 해결할 수 있는데, 관찰 대상 객체의 모든 작업에 아래와 같이 단순히 락(lock)을 추가하면 된다.

```
template <typename T>
struct Observable
{
    void notify(T& source, const string& name)
    {
        scoped_lock<mutex> lock{ mtx };
        ...
    }
    
    void subscribe(Observer<T>* f)
    {
        scoped_lock<mutex> lock{ mtx };
        ...
    }
    
    void ubsubscribe(Observer<T>* o)
    {
        scoped_lock<mutex> lock{ mtx };
        ...
    }
    
private:
    vector<Observer<T>*> observers;
    mutex mtx;
};
```


<br>


## 재진입성(Reentrancy)

앞서의 구현은 언제든 호출될 수 있는 세 가지 주요 메서드에 락을 추가하여 일정 수준의 스레드 안전성을 확보했다.
<br>
하지만 여전히 문제가 있다.
<br>
<br>
예를 들어, 운전면허 과제를 위한 컴포넌트 TrafficAdministration가 있다고 가정한다.
<br>
이 컴포넌트느 운전면허 시험 응시 기준에 맞는지 모든 사람의 연령을 모니터링 한다.
<br>
<br>
그리고 어떤 사람의 나이가 기준 연령 17세에 도달했다면 모니터링을 중단하기 위해 알림 수신 등록을 해제한다.

```
struct TrafficAdministration : Observer<Person>
{
    void TrafficAdministration::field_changed(Person& source, const string& field_name) override
    {
        if(field_name == "age")
        {
            if(source.get_age() < 17)
                cout << "Whoa there, you are not old enough to drive!\n";
        }
        else
        {
            // 기준 연령에 도달했으므로 더 이상 모니터링 할 필요가 없음
            cout << "We no longer care!\n";
            source.unsubscribe(this);
        }
    }
};
```

이 시나리오는 문제를 일으키는데, 17세에 도달했을 때 다음과 같은 호출이 연이어 일어난다.

```
notify() --> field_changed() --> unsubscribe()
```

여기서 unsubscribe()의 실행이 이미 앞에서 락이 점유된 이후에 일어나고 있다.
<br>
이 때문에 이미 점유된 락을 다시 점유하려 들게 된다.
<br>
이러한 문제를 재진입(Reentrancy) 문제라고 한다.
<br>
<br>
이러한 문제에 대한 해결책으로 몇 가지 서로 다른 방법이 있을 수 있다.
<br>
첫 번째 해결책은 그냥 이러한 상황을 금지하는 것이다.
<br>
그러면 최소한 이 상황에서만큼은 문제를 피할 수 있다.
<br>
이 상황에서 재진입이 일어난다는 것은 매우 자명하다.
<br>
<br>
또 다른 방법으로 컬렉션에서 항목을 삭제하는 작업 자체를 우회적으로 처리하는 것이다.
<br>
즉, 다음과 비슷한 방법으로 항목이 삭제된 것과 같은 효과를 낸다.

```
void unsubscribe(Observer<T>* o)
{
    auto it = find(observers.begin(), observers.end(), o);
    if(it != observers.end())
        *it = nullptr; // cannot do this for a set
}
```

이렇게 수정할 경우 당연하게 notify()에서 추가적인 검사가 필요하다.

```
void notify(T& source, const string& name)
{
    for(auto obs : observers)
        if(obs)
            obs->field_changed(source, name);
}
```

위 방법은 notify()와 subsribe()에 일어나는 락 충돌 문제는 해결한다.
<br>
하지만 예를 들어 subscribe()와 unsubscribe()가 병렬로 동시에 호출되어 양쪽에서 수정을 시도할 때 발생할 수 있는 문제는 해결하지 못한다.
<br>
<br>
그 부분에 또 락을 추가하고 관리해야 할 것이다.
<br>
<br>
또 다른 가능한 방법은 notify() 안에서 컬렉션 전체를 복제하여 사용하는 것이다.
<br>
여전히 락이 필요하기는 하지만 알림을 보내는 단계에서 락을 점유할 필요는 없어진다.
<br>
<br>
즉, 아래와 같이 할 수 있다.

```
void notify(T& source, const string& name)
{
    vector<Observer<T>*> observers_copy;
    {
        lock_guard<mutex_t> lock{ mtx };
        observers_copy = observers;
    }
    
    for(auto obs : observers_copy)
        if(obs)
            obs->field_changed(source, name);
}
```

위 구현에서 여전히 락을 사용하기는 하지만 field_changed()를 호출하는 시점에는 락이 해제된 상태가 된다.
<br>
락은 vector를 복제하는 동안만 사용된다.
<br>
<br>
마지막으로 mutex를 recursive_mutex로 바꾸는 방법이 있다.
<br>
대부분의 경우 개발자들은 recursive_mutex를 좋아하지 않는다.

<br>

## 요약

관찰자 패턴의 구현에서 이루어지는 주요한 디자인 선택 사하을 정리할면 다음과 같다.

- 어떤 정보를 모니터링할 수 있게 하여 커뮤니케이션을 도울 것인지 결정해야 한다. 예를 들어 필드 속성의 변경을 모니터링할 수 있게 한다면 그 속성의 이름, 그리고 필요하다면 이전/이후 값을 관찰자에게 제공해야 한다. 단, 값이 전용 타입을 가진다면 타입의 전달에서 어려움을 겪을 수 있다.
- 클래스 전체가 관찰자로 등록되어야 하는지, 아니면 이벤트를 처리할 함수를 등록하는 것만으로도 충분한지
- 관찰자의 알림 수신 해제는 어떻게 해야 하는지
- Observer의 메서드들이 서로 다른 여러 스레드에서 호출될 가능성이 있는지
- 동일한 관찰자가 복수의 수신 등록을 할 수 있어야 하는지
