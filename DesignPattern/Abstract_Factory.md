### Abstract Factory Pattern?

- Abstract Factory = 추상적인 것들을 만드는 공장
- 먼저 만들어야할 컴포넌트들을 추상적으로 정하고 어떤 구체적인 상황이 주어지면 각 컴포넌트들을 구체적으로 만드는 패턴

![](https://velog.velcdn.com/images/sanizzang00/post/2ed280a8-690b-4b1b-a767-5ba537a0ead5/image.png)

예를 들어 Button, CheckBox, TextEdit라는 컴포넌트를 만든다고 가정
이 컴포넌트들은 운영체제에 따라서 만드는 방법이 달라지므로 운영체제가 정해지지 않은 상태에서는 이 컴포넌트들을 구체적으로 만들 수 없다. 하지만 Windows라는 상태가 정해지면 Windows에 대한 Rendering API등을 사용해서 컴포넌트들을 구체적으로 만들 수 있다.

이처럼 일단 만들어야 할 컴포넌트들을 추상적으로 정의하고
그 다음에 구체적인 상황이 정해지면 추상적인 컴포넌트를 구체적으로
생성해 내는 패턴이 Abstract Factory 패턴이다.

![](https://velog.velcdn.com/images/sanizzang00/post/a3b366c9-9403-43df-b0d9-33acfe60922c/image.png)

Button.java

```java
public abstarct class Button {
    protected String caption;

    public Button(String caption) {
        this.caption = caption;
    }

    public void clickEvent() {
        System.out.println(caption + " 버튼을 클릭했습니다.");
    }

    abstract void render();
}
```

CheckBox.java

```java
public abstract class CheckBox {
    protected boolean bChecked;

    public CheckBox(boolean bChecked) {
        this.bChecked = bChecked;
    }

    void setChecked(boolean bChecked) {
        this.bChecked = bChecked;
        render();
    }

    abstract void render();
}
```

TextEdit.java

```java
public abstract class TextEdit {
    protected String value;

    public TextEdit(String value) {
        this.value = value;
    }

    public void setValue(String value) {
        this.value = value;
        render();
    }

    public abstract void render();
}
```

ComponentFactory.java

```java
public abstract class ComponentFactory {
    public abstract Button createButton(String caption);
    public abstract CheckBox createCheckBox(boolean bChecked);
    public abstract TextEdit createTextEdit(String value);
}
```

WindowsFactory.java

```java
public class WindowsButton extends Button {

    public WindowsButton(String caption) {
        super(caption);
    }

    @Override
    void render() {
        System.out.println(
            "Windows 렌더링 API를 이용해 "
            + this.caption
            + " 버튼을 그립니다.");
    }
}
```

LinuxButton.java

```java
public class LinuxButton extends Button {

    public LinuxButton(String caption){
        super(caption);
    }

    @Override
    void render() {
        System.out.println(
            "Linux 렌더링 API를 이용해 "
            + this.caption
            + " 버튼을 그립니다.");
    }

}
```

WindowsCheckBox.java

```java
public class WindowsCheckBox extends Button {

    public WindowsCheckBox(String bChecked){
        super(bChecked);
    }

    @Override
    void render() {
        System.out.println(
            "Windows 렌더링 API를 이용해 "
            + (this.bChecked ? "체크된" : "체크 안된")
            + " 체크 박스를 그립니다.");
    }

}
```

LinuxCheckBox.java

```java
public class LinuxCheckBox extends Button {

    public LinuxCheckBox(String bChecked){
        super(bChecked);
    }

    @Override
    void render() {
        System.out.println(
            "Linux 렌더링 API를 이용해 "
            + (this.bChecked ? "체크된" : "체크 안된")
            + " 체크 박스를 그립니다.");
    }

}
```

WindowsTextEdit.java

```java
public class WindowsTextEdit extends Button {

    public WindowsTextEdit(String value){
        super(value);
    }

    @Override
    void render() {
        System.out.println(
            "Windows 렌더링 API를 이용해 "
            + this.value + " 값을 가진 "
            + " 텍스트에디트를 그립니다.");
    }

}
```

LinuxTextEdit.java

```java
public class LinuxTextEdit extends Button {

    public LinuxTextEdit(String value){
        super(value);
    }

    @Override
    void render() {
        System.out.println(
            "Linux 렌더링 API를 이용해 "
            + this.value + " 값을 가진 "
            + " 텍스트에디트를 그립니다.");
    }

}
```

WindowsFactory.java

```java
public class WindowsFactory extends ComponentFactory {

    @Override
    public Button createButton(String caption) {
        return new WindowsButton(caption);
    }

    @Override
    public CheckBox createCheckBox(boolean bChecked) {
        return new WindowsCheckBox(bChecked);
    }

    @Override
    public TextEdit createTextEdit(String value) {
        return new WindowsTextEdit(value);
    }
}
```

LinuxFactory.java

```java
public class LinuxFactory extends ComponentFactory {

    @Override
    public Button createButton(String caption) {
        return new LinuxButton(caption);
    }

    @Override
    public CheckBox createCheckBox(boolean bChecked) {
        return new LinuxCheckBox(bChecked);
    }

    @Override
    public TextEdit createTextEdit(String value) {
        return new LinuxTextEdit(value);
    }
}
```

MainEntry.java

```java
public class MainEntry {
    public static void main(String[] args) {
        ComponentFactory factory = new LinuxFactory();

        Button button = factory.createButton("확인");
        CheckBox checkBox = factory.createCheckBox(false);
        TextEdit textEdit = factory.createTextEdit("디자인패턴");

        button.render();
        checkBox.render();
        textEdit.render();

        button.clickEvent();
        checkBox.setChecked(true);
        textEdit.setValue("GoF의 디자인패턴");
        // 컴포넌트를 사용하는 코드 어디에도 운영체제의 종속적인 코드가 없다는 것을 알 수 있다.
    }
}
```

어떤 상황에 대해서 그 상항에 맞는 컴포넌트들을 생성해 내는 패턴이다.
각 Factory는 자신이 만들 구체적인 컴포넌트가 무엇인지 명확히 정해져 있다.
