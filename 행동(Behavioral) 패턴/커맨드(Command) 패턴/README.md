# 커맨드 패턴 정리 내용

## 커맨드(Command) 패턴
커맨드는 요청을 요청에 대한 모든 정보가 포함된 독립실행형 객체로 변환하는 행동 디자인 패턴이다.
<br>
이 변환은 다양한 요청들이 있는 메서드들을 인수화 할 수 있도록 하며, 요청의 실행을 지연 또는 대기열에 넣을 수 있도록 하고, 또 실행 취소할 수 있는 작업을 지원할 수 있도록 한다.
<br>
<br>
커맨드 디자인 패턴은 어떤 객체를 활용할 때 직접 그 객체의 API를 호출하여 조작하는 대신, 작업을 어떻게 하라고 명령을 보내는 방식을 제안한다.
<br>
여기에서 명령은 무엇을 어떻게 하라는 지시가 담긴 데이터 클래스 이상도 이하도 아니다.

<br>

## 시나리오
예를 들어 은행의 마이너스 통장이 있으며, 이 통장은 현재 잔고와 인출 한도가 있다.
<br>
입금과 출금 동작을 위해 아래와 같이 deposit()과 withdraw() 메서드를 만든다.

```
struct BankAccount
{
    int balance = 0;
    int overdraft_limit = -500;
    
    void deposit(int amount)
    {
        balance += amount;
        cout << "deposited " << amount << ", balance is now " << balance << '\n';
    }
    
    void withdraw(int amount)
    {
        if(balance - amount >= overdraft_limit)
        {
            balance -= amount;
            cout << "withdraw " << amuotn << ", balance is now " << balance << '\n';
        }
    }
};
```

이 메서드들은 직접 호출하여 입금을 할 수도 있고, 출금을 할 수도 있다.
<br>
그리고 예를 들어 금융 감사가 가능하도록 모든 입출금 내역을 기록해야 하는 상황이다.
<br>
<br>
그런데 이러한 입출금 클래스는 이미 기존에 만들어져 검증되었고 동작 중이어서 수정할 수 없는 상태라고 가정한다.

<br>

## 커맨드 패턴의 구현

먼저 커맨드의 인터페이스를 정의한다.

```
struct Command
{
    virtual void call() const = 0;
};
```

이 인터페이스를 이용해 은행 계좌를 대상으로 한 작업 정보를 캡슐화하는 BankAccountCommand를 정의할 수 있다.

```
struct BankAccountCommand : Command
{
    BankAccount& account;
    enum Action { deposit, withdraw } action;
    int amount;
    
    BankAccountCommand(BankAccount& account, const Action action, const int amount)
        : account(account), action(action), amount(amount) {}
};
```

커맨드에는 다음과 같은 정보들이 포함된다.
- 작업이 적용될 대상 계좌
- 실행될 작업 종류, 해당 작업에 연관된 옵션과 변수들의 집합이 한 번의 선언으로 모두 정의
- 입금 또는 출금할 금액의 크기

클라이언트가 이러한 정보를 제공하면 커맨드를 입금 또는 인출 작업을 실행할 수 있다.

```
void call() const override
{
    switch(action)
    {
        case deposit:
            account.deposit(amount);
            break;
        case withdraw:
            account.withdraw(amount);
            break;
    }
}
```

이러한 접근 방법으로 커맨드를 만들고 커맨드가 지정하고 있는 게좌에 변경 작업을 가할 수 있다.

```
BankAccount ba;
Command cmd{ba, BankAccountCommand::deposit, 100};
cmd.call();
```

위 코드는 주어진 계좌에 100달러를 입금하낟.
<br>
만약 원본 deposit(), withdraw() 메서드가 클라이언트에 노출되는 것을 피하고 싶다면 private으로 선언하고 BankAccountCommand를 friend 클래스로 선언하면 된다.

<br>

## 되돌리기(Undo) 작업

