## 그래프 탐색 알고리즘: DFS/BFS

dfs와 bfs는 대표적인 그래프 탐색 알고르즘이다

- 탐색이란 많의 양의 데이터 중에서 원하는 데이터를 찾는 과정을 말한다.

### 스택 자료구조

- 먼저 들어 온 데이터가 나중에 나가는 형식(선입후출)의 자료구조
- 입구와 출구가 동일한 형태

```python
stack = []

# (1) (2) (2-삭제) (3) (3-삭제) => (1)
stack.append(1)
stack.append(2)
stack.pop()
stack.append(3)
stack.pop()
```

- 파이썬에서 스택 자료구조를 사용하기 위해선 단순히 리스트 자료형을 이용하면 된다.
- 리스트 자료형은 가장 오른쪽에 원소를 삽입하는 append() 메서드와 가장 오른쪽에서 한 원소를 꺼내는 pop() 메서드를 지원하기 때문에 이를 그대로 이용해서 스택을 이용할 수 있다.
- append() 메서드와 pop() 메서드의 시간 복잡도는 O(1)이기 때문에 스택 자료구조로 활용하기에 적합하다.
- 파이썬에서는 별도의 라이브러리 사용 없이 기본적으로 제공되는 리스트를 이용해서 스택을 이용할 수 있다.

### 큐 자료구조

- 먼저 들어 온 데이터가 먼저 나가는 형식(선입선출)의 자료구조
- 입구와 출구가 모두 뚫려 있는 터널과 같은 형태

```python
from collections import deque

queue = deque()

queue.append(1)
queue.append(2)
queue.append(3)
queue.popleft()
queue.append(4)
queue.popleft()

print(queue) # 먼저 들어온 순서대로 출력
queue.reverse() # 역순으로 바꾸기
print(queue) # 나중에 들어온 원소부터 출력
```

- 이런 큐 자료구조를 파이썬에서 이용하고자 할 때는 deque 라이브러리를 사용할 수 있다.
- 리스트로도 큐를 구현할 수는 있지만 시간복잡도가 더 높아서 비효율적으로 동작할 수 있기 때문에 큐를 이용할 때는 꼭 deque 라이브러리를 사용하자
- 엄밀히 말하면 deque라이브러리는 스택과 큐 자료구조의 장점을 모두 합친 자료구조라고 보면 된다.

### 재귀 함수

- 재귀 함수(Recursive Function)란 자기 자신을 다시 호출하는 함수를 의미한다.

```python
def recursive_function():
    print("재귀 함수 호출")
    recursive_function()

recursive_function()
```

- 파이썬에서는 기본적인 재귀를 호출하는 과정에서 깊이 제한이 있기 때문에 별다른 설정을 하지 않고 함수를 재귀적으로 호출하면 오류 메시지가 나올 수 있다.

```python
def recursive_function():
    print("재귀 함수 호출")
    if i == 10:
        print("i = 10 재귀함수 종료")
        return

    recursive_function(i + 1)

recursive_function()
```

- 재귀 함수를 문제 풀이에서 사용할 때는 재귀 함수의 종료 조건을 반드시 명시해야 한다.
- 종료 조건을 제대로 명시하지 않으면 함수가 무한히 호출될 수 있다.

#### 팩토리얼 구현 예제

- n! = 1 x 2 x 3 x ... x (n-1) x n
- 수학적으로 0!과 1!의 값은 1이다.

```python
def factorial_iterative(n):
    result = 1
    # 1부터 n까지의 수를 차례대로 곱하기
    for i in range(1, n + 1):
        result *= i
    return result

# 재귀적으로 구현한 n!
def factorial_recursive(n):
    if n <= 1: # n이 1 이하인 경우 1을 반환
        return 1
    # n! = n * (n - 1)!를 그대로 코드로 작성하기
    return n * factorial_recursive(n - 1)
```

재귀적으로 구현한 팩토리얼은 수학적으로 정의된 점화식을 그대로 이용한다는 점에서 실제로 반복문을 이용해서 작성했을 때 보다 코드가 더 간결하고 직관적일 수 있다는 점을 확인할 수 있다.

