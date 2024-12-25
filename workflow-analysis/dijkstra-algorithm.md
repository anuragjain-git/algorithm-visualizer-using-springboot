# Dijkstra Algorithm Implementation Flow Analysis

## 1. Entry Point: AlgoController.java
```java
@GetMapping("path-find")
public String getPathFindSteps(@RequestParam("name") PathFindType name, Model model)
```
- Triggered when user selects "Dijkstra" in UI
- `@GetMapping`: Maps HTTP GET request to /path-find
- `@RequestParam`: Extracts algorithm name from URL parameter
- Calls `pathFindService.getByType(name)`

## 2. PathFindService.java
```java
@Service
public class PathFindService {
    private final int ROW = 40;
    private final int COL = 40;
    
    @PostConstruct
    public void init() {
        pathFindMap.put(DIJKSTRA, new Dijkstra());
    }
```
- `@Service`: Spring component for business logic
- `@PostConstruct`: Initializes map with algorithm implementations
- Creates 40x40 grid matrix where:
  - 0: Empty cell
  - 1: Wall
  - 2: Target/end point

## 3. Dijkstra.java Core Implementation

### Main Method
```java
public PathFindResult find(int[][] matrix) {
    moves = new ArrayList<>();
    Cell cell = dijkstra(matrix);
    List<int[]> shortestPath = new ArrayList<>();
    
    while(cell != null) {
        shortestPath.add(new int[]{cell.row(), cell.col()});
        cell = cell.prev();
    }
    Collections.reverse(shortestPath);
```
- Input: 40x40 grid matrix
- Creates tracking lists for:
  - `moves`: All visited cells
  - `shortestPath`: Final path from start to end

### Dijkstra Algorithm Core
```java
private Cell dijkstra(int[][] matrix) {
    int row = matrix.length;
    int col = matrix[0].length;
    boolean[][] visited = new boolean[row][col];
    int[][] dist = new int[row][col];
    for(int[] a : dist) Arrays.fill(a, Integer.MAX_VALUE);
    PriorityQueue<Cell> q = new PriorityQueue<>((a,b) -> a.dist() - b.dist());
```
Key Components:
1. `visited[][]`: Tracks explored cells
2. `dist[][]`: Stores shortest distance to each cell
3. `PriorityQueue`: Orders cells by shortest distance

### Cell Processing
```java
while(!q.isEmpty()) {
    Cell cur = q.poll();
    if (visited[cur.row()][cur.col()]) continue;
    moves.add(new int[]{cur.row(), cur.col()});

    for(int[] d : directions) {
        int nr = cur.row() + d[0];
        int nc = cur.col() + d[1];
```
- Explores cells in order of shortest distance
- `directions`: 4 possible moves defined in PathFind.java:
  - Up: [-1, 0]
  - Down: [1, 0]
  - Left: [0, -1]
  - Right: [0, 1]

### Distance Update Logic
```java
if(nr >= 0 && nc >= 0 && nr < row && nc < col && matrix[nr][nc] != 1) {
    if(cur.dist() + 1 < dist[nr][nc]) {
        dist[nr][nc] = cur.dist() + 1;
        Cell cell = new Cell(nr, nc, cur.dist() + 1, cur);
        if (matrix[nr][nc] == 2) {
            return cell;
        }
        q.add(cell);
    }
}
```
1. Boundary check: `nr >= 0 && nc >= 0 && nr < row && nc < col`
2. Wall check: `matrix[nr][nc] != 1`
3. Shorter path check: `cur.dist() + 1 < dist[nr][nc]`
4. End check: `matrix[nr][nc] == 2`

## 4. Frontend Visualization (path-find.js)
```javascript
function animateMoves(result, delay) {
    const highlightCell = (move, className, index, revertDelay) => {
        setTimeout(() => {
            const cellIndex = move[0] * gridSize + move[1];
            const cell = grid.querySelector(`.cell[data-index='${cellIndex}']`);
```
- Receives PathFindResult
- Animates:
  1. Cell exploration (blue)
  2. Final path (green)
- Uses CSS classes:
  - `cur`: Currently exploring
  - `highlight`: Visited cell
  - `path-style`: Final path

## Supporting Classes

### Cell.java
```java
public record Cell(int row, int col, int dist, Cell prev) {
}
```
- Java record: Immutable data class
- `row, col`: Position
- `dist`: Distance from start
- `prev`: Parent cell for path reconstruction

### PathFindResult.java
```java
public record PathFindResult(PathFindType type, List<int[]> moves, 
                           List<int[]> shortestPath, int[][] matrix) {
}
```
- Encapsulates algorithm results
- Used for frontend visualization


Would you like me to elaborate on any specific part of the implementation?