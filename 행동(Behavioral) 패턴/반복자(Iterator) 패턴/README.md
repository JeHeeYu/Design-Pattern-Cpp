# 반복자 패턴 정리 내용

## 반복자(Iterator) 패턴
반복자는 컬렉션 요소들의 기본 표현(리스트, 스택, 트리 등)을 노출하지 않고 그들을 하나씩 순회할 수 있도록 하는 행동 디자인 패턴이다.
<br>
복잡한 데이터 구조를 다루어야 할 때는 항상 데이터 순회(Traversal) 문제에 부딪힌다.
<br>
데이터의 순회는 여러 방법으로 처리될 수 있지만 벡터 형태의 데이터 에서는 반복자라고 불리는 방법이 가장 흔하게 사용된다.
<br>
<br>
반복자는 매우 단순하다.
<br>
어떤 컬렉션의 항목 하나에 접근하는 방법과 그 항목의 다음 항목으로 이동하는 방법을 알고 있는 것을 반복자라고 한다.
<br>
<br>
따라서 반복자는 ++ 연산자와 != 연산자만 구현하면 된다.
<br>
!= 연산자는 두 개의 반복자가 접근하는 항목이 같은 항목인지 아닌지 비교하기 위한 용도이다.

<br>

## 표준 라이브러리의 반복자

예를 들어 다음과 같이 이름의 목록을 가지고 있다고 가정한다.

```
vector<string> names{ "john", "jane", "jill", "jack" };
```

이러한 이름의 컬렉션에서 첫 번째 이름을 얻고 싶을 때 begin() 함수를 호출한다.
<br>
이 함수는 이름의 갑싱나 참조를 넘겨주지 않고, 대신 반복자를 넘겨준다.

```
vector<string>::iterator it = names.begin(); // begin(names)
```

begin()은 vector의 멤버 함수로도 존재하고 전역 함수로도 존재한다.
<br>
전역 함수는 저수준 배열(std::array 가 아닌 C 스타일 배열)을 사욯알 때 편리하다.
<br>
왜냐하면 저수준 배열은 메서드가 존재하지 않기 때문이다.
<br>
<br>
begin()이 포인터를 리턴한다고도 볼 수 있다.
<br>
vector의 반복자는 포인터처럼 동작한다.
<br>
<br>
예를 들어 다음과 같이 반복자를 역참조하여 이름의 값을 출력할 수 있다.

```
cout << "first name is " << *it << "\n";
// 출력 결과 : first name is john
```

얻어진 반복자는 컬렉션을 순회하는 방법을 알고 있다.
<br>
즉, 현재 항목의 다음 항목을 가리키도록 이동하는 방법을 알고 있다.
<br>
<br>
++ 연산자가 포인터의 메모리 주솟값 증가가 아니라 다음 항목으로의 이동을 의미한다는 것을 알아야 한다.

```
++it; // 이제 jane을 가리킴
```

반복자를 이용해 마치 포인터를 이용하듯 접근되는 항목의 값을 바꿀 수도 있다.

```
it->append(" goodall"s);
cout << "full name is " << *it << "\n";

// 출력 결과 : full name is jane goodall
```

begin() 함수의 반대편 짝은 당연하게도 end()이다.
<br>
하지만 end()는 기대와 달리 마지막 항목을 가리키지 않는다.
<br>
대신 마지막 항목 다음 위치를 가리킨다.
<br>
이것을 표현하면 아래와 같다.

```
         1    2    3    4
 begin() ^                 ^ end()
```

end()는 종료 조건으로 활용할 수 있다.
<br>
예를 들어 반복자 it을 이용해 나머지 이름들을 출력할 때 다음과 같이 할 수 있다.

```
while(++it != names.end())
{
    cout << "another name: " << *it << "\n";
}
// 출력 결과 : anoter name: jill
//             another name: jack
```

begin(), end()와 더불어 rbegin(), rend()도 제공된다.
<br>
이 함수들은 컬렉션을 역방향으로 순회한다.
<br>
rbegin()은 마지막 항목을, rend()는 첫 번째 항목의 바로 앞 위치를 가리킨다.

```
for(auto ri = rbegin(names); ri != rend(names); ++ri)
{
    cout << *ri;
    if(ri +1 != rend(names)) // 반복자 위치 계산/비교
        cout << ", ";
}

cout << endl;

// 출력 결과: jack, jill, jane goodall, john
```

