# 인터프리터 패턴 정리 내용


## 인터프리터(Interpreter) 패턴
인터프리터 패턴의 목적은 그 이름이 의미하듯이 입력 데이터를 해석하는 것이다.
<br>
텍스트 입력을 대상으로 할 때가 많지만 꼭 텍스트에 한정되지는 않는다.
<br>
<br>
인터프리터의 개념은 컴파일러 이론과 그에 연관된 대학 교과 과정들에서 중점적으로 다루어진다.
<br>
다음은 인터프리터를 이용해 처리되는 몇 가지 익숙한 예이다.
- 42나 1.234e12와 같은 숫자 리터럴은 바이너리로 저장하면 효율적임
- 텍스트에서 어떤 패턴을 찾고 싶을 때
- CSV, XML, JSON 또는 더 복잡한 경우를 포함하여 어떤 형태로든 구조화된 데이터는 실제 사용하기 위해 인터프리터가 필요
- 인터프리터 응용의 정점으로 프로그래밍 언어가 있는데, C나 파이썬 같은 프로그래밍 언어의 컴파일러 또는 인터프리터는 코드를 실제 실행할 수 있는 형태로 바꾸기 위해 코드를 이해하는 작업이 필요


<br>

## 산술 표현식의 계산

예를 들어 3+(5-4)와 같은 매우 단순한 수식 표현을 구문 분석해야 한다고 가정한다.
<br>
수식이 덧셈, 뺄셈, 괄호만 이용해서 표현되고 있다.

<br>

### 1. 렉싱(Lexing)
표현식을 해석하는 첫 번째 단계는 렉싱이라고 불린다.
<br>
렉싱은 문자열 입력을 토큰(Token)이라 불리는 단위로 나누어 나열한다.
<br>
<br>
토큰은 어떤 문법상에서 의미를 가지는 최소 단위이다.
<br>
산술 표현식은 토큰들의 단순한 나열로 변환될 수 있어야 한다.
<br>
<br>
이 예에서는 다음과 같은 것들이 토큰이 된다.
- 정수
- 연산자(덧셈 기호, 뺼셈 기호)
- 괄호(열림 또는 닫힘)

따라서 아래와 같이 구조를 정의할 수 있다.

```
struct Token
{
    enum Type { integer, plus, minus, lparen, rparen } type;
    string text;
    
    explicit Token(Type type, const string& text)
        : type{type}, text{text} {}
    
    friend ostream& operator<<(ostream& os, const Token& obj)
    {
        return os << obj.text;
    }
};
```

단순히 enum 값이 토큰은 아니다.
<br>
토큰은 값이 아니라 분류에 가깝다.
<br>
<br>
덧셈, 뺄셈 기호는 토큰으로 분류되는 순간 값도 정해진다고 볼 수 있지만, 숫자는 토큰으로 분류되더라도 텍스트에서 해당하는 부분이 별도로 저장되어야 값을 보존할 수 있다.
<br>
<br>
표현식이 std::string으로 주어져 있을 때, 텍스트를 토큰으로 분할한 결과를 Token을 vector형태로 리턴하는 것으로서 렉싱 절차를 정의할 수 있다.

```
vector<Token> lex(const string& input)
{
    vector<Token> result;
    
    for(int i = 0; i < input.size(); i++)
    {
        switch(input[i])
        {
        case '+':
            result.push_back(Token{ Token::plus, "+" });
            break;
        case '-':
            result.push_back(Token{ Token::minus, "-" });
            break;
        case '(':
            result.push_back(Token{ Token::minus, "(" });
            break;
        case ')':
            result.push_back(Token{ Token::minus, ")" });
            break;
        default:
        // 숫자 ???
        }
    }
}
```

덧셈, 뺄셈 기호처럼 확정적인 토큰은 파싱하기 쉽다.
<br>
하지만 숫자를 파싱하는 것은 쉽지 않다.
<br>
<br>
예를 들어 1을 만났을 때 연이어 오는 문자가 있는지 기다려야 한다.
<br>
<br>
이를 위해 별도의 처리 루틴을 만든다.

```
ostringstream buffer;
buffer << input[i];

for(int j = i + 1; j < input.size(); ++j)
{
    if(isdigit(input[j]))
    {
        buffer << input[j];
        ++i;
    }
    else
    {
        result.push_back(Token{ Token::integer, buffer.str() });
        break;
    }
}
```

위 코드가 하는 일은, 기본적으로 숫자가 읽히는 동안에는 계속해서 버퍼에 쌓는 것이다.
<br>
더 이상 숫자가 읽히지 않으면 작업을 종료하고 저장된 버퍼 전체를 토큰으로 하여 결과 벡터에 추가한다.

<br>

### 2. 파싱(Parsing)
파싱은 토큰의 나열을 의미 있는 단위로 바꾼다.
<br>
보통 의미 있는 단위란 객체 지향적인 데이터 구조를 말한다.
<br>
<br>
그러한 데이터 구조 위에 모든 타입을 감싸는, 즉, 토큰 정의 트리에서 최상단에 추상 부모 타입을 두면 편리한 경우가 많다.
<br>
<br>
아래는 그러한 목적으로 정의한 최상위 타입이다.