커맨드는 계좌에서 일어난 작업들에 대한 모든 정보를 담고 있기 때문에 어떤 작업에 의한 변경 내용을 되돌려서 그 작업이 행해지기 이전 상태의 계좌를 리턴할 수 있다.
<br>
<br>
먼저, 되돌리기 작업을 커맨드 인터페이스에 추가 기능으로 넣을지 아니면 또 하나의 커맨드로서 처리할지 결정해야 한다.
<br>
이 부분은 디자인 차원에서의 의사 결정으로 인터페이스 분리 원칙을 따르는 것이 바람직하다.
<br>
<br>
예를 들어 커맨드들 중에 비가역적으로 적용되는 것들이 있다면 커맨드 인터페이스 자체에 되돌리기 인터페이스가 포함될 경우 클라이언트에게 혼란을 준다.
<br>
이런 경우 되돌리기가 가능한 커맨드와 일반 커맨드를 분리하는 것이 좋다.
<br>
<br>
아래는 되돌리기가 추가된 Command 인터페이스이다.
<br>
메서드의 const 속성이 의도적으로 삭제되었음을 눈여겨 봐야 한다.

```
struct Command
{
    virtual void call() = 0;
    virtual void undo() = 0;
};
```

아래는 입금과 출금이 대칭적인 작업이라는 점에서 발상된 매우 단순한 BankAccountCommand::undo()의 구현이다.

```
void undo() override
{
    switch(action)
    {
        case withdraw:
            account.deposit(amount);
            break;
        case deposit:
            account.withdraw(amount);
            break;
    }
}
```

이 구현은 문제가 있다.
<br>
만약 은행 젗네의 잔고보다도 큰 금액을 출금하려 했다면 당연히도 실패했을 것이다.
<br>
하지만 그 작업을 되돌리기할 때는 실패한 작업이라는 사실을 확인하지 않는다.
<br>
<br>
즉, 되돌리기 작업 때문에 은행 자산만큼의 큰 금액이 입금되어 버릴 수 있다.
<br>
<br>
따라서, 출금 작업의 성공 여부를 리턴하도록 아래와 같이 withdraw() 함수를 수정한다.

```
bool withdraw(int amount)
{
    if(balance - amount >= overdraft_limit)
    {
        balance -= amount;
        cout << "withdraw " << amount << ", balance now " << balance << '\n';
        
        return true;
    }
    
    return false;
}
```

이제 BankAccountCommand 전체를 수정하여 아래와 같은 작업을 수행하게 한다.

- 출금 때마다 success 플래그를 내부적으로 저장
- undo()가 호출될 때 이 플래그를 이용

이 작업들의 구현은 아래와 같다.

```
struct BankAccountCommand : Command
{
    ...
    bool withdrawal_succeeded;
    
    BankAccountCommand(BankAccount& account, const Action action, const int amount)
        : ..., withdrawal_succeeded{false} {}
        
    void call() override
    {
        switch(action)
        {
            ...
            case withdraw:
                withdrawal_succeeded = account.withdraw(amount);
                break;
    }
};
```

이제 플래그를 ㄱ가졌으니 undo()의 구현을 개선할 수 있다.

```
void undo() override
{
    switch(action)
    {
        case withdraw:
            if(withdrawal_succeeded)
                account.deposit(amount);
            break;
            ...
    }
}
```

이와 같은 수정으로 출금 커맨드의 되돌리기가 부작용을 일으키지 않게 되었다.

<br>

## 컴포지트 커맨드(Composite Command)

계좌 A에서 계쫘 B로의 이체는 다음의 두 커맨드로 수행될 수 있다.

1. 계좌 A에서 $X만큼 출금
2. 계좌 B에 $X만큼 입금

두 커맨드를 각각 호출하는 대신 하나의 "계좌 이체" 커맨드로 감싸서 처리할 수 있다면 편리할 것이다.
<br>
구조 패턴의 컴포지트 패턴과 지향하는 것이 동일하다.
<br>
<br>
컴포지트 커맨드의 골격은 BankAccountCommand를 벡터로 상속받아 사용한다.
<br>
std::vector에 버추얼 소멸자가 없기 때문에 일반적으로는 바람직하지 않고, 예제에서만 사용한다.

```
struct CompositeBankAccountCommand : vector<BankAccountCommand>, Command
{
    CompositeBankAccountCommand(const initializer_list<value_type>& items)
        : vector<BankAccountCommand>(items) {}
        
    void call() override
    {
        for(auto& cmd : *this)
            cmd.call();
    }
    
    void undo() override
    {
        for(auto it = rbegin(); it != rend(); ++it)
            it->undo();
    }
};
```

