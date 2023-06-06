# 템플릿 메서드 정리 내용

## 템플릿 메서드(Template Method)
템플릿 메서드는 부모 클래스에서 알고리즘의 골격을 정의하지만, 해당 알고리즘의 구조를 변경하지 않고 자식 클래스들이 알고리즘의 특정 단계들을 오버라이드 할 수 있도록 하는 행동 디자인 패턴이다.
<br>
템플릿 메서드는 전략 패턴가 매우 유사하다.
<br>
둘 다 골격을 정의하는 디자인 패턴이다.
<br>
<br>
전략 패턴과 템플릿 메서드 패턴의 차이점은 전략 패턴이 컴포지션을 이용하는 데 반해 템플릿 메서드 패턴은 상속을 이용한다.
<br>
<br>
하지만 핵심 원리는 어떤 알고리즘의 골격을 한 곳에 정의해두고 상세 구현을 다른 곳에 둔다는 점에서 동일하다.

<br>

## 게임 시뮬레이션
예제를 들기 위해 보드 게임류를 생각해 본다.
<br>
게임류는 대부분 비슷한데, 플레이어가 차례로 돌아가며 어떤 액션을 하고, 승자가 결정되면 게임이 종료된다.
<br>
<br>
따라서 아래와 같은 알고리즘을 정의할 수 있다.

```
class Game
{
    void run()
    {
        start();
        while(!have_winner())
            take_turn();
        cout << "Player " << get_winner() << " wins.\n";
    }
};
```

게임을 실행하는 run 메서드는 단지 다른 메서드들을 호출할 뿐이다.
<br>
그 메시지들은 퓨어 버추얼로 정의되어 있고 protected이기 때문에 다른 쪽에서 호출할 수는 없다.

```
protected:
    virtual void start() = 0;
    virtual bool have_winner() = 0;
    virtual void take_turn() = 0;
    virtual int get_winner() = 0;
```

일반적으로 보면 꼭 퓨어 버추얼이어야만 할 이유는 없다.
<br>
특히 리턴 값이 void인 경우는 더욱 그렇다.
<br>
<br>
예를 들어 만약 어떤 게임에 명시적은 start() 단계가 없다면 퓨어 버추얼인 start() 메서드는 인터페이스 분리 원칙(ISP)를 위반을 유도하게 된다.
<br>
왜냐하면 필요 없는 멤버를 구현해야만 하기 때문이다.
<br>
<br>
이제 위 코드에 공통적인 기능을 추가한다.
<br>
플레이어의 수와 현재 플레이어의 인덱스는 모든 게임에서 의미가 있다.

```
class Game
{
public:
    explicit Game(int number_of_playes)
        : number_of_playes(number_of_players) {}
        
protected:
    int current_player{ 0 };
    int number_of_playes;
};  // 다른 멤버들 생략
```

이제 Game 클래스를 확장해서 체스 게임을 구현할 수 있다.

```
class Chess : public Game
{
public:
    explicit Chess() : Game{ 2 } {}
    
protected:
    void start() override {}
    bool have_winner() override { return turns == max_tunrs; }
    void take_turn() override
    {
        turns++;
        currnet_player = (current_player + 1) % number_of_players;
    }
    int get_winner() override { return current_players; }
    
private:
    int turns{ 0 }, max_turns{ 10 };
};
```

체스는 두 명의 플레이어를 필요로 하기 때문에 생성자에서 2명을 파라미터로 전달한다.
<br>
필요한 함수들을 오버라이딩하면서 가상으로 10번의 말을 두고 승자를 가리키도록 한다.
<br>
<br>
아래는 실행 결과이다.

```
Starting a game of chess with 2 players
Turn 0 taken by player 0
Turn 1 taken by player 1
...
Turn 8 taken by player 0
Turn 9 taken by player 1
Player 0 wins.
```

이 예제는 템플릿 메서드 디자인 패턴의 거의 모든 것을 담고 있다.

<br>

## 장단점

### 장점
- 클라이언트들이 대규모 알고리즘의 특정 부분만 오버라이드하도록 하여 그들이 알고리즘의 다른 부분에 발생하는 변경에 영향을 덜 받도록 할 수 있음
- 중복 코드를 부모 클래스로 가져올 수 있음


### 단점
- 일부 클라이언트들은 알고리즘의 제공된 골격에 의해 제한될 수 있음
- 자식 클래스를 통해 Default 단계을 억제하여 리스코프 치환 원칙을 위반할 수 있음
- 템플릿 메서드들은 단계가 많을 수록 유지보수가 어려움

<br>

## 요약

템플릿 메서드는 상속을 사용하기 때문에 정적인 방법만 존재한다.
<br>
이미 생성된 객체의 부모를 바꿀 방법은 없으므로 상속을 이용하면 정적인 방법이 된다.
<br>
<br>
템플릿 메서드 패턴에서는 설계 차웡늬 선택 사항이 한 가지 뿐이다.
<br>
템플릿 메서드에서 이용될 메서드를 퓨어 버추얼로 할 것이냐 실 구현체로 할 것이냐를 선택해야 한다.
<br>
<br>
자식 클래스에서 모든 메서드를 구현할 필요가 없다면, 실 구현체를 선택하고 아무것도 하지 않는 공백 함수를 두는 것이 더 편리할 수도 있다.
