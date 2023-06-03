# 책임 연쇄 패턴 정리 내용

## 책임 연쇄(Chain of Responsibility)
책임 연쇄 패턴은 책임 사슬 패턴이라고도 불린다.
<br>
책임 연쇄 패턴은 핸들러의 체인(사슬)을 따라 요청을 전달할 수 있게 해주는 행동 디자인 패턴이다.
<br>
각 핸들러는 요청을 받으면 요청을 처리할지 아니면 체인의 다음 핸들러로 전달할지를 결정한다.
<br>
<br>
예를 들어 어떤 회사의 임직원이 내부 정보를 이용해 회사 주식을 거래하여 부당 이득을 취했다고 하자.
<br>
이 경우 그 임직원은 처벌을 받을 것이며, 어떤 동료가 공모했다면 그 동료도 처벌을 받을 것이다.
<br>
<br>
이것이 책임 연쇄(사슬)의 예이다.
<br>
어떤 시스템을 구성하고 있는 여러 개의 서로 다른 컴포넌트들이 어떤 메시지를 역할에 따라 주고받으며 처리할 수 있다.


<br>

## 시나리오
예를 들어 어떤 컴퓨터 게임에 크리처(괴물, 유닛 등)들이 있다고 가정한다.
<br>
이 크리처들은 공격력(Attack)과 벙어력(Defense) 두 가지 값을 속성으로 가진다.

```
struct Creature
{
    string name;
    int attack, defense;
    // 생성자와 << 연산자 구현 ...
};
```

이제 게임이 진행되는 동안 크리처가 처리되어야 한다.
<br>
아이템을 습득할 수도 있고, 마법에 당할 수도 있고, 어떤 경우든 공격력 또는 방어력 값이 이벤트에 맞게 변경되어야 한다.
<br>
<br>
이를 위해 CreatureModifier를 호출한다.
<br>
그리고 한 번에 여러 개의 이벤트가 발생하여 CreatureModifier가 여러 번 호출될 수도 있다.
<br>
따라서 변경 작업을 크리처별로 쌓아 놓은 순서대로 속성이 변경될 수 있게 한다.

<br>

