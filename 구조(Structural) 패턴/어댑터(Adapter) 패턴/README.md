# 어댑터 패턴 정리 내용

## 어댑터(Adapter) 패턴
어댑터 패턴은 클래스의 인터페이스를 사용자가 기대하는 다른 인터페이스로 변환하는 패턴으로, 호환성이 없는 인터페이스 때문에 함께 동작할 수 없는 클래스들이 함께 작동하도록 해준다.

## 시나리오
예를 들어 픽셀을 그리는데 적합한 어떤 그리기 라이브러리가 있고 이 라이브러리를 사용해야만 그림을 그릴 수 있다고 가정한다.
<br>
하나하나에 점을 찍는 방식으로 무엇이든 그릴 수는 있지만 기하학적 도형을 그리기에는 너무 저수준 작업이다.
<br>
따라서 기하학적 도형을 픽셀 기반 표현으로 바꾸어주는 어댑터가 필요하다.
<br>
<br>
먼저 아래와 같이 그리기 객체를 정의한다

```
struct Point
{
    int x, y;
};

struct Line
{
    Point start, end;
};
```

이제 기하학적 도형을 모두 담을 수 있도록 일반화 한다.
<br>
가장 일반적인 방법은 선분의 집합으로 표현하는 것이다.
<br>
<br>
아래와 같이 퓨어 버추얼 반복자 메서드의 쌍으로 정의한다.
  
```
struct VectorObject
{
    virtual std::vector<Line>::iterator begin() = 0;
    virtual std::vector<Line>::iterator end() = 0;
};
```

이렇게 하면 사각형 Rectangle을 다음과 같이 그 시작점과 크기를 입력받아 생성하고, 사각형을 구성하는 선분들을 vector 타입 필드에 저장하여 그 꼭짓점들만 노출하는 방식으로 정의할 수 있다.

```
struct VectorRectangle : VectorObject
{
    VectorRectangle(int x, int y, int width, int height)
    {
        lines.emplace_back(Line{ Point{x, y}, Point{x + width, y} });
        lines.emplace_back(Line{ Point{x + width, y}, Point{x + width, y + height} });
        lines.emplace_back(Line{ Point{x, y}, Point{x, y + height} });
        lines.emplace_back(Line{ Point{x, y + height}, Point{x + width, y + height} });
    }
    
    std::vector<Line>::iterator begin() override 
    {
        return lines.begin();
    }
    
    std::vector<Line>::iterator end() override
    {
        return lines.end();
    }
    
private:
    std::vector<Line> lines;
};
```

이제 이 코드를 이용해 화면에 선분을 비롯한 사각형을 그리고 싶다고 가정한다.
<br>
안타깝게도 이 코드로 그릴 수 없는데, 그림을 그리기 위한 인터페이스는 아래와 같은 함수 하나 뿐이다.

```
void DrawPoints(CPaintDC& dc, std::vector<Point>::iterator start, std::vector<Point>::iterator end)
{
    for(auto i = start; i != end; ++i)
    {
        dc.SetPixel(i->x, i->y, 0);
    }
}
```

위코드는 마이크로소프트 MFC 라이브러리의 CPaintDC 클래스이다.
<br>

그리기 인터페이스는 점을 찍는 것밖에 없지만, 선분을 그려야 한다.
<br>
<br>
이러한 이유로 어댑터가 필요


<br>

## 어댑터(Adapter)
사각형 몇 개를 그려야 하는 상황이다.

```
vector<shared_ptr<VectorObject>> vectorObjects {
    make_shared<VectorRectangle>(10, 10, 100, 100),
    make_shared(VectorRectangle>(30, 30, 60, 60)
}
```

이 객체들을 그리기 위해서는 사각형을 이루는 선분의 집합에서 각각의 선분마다 많은 수의 점이 반환되어야 한다.
<br>
이를 위해 별도의 클래스를 만든다.
<br>
<br>
이 클래스는 선분 하나를 점의 집합을 만들어 저장하고 각 점들을 순회할 수 있도록 반복자로 노출한다.

```
struct LineToPointAdapter
{
    typedef vector<Point Points;
    
    LineToPointAdapter(Line& line) { }
    
    virtual Points::iterator begin() { return points.begin(); }
    virtual Points::iterator end() { return points.end(); }
    
private:
    Points points;
};
```

