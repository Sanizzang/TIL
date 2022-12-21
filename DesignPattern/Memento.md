### Memento Pattern?

- Memento의 뜻은 "추억", "과거의 기억"
- 객체의 상태를 기억해 두었다가 필요할 때 기억해둔 상태로 객체를 되돌린다.
- 객체의 상태에 대한 기억은 다른 객체에서도 읽기 전용으로 접근할 수 있다.
- 객체의 상태에 대한 기억에 대한 생성은 오직 해당 객체만 할 수 있다.

![](https://velog.velcdn.com/images/sanizzang00/post/8063ad67-1b50-476e-97c7-4f2fa4463ea8/image.png)

Walker.java

```java
import java.util.ArrayList;

public class Walker {
    // Walker의 현재 위치에 대한 좌표
    private int currentX, currentY;
    // Walker가 도달해야 할 목표 좌표
    private int targetX, targetY;
    // 시작좌표에서 목표좌표로 가기 위해서 어떤 Action을 취해야 하는지 순서대로 저장하기 위한 배열 객체
    private ArrayList<String> actionList = new ArrayList<String>();

    public Walker(int currentX, int currentY, int targetX, int targetY) {
        this.currentX = currentX;
        this.currentY = currentY;
        this.targetX = targetX;
        this.targetY = targetY;
    }

    public double walk(String action) {
        actionList.add(action);

        if(action.equals("UP")) {
            currentY += 1;
        } else if(action.equals("RIGHT")) {
            currentX += 1;
        } else if(action.equals("DOWN")) {
            currentY -= 1;
        } else if(action.equals("LEFT")) {
            currentX -= 1;
        }

        return Math.sqrt(Math.pow(currentX - targetX, 2) + Math.pow(currentY - targetY, 2));
    }

    public class Memento {
        private int x, y;
        private ArrayList<String> actionList;
        private Memento() {}
    }

    public Memento createMemento() {
        Memento memento = new Memento();

        memento.x = this.currentX;
        memento.y = this.currentY;
        memento.actionList = (ArrayList<String>)this.actionList.clone();
        
        return memento;
    }

    public void restoreMemento(Memento memento) {
        this.currentX = memento.x;
        this.currentY = memento.y;
        this.actionList = (ArrayList<String>)memento.actionList.clone();
    }

    @Override
    public String toString() {
        return actionList.toString();
    }
}
```

MainEntry.java

```java
import java.util.Random;

public class MainEntry {
    public static void main(String[] args) {
        Walker walker = new Walker(0, 0, 10, 10);
        String[] actions = {"UP", "RIGHT", "DOWN", "LEFT"};
        Random random = new Random();
        double minDistance = Double.MAX_VALUE;
        Walker.Memento memento = null;

        while(true) {
            String action = actions[random.nextInt(4)];
            double distance = walker.walk(action);
            System.out.println(action + " " + distance);

            if(distance == 0.0) {
                break;
            }

            if(minDistance > distance) {
                minDistance = distance;
                memento = walker.createMemento();
            } else {
                if(memento != null) {
                    walker.restoreMemento(memento);
                }
            }
        }

        System.out.println("Walker's path": + walker);
    }
}
```
