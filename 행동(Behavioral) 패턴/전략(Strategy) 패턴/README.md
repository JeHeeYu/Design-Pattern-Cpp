# 전략 패턴 정리 내용

## 전략(Strategy) 패턴
전략 패턴은 알고리즘들의 패밀리를 정의하고, 각 패밀리를 별도의 클래스에 넣은 후 그들의 객체들의 상호교환을 할 수 있도록 하는 행동 디자인 패턴이다.
<br>
이 디자인 패턴은 런타임에 변경 가능한 동적 형태와 템플릿으로 컴파일 타임에 결정되는 정적 형태가 있다.

<br>

## 동적 전략
먼저 예제로 단순한 텍스트의 목록을 아래의 두 가지 포맷으로 렌더링하는 것이다.

```
enum class OutputFormat
{
    markdown,
    html
};
```

'전략'의 골격은 아래의 베이스 클래스로 정의된다.

```
struct ListStrategy
{
    virtual void start(ostringstream& oss) {};
    virtual void add_list_item(ostringstream& oss, const string& item) {}
    virtual void end(ostringstream& oss) {};
};
```

이제 텍스트 처리 컴포넌트를 구현해야 한다.
<br>
이 컴포넌트는 목록 처리를 위한 전용 메서드 append_list()를 갖는다.

```
struct TextProcessor
{
    void append_list(const vector<string> items)
    {
        list_strategy->start(oss);
        for(auto& item : items)
            list_strategy->add_list_item(oss, item);
        list_strategy->end(oss);
    }
    
private:
    ostringstream oss;
    unique_ptr<ListStrategy> list_strategy;
};
```

위 코드에서 oss는 결과가 출력될 버퍼이다.
<br>
append_list()는 목록을 렌더링 하는 과정을 정의할 뿐만 아니라 렌더링에 적용할 전략도 가진다.
<br>
<br>
이제 목록 렌더링 전략을 구현해야 한다.
<br>
먼저 Html을 렌더링하기 위해 HtmlListStrategy는 다음과 같이 구현할 수 있다.

```
struct HtmlListStrategy : ListStrategy
{
    void start(ostringstream& oss) override
    {
        oss < "<ul>\n";
    }
    
    void end(ostringstream& oss) override
    {
        oss << "</ul>\n";
    }
    
    void add_list_item(ostringstream& oss, const string& item) override
    {
        oss << "<li>" << item << "</li>\n";
    }
};
```

오버라이딩해야 할 메서드들을 구현함으로써 목적을 처리하는 과정의 골격 사이를 채운다.
<br>
Markdown도 구현해야 한다.
<br>
MarkdownListStrategy 전략도 비슷한 방식으로 구현한다.
<br>
<br>
그런데 Markdown 방식에서는 열림/닫힘 태그가 필요 없기 때문에 add_list_item() 메서드의 오버라이딩만 구현하면 된다.

```
struct MarkdownListStrategy : ListStrategy
{
    void add_list_item(ostringstream& oss, const string& item) override
    {
        oss << " * " << item << endl;
    }
};
```

이제 TextProcessor를 이용하여 서로 다른 절냑에 목록을 입력하여 서로 다른 렌더링 결과를 얻을 수 있다.
<br>
예를 들어 다음과 같이 이용한다.


```
TextProcessor tp;
tp.set_output_format(OutputFormat::markdown);
tp.append_list({"foo", "bar", "baz"});
cout << tp.str() << endl;

// 출력 결과:
// * foo
// * bar
// * baz
```

전략을 런타임에 목적에 따라 선택할 수도 있다.
<br>
이 때문에 이 구현을 '동적 전략 패턴' 이라고 부른다.
<br>
<br>
set_output_format() 함수에서 전략의 선택이 이루어진다.
<br>
이 함수의 구현은 아래와 같이 단순하다.

```
void set_output_format(const OutputFormat format)
{
    switch(format)
    {
        case OutputFormat::markdown:
            list_strategy = make_unique<MarkdownListStrategy>();
            break;
        case OutputFormat::html:
            list_strategy = make_unique<HtmlListStrategy>();
            break;
    }
}
```

클라이언트에서 아래와 같이 렌더링 전략을 바꿀 수 있다.

```
tp.clear();
tp.set_output_format(OutputFormat::html);
tp.apend_list({"foo", "bar", "baz")};
cout << tp.str() << endl;

// 출력 결과:
// <ul>
//   <li>foo</li>
//   <li>bar</li>
//   <li>baz</li>
//  </ul>
```

<br>

## 정적 전략
템플릿 덕분에 어떤 전략이든 자동으로 타입에 맞추어 적용할 수 있다.
<br>
아래와 같이 TextProcessor 클래스에 약간의 수정만 가하면 된다.

```
template <typename LS>
struct TextProcessor
{
    void append_list(const vector<string> items)
    {
        list_strategy.start(oss);
        for(auto& item : items)
            list_strategy.add_list_item(oss, item);
        list_strategy.end(oss);
    }
    
private:
    ostringstream oss;
    LS list_strategy;
};
```

위 코드에서 정적으로 전략이 연결될 수 있게 하는 부분은 템플릿 파라미터 LS가 거의 전부이다.
<br>
그리고 앞서 포인터를 사용했던 부분을 타입 LS를 사용하도록 바꾸었다.
<br>append_list()의 결과는 동일하다.

```
// markdown 렌더링 전략 사용
TextProcessor<MarkdownListStrategy> tpm;
tpm.append_list({"foo", "bar", "baz"});
cout << tmp.str() << endl;

TextProcessosr<HtmlListStrategy> tph;
tph.append_list({"foo", "bar", "baz"});
cout << tph.str() << endl;
```

위 예제의 출력 결과는 동적 전략의 예제와 동일하다.
<br>
하지만 두 개의 TextProcessor 인스턴스를 만들고 있는 점을 유의해야 한다.
<br>
목록 렌더링 전략마다 개별적으로 인스턴스를 가져야만 한다.

<br>

## 장단점

### 장점
- 런타임에 한 객체 내부에서 사용되는 알고리즘들을 교환할 수 있음
- 알고리즘을 사용하는 코드에서 알고리즘의 구현 세부 정보들을 고립할 수 있음
- 상속을 합성으로 대체할 수 있음
- Context를 변경하지 않고도 새로운 전략들을 도입할 수 있음

### 단점
- 알고리즘이 몇 개밖에 되지 않고 거의 변하지 않는다면 프로그램을 지나치게 복잡하게 만들 수 있음
- 클라이언트들은 적절한 전략을 선택할 수 있어야 함


<br>

## 요약
전략 디자인 패턴은 알고리즘의 골격만을 정의하고 세부 구현은 컴포지션으로서 특정 전략을 선택적으로 채워 넣을 수 있게 한다.
<br>
이 접근 방법은 아래와 같이 두 가지 실현 방법이 있다.
- 동적 전략은 사용될 전략을 단순히 포인터 또는 참조자를 가지며 전략을 바꾸고 싶을 때는 참조를 변경하면 됨
- 정적 전략은 컴파일 시점에 전략이 선택되어 고정되도록 하며 나중에 전략을 바꿀 수 없음

마지막으로 적용할 수 있는 전략의 목록을 제한할 수도 있다.
<br>
일반화된 ListStraegy 인자 대신, 적용 가능한 전략의 목록을 지정하여 std::variant로 전략을 전달할 수도 있다.