### 재귀 함수 사용의 유의 사항

- 재귀 함수를 잘 활용하면 복잡한 알고르짐을 간결하게 작성할 수 있다.
  - 단, 오히려 다른 사람이 이해하기 어려운 형태의 코드가 될 수도 있으므로 신중하게 사용해야 한다.
- 모든 재귀 함수는 반복문을 이용하여 동일한 기능을 구현할 수 있다.
- 재귀 함수가 반복문보다 유리한 경우도 있고 불리한 경우도 있다.
- 컴퓨터가 함수를 연속적으로 호출하면 컴퓨터 메모리 내부의 스택 프레임에 쌓인다.
  - 그래서 스택을 사용해야 할 때 구현상 스택 라이브러리 대신에 재귀 함수를 이용하는 경우가 많다.

### DFS (Depth-First Search)

- DFS는 깊이 우선 탐색이라고도 부르며 그래프에서 깊은 부분을 우선적으로 탐색하는 알고리즘이다.
- DFS는 스택 자료구조(혹은 재귀 함수)를 이용하며, 구체적인 동작 과정은 다음과 같다.
  1. 탐색 시작 노드를 스택에 삽입하고 방문 처리를 한다.
  2. 스택의 최상단 노드에 방문하지 않은 인접한 노드가 하나라도 있으면 그 노드를 스택에 넣고 방문처리한다. 방문하지 않은 인접노드가 없으면 스택에서 최상단 노드를 꺼낸다.
  3. 더 이상 2번의 과정을 수행할 수 없을 때까지 반복한다.

```python
def dfs(graph, v, visitied)
    # 현재 노드를 방문 처리
    visited[v] = True
    print(v, end=' ')
    # 현재 노드와 연결된 다른 노드를 재귀적으로 방문
    for i in graph[v]:
        if not visited[i]:
            dfs(graph, i, visited)

# 각 노드가 연결된 정보를 표현
graph - [
    [],
    [2, 3, 8],
    [1, 7],
    [1, 4, 5],
    [3, 5],
    [3, 4],
    [7],
    [2, 6, 8],
    [1, 7],
]

# 각 노드가 방문된 정보를 표현
# 0번 노드를 사용하지 않을거기 때문에 크기를 9로 지정
visited = [False] * 9

# 정의된 DFS 함수 호출 (1번 노드부터 시작)
dfs(graph, 1, visited)
```

## BFS(Breadth-First Search)

- BFS는 너비 우선 탐색이라고도 부르며, 그래프에서 가까운 노드부터 우선적으로 탐색하는 알고리즘이다.
- BFS는 큐 자료구조를 이용하며, 구체적인 동작 과정은 다음과 같다.
  1. 탐색 시작 노드를 큐에 삽입하고 방문 처리를 한다.
  2. 큐에서 노드를 꺼낸 뒤에 해당 노드의 인접 노드 중에서 방문하지 않은 노드를 모두 큐에 삽입하고 방문처리한다.
  3. 더 이상 2번의 과정을 수행할 수 없을 때까지 반복한다.
- BFS는 특정 조건에서의 최단 경로 문제를 해결하기 위한 목적으로도 효과적으로 사용될 수 있다.

```python
from collections import deque

def bfs(graph, start, visited):
    # 큐 구현을 위해 deque 라이브러리 사용
    queue = deque([start])
    # 현재 노드를 방문 처리
    visited[start] = True
    # 큐가 빌 때까지 반복
    while queue:
        # 큐에서 하나의 원소를 뽑아 출력하기
        v = queue.popleft()
        print(v, end=' ')
        # 아직 방문하지 않은 인접한 원소들을 큐에 삽입
        for i in graph[v]:
            if not visited[i]:
                queue.append(i)
                visited[i] = True

# 각 노드가 연결된 정보를 표현
graph - [
    [],
    [2, 3, 8],
    [1, 7],
    [1, 4, 5],
    [3, 5],
    [3, 4],
    [7],
    [2, 6, 8],
    [1, 7],
]

# 각 노드가 방문된 정보를 표현
# 0번 노드를 사용하지 않을거기 때문에 크기를 9로 지정
visited = [False] * 9

# 정의된 BFS 함수 호출 (1번 노드부터 시작)
bfs(graph, 1, visited)
```