```
struct Element
{
    virtual int eval() const = 0;
};
```

여기서 eval() 멤버 함수는 해당 항목의 숫자 값을 구한다.
<br>
다음으로 숫자값을 저장하는 자식 타입을 정의한다.

```
struct Integer : Element
{
    int value;
    
    explicit Integer(const int value)
        : value(value) {}
        
    int eval() const override { return value; }
};
```

만약 어떤 Element가 Integer가 아니라면 반드시 연산자여야 한다.
<br>
이 예제에서의 덧셈, 뺄셈은 모두 이항 연산자이므로 모든 연산자 항목은 두 개의 하위 항목을 가진다.
<br>
<br>
예를 들어 2+3은 이 모델에서 다음과 같은 의사 코드로 표현될 수 있다.

```
struct BinaryOperation : Element
{
    enum Type { addition, subtraction } type;
    shared_ptr<Element> lhs, rhs;
    
    int eval() const override
    {
        if(type == addition)
            return lhs->eval() + rhs->eval();
            
        return lhs->eval() - rhs->eval();
    }
};
```

위 코드에서는 enum 클래스 대신 enum이 사용되고 있다.
<br>
이렇게 한 이유는 나중에 BinaryOperation::addition을 쓸 수 있게 하기 위함이다.
<br>
<br>
이후 이 파싱 과정이 해야 할 일은 토큰의 나열을 표현식에 대한 이진 트리로 바꾸는 것이다.
<br>
이 작업은 아래와 같이 구현할 수 있다.

```
shared_ptr<Element> parse(const vector<Token>& tokens)
{
    auto result = make_unique<BinaryOperation>();
    bool have_lhs = false;
    for(size_t i = 0; i < tokens.size(); i++) 
    {
        auto token = tokens[i];;
        switch(token.type)
        {
            // 각 토큰을 차례대로 처리
        }
    }
    
    return result;
}
```

위 코드에서 추가적인 설명이 필요한 부분은 have_lhs 변수 하나뿐이다.
<br>
이 코드는 트리를 만드려고 하는데, 이때 트리의 뿌리는 BinaryExpression 이어야 한다.
<br>
<br>
BinaryExpression은 정의에 따라 왼쪽과 오른쪽 항목들을 가진다.
<br>
그런데 숫자의 경우 표현식의 왼쪽에 위치해야 할지 오른쪽에 위치해야 할지 알 수가 없다.
<br>
따라서 have_lhs 변수를 두어 어느쪽에 위치해야 하는지 기록해두어 활용한다.
<br>
<br>
이제 토큰을 하나씩 살펴봐야 하는데, 먼저 숫자의 경우 정수 생성 코드로 넘겨져 텍스트로 표현된 정수가 숫자 값으로 변환된다.

```
case Token::integer:
{
    int value = boost::lexical_case<int>(token.text);
    auto integer = make_shared<Integgr>(value);
    
    if(!have_lhs) {
        result->lhs = integer;
        have_lhs = true;
    }
    else result->rhs = integer;
}
```

덧셈, 뺄셈 토큰은 단순히 현재 처리 중인 연산의 타입을 결정하기 때문에 처리하기 쉽다.

```
case Token::plus:
    result->type = BinaryOperation::addition;
    break;
case Token::minus:
    result->type = BinaryOperation::subtraction;
    break;
```

이후, 괄호 차례이다.
<br>
먼저 왼쪽 괄호는 오른쪽 괄호를 명시적으로 찾지 않는다.
<br>
즉, 닫힘 괄호를 만나면 그 괄호 쌍으로 구분되는 부분 표현식을 꺼내어 재귀적으로 parse()를 적용한다.
<br>
<br>
그리고 현재 처리 중인 표현식의 왼쪽 또는 오른쪽 항목으로 설정한다.

```
case Token::lparen:
{
    int j = i;
    for(; j < tokens.size(); ++j)
        if(tokens[j].type == Token::rparen)
            break; // 오른쪽 괄호 발견
            
    vector<Token> subexpression(&toknes[i + 1], &toknes[j]);
    auto element = parse(subexpression); // 재귀 호출
    
    if(!have_lhs)
    {
        result->lhs = element;
        have_lhs = true;
    }
    else result->rhs = element;
    i = j; // 
}
```


<br>


### 3. 렉서와 파싱의 사용
lex()와 parse()가 모두 구현되고 나면, 표현식을 파싱하여 그 값을 계산해낼 수 있다.

```
string input{ "(13-4)-(12+1)" };
auto tokens = lex(input);
auto parsed = parse(tokens);
cout << input << " = " << parsed->eval() << endl;
// 출력 결과 "(13-4)-(12+1) = -4"
```


<br>

## 요약
인터프리터 디자인 패턴은 그리 흔하게 사용되지는 않는다.
<br>
파서를 직접 만들어야 할 상황이 그렇게 많지는 않다.
<br>
<br>
심지어 컴퓨터 과학 전공 정규 교과에서 관련 과목이 없어지는 경우도 생기고 있다.
<br>
더욱이, 언어 설계나 정적 코드 분석 툴을 만드는 일을 할 계획이 아니라면 파서 개발 전문가를 위한 일자리도 드물다.








