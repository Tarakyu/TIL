# Getter, Setter and OOP

getter와 setter가 객체지향에 적합하지 않다는 글을 보았다. Javabean 컨벤션을 공부하면서 당연히 getter와 setter는 필수인 것으로 생각했는데, 헷갈리기 시작했다. 그래서 이런 말들이 왜 나왔는지 알아보았다.



## 1. 애초에 Getter랑 Setter는 왜 필요한데?

```java
// #1
public int level;
// #2
private int level;
public int getLevel() { return this.level; }
```

처음에는 위 두개의 코드가 뭐가 다른 건지 전혀 이해하지 못했다. 어차피 `getLevel()`하면 똑같이 public한거 아닌가?

그래서 getter와 setter를 왜 사용해야 하는지부터 조사해보았다.

#### 1-1) Validation이 가능하다

```java
private int level;
public void setLevel(int level) {
    if (level > 0) { this.level = level; }
}
```

level은 음수가 될 수 없다고 가정해보자. level 변수를 public으로 열어두게 되면 누군가 여기에 음수 값을 집어넣을 수도 있다.

그런데 level을 private 변수로 바꿔주고 setLevel이라는 setter를 통해 값을 변경하도록 하면 들어오는 값이 양수인지 음수인지 사전에 체크할 수 있다.

#### 1-2) 다형성이 가능하다

```java
public class Man {
    protected int level;
    public void setLevel(int level) {
        this.level = level;
    };
}

public class Wizard extends Man {
    @Override
    public void setLevel(int level) {
        if (level > 10) {
            this.level = level;
        }
    }
}

public void levelUp(int level, Man man) {
    man.setLevel(level);
}
```

Man의 레벨은 아무 숫자나 될 수 있지만 이를 상속한 Wizard의 레벨은 10 이상이어야 한다고 가정해보자.

같은 setLevel메소드에 대해 Man과 Wizard는 다른 구현을 가지고 있는데, setter는 이를 활용하기 편하게 만들어준다.

#### 1-3) (중요!) 클라이언트쪽 코드의 수정 없이 구현을 바꿀 수 있다.

```java
// #1
private boolean alive = true;

public boolean isAlive() { return alive; }
public void setAlive(boolean alive) { this.alive = alive; }

// client
kim.setAlive(false);

// #2
private int hp; // change!

public boolean isAlive() { return hp > 0; }
public void setAlive(boolean alive) { this.hp = alive ? 100 : 0; }
```

1번 코드는 alive 프로퍼티를 가지고, 얘가 죽었는지 살았는지 설정하는 setter를 가지고 있다. 그런데 나중에 boolean인 alive 변수를 쓰기보다는 int의 hp변수를 사용하고 싶어졌다고 가정하자. 그렇다면 2번처럼 코드를 바꿀 수 있다. 그런데 놀랍게도 이렇게 내부 구현을 바꾸었지만 클라이언트 코드는 하나도 손댈 필요가 없다. isAlive 메소드와 setAlive 메소드의 사용법을 변하지 않았기 때문이다.



#### 1-4) 그 외

그 외에도 컨벤션을 지킬 수 있고, 디버깅 시 브레이크 포인트가 메소드 내부에 위치해 디버깅에 유리하다는 장점이 있다.



## 2. 근데 왜 Getter와 Setter를 쓰지 말라는 걸까

위에서 설명한 장점들 덕분에 우리는 Getter와 Setter를 사용하곤 하는데, getter와 setter가 OOP의 원칙을 깬다는 얘기는 왜 나온 것일까?



#### 2-1) getter와 setter를 쓰면 안 되는 이유: 클라이언트쪽 코드의 수정 없이 구현을 바꿀 수 있도록 하기 위해.

오잉? 분명 getter와 setter를 쓰는 이유가 저거였는데, 쓰면 안 되는 이유도 이거라고 한다. 사실 getter setter를 쓰자는 사람하고 쓰지 말자는 사람 모두 같은 목적을 가지고 얘기하는 것이다. 우선 이에 대해 내가 이해한 바를 짧게 표현하자면, `내부 구현의 변화에도 클라이언트 코드가 변하지 않도록 하기 위해 getter와 setter를 쓸 수 있는데, 사실 이 방법으로도 조금 부족하고 이보다 더 좋은 대안이 필요하다.`이다.

포켓몬 클래스가 하나 있다고 가정해보자.

```java
public class Pokemon {
    private int level;
    private int power;
    public int getLevel() { return this.level; }
    public int getPower() { return this.power; }
}
```

