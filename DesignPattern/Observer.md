### Observer?

![](https://velog.velcdn.com/images/sanizzang00/post/b1e09867-4443-48a1-9d70-709ab03f7c2c/image.png)

Observer는 관찰자라는 뜻인데, 어떤 상태가 변경되었을 때 이 상태의 변경에
관심이 있는 관찰자들에게 알려주는 패턴이다.
대상의 상태가 변경되면 자신의 상태가 변경되었음을 알린다.

![](https://velog.velcdn.com/images/sanizzang00/post/bd80755d-d9e9-4a19-b6f6-72355dd6b754/image.png)

DiceGame은 주사위 게임에 참가하는 이 플레이어의 클래스의 객체를 여러개 추가할 수 있고
DiceGame은 주사위를 던지면 주사위의 갯수를 게임에 참가한 플레이어에게 알리는데 던진 주사위의 수가
바로 상태 변화에 해당한다.

Player.java

```java
public abstract class Player {
    private String name;

    public Player(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public abstract void update(int diceNumber);
}
```

DiceGame.java

```java
import java.util.LinkedList;

public abstract class DiceGame {
    protected LinkedList<Player> playerList = new LinkedList<Player>();

    public void addPlayer(Player player) {
        playerList.add(player);
    }

    public abstract void play();
}
```

FairDiceGame

```java
import java.util.Iterator;
import java.util.Random;

public class FairDiceGame extends DiceGame {
    private Random random = new Random();

    @Override
    public void play() {
        int diceNumber = random.nextInt(6) + 1;
        System.out.println("주사위 던졌다~ " + diceNumber);

        Iterator<Player> iter = playList.iterator();
        while(iter.hasNext()) {
            iter.next().update(diceNumber);
        }
    }
}
```

OddBettingPlayer.java

```java
public class OddBettingPlayer extends Player {
    public OddBettingPlayer(String name) {
        super(name);
    }

    @Override
    public void update(int diceNumber) {
        if(diceNumber % 2 ==1) {
            System.out.println(getName() + "win!");
        }
    }
}
```

EvenBettingPlayer.java

```java
public class EvenBettingPlayer extends Player {
    public EvenBettingPlayer(String name) {
        super(name);
    }

    @Override
    public void update(int diceNumber) {
        if(diceNumber % 2 == 0) {
            System.out.println(getName() + "win!");
        }
    }
}
```

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        DiceGame diceGame = new FairDiceGame();

        Player player1 = new EvenBettingPlayer("짝궁댕이");
        Player player2 = new OddBettingPlayer("홀아비");
        Player player3 = new OddBettingPlayer("홀쭉이");

        diceGame.addPlayer(player1);
        diceGame.addPlayer(player2);
        diceGame.addPlayer(player3);

        for(int i = 0;i < 5;i++){
            diceGame.play();
            System.out.println();
        }
    }
}
```
