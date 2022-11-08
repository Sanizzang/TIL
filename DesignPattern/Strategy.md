### Strategy 패턴?

Gof의 디자인 패턴 중 상대적으로 가장 단순한 패턴
어떤 하나의 기능을 구성하는 특정 부분을 실행중에 다른것으로 효과적으로 변경할 수 있는
방안을 제공 즉 필요할 경우 전략을 바꿀 수 있는 패턴이다.

![](https://velog.velcdn.com/images/sanizzang00/post/e99bb966-7be2-491d-aea7-36c83432095d/image.png)

![](https://velog.velcdn.com/images/sanizzang00/post/5eca3096-39b7-4c9b-92af-dabb0ad38c84/image.png)

SumPrinter는 SumStrategy 인터페이스만을 알 뿐 실제 총 합을 계산하는 SimpleSumStrategy와 GaussSumStrategy의
존재를 모른다.
-> 추후에 총합을 계산하는 다른 방법이 추가되었을 때 기존의 SumPrinter 클래스의 코드를 전혀 변경할 필요가 없어진다.

SumPrinter.java

```java
public class SumPrinter {
    void print(SumStrategy strategy, int N){
        System.out.printf("The Sum of 1 - %d : ", N);
        System.out.println(strategy.get(N));
    }
}
```

n까지의 총 합을 출력하는 프린트 메소드가 있음

SumStrategy.java

```java
public interface SumStrategy {
    int get(int N);
}
```

1부터 n까지의 총합을 얻는 메서드만을 제공

SimpleSumStrategy

```java
public class SimpleSumStrategy implements SumStrategy {
    @Override
    public int get (int N) {
        int sum = N;

        for(long i = 1; i< N; i++) {
            sum += i;
        }

        return sum;
    }
}
```

단순히 1부터 N까지 반복해서 총합을 구함

GaussSumStrategy.java

```java
public class GaussSumStrategy implements SumStrategy {

    @Override
    public int get(int N) {
        return (N+1)*N/2;
    }
}
```

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        SumPrinter cal = new SumPrinter();

        cal.print(new SimpleSumStrategy(), 10);
        cal.print(new GaussSumStrategy(), 10);
    }
}
```