CompositeBankAccountCommand는 vector이면서 Command이다.
<br>
즉, 컴포지트 디자인 패턴을 따르고 있다.
<br>
<br>
생성자는 편리한 initializer_list를 이용해 인자를 받고 있고, undo(), redo() 두 개의 작업이 구현되어 있다.
<br>
<br>
이제 계좌 이체 작업을 컴포지트 패턴으로 구현해야 한다.
<br>
먼저 아래와 같은 코드가 있다.

```
struct MoneyTransferCommand : CompositeBankAccountCommand
{
    MoneyTransfer(Command(BankAccount& from, BankAccount& to, int amount) : CompositeBankAccountCommand
    {
        BankAccountCommand{from, BankAccountCommand::withdraw, amount},
        BankAccountCommand(to, BankAccountCommand::deposit, amount}
    } {}
}
```

베이스 클래스의 생성자를 재사용해 두 개의 커맨드로 "계좌 이체 커맨드" 객체를 초기화하고 있다.
<br>
그리고 베이스 클래스의 call()/undo() 구현도 재사용 한다.
<br>
<br>
하지만 앞서 이야기되었듯이 문제가 있다.
<br>
베이스 클래스의 구현은 명령이 실패할 수도 있다는 것을 고려하여야 한다.
<br>
만약 계좌 A에서의 출금이 실패했다면 계좌 B로의 입금이 실행되어서는 안 된다.
<br>
<br>
이때는 전체 명령 사슬이 취소 되어야 한다.
<br>
<br>
이러한 예외 처리를 하려면 아래와 같이 많은 변화가 필요하다.
- Command에 success 플래그 추가
- 각 작업의 수행마다 성공, 실패 여부 기록
- 성공한 명령에 대해서만 되돌리기 명령 수행 가능
- 명령의 되돌림을 주의 깊게 수행하는 클래스 DependentCompositeCommand를 커맨드 클래스 간에 둠

각 커맨드가 호출될 때, 직전의 커맨드가 성공적으로 수행되었을 때만 실제 수행되게 한다.
<br>
그렇지 않은 경우 success 플래그를 false로 세팅하고 커맨드를 실행하지 않는다.

```
void call() override
{
    bool ok = true;
    
    for(auto& cmd :*this)
    {
        if(ok)
        {
            cmd.call();
            ok = cmd.succeeded;
        }
        else
        {
            cmd.succeeded = false;
        }
    }
}
```

undo() 메서드를 오버라이딩할 필요는 없다.
<br>
왜냐하면 개별 커맨드가 자체적으로 자신의 success 플래그를 확인하고 이 플래그가 true일 때만 되돌리기 작업을 실행하기 때문이다.
<br>
<br>
구성하고 있는 모든 커맨드가 성공적으로 실행되어야만 성공하는 더 강한 조건의 컴포지트 커맨드를 생각해볼 수 있다.

<br>

## 명령과 조회의 분리

명령과 조회를 분리(CQS, Command Query Separatinon) 한다는 개념은 어떤 시스템에서의 작업이 크게 보았을 때 다음의 두 종류 중 한 가지로 분류될 수 있다는 것에서 제안되었다.

- 명령 : 어떤 시스템의 상태 변화를 야기하는 작업 지시들로 어떤 결괏값의 생성이 없는 것
- 조회 : 어떤 결괏값을 생성하는 정보 요청으로, 그 요청을 처리하는 시스템의 상태 변화를 일으키지 않는 것

직접적으로 상태에 대한 읽기, 쓰기 작업을 노출하고 있는 객체라면 그러한 것들을 private으로 바꾸고 각 상태에 대한 get/set 메서드들 대신 단일 인터페이스로 바꿀 수 있다.
<br>
<br>
예를 들어 체력과 빠르기 두 개의 속성을 가지는 크리처 클래스가 있다고 할 때, 각 속성에 대한 get/set 메서드 대신 아래처럼 단일한 커맨드 인터페이스를 제공할 수 있다.

```
class Creature
{
    int strength, agility;

public:
    Creature(int strength, int agility)
        : strength{strength}. agility{agility} {}
    
    void process_command(const CreatureCommand& cc);
    int process_query(const CreatureQuery& q) const;
};
```

위 코드에서 get/set 메서드가 없다.
<br>
대신 두 개의 API 멤버 함수 process_command()와 process_query()가 있다.
<br>
<br>
Creature가 제공해야 하는 속성과 기능이 아무리 늘어나더라도 이 API 두 개만으로 처리된다.
<br>
즉, 커맨드만으로 크리처와의 상호작용을 수행할 수 있다.
<br>
<br>
이 API들은 각각 전용 클래스로 정의되고 enum 타입 클래스 CreatureAbility와 연관된다.