선분을 점의 집합으로 변환하는 작업은 생성자에서 일어난다.
<br>
즉, 이 어댑터는 '성급한 접근법' 을 취한다.
<br>
실제 변환 코드는 복잡하지 않다.

```
LineToPointAdapter(Line& line)
{
    int left = min(line.start.x, line.end.x);
    int right = max(line.start.x, line.end.x);
    int top = min(line.start.y, line.end.y);
    int botton = max(line.start.y, line.end.y);
    int dx = right - left;
    int dy = line.end.y - line.start.y;
    
    // 가로 또는 세로 선분들
    if(dx == 0)
    {
        // 세로
        for(int y = top; y <= bottom; ++y)
        {
            points.emplace_back(Point{ left, y });
        }
    }
    else if(dy == 0)
    {
        for(int x = left; x <= right; ++x)
        {
            points.emplace_back(Point{ x, top });
        }
    }
}
```

위 코드는 매우 쉽다.
<br>
수직, 수평, 선분만 다루고 나머지는 무시한다.
<br>
<br>
이 어댑터를 이용하면 몇몇 기하 도형들을 그릴 수 있다.
<br>
앞서 예제 코드에서 정의한 사각형 두 개를 아래와 같이 단순한 코드로 그릴 수 있다.

```
for(auto& obj : vectorObjects)
{
    for(auto& line : *obj)
    {
        LineToPointAdapter lpo{ line };
        DrawPoints(dc, lpo.begin(), lpo.end());
    }
}
```

이 코드는 기하 도형에서 선분 집합을 정의하면 그 선분들로 LineToPointAdapter를 생성하여 점들의 집합으로 변환하고, 그 점둘을 순회할 수 있는 시작 반복자와 끝 반복자를 DrawPoints에 넘겨주어 그림을 그린다.

<br>

## 일시적 어댑터

아래 코드는 정상적으로 동작하지만, 문제가 되는 상황이 있다.
```
for(auto& obj : vectorObjects)
{
    for(auto& line : *obj)
    {
        LineToPointAdapter lpo{ line };
        DrawPoints(dc, lpo.begin(), lpo.end());
    }
}
```

화면이 업데이트 될 때마다 DrawPoints()가 불리는 경우를 생각할 수 있다.
<br>
전혀 바뀐 겂이 없더라도 도형의 선분들이 어댑터에 의해 점으로 변환된다.
<br>
즉, 비효율적인 중복 작업이 발생하게 된다.
<br>
<br>
이러한 작업을 피하기 위해 먼저 생각할 수 있는 방법은 캐싱을 이용하는 것이다.
<br>
즉, 모든 Point를 애플리케이션이 기동할 때 미리 어댑터를 이용해 정의해 두고 재활용하는 것이다.

```
vector<Point> points;
for(auto& o : vectorObjects)
{
    for(auto& l : *o)
    {
        LineToPointAdapter lpo{ l };
        for(auto& p : lop)
            points.push_back(p);
    }
}
```

이렇게 준비해둔 Point들의 반복자를 DrawPoints()에 넘긴다.

```
DrawPoints(dc, points.begin(), points.end());
```

이제 반복 변환 작업은 없어졌다.
<br>
하지만 또 다른 문제가 생기는데, 원본 vectorObjects가 바뀌어 선분을 다시 점으로 변환해야 하는 경우의 문제이다.
<br>
이 경우는 미리 만들어 두고 재활용하는 방식을 사용할 수 없다.
<br>
비효율적인 반복 변환 작업을 피하기 위해 다시 캐싱을 활용한다.
<br>
<br>
먼저, 반복 변환을 피하기 위해 각각의 선분을 유일하게 식별할 방법이 필요하다.
<br>
즉, 선분의 시작/끝점을 유일하게 식별할 수 있어야 한다.

```
struct Point
{
    int x, y;
    
    friend std::size_t hash_value(const Point& obj)
    {
        std::size_t seed = 0x725C686F;
        boost::hash_combine(seed, obj.x);
        boost::hash_combine(seed, obj.y);
        
        return seed;
    }
};

struct Line
{
    Point start, end;
    
    friend std::size_t hash_value(const Line& obj)
    {
        std::size_t seed = 0x719E6B16;
        boost::hash_combine(seed, obj.start);
        boost::hash_combine(seed, obj.end);
        
        return seed;
    }
};
```

