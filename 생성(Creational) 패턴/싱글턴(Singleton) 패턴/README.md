# 싱글턴 패턴 정리 내용

## 싱글턴(Singleton) 패턴
싱글턴 패턴은 클래스에 인스턴스가 하나만 있도록 하면서 이 인스턴스에 대한 전역 접근을 허용하는 생성 디자인 패턴이다.
<br>
<br>
싱글턴 패턴은 디자인 패턴의 역사상 가장 많은 미움을 받고 있는 디자인 패턴이다.
<br>
하지만 상황에 따라 싱글턴 패턴을 필요로 하는 상황이 있다.
<br>
<br>
싱글턴 디자인 패턴은 어떤 특정 컴포넌트의 인스턴스가 애플리케이션 전체에서 단 하나만 존재해야 하는 상황을 처리하기 위해 고안되었다.
<br>
<br>
예를 들어 메모리에 데이터베이스를 로딩하고 읽기 전용 인터페이스를 제공하는 경우는 싱글턴 패턴을 활용하기 딱 좋은 상황이다.
<br>
왜냐하면 동일한 데이터를 여러번 로딩하여 메모리를 낭비할 필요가 없기 때문이다.
<br>
어쩌면 애당초 두 개 이상의 데이터베이스가 올라갈 만큼의 메모리가 크지 않을 수도 있고, 또는 메모리에 여유가 없으면 프로그램이 이상 동작을 할 수도 있다.

<br>

## 전역 객체로서의 싱글턴
이 문제의 대한 손쉬운 접근 방법은 객체를 단 한번만 인스턴스화하도록 약속하는 것이다.
<br>
예를 들어 아래와 같은 코드가 있다.

```
struct Database
{
    // 이 객체를 두 개 이상 인스턴스화 하지 말 것
    Database();
};
```

이렇게 약속으로 한다면 어떤 개발자는 이 주석을 무시하고 사용하는 경우 말고도, 눈에 보이지지도, 의도하지도 않은 은밀한 방식으로 생성자가 호출되어 버릴 수도 있다.
<br>
복제 생성자, 복제 대입 연산자에 의한 것일 수도 있고, make_unique()의 호출, 또는 제어 역전(IoC) 컨테이너의 사용에 따른 것일 수도 있다.
<br>
<br>
이를 극복하는 방법으로 가장 간단하게 static 전역 객체를 두는 것이다.

```
static Database database();
```

static 전역 객체의 문제점은 각각의 컴파일 단위 바이너리들에서 초기화 순서가 정의되어 있지 않다는 것이다.
<br>
이 부분은 까다로운 문제를 일으킨다.
<br>
<br>
static 전역 객체가 여러 개 사용된다면 어느 한 모듈에서 전역 객체를 참조할 때 그 전역 객체가 참조하는 또 다른 전역 객체가 아직 초기화된 상태가 아닐 수 있다.
<br>
그리고 사용자가 전역 객체가 있다는 사실을 어떻게 알 수 있느냐 하는 문제도 존재한다.
<br>
<br>
사용자가 조금 더 알기 쉽도록, 필요한 객체를 리턴하는 전역 함수를 제공하는 방법이 있다.

```
Database& get_database()
{
    static Database database;
    
    return database;
}
```

이 함수를 호출하면 데이터베이스에 접근할 수 있는 참조를 얻을 수 있다.
<br>
하지만 위 함수의 스레드 안전성이 C++ 11 이상 버전에서만 안전이 보장된다는 점에 유의해야 한다.
<br>
<br>
static 객체를 초기화하는 코드 앞뒤로 컴파일러가 락을 삽입하여 초기화 와중에 동시에 다른 스레드에서 접근하는 것을 방지해 주는지 확인해야 한다.
<br>
<br>
물론 이 방식도 문제가 생길 수 있다.
<br>
만약 데이터베이스의 소멸자에서 다른 싱글턴 모듈을 참조한다면 프로그램이 Crash가 발생할 가능성이 높다.
<br>

## 전통적인 구현 방법

전역 객체를 static 전역 변수로 관리한다고 해서 인스턴스의 추가 생성을 막을 수는 없다.
<br>
이에 대한 쉬운 대책은 생성자 안에 static 카운터 변수를 두고 값이 증가될 경우 Exception을 발생 시키는 것이다.

```
struct Database
{
    Database()
    {
        static int instance_count { 0 };
        if(++instance_count > 1) 
            throw std::exception("Cannot make > 1 database!");
    }
};
```

