# 메멘토 패턴 정리 내용

## 메멘토(Memento) 패턴
메멘토는 객체의 구현 세부 사항을 공개하지 않으면서 해당 객체의 이전 상태를 저장하고 복원할 수 있게 해주는 행동 디자인 패턴이다.
<br>
메멘토는 특정 시점의 시스템 상태를 전용 객체에 저장하여 리턴한다.
<br>
<br>
이 객체는 읽기 전용 속성을 가지며 그 자체적으로는 아무런 동작도 하지 않는다.
<br>
이러한 객체를 토큰(Token)이라고 부르기도 한다.
<br>
<br>
이 객체는 필요할 때 시스템에 주입되어 있던 상태로 되돌린다.

<br>

## 은행 계좌
먼저 은행 계좌를 표현하는 BankAccount 클래스가 있다.

```
class BankAccount
{
    int balance = 0;

public:
    explicit BankAccount(const int balance)
        : balance(balance) {}
        
    Memento deposit(int amount)
    {
        balance += amount;
        return { balance };
    }
};
```

Memento 객체는 계좌를 Memento에 저장된 상태로 되돌릴 때 사용될 수 있다.

```
void restore(const Memento& m)
{
    balance = m.balance;
}
```

Memento 클래스는 아래와 같이 쉽게 구현한다.

```
class Memento
{
    int balance;

public:
    Memento(int balance)
        : balance(balance) {}
        
    friend class BankAccount;
};
```

Memento 클래스는 두 가지 특징이 있다.
- 불변 속성을 가지며, 저장된 잔고 값이 나중에 변경될 수 있다면 존재한 적 없는 과거 상태로 되돌리는게 가능해져 버림
- BankAccount를 friend로 선언하며, balance 필드 변수에 접근하기 위해 필요함<br>friend로 선언하는 대신 Memento 클래스를 BankAccount 내부의 중첩 클래스로 만드는 방법도 있음

이제 다음과 같이 계좌 상태를 기억해 두었다가 되돌릴 수 있다.

```
void memento()
{
    BankAccount ba{ 100 };
    auto m1 = ba.deposit(50);
    auto m2 = ba.deposit(25);
    cout << ba << "\n"; // 잔고: 175
    
    // m1으로 되돌리기(undo)
    ba.restore(m1);
    cout << ba << "\n"; // 잔고: 150
    
    // m2 다시 수행(redo)
    ba.restore(m2);
    cout << ba << "\n"; // 잔고: 175
}
```

이 구현은 충분히 제 역할을 하지만 예외 상황 처리가 부족하다.
<br>
예를 들어 계좌를 만든 초기 상황으로는 돌아갈 수가 없다.
<br>
<br>
왜냐하면 생성자에서는 balance의 값을 리턴할 방법이 없기 때문이다.

<br>

## Undo와 Redo

새로운 은행 계좌 클래스 BankAccount2를 만든다.
<br>
BankAccount2 클래스에서는 생성되는 모든 Memento를 저장한다.

```
class BankAccount2 // undo/redo를 지원하는 은행 계좌 클래스
{
    int balance = 0;
    vector<shared_ptr<Memento>> changes;
    int current;
    
public:
    explicit BankAccount2(const int balance) : balance(balance)
    {
        changes.emplace_back(make_shared<Memento>(balance));
        current = 0;
    }
};
```

이제 계좌 생성 초기 잔고 값이 저장되어 생성자에서 리턴할 수 없는 문제가 해결된다.
<br>
위 코드에서 메멘토를 저잫알 때 shared_ptr를 사용하고, 리턴할 떄도 shared_ptr를 사용한다.
<br>
<br>
그리고 현재 위치를 나타내는 current 필드를 두어 방금 수행한 작업을 기준으로 뒤 또는 앞으로 순회할 수 있게 한다.
<br>
이것을 이용하여 되돌리기(Undo)와 다시하기(Redo) 작업을 쉽게 구현할 수 있다.
<br>
<br>
다음은 deposit 메서드의 구현이다.

