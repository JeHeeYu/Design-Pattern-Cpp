# 상태 패턴 정리 내용

## 상태(State) 패턴
상태 패턴은 객체의 내부 상태가 변경될 때 해당 객체가 그의 행동을 변경할 수 있도록 하는 행동 디자인 패턴이다.
<br>
객체가 행동을 변경할 때 객체가 클래스를 변경한 것처럼 보일 수도 있다.
<br>
<br>
상태 디자인 패턴은 "상태가 동작을 제어하고 상태는 바뀔 수 있다"라는 매우 단순한 아이디어이다.
<br>
이 패턴은 상태 변경 주체가 누구인지 규정하고 있지 않다.
<br>
<br>
이 패턴의 구현에는 기본적으로 다음의 두 가지 방법이 있다.
- 동작을 가지는 실제 클래스 상태를 정의하고, 그 동작들은 상태가 이전될 때 클래스의 변화에 따라 변경됨
- 상태와 상태 전이를 단순히 enum 타입처럼 식별자의 나열로 정의하고 실제 상태 변화는 상태 머신(State Machine)이라는 특별한 컴포너트를 두어 수행

실제로는 두 방법 중 두 번째 방법이 훨씬 더 흔하게 사용된다.

<br>

## 상태 기반 상태 전이
가장 쉬운 예로 전등이 있다.
<br>
전등은 꺼짐, 켜짐 두 상태가 있다.
<br>
<br>
먼저 전등 스위치를 모델링해야 한다.
<br>
상태 값과 다른 상태로 전이할 수단을 가지면 된다.

```
class LightSwitch
{
    State *state;

public:
    LightSwitch()
    {
        state = new OffState();
    }
    
    void set_state(State* state)
    {
        this->state = state;
    }
};
```

이제 상태를 정의해야 한다.
<br>
다음과 같이 하나의 클래스로 정의한다.

```
struct State
{
    virtual void on(LightSwitch *ls)
    {
        cout << "Light is already on\n";
    }
    virtual void off(LightSwitch *ls)
    {
        cout << "Light is Already off\n";
    }
};
```

이 구현은 직관적인 것과는 거리가 멀며, 이해하려면 따로 설명이 필요하다.
<br>
전등을 켜고 끄는 시나리오에서는 그 상태를 별도의 클래스로 정의하는 것 자체가 그렇게 상식적이지 않다.
<br>
<br>
왜냐하면 첫 번째로 State가 추상 타입이 아니다.
<br>
언뜻 생각하기에는 추상 타입의 상태에는 접근할 방법이 없다고 생각할 수도 있지만 실제로는 그렇지 않다.
<br>
<br>
두 번째로 State 자체적으로 상태 전이를 할 수 있도록 하고 있다.
<br>
이것은 전이에 대한 상식적인 개념과 거리가 멀다.
<br>
<br>
전등 스위치는 상태를 변경 시킨다.
<br>
상태 자체가 상태를 변경시키지는 않는다.
<br>
<br>
세 번째로 State::on/off 동작은 이미 그 상태가 있다는 것을 가정하고 있다.
<br>
이 부분은 직관적인 것을 떠나서 가장 큰 문제가 된다.
<br>
<br>
이후 on/off 각각에 해당하는 State를 정의한다.

```
struct OnState : State
{
    OnState() { cout << "Light turned on\n"; }
    void off(LightSwitch* ls) override;
};

struct OffState : State
{
    OffState() { cout << "Light turned off\n"; }
    void on(LightSwitch* ls) override;
};
```

OnState::off와 OnState::on의 구현에서 상태 스스로 상태 전환을 할 수 있게 하고 있다.
<br>
즉, 아래와 같은 구현이 가능하다.

```
void OnState::off(LightSwitch* ls)
{
    cout << "Switching light off...\n";
    ls->set_state(new OffState());
    delete this;
} // on도 동일
```

여기서 전환이 일어난다.
<br>
이 구현은 보통의 C++에서 보기 힘든 delete this; 구문을 가지고 있다.
<br>
이 구문은 해당 객체가 이미 생성이 완료되었다는 대단히 위험한 가정을 하고 있다.
<br>
<br>
이 부분을 스마트 포인터를 이용해서 수정할 수는 있지만 포인터의 사용과 힙 메모리 할당이 일어난다는 것 자체가 여기서 직접적인 상태 소멸이 일어남을 명확히 보여준다.
<br>
<br>
만약 상태가 소멸자를 가진다면 여기서 호출될 것이고 그 안에서 추가적인 정리 작업을 할 수도 있다.
<br>
<br>
다연하지만 전등 스위치를 통해서 상태 전환도 가능해야 한다.
<br>
따라서 아래와 같이 구현한다.

```
class LightSwitch
{
    void on() { state->on(this); }
    void off() { sate->off(this); }
};
```

이러한 구현을 이용하여 아래와 같은 시나리오를 실행할 수 있다.

```
LightSwitch ls; // 전등 꺼짐
ls.on();     // 전등 켜기 상태로 전환
             // 전등 켜짐
ls.off();    // 전등 끄기 상태로 전환
             // 전등 꺼짐
ls.on();     // 전등이 이미 꺼져 있음
```

이러한 방법은 직관적이지 않다.
<br>
목적하는 상태를 알려주면 그 상태로 바뀌는 것이 더 직관적이다.
<br>
<br>
OffState에서 OnState로의 상태 전이를 도식화해서 표현하면 다음과 같다.

