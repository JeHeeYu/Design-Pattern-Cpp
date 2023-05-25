# 빌더 패턴 정리 내용

## 빌더(Build) 패턴
빌더 패턴이란 생성이 까다로운 객체를 쉽게 처리하기 위한 패턴을 말한다.
<br>
즉, 생성자 호출 코드 단 한 줄로 객체를 생성할 수 없는 객체를 다룬다.
<br>
<br>
그러한 타입의 객체들은 조합으로 구성되거나, 상식적인 것을 벗어난 까다로운 로직을 필요로 한다.
<br>


## 시나리오
예를 들어 웹 페이즈를 그리기 위한 컴포넌트들을 생성해야 한다.
<br>
먼저 단순하게 단어를 나타내는 항목 두 개를 html의 비순차('< ul >') 리스트('< li >') 태그로 출력해 보면 아래와 같이 간단하게 출력할 수 있다.

```
string words[] = { "hello", "world" };
ostring oss;
oss << "<ul>";
for(auto w : words)
    oss << " <li>" << w << "</li>";
oss << 
```
이 코드는 목적한 대로 출력을 하기는 하나, 융통성이 없는 코드이다.
<br>
항목마다 앞에 점을 찍거나 순서대로 번호를 매겨야 한다면 이 코드를 손쉽게 수정하기는 어렵다.
<br>
<br>
대안으로 객체 지향(OOP) 스타일을 적용할 수 있는데, HtmlElement 클래스를 정의하여 각 html 태그에 대한 정보를 저장한다.

```
struct HtmlElement
{
    string name;
    string text;
    vector<HtmlElement> elements;
    
    HtmlElement() {}
    HtmlElement(const string& name, const string& text)
      : name(name), text(text) { }
      
    string str(int indent = 0) const
    {
        //  컨텐츠를 양식에 맞추어 출력
    }
};
```

이러한 접근 방법을 활용하여 출력 양식이 좀 더 쉽게 드러나도록 리스트를 생성할 수 있다.

```
string words[] = { "hello", "world" };
HtmlElement list{"ul", ""};
for(auto w : words)
    list.elements.emplace_back{HtmlElement{"li", w}};
printf(list.str().c_str());
```

이 코드는 OOP에 기반하여 항목 리스트를 표현하고 있다.
<br>
양식을 제어하기가 좀 더 쉬우면서도 목적하는 출력을 할 수 있다.
<br>
<br>
하지만 각각의 HtmlElement를 생성하는 작업이 편하다고 볼수 없는데, 이 부분을 빌더 패턴을 활용하여 개선할 수 있다.

<br>

## 단순한 빌더

빌더 패턴은 단순하게 개별 객체의 생성을 별도의 다른 클래스에 위임한다.
<br>
아래 코드는 빌더 패턴으로 구현하는 예제 코드이다.

```
struct HtmlBuilder
{
    HtmlElement root;

    HtmlBuilder(string root_name) { root.name = root_name; }

    void add_child(string cihld_name, string child_text)
    {
        HtmlElement e{ child_name, child_text };
        root.elements.emplace_back(e);
    }

    string str() { return root.str(); }
};
```

HtmlBuilder는 HTML 구성 요소의 생성만을 전담하는 클래스이다.
<br>
add_child() 메서드는 현재 요소에 자식 요소를 추가하는 목적으로 사용된다.
<br>
각 자식 요소는 이름/텍스트 쌍을 가지며, 아래와 같이 사용할 수 있다.

```
HtmlBuilder builder{ "ui " };
builder.add_child("li", "hello");
builder.add_child("li", "world");
cout << builder.str() << endl;
```

그런데 add_child() 메서드의 리턴 값은 사용되는 곳 없이 void로 선언되어 있다.
<br>
여기서 리턴 값을 활용하면 좀 더 편리한 흐름식 인터페이스(Fluent Infertace) 스타일의 빌더를 만들 수 있다.