```
shared_ptr<Memento> deposit(int amount)
{
    balance += amount;
    auto m = make_shared<Memento>(balance);
    changes.push_back(m);
    ++current;
    return m;
}
```

이제 특정 메멘토에 맞추어 계좌의 상태를 되돌리는 메서드를 구현한다.

```
void restore(const shared_ptr<Memento>& m)
{
    if(m)
    {
        balance = m->balance;
        changes.push_back(m);
        current = changes.size() - 1;
    }
}
```

위 코드의 되돌리기 절차는 앞서 보았던 것과는 크게 다르다.
<br>
첫 번째로, 인자로 받은 shared_ptr가 유효한지 검사한다.
<br>
이러한 검사를 하는 이유는 아무것도 하지 않는 기능을 제공하기 위해서이다.
<br>
이렇게 하면 어떤 선행 작업에서 아무것도 하지 않는 디폴트 값을 리턴할 수도 있다.
<br>
<br>
두 번째로, 메멘토를 복구할 때, 복구로 인한 변경 작업 자체도 변경 리스트에 추가하고 있다.
<br>
이렇게 함으로써 방금 수행한 되돌리기를 다시 되돌리는 것도 가능하게 한다.
<br>
<br>
이제 undo()의 구현이다.

```
shared_ptr<Memento> undo()
{
    if(current > 0)
    {
        --current;
        auto m = changes[current];
        balance = m->balance;
        return m;
    }
    
    return{};
}
```

변경 목록에 대한 현재 위치(current)가 0보다 클 때만 undo()를 수행할 수 있다.
<br>
현재 위치가 0보다 크다면, 현재 위치를 한 칸 뒤로 옮기고 그 위치의 변경 사항을 얻어서 상태에 적용하고 그 변경을 리턴한다.
<br>
만약 이전 메멘토로 되돌릴 수 없다면 shared_ptr의 디폴트 생성자를 통해 공백 포인터 객체를 리턴한다.
<br>
공백 포인터는 restore()에서 검사되어 무시된다.
<br>
<br>
redo()도 비슷하게 구현된다.

```
shared_ptr<Memento> redo()
{
    if(current + 1 < changes.size())
    {
        ++current;
        auto m = changes[current];
        balance = m->balance;
        return m;
    }
    
    return{};
}
```

이제, 되돌리기를 안전하게 수행할 수 있다면 수행하고, 그렇지 않다면 공백 포인터를 리턴한다.
<br>
따라서 아래와 같은 Undo/Redo 작업을 Crash 없게 수행할 수 있다.

```
BankAccount2 ba{ 100 };
ba.deposit(50);
ba.deposit(25); // 누적 잔고 125
cout << ba << "\n";

ba.undo();
cout << "Undo 1: " << ba << "\n"; // Undo 1: 150
ba.undo();
cout << "Undo 2: " << ba << "\n"; // Undo 2: 100
ba.redo();
cout << "Redo 2: " << ba << "\n"; // Redo 2: 150
ba.undo(); // 다시 잔고가 100이 됨
```

<br>

## 장단점

### 장점
- 캡슐화를 위반하지 않고 객체의 상태의 스냅샷들을 생성할 수 있음

### 단점
- 클라이언트들이 메멘토들을 너무 자주 생성하면 많은 메모리를 소모할 수 있음

<br>

## 요약

메멘토 디자인 패턴은 시스템의 상태를 이전으로 되돌리기 위한 토큰을 어떻게 정의하고 어떻게 관리할 수 있는지 보여준다.
<br>
보통 그러한 토큰을 시스템을 특정 상태로 옮기기에 딱 필요한 만큼의 작은 정보를 저장한다.
<br>
<br>
이러한 토큰들을 이용하여 시스템을 임의의 상태로 재설정하는 대신 시스템의 모든 정보를 저장해서 뒤(Undo) 또는 앞(Redo) 상태로의 이동 과정을 하나하나 따져보고 통제할 수도 있다.
