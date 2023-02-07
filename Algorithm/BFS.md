## 개념

### 그래프 탐색: 어떤것들이 연속해서 이어질때, 모두 확인하는 방법

- Graph: Vertex(어떤것) + Edge(이어지는것)

### 그래프 탐색 종류

- BFS: Breadth-first search(너비 우선 탐색)
- DFS: Depth-first search(깊이 우선 탐색)

## 아이디어

- 시작점에 연결된 Vertex 찾기
- 찾은 Vertex를 Queue에 저장
- Queue의 가장 먼저 것 뽑아서 반복
- BFS: Queue
- DFS: Stack

## 시간복잡도

- 알고리즘이 얼마나 오래걸리는지
- BFS: O(V+E)
- O: 빅(O) 표기법

## 자료구조

- 검색할 그래프
- 방문여부 확인(재방문 금지)
- Queue: BFS 실행