<br>

## 흐름식 빌더
예를 들어 아래와 같은 코드가 있다.

```
struct HtmlBuilder
{
    HtmlElement root;

    HtmlBuilder(string root_name) { root.name = root_name; }

    void add_child(string cihld_name, string child_text)
    {
        HtmlElement e{ child_name, child_text };
        root.elements.emplace_back(e);

        return *this;
    }

    string str() { return root.str(); }
};
```

이렇게 빌드 자기 자신이 참조로서 리턴되기 때문에 다음과 같이 메서드들이 꼬리를 무는 호출이 가능해진다.
<br>
이러한 형태로 호출하는 것을 흐름식 인터페이스(Fluent Interface)라고 한다.

```
HtmlBuilder builder{ "ui: };
builder.add_child("li", "hello").add_child("li", "world");
cout << builder.str() << endl;
```

리턴을 참조로 할지 포인터로 할지는 개발자의 자유이다.
<br>
호출 체인에 -> 연산자를 사용하고 싶다면 아래와 같이 add_child() 함수를 수정할 수 있다.

```
HtmlBuilder* add_child(string cihld_name, string child_text)
{
HtmlElement e{ child_name, child_text };
root.elements.emplace_back(e);

return *this;
}
```

위와 같이 수정한다면 아래와 같이 -> 연산자를 사용할 수 있다.

```
HtmlBuilder *builder = new HtmlBuilder("ul");
builder->add_child("li", "hello")->add_child("li", "world");
```

<br>

## 빌더 패턴의 이미

사용자가 빌더 클래스를 사용해야 한다는 것을 알 수 있게 하는 방법은 두 가지가 있다.
<br>
첫 번째 방법은 사용 안하면 객체 생성이 불가능하도록 강제하는 것으로, 아래와 같이 사용할 수 있다.

```
struct HtmlElement
{
    string name;
    string text;
    vector<HtmlElement> elements;
    const size_t index_size = 2;

static unique_ptr<HtmlBuilder> build(const string& root_name)
{
    return make_unique<HtmlBuilder>(root_name);
}

protected: // 모든 생성자 숨기기
    HtmlElement() {}
    HtmlElement(const string& name, const string& text)
      : name{name}, text{text} {}
};
```

이러한 접근 방법은 두가지 방법으로 이루어진다.
<br>
첫 번째는 모든 생성자를 숨겨서 사용자가 접근할 수 없게 하는 것이다.
<br>
두 번째는 생성자를 숨긴 대신 HtmlLement 자체에 팩터리 메서드를 두어 빌더를 생성할 수 있게 한다.
<br>
패터리 메서드 역시 static 메서드로 만든다.
<br>
<br>
아래는 사용자 입장에서의 활용 예시이다.

```
auto builder = HtmlElement::build("ui");
builder.add_child("li", "hello").add_child("li", "world");
cout << builder.str() << endl;
```

그리고 아래는 HtmlElement 연산자(피연산자는 빌더임)의 구현 방법이다.

```
struct HtmlBuilder
{
    operator HtmlElement() const { return root; }
    HtmlElement root;
}
```

단순히 root를 리턴하는 대신 return std::move(root)와 같이 이동 시멘틱을 이용할 수도 있다.
<br>
<br>
위와 같이 연산자가 추가됨으로써 사용자는 다음과 같은 코드를 작성할 수 있게 된다.

```
HtmlElement e = HtmlElement::build("ul")
    .add_child("li", "hello")
    .add_child("li", "world");
cout << e.str() << endl;
```

<br>

## 그루비-스타일 빌더(Groovy-Style Builder)
Groovy, Kotlin 등과 같은 프로그래밍 언어들은 도메인에 특화된 언어(DSL, Domain Specific Language)의 생성을 지원한다.
<br>
즉, 어떤 절차를 해당 응용의 성격에 맞추어 쉽게 기술할 수 있도록 새로운 언어를 정의할 수 있는 문법적 기능이 제공된다.
<br>
<br>
여기서 C++도 초기화 리스트 기능을 사용한다면 DSL을 효과적으로 만들 수 있다.
<br>
<br>
예를 들어 아래의 HTML 태그를 구현한 C++ 코드가 있다.

