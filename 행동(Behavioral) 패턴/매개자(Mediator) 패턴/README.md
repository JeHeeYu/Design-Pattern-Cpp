# 매개자 패턴 정리 내용

## 매개자(Mediator) 패턴
매개자는 객체 간의 혼란스러운 의존 관계들을 줄일 수 있는 행동 디자인 패턴이다.
<br>
이 패턴은 객체 간의 직접 통신을 제한하고 중재자 객체를 통해서만 협력하도록 한다.
<br>
<br>
보통 작성하는 코드의 상당 부분이 서로 다른 컴포넌트(클래스) 간에 포인터나 직접적인 참조를 통한 커뮤니케이션에 소요된다.
<br>
하지만 어떤 경우에는 컴포넌트 간에 명시적으로 상대방 객체의 존재를 알아야 하는 상황이 불편할 수 있다.
<br>
<br>
또는 상대방 객체를 알더라도 객체의 생성/소멸 시점에 대한 관리 때뭉네 포인터나 참조로 접근되는 것이 싫을 수도 있다.
<br>
<br>
매개자는 컴포넌트 간의 커뮤니케이션을 돕기 위한 메커니즘이다.
<br>
당연하게도 매개자 자체는 커뮤니케이션에 동반되는 모든 컴포넌트로부터 접근 가능해야 한다.
<br>
<br>
즉, 매개자는 전역 정적 변수이거나 모든 컴포넌트에 그 참조가 노출되어야 한다.

<br>

## 채팅 룸

인터넷 채팅 룸은 매개자 디자인 패턴이 적용될 수 있는 가장 전형적인 예제이다.
<br>
아래는 채팅 룸의 참여자를 추상화하는 가장 단순한 구현이다.

```
struct Person
{
    string name;
    ChatRoom* room = nullptr;
    vector<string> chat_log;
    
    Person(const string& name);
    
    void receive(const string& origin, const string& message);
    void say(const string& message) const;
    void pm(const string& who, const string& message) const;
};
```

참여자의 이름(name), 대화 내용(chat_log), 채팅 룸 객체로의 참조를 가진다.
<br>
그리고 생성자와 세 개의 메서드가 있다.

- receive() : 메시지를 수신 받는데, 보통 이러한 함수는 화면에 수신받은 메시지를 출력하고 대화 내용을 업데이트 함
- say() : 채팅 룸의 모든 참여자에게 메시지 전송
- pm() : 개인 메시지(PM, Private Message) 기능으로 채팅 룸의 특정 참여자를 이릉므로 지정하여 그 참여자에게만 메시지 전송

say()와 pm() 모두 채팅 룸에서 수행되어야 하는 작업을 중계하는 역할이다.
<br>
이제 채팅 룸을 구현하며, 코드는 다음과 같다.

```
struct ChatRoom
{
    vector<Person*> people; // 추가만 된다고 가정함
    
    void join(Person* p);
    void broadcast(const string& origin, const string& message);
    void message(const string& origin, const string& who, const string& message);
};
```

포인터, 참조, shared_ptr 등등 어떤 방식으로 객체에 접근할지는 구현하는 사람의 취향에 달려있다.
<br>
하지만 std::vector를 사용할 경우 참조를 저장할 수 없다는 제약 사항이 따른다.
<br>
따라서 여기서는 포인터를 사용한다.
<br>
<br>
ChatRoom의 API는 아래와 같이 단순하게 정의된다.
- join() : 채팅 룸에 사용자가 입장할 수 있게 함
- broadcast() : 채팅 룸의 모든 참여자에게 메시지 전송
- message() : 개인 메시지 전송

join()은 아래와 같이 구현한다.

```
void ChatRoom::join(Person* p)
{
    string join_msg = p->name + " joins the chat";
    broadcast("room", join_msg);
    p->room = this;
    people.push_back(p);
}
```

이제 broadcast()를 구현하며, 아래와 같이 쉽게 구현할 수 있다.

```
void broadcast(const string& origin, const string& message)
{
    for(auto p : people)
        if(p->name != origin)
            p->receive(origin, message);
}
```

마지막으로 개인 메시지 처리하는 message() 함수를 구현한다.

```
void message(const string& origin, const string& who, const string& message)
{
    auto target = find_if(begin(people), end(people), [&](const Person* p) { return p->name == who; });
    
    if(target != end(people))
    {
        (*target)->receive(origin, message);
    }
}
```

이제 채팅 룸 API가 준비되었으므로 Person의 say()와 pm() 메서드를 아래와 같이 구현할 수 있다.

```
void Person::say(const string& message) const
{
    room->broadcast(name, message);
}

void Person::pm(const string& who, const string& message) const
{
    room->message(name, who, message);
}
```

receive() 메서드는 수신되는 메시지를 화면에 출력하고 대화 내용에 추가하기에 적합한 위치이다.

```
void Person::receive(const string& origin, const string& message)
{
    string s{ origin + ": \"" + message + "\"" };
    cout << "[" << name << "'s chat session] " <<  s << "\n";
    chat_log.emplace_back(s);
}
```

여기에 현재 채팅의 채팅 룸 세션에서 메시지가 언제 어디서 왔는지 기록하는 기능을 추가한다.
<br>
아래는 지원해야 할 시나리오의 예제이다.