### <문제>음료수 얼려 먹기: 문제 설명

![](https://velog.velcdn.com/images/sanizzang00/post/e95f83b8-e7d7-4fdb-b194-05487bd2a66d/image.png)

#### 문제 해결 아이디어

1. 특정한 지점의 주변 상, 하, 좌, 우를 살펴본 뒤에 주변 지점 중에서 값이 '0'이면서 아직 방문하지 않은 지점이 있다면 해당 지점을 방문한다.
2. 방문한 지점에서 다시 상, 하, 좌, 우를 살펴보면서 방문을 진행하는 과정을 반복하면, 연결된 모든 지점을 방문할 수 있다.
3. 모든 노드에 대하여 1 ~ 2번의 과정을 반복하며, 방문하지 않은 지점의 수를 카운트한다.

```python
# DFS로 특정 노드를 방문하고 연결된 모든 노드들도 방문
def dfs(x, y):
    # 주어진 범위를 벗어나는 경우에는 즉시 종료
    if x <= -1 or x >= n or y <= -1 or y >=m:
        return False
    # 현재 노드를 아직 방문하지 않았다면
    if graph[x][y] == 0:
        # 해당 노드 방문 처리
        graph[x][y] = 1
        # 상, 하, 좌, 우의 위치들도 모두 재귀적으로 호출
        dfs(x - 1, y)
        dfs(x, y - 1)
        dfs(x + 1, y)
        dfs(x, y + 1)
        return True
    return False

# N, M을 공백으로 기준으로 구분하여 입력 받기
n, m = map(int, input().split())

# 2차원 리스트의 맵 정보 입력 받기
graph = []
for i in range(n):
    graph.append(list(map(int, input())))

# 모든 노드(위치)에 대하여 음료수 채우기
result = 0
for i in range(n):
    for j in range(m):
        # 현재 위치에서 DFS 수행
        if dfs(i, j) == True:
            result += 1
print(result) # 정답 출력
```

### <문제> 미로 탈출: 문제 설명

![](https://velog.velcdn.com/images/sanizzang00/post/6ea2c2de-30c5-49ef-bd89-92827696c532/image.png)

#### 문제 해결 아이디어

- BFS는 시작 지점에서 가까운 노드부터 차례대로 그래프의 모든 노드를 탐색한다.
- 상, 하, 좌, 우로 연결된 모든 노드로의 거리가 1로 동일하다.
  - 따라서 (1, 1) 지점부터 BFS를 수행하여 모든 노드의 최단 거리 값을 기록하면 해결할 수 있다.

```python
from collections import deque

# N, M을 공백을 기준으로 구분하여 입력 받기
n, m = map(int, input().split())
graph = []
for i in range(n):
    graph.append(list(map(int, input())))

# 이동할 네 가지 방향 정의 (상, 하, 좌, 우)
dx = [-1, 1, 0, 0]
dy = [0, 0, -1, 1]

# BFS를 수행한 결과 출력
print(bfs(0,0))

def bfs(x, y):
    # 큐(Queue) 구현을 위해 deque 라이브러리 사용
    queue = deque()
    queue.append((x, y))
    # 큐가 빌 때까지 반복하기
    while queue:
        x, y = queue.popleft()
        # 현재 위치에서 4가지 방향으로의 위치 확인
        for i in range(4):
            nx = x + dx[i]
            ny = y + dy[i]
            # 미로 찾기 공간을 벗어난 경우 무시
            if nx < 0 or nx >= n or ny < 0 or ny >= m:
                continue
            # 벽인 경우 무시
            if graph[nx][ny] == 0:
                continue
            # 해당 노드를 처음 방문하는 경우에만 최단 거리 기록
            if graph[nx][ny] == 1:
                graph[nx][ny] = graph[x][y] + 1
                queue.append((nx, ny))
    # 가장 오른쪽 아래까지의 최단 거리 반환
    return graph[n - 1][m - 1]



def bfs(x, y):
```