이 방법은 인스턴스가 한 개보다 더 많이 만들어지려고 할 때 Exception을 발생시켜 여러 개의 인스턴스가 생기는 것은 막을 수 있다.
<br>
하지만 다른 문제가 있는데, 사용자와의 소통 부분에서는 실패한 코드이다.
<br>
사용자 관점에서 인터페이스상으로는 Database의 생성자가 단 한 번만 호출되어야 한다는 것을 알 수가 없다.

<br>
<br>

Database를 사용자가 명시적으로 생성하는 것을 막는 방법은 생성자를 private으로 선언하고 인스턴스를 리턴받기 위한 멤버 함수를 만드는 것이다.
<br>
이 멤버 함수는 유일하게 하나만 존재하는 인스턴스를 리턴한다.

```
struct Database
{
protected:
    Database()
    
public:
    static Database& get()
    {
        static Database database;
        
        return database;
    }
    
    Database(Database const&) = delete;
    Database(Database&&) = delete;
    Database& operator=(Database const&) = delete;
    Database& operator=(Database &&) = delete;
};
```

C++ 11 이전에는 생성자/연산자를 private으로 만들어서 비슷한 효과를 볼 수 있었다.
<br>
위 코드와 같은 수작업이 싫다면 boost::noncopyable 클래스를 사용해 볼 수도 있다.
<br>
이 클래스를 상속받아 구현하면 이동 생성자/연산자를 제외하고 모두 숨길 수 있다.

<br>
<br>

그리고 Database가 다른 static 또는 전역 객체에 종속적이고, 소멸자에서 그러한 객체를 참고하고 있다면 아주 위험하다.
<br>
static 객체, 전역 객체의 소멸 순서는 결정적이지 않기 때문에 접근하는 시점에 이미 소멸되어 있을 수도 있다.

<br>
<br>

마지막으로 get() 함수에서 힙 메모리 할당으로 객체를 생성하게 한다.
<br>
이렇게 함으로써 전체 객체가 아니라 포인터만 static으로 존재할 수 있다.

```
static Database& get() {
    static Database* database = new Database();
    
    return *database;
}
```

이렇게 Database가 프로그램이 종료될 때까지 살아있어 소멸자의 호출이 필요 없는 것으로 가정하고 있다.
<br>
따라서 참조 대신 포인터를 사용해 public 생성자가 호출될 일이 없게 한다.
<br>
그리고 위 코드는 static 변수의 초기화는 전체 런타임 중 한 번만 수행되므로 메모리 누수를 일으키지도 않는다.

<br>

## 멀티스레드 안전성

C++ 11 버전 이상부터 스레드 세이프하다.
<br>
즉, 두 개의 스레드가 동시에 get()을 호출하더라도 Database가 두 번 생성되는 일은 없다.
<br>
<br>
하지만 C++ 11 이전 버전에서는 이중 검증 락킹(Double-Chekced Locking) 방법으로 생성자를 보호해야 한다.
<br>
이 구현은 아래 코드와 같다.

```
struct Database
{
    static Database& instance();
    
private:
    static boost::atomic<Database*> instance;
    static boost::boost::mutex mtx;
};

Database& Database::instance()
{
    Database* db = instance.load(boost::memory_order_consume);
    
    if(!db)
    {
        boost::mutex::scoped_lock lock(mtx);
        db = instance.load(boost::memory_order_consume);
        if(!db)
        {
            db = new Database();
            instance.store(db, boost::memory_order_release);
        }
    }
}
```


<br>

## 싱글턴의 문제

Database가 도시의 이름과 그 인구수의 목록을 담고 있다고 가정한다.
<br>
그러면 다음과 같이 도시의 이름에서 인구수를 얻는 인터페이스를 가질 수 있다.

```
class Database
{
public:
    virtual int get_population(const std::string& name) = 0;
};
```

Database는 도시의 인구수를 알려주는 메서드를 하나 가진다.
<br>
이제 Database를 상속받는 싱글턴 구현 클래스 SingletonDatabase가 있다고 가정한다.

```
class SingletonDatabase : public Database
{
    SingletonDatabase()
    std::map<std::string, int> capitals;

public:
    SingletonDatabase(SingletonDatabase const&) = delete;
    void operator=(SingletonDatabase const&) = delete;
    
    static SingletonDatabase& get()
    {
        static SingletonDatabase db;
        
        return db;
    }
    
    int get_population(const std::string& name) override
    {
        return capitals[name];
    }
};
```

그리고 위 코드를 기반으로 서로 다른 여러 도시의 인구수의 합을 계산하는 싱글턴 컴포넌트를 아래와 같이 만들었다고 가정한다.