```
ChatRoom room;

Person john{ "john" };
Person jane{ "jane" };
room.join(&john);
room.join(&jane);
john.say("hi room");
jane.say("oh, hey john");

Person simon("simon");
room.join(&simon);
simon.say("hi everyone!");

jane.pm("simon", "glad you could join us, simon");
```

위와 같은 채팅에서 다음과 같은 출력이 일어나야 한다.

```
[john's chat session] room: "jane joins the chat"
[jane's chat session] joihn: "hi room"
[john's chat session] jane: "oh, hey john"
[john's chat session] room: "simon joins the chat"
[jane's chat session] room: "simon joins the chat"
[john's chat session] simon: "hi everyone!"
[jane's chat session] simon: "hi everyone!"
[simon's chat session] jane: "glad you could join us, simon"
```

<br>

## 매개자와 이벤트

채팅 룸 예제로 돌아가기 전에 더 단순한 예제로 축구 게임을 예로 가정한다.
<br>
축구에는 선수와 코치가 있고, 선수가 득점을 하면 코치는 칭찬을 한다.
<br>
이때 누가 골을 넣었는지, 몇 골을 넣었는지에 대한 정보가 전달되어야만 한다.
<br>
<br>
이러한 정보를 전달하기 위해, 이벤트 데이터를 일반화한 베이스 클래스를 아래와 같이 정의한다.

```
struct EventData
{
    virtual ~EventData() = default;
    virtual void print() const = 0;
};
```

이제 이 클래스를 상속받아 골 득점 정보를 저장할 클래스를 만들 수 있다.

```
struct PlayerScoredData : EventData
{
    string player_name;
    int goals_scored_so_far;
    
    PlayerScoredData(const string& player_name, const int goals_scored_so_far)
        : player_name(player_name), goals_scored_so_far(goals_scored_so_far) {}
        
    void print() const override
    {
        cout << player_name << " has scored! (their " << goals_scored_so_far << " goal)" << "\n";
    }
};
```

이제 다시 한번 매개자를 만든다.
<br>
이 매개자는 동작을 하지 않는데, 이벤트 기반 구조에서는 매개자가 직접 일을 수행하지 않기 때문이다.

```
struct Game
{
    signal<void(EventData*)> events; // 관찰자
};
```

이제 Player 클래스를 만들 수 있다.
<br>
Player는 선수 이름, 게임에서의 골 득점 그리고 매개자인 Game의 참조를 가진다.

```
struct Player
{
    string name;
    int goals_scored = 0;
    Game& game;
    
    Player(const string& name, Game& game)
        : name(name), game(game) {}
        
    void score()
    {
        goals_scored++;
        PlayerScoredData ps{name, goals_scored};
        game.events(&ps);
    }
};
```


Player::score() 메서드는 이벤트를 이용해 PlayerScoredData를 생성하고 모든 이벤트의 수신처로 등록된 객체들에 알림을 전송한다.
<br>
이 이벤트를 받을 객체를 아래와 같이 만들 수 있다.

```
struct Coach
{
    Game& game;
    explicit Coach(Game& game) : game(game)
    {
        // 선수의 누적 득점이 3정 미만인 동안은 계속해서 칭찬을 함
        game.events.connect([](EventData* e)
        {
            PlayerScoredData* ps = dynamic_cast<PlayerScoredData*>(e);
            if(ps && ps->goals_scored_so_far < 3)
            {
                cout << "coach says: well done, " << ps->player_name << "\n";
            }
        });
    }
};
```

Coach 클래스의 구현은 단순하다.
<br>
코치의 이름 조차도 저장하지 않는다.
<br>
생성자에서 game.events에 수신 등록을 할 뿐이다.
<br>
<br>
game에서 어떤 이벤트가 발생하면 알림을 받게 되어 필요한 처리를 할 수 있다.
<br>
여기서는 골 득점 이벤트 발생 시 처리할 내용을 람다 함수로 지정한다.
<br>
<br>
골 득점 이벤트에 대해서만 처리를 하고 싶지만, 어떤 이벤트가 올지 알 수 없다.
<br>
따라서 dynamic_cast를 이용해 목적하는 골 득점 이벤트 타입이 수신되었는지 확인한다.

<br>


## 요약

매개자 디자인 패턴은 시스템 내 컴포넌트 모두가 참조할 수 있는 어떤 중간자를 컴포넌트 간에 서로 직접적으로 참조하지 않더라도 커뮤니케이션을 할 수 있게 한다는 것을 기본 아이디어로 한다.
<br>
<br>
매개자를 통해 직접적인 메모리 접근 대신 식별자로 커뮤니케이션을 할 수 있게 한다.
<br>
<br>
매개자의 가장 단순한 구현 형태는 메서드로 리스트를 두고 그 리스트를 검사하여 필요한 항목만 선택적으로 처리하는 함수를 만드는 것이다.
<br>
<br>
매개자의 좀 더 정교한 구현 형태는 시스템에서 이벤트들에 대해서 이벤트를 받길 원하는 객체가 개별적으로 수신 등록을 할 수 있게 하는 것이다.
<br>
이렇게 함으로써 컴포넌트 간에 전달되는 메시지를 이벤트로써 다룰 수 있게 한다.
<br>
<br>
이러한 구현에서는 어떤 컴포넌트가 시스템에서 소멸되거나 더 이상 이벤트를 처리할 필요가 없을 때 손쉽게 수신 해제를 할 수 있다.
