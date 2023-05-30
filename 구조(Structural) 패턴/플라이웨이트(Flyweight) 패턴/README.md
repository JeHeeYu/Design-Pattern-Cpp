# 플라이웨이트 패턴 정리 내용

## 플라이웨이트(Flyweight) 패턴
플라이웨이트 패턴은 각 객체에 모든 데이터를 유지하는 대신 여러 객체들 간에 상태의 공통 부분들을 공유하여 사용할 수 있는 RAM에 더 많은 객체들을 포함할 수 있도록 하는 구조 디자인 패턴이다.
<br>
플라이웨이트 패턴은 많은 수의 가벼운 임시 객체들을 '스마트 참조'로 사용하는 것을 말하며 그러한 객체들을 플라이웨이트라고 부른다.
<br>
<br>
즉, 플라이웨이트는 대단히 많은 수의 매우 비슷한 객체들이 사용되어야 할 때 메모리 사용량을 절감하는 방법으로서 자주 사용된다.

## 시나리오
예를 들어 대규모 멀티 플레이가 지원되는 온라인 게임이 있다.
<br>
예를 들어 John Smith라는 이름은 너무 흔해서 두 명 이상 있을 가능성이 매우 높다.
<br>
<br>
만약 그러한 이름들을 계속해서 각각 따로 저쟁하 나간다면(ASCII 코드로) 매번 11 바이트의 공간을 낭비하게 된다.
<br>
<br>
대신에 이름을 한 번만 저장하고 같은 이름을 가진 사용자들이 그 이름을 포인터로 가지게 할 수 있다.
<br>
(그러면 64비트 머신 기준으로 포인터 용량 8바이트만 사용)
<br>
이것은 적지 않은 메모리 절감이다.
<br>
<br>
더 나아가 '성'과 '이름'으로 분할하는 것도 고려해볼만은 하다.
<br>
그렇게 하면 예를 들어, Fitzgerald Smith라는 이름이 각각 'Fitzerald'와 'Smith'라는 두 개의 포인터(16 바이트)로 표현된다.
<br>
<br>

이렇게 포인터가 소요하는 공간이 적지 않다.
<br>
2^64개의 서로 다른 이름이 존재하지는 않을 것이기 때문에 포인터 대신 인덱스를 사용하면 훨씬 더 공간을 절약할 수 있다.

<br>
<br>
그렇게 하기 위해 다음과 같이 인덱스로 사용될 타입을 typedef 한다.

```
typedef uint32_t key;
```

이 타입을 이용하면 User가 다음과 같이 정의된다.

```
struct User
{
    User(const string& first_name, const string& last_name)
        : first_name{add(first_name)}, last_name{add(last_name)} {}
        
    ...

protected:
    key first_name, last_name;
    static bimap<key, string> names;
    static key seed;
    static key add(const string& s) { ... }
};
```

이 코드에서 볼 수 있듯 생성자는 멤버 변수 first_name과 last_name을 add() 메서드의 리턴 값으로 초기화한다.
<br>
add()는 키/값 쌍을(키는 seed로 부터 생성) names 데이터 구조에 추가한다.
<br>
여기서는 names를 boost::bitmap(양방향 map)으로 만들고 있다.
<br>
<br>
이 데이터 구조는 중복을 피하고 새로운 것만 저장하기에 용이하다.
<br>
만약 이름이 bimap에 존재한다면 그 인덱스만 리턴하면 된다.
<br>
<br>
아래는 add()의 구현이다.

```
static key add(const string& s)
{
    auto it = names.right.find(s);
    if(it == names.right.end())
    {
        // 새로운 이름으로 추가
        names.insert({++seed, s});
        return seed;
    }
    return it->second;
}
```

이 코드는 '가져오거나 추가하기' 메커니즘의 표준적인 구현이다.
<br>
<br>
이제 key 타입의 protected 인덱스 변수가 아니라 실제 성과 이름을 제공하기 위해 적절한 get/set 메서드를 만들어야 한다.

```
const string& get_first_name() const
{
    return names.left.find(last_name)->second;
}

const string& get_last_name() const
{
    return names.left.find(last_name)->second;
}
```

이 멤버 함수들을 이용하면, 예를 들어 User의 스트림 출력 연산자를 만들고 싶을 때 아래와 같이 쉽게 구현할 수 있다.

```
friend ostream& operator<<(ostream& os, const User& obj)
{
    return os
        << "first_name: " << obj.get_first_name()
        << " last_name: " << obj.get_last_name();
}
```

<br>

## Boost.Flyweight
Boost 라이브러리를 이용하면 플라이웨이트 객체를 손쉽게 만들 수 있다.
<br>
boost::flyweight 타입은 그 이름이 암시 하듯이 플라이웨이트를 위한 것이다.
<br>
<br>
이 타입을 이용하면 공간을 절약하기 위한 플라이웨이트를 쉽게 생성할 수 있다.
<br>
<br>
User 클래스의 이름 멤버 변수에 boost::flyweight을 이용하면 아래와 같이 너무 간단하게 플라이웨이트가 적용된다.

```
struct User2
{
    flyweight<string> first_name, last_name;
    
    User2(const string& first_name, const string& last_Name)
        : first_name{first_name}, last_name{last_name} {}
};
```