```
struct Tag
{
    std::string name;
    std::string text;
    std::vector<Tag> children;
    std::vector<std::pair<std::string, std::string>> attributes;

    friend std::ostream& operator<<(std::ostream& os, const Tag& tag)
    {
        // 구현부
    }
};
```

여태까지 이름과 텍스트, 자식 태그(내부 태그) 또는 HTML 속성을 저장할 수 있는 태그 클래스이다.
<br>
그리고 포맷팅하여 출력하는 기능도 있다.
<br>
<br>
클래스를 생성하기 위해 두 가지 경우를 고려해야 한다.

- 태그가 이름과 텍스트로 초기화되는 경우(예 : 리스트 항목)
- 태그가 이름과 자식의 집합으로 초기화되는 경우

두 번째 경우, 자식의 집합을 처리하기 위해 std::vector 타입을 파라미터로 사용한다.
<br>
다음은 구현한 예제 코드이다.

```
struct Tag
{
    ...

protected:
    Tag(const std::string& name, const std::string& text)
      : name{name}, text{text} {}

    Tag(const std::string& name, const std::vector<Tag>& children)
      : name{name}, children{children} {}
};
```

이제 이 Tag 클래스를 상속받을 준비가 되어 있으며, 상속받을 때 유효한 HTML 태그여야만 한다.
<br>
<br>
이제 아래와 같이 사용할 수 있으며 하나는 문단을, 다른 하나는 이미지를 나타낸다.

```
struct P : Tag
{
    explicit P(const std::string& text)
      : Tag{"p", text} {}

    P(std::initializer_list<Tag> children)
      : Tag("p", children) {}
};

struct IMG: Tag
{
    explicit IMG(const std::string& url)
      : Tag{"img", ""}
    {
        attributes.emplace_back({"src", url});
    }
};
```

위 코드에 따르면 p 태그의 경우 텍스트 또는 자식 태그의 집합만 가질 수 있다.
<br>
반면에 이미지 태그는 다른 태그를 가질 수 없고 이미지의 주소를 가리키는 속성 img를 가져야만 한다.
<br>
<br>
앞서 구현한 생성자들과 모던 C++의 초기화 문법 덕분에 아래와 같이 C++ DSL로 HTML을 표현할 수 있다.


```
std::cout <<

    P {
        IMG { "http://url" }
    }

    << std::endl;
```

이러한 DSL 태그는 add_child() 호출이 전혀 필요 없으며, 이 DSL은 다른 태그들로도 쉽게 확장할 수 있다.

<br>

## 컴포지트 빌더(Composite Builder)
빌더에 관련한 마지막 예제로 객체 하나를 생성하는 복수의 빌더가 사용되는 경우이다.
<br>
아래 코드는 개인 신상 정보를 저장하는 프로그램의 클래스이다.

```
class Person
{
    // 주소
    std::string street_address, post_code, city;

    // 직업
    std::string company_name, position;
    int annual_income = 0;

    Person() {}
};
```

Person 클래스에는 두 종류의 데이터 주소와 직업 정보가 있다.
<br>
만약 빌더를 각 정보마다 따로 두고 싶다면 이를 위해 컴포지트 빌더를 고안해야 한다.
<br>
<br>
이를 위해 4개의 클래스로 빌더를 정의해야 하며 아래는 클래스 관계도를 설명한 UML 이다.

<br>