```
            LightSwitch::on() -> OffState::on()
OffState -----------------------------------------> OnState
```

그리고 OnState에서 OnState으로의 상태 전이는 베이스 클래스 State를 이용하여 이미 그 상태에 있다는 것을 알려주기 때문에 아래와 같이 도식화될 수 있다.

```
          LightSwitch::on() -> State::on()
OnState ----------------------------------> OnState
```

<br>

## 수작업으로 만드는 상태 머신
예를 들어 전화기를 위한 상태 머신을 정의한다.
<br>
이 전화기는 스마트폰이 아닌 구식 전화기이다.
<br>
<br>
이 예제에서는 상태와 상태 전이를 enum 값으로 정의하는 예제이다.
<br>
먼저 전화기의 상태들을 나열하여 정의해본다.

```
enum class State
{
    off_hook,   // 수화기 든 상태
    connection, // 연결 시도 상태
    connected,  // 연결된 상태
    on_hold,    // 대기 상태
    on_hook     // 수화기 내린 상태
};
```

이제 상태 간의 전이를 나열하여 정의한다.

```
enum class Trigger
{
    call_dialed,      // 전화 걸기
    hung_up,          // 전화 끊기
    call_connected,   // 전화 연결됨
    placed_on_hold,   // 대기
    taken_off_hold,   // 대기 종료
    left_message,     // 메시지 남기기
    stop_using_phone  // 전화 사용 종료
};
```

상태 머신에서 상태 간 전이가 어떤 규칙으로 이루어져야 하느지에 대한 정보를 어딘가에 저장해야 한다.

```
map<State, vector<pair<Trigger, State>>> rules;
```

이 부분은 그다지 매끄럽지 않은데, map의 키는 상태 전이의 출발 상태이고, 값은 트리거와 도착 상태 쌍들의 집합이다.
<br>
트리거와 도착 상태의 쌍들은 출발 상태에서 전이할 수 있는 상태들과 그 트리거에 대한 규칙이 된다.
<br>
<br>
이 데이터 구조는 다음과 같이 초기화할 수 있다.

```
rules[State::off_hook] = {
    {Trigger::call_dialed, State::connecting},
    {Trigger::stop_using_phone, State::on_hook}
};

rules[State::connecting] = {
    {Trigger::hung_up, State::off_hook},
    {Trigger::call_connected, State::Connected}
};

// 다른 규칙들 생략
```

전이 규칙들 말고도 시작 상태가 필요하다.
<br>
그리고 상태 머신이 특정 상태에 도달한 후 멈추기를 바란다면 종료 상태도 필요할 수 있다.

```
State currentState{ State::off_hook },
exitState{ State::on_hook };
```

이러한 준비를 기반으로 하면 상태 머신의 구동에 별도의 컴포넌트를 만들지 않아도 된다.
<br>
즉, 전체적으로 관리 통제할 모듈이 필요 없다.
<br>
<br>
예를 들어 전화기와 사용자의 상호 작용 모델이 다음과 같이 구현될 수 있다.

```
while(true)
{
    cout << "The phone is currently " << currentState << endl;
    
select_trigger:
    cout << "Select a trigger:" << "\n";
    
    int i = 0;
    
    for(auto item : rules[currentState])
    {
        cout << i++ << ". " << item.first << "\n";
    }
    
    int input;
    cin >> input;
    if(input < 0 || (input+1) > rules[currentState].size())
    {
        cout << "Incorrect option. Please try again." << "\n";
        goto select_trigger;
    }
    
    currentState = rlues[currentState][input].second;
    if(currentState == exitState) break;
}
```

먼저, goto가 있다는 것에 거슬릴 수 있지만 이 코드는 goto가 적합하게 사용된 좋은 예이다.
<br>
사용자에게 현재 상태에서 가용한 트리거 중 하나를 선택하게 하고 트리거가 유효한 경우 앞서 생성한 map의 규칙에 따라 상태를 전이한다.
<br>
<br>
만약 전이가 종료 상태인 경우 루프를 탈출한다.

<br>


## 장단점

### 장점
- 특정 상태들과 관련된 코드를 별도의 클래스로 구성할 수 있음
- Context를 변경하지 않고 새로운 상태를 도입할 수 있음
- 거대한 상태 머신 조건문들을 제거하여 Context의 코드를 단순화할 수 있음


### 단점
- 상태 머신에 몇 가지 상태만 있거나 머신이 거의 변경되지 않을 경우 상태 패턴의 적용은 불필요할 수 

<br>

## 요약
단순한 상태 머신 이상의 고차원 상태 머신들이 많이 있다는 것을 알아 두어야 한다.
<br>
예를 들어 많은 라이브러리들이 "계측적 상태 머신"을 지원한다.
<br>
<br>
계측정 상태 머신이란 예를 들어 몸이 아픈 상태는 감기, 근육통 등 수많은 하위 계층의 상태들을 가질 수 있다.
<br>
즉, 감기 상태에 있다면 당연하게도 아픈 상태이기도 하다.
<br>
<br>
마지막으로 현대의 상태 머신은 과거의 전통적인 상태 머신 디자인 패턴과 비교해 훨씬 더 진보되어 있다는 점을 알아얗 ㅏㄴ다.
<br>
전통적인 방식은 중복된 API 문제, 자기 자신을 삭제하는 문제 등 바람직하지 않은 특성들을 가진다.