```
struct SingletonRecordFinder
{
    int total_population(std::vector<std::string> names)
    {
        int result = 0;
        for(auto& name : names)
            result += SingletonDatabase::get().get_population(name);
            
            return result;
    }
};
```

여기서 문제는 SingletonRecordFinder가 SingletonDatabase에 밀접하게 의존한다는 데서 발생한다.
<br>
SingletonRecordFinder에 대한 단위 테스트를 한다고 생각해보면 알 수 있다.
<br>
아래와 같이 실제 데이터를 이용해서 테스트해야 한다.

```
TEST(RecordFinderTests, SingletonTotalPopulationTest)
{
    SingletonRecordFinder rf;
    std::vector<std::string> names{ "Seoul", "Mexico City" };
    
    int tp = rf.total_population(names);
    EXPECT_EQ(17500000 + 17400000, tp);
}
```

그런데 실제 데이터는 언제든 바뀔 수 있는데 그때마다 테스트 코드를 수정하는 것은 비효율적이다.
<br>
실제 데이터 대신 더미 데이터를 이용하여 테스트를 하는것은 불가능하다.
<br>
이러한 유연성 부족은 싱글턴의 전형적인 단점이다.

<br>

<br>
그렇다면 할 수 있는 방법으로 한 가지 방법은 싱글턴 Database에 대한 명시적 의존을 제거하는 것이다.
<br>
꼭 싱글턴 객체가 아니어도 Database 인터페이스를 구현한 객체만 있으면 RecordFinder를 구동할 수 있다.
<br>
<br>
따라서 데이터를 어디서 얻을지 지젛알 수 있게 하는 ConfigurableRecordFinder를 만든다.

```
struct ConfigurableRecordFinder
{
    explicit ConfigurableRecordFinder(Database& db)
        : db{db} ()
        
    int total_population(std::vector<std::string> names)
    {
        int result = 0;
        
        for(auto& name: names)
            result += db.get_population(name);
        return result;
    }
    
    Database& db;
};
```

이렇게 하면 명시적으로 싱글턴에 의존하는 대신 참조 변수 db를 사용한다.
<br>
이렇게 함으로써 ConfigurableRecordFinder를 테스트할 때는 아래와 같은 더미 데이터를 지정할 수 있게 된다.

```
class DummyDatabase : public Database
{
    std::map<std::string, int> capitals;
    
public:
    DummyDatabase()
    {
        capitals["alpha"] = 1;
        capitals["beta"] = 2;
        capitals["gamma"] = 3;
    }
    
    int get_population(const std::string& name) oveeride
    {
        return capitals[name];
    }
};
```

이제 DummyDatabase를 사용하도록 단위 테스트 코드를 수정할 수 있다.

```
TEST(RecordFinderTests, DummyTotalPopulationTest)
{
    DummyDatabase db{};
    ConfigurableRecordFinder rf{ db };
    EXPECT_EQ(4, rf.total_population(std::vector<std::string>{"alpha", "gamma"));
}
```

<br>

## 싱글턴과 제어 역전(Inversion Of Control)

어떤 컴포넌트를 명시적으로 싱글턴으로 만드는 것은 과도하게 깊은 종속성을 유발한다.
<br>
이 때문에 싱글턴 클래스를 다시 일반 클래스로 만들 때 그 사용처 깊숙이 손대야 하여 많은 수정 비용이 든다.
<br>
<br>
또 다른 방법은 클래스의 생성 소멸 시점을 직접적으로 강제하는 대신 IoC 컨테이너에 간접적으로 위임하는 것이다.
<br>
<br>
종속성 주입 프레임워크 Boost.DI를 이용하면 아래와 같이 싱글턴 컴포넌트를 IoC 관례에 맞추어 정의할 수 있다.

```
auto injector = di::make_injector(di::bind<IFoo>.to<Foo>.in(di::singleton);
```