위 코드에서 첫 번째로, vector를 역방향 순회함에도 불구하고 ++ 연산자를 사용하고 있다.
<br>
그리고 두 번째로, 반복자를 대상으로 산술 연산이 가능하다.
<br>
위 코드에서 ri + 1 부분은 ri 바로 앞 항목 위치를 가리킨다.
<br>
<br>
객체에 대한 변경을 허락하지 않는 const 반복자도 이용할 수 있다.
<br>
순방향 const 반복자는 cbegin()/cend()를, 역방향 const 반복자는 crbegin()/crend()를 사용한다.

```
vector<string>::const_reverse_iterator jack = crbegin(names);
// 아래의 코드는 빌드 에러가 발생
*jack += "reacher";
```
그리고 모던 C++의 범위 기반 for문을 사용할 수 있다.
<br>
범위 기반 for 루프는 begin()에서 end() 직전까지 순회하는 축약 코드이다.

```
for(auto& name : names)
    cout << "name = " << name << "\n";
```

여기서 변수 name이 참조 타입으로 선언되어 있지만, 값으로서도 반복자를 순회할 수 있다.

<br>

## 이진 트리의 탐색

먼저 트리의 노드를 다음과 같이 정의한다.

```
template <typename T> struct Node
{
    T value;
    Node<T> *left = nullptr;
    Node<T> *right = nullptr;
    Node<T> *parent = nullptr;
    BinaryTree<T> *tree = nullptr;
};
```
각 노드는 왼쪽/오른쪽 가지와 부모, 그리고 전체 트리에 대한 참조를 가진다.
<br>
노드는 그 자체에 대한 정의로 생성될 수 있고, 자식들에 대한 정의를 통해 생성될 수 있다.

```
explicit Node(const T& value)
    : value(value) {}
    
Node(const T& value, Node<T>* const left, Node<T>* const right)
    : value(value), left(left), right(right)
{
    this->left->tree = this->right->tree = tree;
    this->left->parent = this->right->parent = this;
}
```

마지막으로 트리 전체에 대한 포인터를 세팅하는 편의 메서드를 만든다.
<br>
이 메서드는 노드의 모든 자식들에게서 재귀적으로 호출된다.

```
void set_tree(BinaryTree<T>* t)
{
    tree = t;
    if(left) left->set_tree(t);
    if(right) right->set_tree(t);
}
```

이러한 준비를 기반으로 하여 트리 순회를 가능하게 하는 BinaryTree 데이터 구조를 만들 수 있다.

```
template <typename T> struct BinaryTree
{
    Node<T>* root = nullptr;
    
    explicit BinaryTree(Node<T>* const root)
        : root{ root }
    {
        root->set_tree(this);
    }
};
```

이제 트리에 대한 반복자를 정의해야 한다.
<br>
이진 트리의 탐색에는 크게 전위, 중위, 후위 탐색이 있다.
<br>
<br>
여기 예제에서는 전위(Preorder) 탐색 방법으로 구현한다.
<br>
전위 탐색은 다음과 같이 정의된다.
- 항목(잎 노드)을 만나마자 바로 리턴
- 재귀적으로 왼쪽 서브트리 순회
- 재귀적으로 오른쪽 서브트리 순회

전위 탐색 반복자의 생성자 구현은 다음과 같다.

```
template <typename U>
struct PreOrderIterator
{
    Node<U>* current;
    
    explicit PreOrderIterator(Node<U>* current)
        : current(current)
    {
    }
    
    // 다른 멤버 정의
};
```

다른 반복자와 비교 작업이 가능하도록 != operator도 구현해야 한다.
<br>
여기서 만들고 있는 반복자는 포인터를 대상으로 하기 때문에 다음과 같이 쉽게 구현할 수 있다.

```
bool operator!=(const PreOrderIterator<U>& other)
{
    return current != other.current;
}
```

역참조를 위해 * operator도 구현한다.

```
Node<U>& operator*() { return *current; }
```

이제 트리 순회 구현만 남았는데, 이 부분은 간단하지 않다.
<br>
탐색 알고리즘은 재귀적이지만, 순회는 ++ 연산자로 이루어져야 하기 때문에 재귀 상태를 기억하면서 순회가 진행되어야 한다.
<br>
<br>
따라서 다음과 같이 구현한다.