<br>

## 섣부른 접근 방법

매우 단순하고 비효율적인 방법 중 하나는 모든 문자와 매칭되는, 즉 원본 텍스트와 같은 크기의 이진 배열을 만들어 두고 대문자로 만들 문자를 표시하는 것이다.
<br>
이 방법은 다음과 같이 구현될 수 있다.

```
class FOrmattedText
{
    string plainText;
    bool *caps;
    
public:
    explicit FormattedText(const string* plainText)
        : plainText{plainText}
    {
        caps = new bool[plainText.length()];
    }
    
    ~FormattedText()
    {
        delete[] caps;
    }
};
```

이를 기반으로 아래와 같이 특정 범위의 문자들을 대문자로 만드는 편의 함수를 만들 수 있다.

```
void capitalize(int start, int end)
{
    for(int i = start; i <= end; ++i)
        caps[i] = true;
}
```

그다음 스트림 출력 연산자가 대문자 변환을 표시하는 이진 마스크를 이용하게 한다.

```
friend std::ostream& operator<<(std::ostream& os, const FormattedText& obj)
{
    string s;
    
    for(int i = 0; i < obj.plainText.length(); ++i)
    {
        char c = obj.plainText[i];
        s += (obj.caps[i] ? toupper(c) : c);
    }
    
    return os << s;
}
```

이렇게 되면 어쨋든 목적하는 동작은 한다.

```
FormattedText ft("This is a brave new world");
ft.capitalize(10, 15);
cout << ft << endl;
// 출력 결과 "This is a BRAVE new World"
```

하지만 이 코드는 범위에 대한 시작 /끝 표시만을 요구 조건을 나타내고 있다.
<br>
텍스트의 모든 문자를 이진 플래그로 만든다는 것은 매우 큰 낭비이다.

<br>

## 플라이웨이트 구현

예를 들어 BetterFormattedText 클래스를 만든다.
<br>
상위 클래스와 플라이웨이트 클래스를 정의한다.
<br>
플라이웨이트 클래스는 상위 클래스 안에서 중첩 클래스로서 정의한다.

```
class BetterFormattedText
{
public:
    struct TextRange
    {
        int start, end;
        bool capitalize;
        
        bool covers(int position) const
        {
            return position >= start && position <= end;
        }
    };
    
private:
    string plain_text;
    vector<TextRange> formatting;
};
```

코드에서 볼 수 있듯이, TextRange는 포맷을 적용할 범위의 시작과 끝점만 저장한다.
<br>
그리고 대문자/소문자와 같이 적용할 포맷의 속성 정보도 저장한다.
<br>
<br>
포맷을 적용할지 여부에 대한 판단은 멤버 함수 covers() 하나에만 의존한다.
<br>
이 함수는 주어진 위치의 문자가 포맷의 적용을 받을 필요가 있는지 여부를 결정해준다.
<br>
<br>
BetterFormattedText는 플라이웨이트인 TextRange 객체들을 vector에 담고, 새로운 포매팅 요청을 적용하기 위한 준비를 한다.

```
TextRange& get_range(int start, int end)
{
    formatting.emplace_back(TextRange{ start, end });
    return *formatting.rbegin();
}
```
 
이 코드에서는 아래의 세 가지 작업이 일어난다.

1. 새로운 TextRange 생성
2. 생성된 TextRange가 vector로 이동
3. 마지막 항목에 대한 참조 리턴

이제 BetterFormattedText를 위한 operator<<를 구현할 수 있다.

```
friend std::ostream& operator<<(std::ostream& os, const BetterFormattedText& obj)
{
    string s;
    
    for(size_t i = 0; i < obj.plain_text.length(); i++)
    {
        auto c = obj.plain_text[i];
        for(const auto& rng : obj.formatting)
        {
            if(rng.covers(i) ** rng.capitalize)
                c = toupper(c);
            s += c;
        }
    }
    
    return os << s;
}
```

이 구현에서도 각각의 문자에 대해 속하는 범위가 있는지 검사한다.
<br>
어떤 범위든 속하는 범위가 있다면 대문자로 바꾼다.
<br>
<br>
이 구현은 섣부른 구현이 할 수 있는 것처럼 대문자 변환을 해내면서도 약간의 변화만으로 더 유연한 API를 제공한다.

```
BetterFormattedText bft("this is a brave new world");
btf.get_range(10, 15).capitalize = true;
cout << bft << endl;
// 출력 결과 "This is a BRAVE new world"
```

<br>

## 장단점

### 장점
- 프로그램에 유사한 객체들이 많을 경우 메모리를 절약할 수 있음

### 단점
- 코드가 복잡해지므로 개체(Entity)의 상태가 분리되었는지 알기 힘들 수도 있음


<br>

## 요약
플라이웨이트 패턴은 기본적으로 공간 절약을 위한 테크닉이다.
<br>
플라이웨이트가 실제 구현되는 형태는 매우 다양할 수 있다.
<br>
<br>
어떤 경우에는 플라이웨이트를 API에 사용하기 위한 토큰으로서 전달받아 플라이웨이트가 어디서 생성되었든 관계없이 활용하고 수정하게 할 수도 있고, 또는 사용자도 모르게 암묵적으로 플라이웨이트가 활용될 수도 있다.