![image](https://github.com/JeHeeYu/Book-Reviews/assets/87363461/245b7400-752c-4204-86dd-5b83c4258d13)


<br>

첫 번째 빌더 클래스는 PersonBuilderBase이다.

```
class PersonBuilderBase
{
protected:
    Person& person;
    explicit PersonBuilderBase(Person& person)
      : person{ person } {}
      
public:
    operator Person() 
    {
        return std::move(person);
    }
    
    // 빌더의 한 측면
    PersonAddressBuilder lives() const;
    PersonJobBuilder works() const;
};
```

각 멤버들의 설명은 다음과 같다.
- 참조 변수 person은 현재 생성되고 있는 객체에 대한 참조를 담는다. 이 변수는 하위 빌더들을 위해 의도적으로 만들어 졌다. 또한 베이스 클래스는 단지 참조만 가질 뿐 생성된 객체는 가지지 않는다.
- 참조 대입을 하는 생성자는 protected로 선언하여 그 자식 클래스들(PersonAddressBuilder, PersonJobBuilder)에서만 이용할 수 있게 한다.
- operator Person은 테크닉이다.
- lives()와 works() 함수는 하위 빌더의 인터페이스를 리턴하며 하위 빌더들은 각각 주소와 직업 정보를 초기화한다.

이 빌더 클래스에서 마지막으로 남아 있는 구멍은 생성하고 있는 객체 자체이다.
<br>
생성 중인 객체는 PersonBuilderBase 클래스를 상속받는 PersonBuilder에 존재한다.
<br>
<br>
즉, PersonBuilder가 실제 사용자가 이용할 클래스이다.

```
class PersonBuilder : public PersonBuilderBase
{
    // 생성 중인 객체
    Person p;   
    
public:
    PersonBuilder() : PersonBuilderBase{p} {}
};
```

여기서 실체 객체가 생성된다.
<br>
이 클래스는 사실 빌더 베이스를 상속받을 필요는 없지만 빌더 구동 절차를 초기화하기 쉽도록 편의상 이렇게 사용하고 있다.
<br>
public 생성자의 protected 생성자를 다르게 둔 이유를 알기 위해 빌더의 구현을 다르게 구현하여 살펴봐야 한다.

```
class PersonAddressBuilder : public PersonBuilderBase
{
    typedef PersonAddressBuilder self;
public:
    explicit PersonAddressBuilder(Person& person)
      : PersonBuilderBase{ person } {}
      
      self& at(std::string street_address)
      {
          person.street_address = street_address;
          
          return *this;
      }
      
      self& with_postcode(std::string post_code) { ... }
      self& in(std::string city) { ... }
};
```

코드에서 보여지는 바와 같이 PersonAddressBuilder는 person의 주소를 생성하는데 Fluent Interface 스타일을 지원한다.
<br>
PersonAddressBuilder는 PersonBuilderBase를 상속받고 있고 Person의 참조를 인자로 하여 베이스의 생성자를 호출한다.
<br>
<br>
PersonAddressBuilder가 PersonBuilder를 상속받을 것만 같지만 그렇게 하지 않는다.
<br>
만약 그렇게 한다면 한 개의 객체만 생성해야 함에도 불구하고 여러 개의 Person이 인스턴스화 되어 버린다.
<br>
<br>
PersonJobBuilder도 같은 방식으로 구현된다.
<br>
이 두 클래스와 PersonBuilder 모두 Person 클래스 안에서 frient로 선언되어 Person이 private 멤버에 접근할 수 있게 한다.
<br>
<br>
아래 방법은 사용자가 이 빌더를 사용하는 방법이다.

```
Person p = Person::create()
  .lives().at("123 London Road")
          .with_postcode("SW1 1GB")
          .in("London")
  .works().at("ProgramSoft")        
          .as_a("Consultant")
          .earning(10e6);
```

create 함수를 이용해 빌더를 얻고, lives() 함수를 이용해 PersonAddressBuilder를 얻는다.
<br>
그리고 주소 정보를 설정한 다음, 바로 works()를 호출하여 PersonJobBuilder로 전환하고 직업 정보를 설정한다.
<br>
<br>
생성 절차가 완료되면 이전에 사용했던 것과 같은 테크닉으로 생성이 완료된 Person 객체를 얻는다.
<br>
생성된 객체가 이전될 때 std::move()를 사용하기 때문에, 이전한 빌더에서는 더 이상 그 인스턴스에 접근할 수 없다

<br>

## 장단점

### 장점

- 객체들을 단계별로 생성하거나 생성 단계들을 연기하거나 재귀적으로 단계들을 실행할 수 있음
- 제품들의 다양한 표현을 만들 때 같은 생성 코드를 재사용할 수 있음
- 단일 책임 원칙. 제품의 비즈니스 로직에서 복잡한 생성 코드를 고립시킬 수 있음

### 단점
- 패턴이 여러 개의 새 클래스들을 생성해야 하므로 코드의 전반적인 복잡성이 증가함


<br>


## 다른 패턴과의 관계

- 많은 디자인은 복잡성이 낮고 자식 클래스들을 통해 더 많은 커스터마이징이 가능한 팩토리 메서드로 시작해 더 유연하면서도 더 복잡한 추상 팩토리, 프로토타입 또는 빌더 패턴으로 발전해 나감
- 빌더는 복잡한 객체들을 단계별로 생성하는 데 중점을 두며 추상 팩토리는 관련된 객체들의 패밀리들을 생성하는 데 중점을 둠.<br>추상 팩토리는 제품을 즉시 반환하지만 빌더는 제품을 가져오기 전에 몇 가지 추가 생성 단계들을 실행할 수 있도록 함
- 복잡한 복합체 패턴 트리를 생성할 때 빌더를 사용할 수 있는데, 그 이유는 빌더의 생성 단계들을 재귀적으로 작동하도록 프로그래밍할 수 있기 때문
- 빌더를 브리지와 조합할 수 있으며, 디렉터 클래스는 추상화의 역할을 하고 다양한 빌더들은 구현의 역할을 함
- 추상 팩토리, 빌더, 프로토 타입은 모두 싱글턴으로 구현할 수 있음

<br>

## 요약

빌더 패턴의 목적은 여러 복잡한 요소들의 조합이 필요한 객체를 생성해야 하거나 또는 여러 개의 다양한 객체 집합을 생성해야 할 때 객체 생성만을 전담하는 컴포넌트를 정의하여 객체 생성을 간편하게 하는 것이다.
<br>
<br>
빌더 패턴에서 빌더의 특징을 정리하면 다음과 같다.

- 흐름식 인터페이스를 이용하면 복잡한 생성 작업을 한 번의 호출 체인으로 처리할 수 있다.<br>Fluent Interface를 지원하려면 빌더 함수가 this 또는 *this를 리턴해야만 한다.
- 사용자에게 빌더 API의 사용을 강제하기 위해, 타겟 객체의 생성자를 외부에 접근하지 못하게 설정하고 빌더에 static create() 함수를 추가하여 생성된 객체를 리턴하게 한다.
- 적절한 연산자를 정의하여 객체 자체적으로 빌더의 사용을 강제할 수도 있다.
- 유니폼 초기화 문법을 이용하여 C++에서도 그루비 스타일 빌더를 만들 수 있다.<br>이러한 접근 방법은 매우 일반적이며 다양한 DSL(도메인 특화 언어)을 만들 수 있다.
- 빌더 하나의 인터페이스가 여러 하위 빌더를 노출할 수도 있다.<br>상속과 Fluent Interface를 요령있게 활용하면, 여러 빌더를 거치는 객체 생성을 쉽게할 수 있다.

빌더 패턴의 활용은 <b>객체의 생성 과정이 충분히 복잡할 때</b>에만 의미가 있다.
<br>
<b>혼동될 여지가 없는 쉬운 생성자들과 몇 가지 정도의 파라미터만으로 생성할 수 있는 객체라면 굳이 빌더를 사용할 필요가 없다.</b>