```
PreOrderIterator<U>& operator++()
{
    if(current->right)
    {
        current = current->right;
        
        while(current->left)
            current = current->left;
    }
    else
    {
        Node<T>* p = current->parent;
        
        while(p && current == p->right)
        {
            current = p;
            p = p->parent;
        }
        current = p;
    }
    return *this;
}
```

이 코드는 꽤 난잡한 코드이다.
<br>
특히 재귀 호출 부분이 없어 전통적인 트리 탐색 구현처럼 보이지 않는다.
<br>
<br>
이제 마지막으로 BinaryTree에서 이 반복자를 어떻게 클라이언트에 노출하냐 이다.
<br>
만약 이 방식을 트리의 디폴트 탐색 방법으로 한다면 그냥 아래처럼 메서드를 만들면 된다.

```
typdef PreOrderIterator<T> iterator;

iterator begin()
{
    Node<T>* n = root;
    
    if(n)
        whilte(n->left)
            n = n->left;
        return iterator{ n };
}

iterator end()
{
    return iterator{ nullptr };
}
```

이제 아래처럼 반복자를 이용해 트리를 순회할 수 있다.

```
BinaryTree<string> family {
    new Node<string> {"me",
        new Node<string>{"mother",
            new Node<string>{"mother's mother"},
            new Node<string>{"mother's father"}
        },
        new Noew<string>{"father"}
    }
};
```

이러한 순회 방법을 별도의 객체로 분리하여 제공할 수도 있다.
<br>
즉, 아래와 같이 전위 순회 클래스를 정의할 수 있다.

```
class pre_order_traversal
{
    BinaryTree<T>& tree;

public:
    pre_order_traversal(BinaryTree<T>& tree) : tree{tree} {}
    iterator begin() { return tree.begin(); }
    iterator end() { retutn tree.end(); }
} pre_order;
```

이 코드를 아래와 같이 사용할 수 있다.

```
for(const auto& it: family.pre_order)
{
    cout << it.value << "\n";
}
```

<br>

## 코루틴(Coroutine)을 이용한 순회

코루틴(coroutine)은 루틴의 일종으로서, 협동 루틴이라 할 수 있다(코루틴의 "Co"는 with 또는 together를 뜻한다). 
<br>
상호 연계 프로그램을 일컫는다고도 표현가능하다. 
<br>
<br>
코루틴을 이용하면 트리의 후위 탐색을 아래와 같이 구현할 수 있다.

```
generator<Node<T>*> post_order_impl(Node<T>* node)
{
    if(node)
    {
        for(auto x : post_order_impl(node->left))
            co_yield x;
        for(auto y : post_order_impl(node->right))
            co_yield y;
        co_yield node;
    }
}
```

이제 아래와 같이 반복자 인터페이스를 제공할 수 있다.

```
for(auto it: family.post_order())
{
    cout << it->value << endl;
}
```

<br>

## 장단점

### 장점
- 부피가 큰 순회 알고리즘들을 별도의 클래스들로 추출하여 클라이언트 코드와 컬렉션들을 정리할 수 있음
- 새로운 유형의 컬렉션들과 반복자들을 구현할 수 있으며 이들을 아무것도 훼손하지 않은 채 기존의 코드에 전달할 수 있음
- 컬렉션을 병렬로 순회할 수 있음
- 순회를 지연하고 필요할 때 계속 진행할 수 있음


### 단점
- 앱이 단순한 컬렉션들과만 작동하는 경우 반복자 패턴이 필요 없을 수 있음
- 일부 특수 컬렉션들의 요소들을 직접 탐색하는 것보다 비효율적일 수 있음


<br>

## 요약

C++에서 반복자 디자인 패턴을 명시적인 형태와 묵시적인 형태를 모두 지원한다.
<br>
서로 다른 객체의 순회를 위해 서로 다른 반복자 타입이 존재한다.
<br>
<br>
예를 들어 vector에서는 역방향 반복자가 지원되지만 싱글 연결 리스트에서는 지원되지 않는다.
<br>
<br>
반복자를 직접 구현하는 것은 ++, != 연산자를 구현하는 것만큼 간단하다.
<br>
대부분 반복자들은 컬렉션을 순회하기 위한 목적으로 만들어진 포인터 연산의 단순한 퍼사드 인터페이스로 볼 수 있다.
<br>
<br>
코루틴은 반복자가 가진 문제를 해결할 수 있다.
<br>
코루틴을 이용하면 반복자의 호출 간에 상대를 보존할 수 있어 재귀 알고리즘을 구현하는 것이 가능하다.