위 코드는 Boost의 해시 편의 기능을 이용하고 있다.
<br>
이제 점들을 캐싱하여 꼭 필요할 때만 선분에서 점을 생성하는 LineToPointCachingAdapter를 만들 수 있다.
<br>
캐시를 가졌다는 것을 제외하면 기존 LineToPointAdapter 구현과 거의 동일하다.

<br>
<br>

먼저, 맵을 이용해 해시값과 점들을 연관시켜 캐시를 만든다.

```
static map<size_t, Points) cache;
```

size_t 타입은 Boost의 해시 함수가 리턴하는 타입과 동일하다.
<br>
이제 생성된 점을 순회하는 반복자를 다음과 같이 해시 값을 이용해 캐시에서 얻는다.

```
virtual Points::iterator begin() { return cache[line_hash].begin(); }
virtual Points::iterator end() { return cache[line_hash].end(); }
```

구현하려는 알고리즘은 다음과 같다.
<br>
점들을 생성하기 전에 이미 생성되었는지 검사하고 만약 이미 생성되어 있으면 그냥 종료하고, 없으면 생성한 다음 캐시에 등록한다.

```
LineToPointCachingAdapter(Line& line)
{
    static boost::hash<Line> hash;
    line_hash = hash(line); // line_hash는 이 클래스의 필드임
    
    iF(cache.find(line_hash) != cache.end())
        return; // 이미 존재하므로 그냥 리턴
        
    Points poiunts;
    
    // 이전과 동일한 코드
    
    cache[line_hash] = points;
}
```

이제 해시 함수와 캐싱 덕분에 변환 생성 작업 횟수를 드라마틱하게 줄일 수 있다.
<br>

## 장단점

### 장점
- 프로그램의 기본 비즈니스 로직에서 인터페이스 또는 데이터 변환 코드를 분리할 수 있음
- 클라이언트 코드가 클라이언트 인터페이스를 통해 어댑터와 작동하지 안는 한, 기존의 클라이언트 코드를 손상시키지 않고 새로운 유형의 어댑터들을 프로그램에 도입할 수 있음

### 단점
- 다수의 새로운 인터페이스와 클래스들을 도입해야 하므로 코드의 전반적인 복잡성이 증가함
- 때로는 코드의 나머지 부분과 작동하도록 서비스 클래스를 변경하는 것이 더 간단함

<br>

## 다른 패턴과의 관계
- 어댑터는 기존 객체의 인터페이스를 변경하는 반면 데코레이터는 객체를 해당 객체의 인터페이스를 변경하지 않고 향상됨<br>또한 데코레이터는 어댑터를 사용할 때는 불가능한 재귀적 합성을 지원함
- 어댑터는 일반적으로 기존 앱과 사용되어 원래 호환되지 않던 일부 클래스들이 더 잘 작동하도록 함<br>반면 브리지는 일반적으로 사전에 설계되며, 앱의 다양한 부분을 독립적으로 개발할 수 있도록 함
- 어댑터는 기존의 인터페이스를 사용할 수 있게 만드는 반면 퍼사드는 기존 객체들을 위한 새 인터페이스를 정의함

<br>

## 요약
어댑터는 매우 단순한 개념의 패턴이다.
<br>
사용할 수밖에 없는 인터페이스를 사용하고 싶은 인터페이스로 감쌀 수 있게 해준다.
<br>
<br>
어댑터가 가진 유일한 난관은 서로 다른 인터페이스를 이어 붙여 나가는 과정에 있다.
<br>
어떤 경우에는 데이터 표현 방식을 맞추기 위해 임시 데이터를 만들어야 할 수도 있다.
<br>
그런 경우에는 캐싱을 활용하여 꼭 필요한 때만 임시 데이터가 생성되게 하여 변환 작업이 일어나는 것을 피한다.
<br>
<br>
추가로, 캐시된 객체가 변경되었을 때 무효해진 데이터를 제거하는 기능도 구현할 필요가 있다.

