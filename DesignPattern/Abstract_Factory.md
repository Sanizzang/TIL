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