```
enum class CreatureAbility { strength, agility };

struct CreatureCommand
{
    enum Action { set, increaseBy, decreaseBy } action;
    CreatureAbility ability;
    int amount;
};

struct CreatureQuery
{
    CreatureAbility ability;
};
```

위 코드처럼 명령 객체는 어떤 멤버 변수를 어떻게 얼마만큼 바꿀 것인가를 지시한다.
<br>
조회 객체는 조회할 대상만 지정한다.
<br>
<br>
여기서는 조회 결과가 함수 리턴 값으로 전달되는 것으로 가정하고 조회 객체 자체에 따로 저장하지 않는다.
<br>
<br>
다음은 process_command()의 정의이다.

```
void Creature::process_command(const CreatureCommand &cc)
{
    int* ability;
    switch(cc.ability)
    {
        case CreatureAbility::strength:
            ability = &strength;
            break;
        case CreatureAbility::agility:
            ability = &agility;
            break;
    }
    
    switch(cc.action)
    {
        case CreatureCommand::set:
            *ability = cc.amount;
            break;
        case CreatureCommand::increaseBy:
            *ability += cc.amount;
            break;
        case CreatureCommand::decreaseBy:
            *ability -= cc.amount;
            break;
    }
}
```

명령과 조회에 로깅이 필요하거나 객체를 유지해야 한다면 위 두 메서드만 수정하면 된다.
<br>
유일한 문제는 익숙한 get/set 방식의 API를 고집하는 클라이언트를 어떻게 지원하느냐 이다.
<br>
<br>
다행히도 아래와 같이 process_..() 메서드의 호출을 감싸고 적절히 인자를 전달함으로써 get/set 함수를 쉽게 만들 수 있다.


```
void Creature::set_strength(int value)
{
    process_command(CreatureCommand{ CreatureCommand::set, CreatureAbility::strength, value });
}

int Creature::get_strength() const
{
    return process_query(CreatureQuery{ CreatureAbility::strength });
}
```


위 코드는 명령과 조회를 분리 구현하는 시스템의 내부에서 무슨 일이 일어나고 있는지 매우 간단하게 보여주는 예이다.
<br>
그리고 객체의 모든 인터페이스를 명령과 조회 두 종류로 구분하는 방법을 보여준다.

<br>

## 장단점

### 장점
- 기존 클라이언트 코드를 손상하지 않고 앱에 새 커맨드들을 도입할 수 있음
- 실행 취소/다시 실행을 구현할 수 있음
- 작업들의 지연된 실행을 구현할 수 있음
- 간단한 커맨드들의 집합을 복잡한 커맨드로 조합할 수 있음


### 단점
- 발송자와 수신자 사이에 완전히 새로운 레이어를 도입하기 때문에 코드가 복잡해질 수 있음


<br>

## 요약

커맨드 디자인 패턴은 단순하다.
<br>
인자를 전달하여 메서드를 호출하는 직접적인 방법으로 객체에 일을 시키는 대신, 작업 지시 내용을 감싸는 특별한 객체를 두어 객체와 커뮤니케이션하게 한다.
<br>
<br>
어떤 경우에는 대상 객체의 변화를 일으키거나 어떤 작업을 시키는 것이 아니라 단순히 대상 객체로부터 어떤 정보를 얻고 싶을 수도 있다.
<br>
이러한 경우를 처리하는 커맨드 객체를 조회 객체라고 부른다.
<br>
많은 경우 조회 객체는 메서드의 리턴 값으로 정보를 전달하는 불변 객체이다.
<br>
<br>
커맨드 디자인 패턴은 많은 UI 시스템에서 '복사하기', '붙여넣기' 와 같은 익숙한 동작들을 캡슐화하는 데 사용되고 있다.
<br>
커맨드는 여러 경로로 실행될 수 있다.
<br>
<br>
예를 들어 '복사하기'는 메뉴에서 실행될 수도 있고, 툴바 아이콘에서 실행될 수도 있고, 키보드에서 실행될 수도 있다.
<br>
그리고 여러 작업들의 시퀀스를 저장해 두었다가 단일 명령으로서 일괄처리 할 수도 있다.
