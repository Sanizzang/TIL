### Flyweight의 목적?

- 어떤 객체를 사용하기 위해 매번 생성하지 않고 한번만 생성하고 다시 필요해 질때는
  이전에 생성된 객체를 재사용할 수 있음
- 객체를 생성할 때 많은 자원을 소모될 경우 플라이웨이트 패턴을 적용하여 훨씬 적은
  자원만으로 필요한 객체를 재사용할 수 있음

![](https://velog.velcdn.com/images/sanizzang00/post/75efd5fe-9743-4389-9ffd-4be3a97996fc/image.png)

Digit: 0부터 9까지의 단 하나의 숫자를 매우 멋지게 화면에 표시하는데 이 멋진 하나의
숫자를 표시하기 위해 상당한 량의 메모리를 사용한다고 가정

이때 사용하는 데이터는 파일명이 0부터 9까지 정해진 총 10개의 텍스트 파일이다.

Digit 클래스는 생성될 때 정해진 하나의 파일을 오픈하고
파일의 데이터를 모두 읽어서 메모리에 적재한 뒤 이를 이용해 화면에 출력한다.

결과적으로 Digit 객체를 생성할 때는 파일을 열고 데이터를 읽어 메모리에 모두 적재해야해서
상당한 비용이 발생한다고 할 수 있다.

그리고 DigitFactory는 원하는 숫자에 대한 Digit 객체를 요청하면 해당되는 Digit 객체를
반환해주는 역할을 한다. 이 DigitFactory는 똑똑하게도 이 Digit객체를 미리 생성해 두지 않고,
처음 요청 받았을 때 한번만 생성하고 다음에 동일한 숫자의 요청을 받으면 이전에 생성했던
Digit 객체를 반환한다.

Number클래스는 긴 숫자값을 화면에 출력하는 역할을 수행하는데 긴 숫자 값을 구성하는 각 숫자에 대한
Digit객체를 DigitFactory를 사용해서 작성한다.

![](https://velog.velcdn.com/images/sanizzang00/post/85d10e6f-106d-4911-9548-0d4d17a38ceb/image.png)

이 숫자는 6자리 이므로 6개의 Digit 객체를 이용함에도 이 6개의 Digit객체가 모두 생성된것은 아니고
Flyweight 패턴이 적용되어 실제로는 생성된 3개의 Digit 객체만이 사용된다.

Digit.java

```java
import java.util.ArrayList;

public class Digit {
    private ArrayList<String> data = new ArrayList<String>();

    public Digit(String fileName) {
        FileReader fr = null;
        BufferedReader br = null;

        try {
            fr = new FileReader(fileName);
            br = new BufferedReader(fr);

            for(int i = 0;i < 8;i++) {
                data.add(br.readLine());
            }
        } catch(IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if(fr != null) fr.close();
                if(br != null) br.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    public void print(int x, int y) {
        for(int i = 0;i < 8;i++) {
            String line = data.get(i);
            System.out.print(String.format("%c[%d;%df", 0x1B,y+i,x));
            System.out.print(line);
        }
    }
}
```

DigitFactory.java

```java
public class DigitFactory {
    private Digit[] pool = new Digit[] {
        null, null, null, null, null, null, null, null, null, null
    };

    public Digit getDigit(int n) {
        if(pool[n] != null) {
            return pool[n];
        } else {
            String fileName = String.format("./data/%d.txt", n);
            Digit digit = new Digit(fileName);
            pool[n] = digit;
            return digit;
        }
    }
}
```

Number.java

```java
public class Number {
    private ArrayList<Digit> digitList = new ArrayList<Digit>();

    public Number(int number) {
        String strNum = Integer.toString(number);
        int len = strNum.length();

        DigitFactory digitFactory = new DigitFactory();
        for(int i = 0;i < len; i++) {
            int n = Character.getNumericValue(strNum.charAt(i));
            Digit digit = digitFactory.getDigit(n);
            digitList.add(digit);
        }
    }

    public void print(int x, int y) {
        int cntDigits = digitList.size();
        for(int = 0;i < cntDigits;i++) {
            Digit digit = digitList.get(i);
            digit.print(x+(i*8), y);
        }
    }
}
```

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        Number number = new Number(434331);
        number.print(5, 5);

        System.out.println();
        System.out.println();
        System.out.println();
    }
}
```