그리고 여기에 있는 getPower 메소드가 여러 클라이언트에서 사용되었다.

```java
// client1
int a = pikachu.getPower();
// client2
int b = tarakyu.getPower();
// ......
// client1000
int z = raichu.getPower();
```

그런데 두둥. 갑자기 power를 정수가 아닌 long으로 바꾸자고 한다. 내부 구현을 바꿔야 하는 상황이다. 그러면 우리는 getPower 메소드가 사용된 1000개의 코드를 찾아 저 int를 모두 long으로 바꿔줘야 한다. 정말 창문 열고 뛰어내리고 싶은 상황이다.

구현 은닉 원칙(implementation hiding principle)은 OOP의 꽃이다. 우리의 목표는 포켓몬 클래스의 내부를 완전히 뜯어고치고도, 포켓몬 클래스를 이용하고있는 클라이언트의 코드는 하나도 바꾸지 않아도 되도록 하는 것이다. 그렇지 않으면 우리는 객체지향의 의미를 전혀 살리지 못하게 되는 것이다.

내가 처음으로 캡슐화의 설명에서 '정보 은닉'이라는 표현을 들었을 때, '정보를 누구한테서 숨긴다는 거지? 해커한테서 숨긴다는 건가?'라고 생각했다. 사실 나는 hiding(은닉)이라는 표현이 그 의도를 잘 설명하지 못하는 표현인 것 같다. 우리의 목표는 클라이언트에게서 정보나 구현을 숨기는 것이 아니라, '몰라도 되게'하는 것이다. 



#### 2-2) 그럼 어쩌라는 겨?

그렇다고 해서 getter setter를 아예 쓰지 말라던가, 정보 흐름을 아예 차단하라는 의미는 아니다. 물론 정보 흐름은 최대한 줄이는 것이 좋다.  중요한 원칙은 `어떤 작업을 하기 위해 필요한 정보를 묻지 말고, 그 정보를 가지고 있는 객체한테 그 일을 대신해달라고 하라`이다. 애초에 클래스를 구현할 때 클래스간에 어떤 정보를 주고받아야 하는지 잘 계획해야 한다. 명확한 계획 없이 클래스를 만들다보면 이게 필요할지 필요하지 않을지 애매해서 getter와 setter를 무지성으로 만들게 되는 것이다. 



#### 2-3) 예시 

위의 포켓몬 예시에서 포켓몬이 주는 데미지를 계산해야한다고 해보자.

```java
public class Pokemon {
    private int level;
    private int power;
    public int getLevel() { return this.level; }
    public int getPower() { return this.power; }
}
```

처음에는 클라리언트에서 데미지가 파워의 5배라고 구현하였다.

```java
int damage = pikachu.getLevel() * 5;
```

근데 나중에보니 데미지에는 레벨도 중요한 것 같아서, 레벨도 더하기로 했다. 그러면 모든 클라이언트들에서 코드를 다음과 같이 수정해야 한다.

```java
int damage = pikachu.getLevel() * 5 + pikachu.getLevel();
```

근데 이러면 클라이언트의 코드를 바꿔야 하니, 이러지 말고 데미지를 계산하는 메소드를 포켓몬 안에 집어 넣자는 것이다.

```java
public class Pokemon {
    private int level;
    private int power;
    public int getDamage() { return this.power * 5 + this.level; }
}

// client
int damage = pikachu.getDamage();
```

이렇게 하면 나중에 데미지 공식(내부 구현)을 바꾸더라도 클라이언트의 코드를 바꿀 필요가 없다!

즉 클라이언트의 코드는 "나 데미지 구하려면 너가 가진 레벨과 파워라는 정보가 필요하니까 알려줘!"가 되면 안 되고, "너는 레벨과 파워라는 정보를 알고있으니 데미지를 구할 수 있겠네? 너가 데미지 구해서 알려줘!"가 되어야 한다는 것이다.



## 3. 후기

스프링 강의를 보면 getter와 setter가 많이 쓰였다. 이것은 잘못된 구현이었을까? 아니면 필요한만큼 최소한으로 사용한 것일까? 혹은 강의의 단순함을 위해 일부러 간단하게 작성한 것일까? 

내공을 더 쌓은 뒤 나중에 다시 생각해봐야겠다..



## 참고

https://stackoverflow.com/questions/11071407/advantage-of-set-and-get-methods-vs-public-variable

https://stackoverflow.com/questions/2747721/getters-and-setters-are-bad-oo-design

https://www.infoworld.com/article/2073723/why-getter-and-setter-methods-are-evil.html