## 포인터 사슬
아래의 CreatureModifier 구현은 책임 연쇄(사슬(의 전통적인 구현 방법을 보여준다.

```
class CreatureModifier
{
    CreatureModifier* next{nullptr};

protected:
    Creature& creature; // 참조 대신 포인터나 shared_ptr 사용 가능

public:
    explicit CreatureModifier(Creature& creature)
        : creature(creature) {}

    void add(CreatureModifier* cm)
    {
        if(next) next->add(cm);
        else next = cm;
    }

    virtual void handle()
    {
        if(next) next-> handle(); // 핵심 부분
    }
};
```

- 이 클래스는 Creature의 참조를 넘겹다아 저장하고 변경할 준비를 함
- 이 클래스는 실질적으로 하는 일이 많지 않지만 추상 클래스는 아니며 모든 멤버가 구현되어 있음
- 멤버 변수 next는 부가적으로 현재의 변경 작업에 뒤따르는 또 다른 CreatureModifier를 가리키며, 이 포인터의 실 구현체는 CreatureModifier를 상속받는 무엇이든 가능함
- add() 메서드는 다른 크리처에 대한 변경 작업을 현재의 변경 작업 사슬에 연결하여 추가하며, 이러한 연결은 재귀적으로 일어남<br>만약 현재 사슬이 nullptr를 가리키고 있으면 인자로 주어진 변경 작업을 그대로 설정하고 재귀 호출에는 진입하지 않음
- handle() 메서드는 단순히 사슬의 다음 항목을 처리하며 이 함수는 그 자체로는 아무런 작업을 수행하지 않음<br>이 함수는 목적에 따라 오버라이딩될 수도 있기 때문에 virtual로 선언되어 있음

이런 구현은 단지 연결 리스트에 항목을 추가하는 정도밖에는 특별할 게 없다.
<br>
하지만 이 크래스를 상속받아 실질적인 작업들이 추가되기 시작하면 이 구현의 이미가 더 명확해진다.
<br>
<br>
예를 들어 크리처의 공격력을 두 배로 키우는 변경 작업이 다음과 같이 정의될 수 있다.

```
class DoubleAttackModifier : public CreatureModifier
{
public:
    explicit DoubleAttackModifier(Creature& creature)
        : CreatureModifier(creature) {}

    void handle() override
    {
        creature.attack *= 2;
        CreatureModifier::handle();
    }
};
```

이 클래스는 CreatureModifier를 상속받아 handle() 메서드 안에서 두 가지 작업을 한다.
<br>
하나는 공격력을 두배로, 다른 하나는 부모 클래스의 handle() 메서드를 호출하는 것이다.
<br>
<br>
변경 작업이 연이어질 수 있으려면 중간의 어느 클래스에서도 handle()의 구현부 마지막에서 부모의 hnadle()을 호출하는 것을 빠뜨리지 않아야 한다.
<br>
<br>
이제 클래스를 복잡하게 변경하려 한다.
<br>
아래의 클래스는 공격력이 2 이하인 크리처의 방어력을 1 증가시키는 변경 작업을 수행한다.

```
class IncreaseDefenseModifier : public CreatureModifier
{
public:
    explicit IncreaseDefenseModifier(Creature& creature)
        : CreatureModifier(creaure) {}

    void handle() override
    {
        if(creature.attack <= 2)
            creature.defense += 1;
        CreatureModifier::handle();
    }
};
```

이번에도 변경 작업 구현의 마지막에서 부모를 호출하고 있다.
<br>
이러한 변경 작업의 정의를 활용하여 다음과 같이 복합적인 변경 작업을 크리처에 적용할 수 있다.

```
Creature goblin{ "Goblin", 1, 1 };
CreatureModifier root{ goblin };
DoubleAttackModifier r1{ goblin };
DoubleAttackModifier r1_2{ goblin };

IncreaseDefenseModifier r2{ goblin };

root.add(&r1);
root.add(&r1_2);
root.add(&r2);

root.handle();

cout << goblin << endl;
// 출력 결과 "name: Goblin attack: 4 defennse 1"
```

위 코드에서 보듯이 고블린의 공격력이 두 배로 커졌고, 방어력은 공격력이 조건에 부합하지 않기 때문에 아무런 변경이 없다.
<br>
<br>
또 다른 예제로 크리처에 마법을 걸어 어떤 보너스(공격력 또는 방어력)도 받을 수 없게 만드는 변경 작업을 적용하려고 한다.
<br>
이 부분은 간단하게 구현할 수 있는데, 단순히 부모의 handle()을 호출하지 않기만 하면 된다.
<br>
<br>
그렇게 하면 전체 책임 사슬의 호출이 생략된다.

```
class NoBonusesModifier : public CreatureModifier
{
public:
    explicit NoBonusesModifier(Creature& creature)
        : CreatureModifier(creature) {}

    void handle() override
    {
        // 아무 동작도 하지 않음
    }
};
```

이제 책임 사슬의 제일 앞에 있는 NoBonusModifier가 들어 있기만 하면 뒤에 연결된 변경 작업들이 전혀 반영되지 않는다.

<br>

## 브로커 사슬
실제 게임에서는 크리퍼가 임의로 보너스를 얻거나 잃을 수 있어야 한다.
<br>
이어 붙이는 것만 가능한 연결 리스트로는 임의의 변경 작업을 지원할 수는 없다.
<br>
<br>
게다가 일반적인 게임이라면 크리처의 상태를 영구적으로 변경하는 것이 아니라 원본은 남겨두고 임시로 변경 작업이 적용되어야 한다.
<br>
<br>
책임 사슬을 구현하는 또 다른 방법은 중앙 집중화된 컴포넌트를 두는 것이다.
<br>
이 컴포넌트는 게임에서 발생할 수 있는 모든 변경 작업의 목록을 관리하고 특정 크리처의 공격력 또는 방어력의 상태를 그간의 변경 작업 이력이 모두 반영된 상태로 구할 수 있게 한다.
<br>
<br>
이러한 컴포넌트를 이벤트 브로커라고 부른다.
<br>
그 이유는 이 컴포넌트는 참여하는 모든 컴포넌트를 연결하는 역할을 하기 때문이다.
<br>
<br>
이것은 매개자(Mediator) 디자인 패턴이기도 하면서 모든 이벤트를 모니털이한 결과를 조회할 수 있기 때문에 관찰자(Observer) 디자인 패턴이기도 하다.
<br>
<br>
이제 브로커를 만들기 위해 Game 클래스를 만든다.
<br>
이 클래스는 게임의 실행에 대한 모든 것을 담는다.

```
struct Game // 매개자
{
        signal<void(Query&)> queries;
};
```

상태 조회 명령을 전송하기 위해 Boost.Signals2 라이브러리를 사용한다.
<br>
이 라이브러리는 어떤 신호를 발생시키고 그 신호를 기다리고 있는 모든 수신처가 신호를 처리할 수 있게 한다.

<br>
<br>

어떤 크리처의 상태를 임의의 시점에 조회할 수 있게 하고 싶다고 가정한다.
<br>
단순히 크리처의 필드를 읽을 수도 있지만, 문제가 있다.
<br>
크리처에 가해질 변경 작업이 모두 완료되어 결괏값이 확정된 이후에 읽어야 한다.
<br>
<br>
따라서 조회 작업을 별도의 객체에 캡슐화하여 처리하기로 한다. (이것이 커맨드 패턴이라고 함)
<br>
<br>
이 객체는 다음과 같이 정의된다.

```
struct Query
{
    string creature_name;
    enum Argument { attack, defense } argument;
    int result;
};
```

위 클래스는 어떤 크리처의 특정 상태 값에 대한 조회라는 개념을 캡슐화하고 있다.
<br>
이 클래스를 사용하기 위해 필요한 것은 크리처의 이름과 조회할 상태 값의 종류 뿐이다.
<br>
<br>
Game::queries에서 변경 작업들을 적용하고 최종 결괏값을 리턴하기 위해 Query 객체에 대한 참조를 사용한다.

<br>
<br>

이제 Crature의 정의를 보면, 앞서 보았던 정의와 거의 같다.
<br>
필드 변수에서의 차이점은 Game의 참조가 추가되었다는 것 하나 뿐이다.

```
class Creature
{
    Game& game;
    int attack, defense;

public:
    string name;
    Creature(Game& game, ...) : game{game}, ... { ... }
    // 다른 멤버들
};
```

attack, defense 변수가 private으로 선언되었다.
<br>
이 값들이 private이라는 것은 변경 작업들이 반영된 최종 값이 별도로 get 메서드로 얻어져야 한다는 것을 의미한다.
<br>
예를 들어 아래와 같이 이용된다.

```
int Creature::get_attack() const
{
    Query q { name, Query::Argument::attack, attack };
    game.queries(q);
    return q.result;
}
```

여기서 단순히 값을 리턴학서나 포인터 기반의 정적 책임 사슬을 이용하는 대신, 목적하는 인자로 Query 객체를 만든 다음 Game::queries에 넘기면 등록된 수신처들이 각각 조회 객체를 검사하여 자신이 처리 가능한 경우 결과를 채워준다.
<br>
<br>
이제 변경 작업을 구현하기 위해 베이스 클래스를 만드는데, 이번에는 handle() 메서드가 없다.

```
class CreatureModifier
{
    Game& gmae;
    Creature& creature;

public:
    CreatureModifier(Game& game, Creature& creature)
        : game(game), creature(creature) {}
};
```

이 클래스가 하는 일은 오직 생성자가 올바른 인자로 호출되는 것을 보증하는 역할 뿐이다.
<br>
이제 CreatureModifier를 상속받아 변경 작업 클래스를 구현한다.

```
class DoubleAttackModifier : public CreatureModifier
{
    Connection conn;

public:
    DoubleAttackModifier(Game& game, Creature& creature)
        : CreatureModifier(game, creature)
    {
        conn = game.queris.connect([&](Query& q)
        {
            if(q.creature_name == creature.name && q.argument == Query::Argument::attack)
                q.result *= 2;
        });
    }

    ~DoubleAttackModifier() { conn.disconnect(); }
};
```

위 코드에서 볼 수 있듯이 생성자와 소멸자에서 주요 작업들이 수행되고 추가적인 메서드는 필요하지 않다.
<br>
생성자에서는 Game에 대한 참조를 통해 Game::queries에 접근하여, 공격력을 두 배 증가시키는 람다 함수를 조회 이벤트에 연결한다.
<br>
<br>
나중에 객체가 소멸되었을 때 이벤트의 연결을 해제할 수 있도록 연결 정보도 저장해야 한다.
<br>
그렇게 함으로써 변경 작업이 임시적으로 적용되고 유효한 조건을 벗어났을 때 더 이상 적용되지 않게 할 수 있다.
<br>
<br>
이러한 것들을 모두 포함하면 다음과 같은 구현이 가능하다.

```
Game game;

Creagure goblin{ game, "Strong Goblin", 2, 2 };
cout << goblin << endl;
// 이름 : Strong Goblin 공격력 : 2 방어력 : 2
{
    DoubleAttackModifier dam{ game, goblin };
    cout << goblin << endl;
// 이름 : Strong Goblin 공격력 : 4 방어력 : 2 
}
cout << goblin << endl;
// 이름 : Strong Goblin 공격력 : 2 방어력 : 2
}
```

변경 작업이 적용되기 전 고블린의 공격력과 방어력은 각각 2, 2 였다.
<br>
그 다음 게임 중에 공격력이 2배로 커진 현상이 발생하여 DoubleAttackModifier를 적용한다.
<br>
<br>
그러면 고블린의 공격력과 방어력이 각각 4, 2가 된다.
<br>
그리고 그 범위를 벗어나면 자동으로 변경 작업의 소멸자가 호출되어 브로커와의 연결을 끊고 변경 작업이 더 이상 적용되지 않게 된다.
<br>
<br>
즉, 고블린의 공격력과 방어력이 2, 2로 원래대로 돌아간다.

<br>

## 장단점

### 장점
- 요청의 처리 순서를 제어할 수 있음
- 호출하는 클래스와 작업을 수행하는 클래스를 분리할 수 있음
- 기존 클라이언트 코드를 손상하지 않고 앱에 새 핸들러를 도입할 수 있음


### 단점
- 일부 요청들이 처리되지 않을 수 있음


### 단점
- 플라이웨이트 메서드 호출 시 Context 데이터의 일부를 다시 게산해야 한다면 

<br>

## 요약
책임 사슬은 컴포넌트들이 어떤 명령이나 조회 작업을 차례로 처리할 수 있게 하는 매우 단순한 디자인 패턴이다.
<br>
책임 사슬의 가장 단순한 구현은 포인터 사슬이다.
<br>
<br>
이론적으로 포인터 사슬은 보통의 vector나 list로 대체될 수 있다.
<br>
<br>
좀 더 복잡한 브로커 사슬의 구현에서는 매개자 패턴과 관찰자 패턴을 활용한다.
<br>
이를 통해 어떤 조회 이벤트(신호)에 수신처로 등록된 변경 작업들이, 최종 값을 클라이언트에게 리턴하기 전에, 전달된 원본 객체를 상황에 맞게 수정할 수 있다.