위 코드에서는 타입 이름의 앞첨자 I를 붙여 인터페이스 목적의 타입임을 나타내고 있다.
<br>
여기서 di::bind 코드가 의미하는 바는, IFoo 타입 변수를 멤버로 가지는 컴포넌트가 생성될 때마다 IFoo 타입 멤버 변수를 Foo의 싱글턴 인스턴스로 초기화한다는 것을 뜻한다.
<br>
<br>
이렇게 종속성 주입 컨테이너를 활용하는 방식만이 바람직한 싱글턴 패턴의 구현 방법이라고 할 수 있다.
<br>
이렇게 하면 싱글턴 객체를 뭔가 다른 것으로 바꾸어야 할 때 코드 한 군데만 수정하면 됟나.
<br>
<br>
즉, 컨테이너 설정 코드만 수정하면 원하는 데로 객체를 바꿀 수 있다.
<br>
추가적인 장점으로 싱글턴을 직접 구현할 필요없이 Boost.DI 프레임워크에서 자동으로 처리해주기 때문에 싱글턴 로직을 잘못 구현할 오류의 여지를 없애준다.
<br>
<br>
또한 Boost.DI는 스레드 세이프하다.

<br>

## 모노스테이트(Monostate)
모노스테이트는 싱글턴 패턴의 변형이다.
<br>
모노스테이트는 겉보기에는 일반 클래스와 동일하나 동작은 싱글턴처럼 동작한다.

```
class Printer
{
    static int id;
    
public:
    int get_id() const { return id; }
    void set_Id(int value) { id = value; }
};
```

언뜻 보기에는 보통 클래스처럼 보이지만 static 데이터를 이용하여 get/set 메서드가 구현되어 있다.
<br>
<br>
하지만 이 코드는 문제가 있는 코드이다.
<br>
사용자는 일반 클래스인 것으로 알고 Printer의 인스턴스를 만들지만, 실제로는 모든 인스턴스가 같은 데이터를 바라본다.
<br>
<br>
사용자는 아무런 의심 없이 두 개의 Printer 인스턴스를 만들어 서로 다른 id를 부여하였으나, 두 Printer의 id가 같아져 버리는 현상이다.
<br>
<br>
모노스테이트 방식은 어느 수준에서는 잘 동작하고 몇 가지 장점도 있다.
<br>
예를 들어 상속받기가 쉬워 다형성을 활용할 수 있다.
<br>
생존 주기도적절히잘 정의된다.
<br>
<br>
그래도 모노스테이트의 가장 큰 장점은 시스템에서 사용 중인, 이미 존재하는 객체를 이용할 수 있다는 점이다.
<br>
복수의 객체가 존재하는 것이 문제 되지 않는다면, 대규모 구조 변경 없이 기존 코드를 조금만 수정하여 기종네 잘 동작하던 기능에 변화를 일으키지 않고서도 싱글턴 특성을 추가할 수 있다.
<br>
<br>
반면 모노스테이트 방식의 단점은 명확한데, 이 방식은 코드 깊숙이 손을 대야 한다.
<br>
그리고 static 멤버를 사용하기 때문에 실제 객체가 인스턴스화되어 사용되는지와 관계없이 항상 메모리를 차지한다.
<br>
<br>
마지막으로 모노스테이트의 가장 큰 단점은 클래스의 필드들이 항상 get/set 메서드를 통해서만 접근되는 것으로 가정하는 점이다.

<br>

## 장단점

### 장점
- 클래스가 하나의 인스턴스만 갖는다는 점이 보장됨 
- 인스턴스에 대한 전역 접근 지점을 얻을 수 있음
- 싱글턴 객체는 처음 요청될 때만 초기화 됨

### 단점
- 단일 책임 원칙을 위반함
- 잘못된 디자인(컴포넌트들이 서로 엉켜 있는 경우)을 가릴 수 있음
- 다중 스레드 환경에서 여러 스레드가 싱글턴 객체를 여러 번 생성하지 않도록 특별한 처리가 필요함
- 클라이언트 코드를 유닛 테스트 하기 어려움

<br>

## 다른 패턴과의 관계


- 대부분의 경우 하나의 퍼사드 객체만 있어도 충분하므로 퍼사드 패턴의 클래스는 종종 싱글턴으로 변환될 수 있음
- 만약 객체들의 공유된 상태들이 단 하나의 플라이웨이트 객체로 줄일 수 있다면 플라이웨이트는 싱글턴과 유사해질 수 있음
- 추상 팩토리, 빌더, 프로토타입은 싱글턴으로 구현할 수 있음

<br>



## 요약
신중하게만 사용된다면 싱글턴 패턴 자체가 나쁜 패턴은 아니다.
<br>
하지만 일반적으로는 테스트와 리팩터링 용이성을 해칠 수 있다.
<br>
<br>
싱글턴을 꼭 사용해야만 한다면 직접적인 사용법을 피한다.
<br>
대신, 종속성을 주입하는 방식을으로 몬든 종속성이 전체 코드의 한 곳에서 관리될 수 있도록 형태를 활용하는 것이 바람직하다.





