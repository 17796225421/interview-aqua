## 雷的生成

雷的生成要保证随机，游戏体验才会好。游戏刚开始的阶段，也可以快速标记雷的位置。

### 替换法

替换法也就是把一个方块随机替换为雷，知道所有雷都放置完成。但是实际玩耍起来，体验不是很好，根本看不出哪个是雷，全靠盲猜。这样肯定要怀疑这个算法（特指下面实现方法）的随机性的，也不知道是不是`Random`的锅。

```java
  private void genRandomMines(int mineSize) {
    Random random = new Random(System.currentTimeMillis());
    while (mines.size() != mineSize) {
      // [0, 63]
      int nextMine = random.nextInt(size * size);
      Point point = oneD2point(nextMine);
      if (!mines.contains(point)) {
        board[point.x][point.y] = true;
        mines.add(point);
      }
    }
  }
```

### 洗牌法

然后一年过去，在知乎上偶然看到关于雷生成的算法，里面提到了洗牌法（**shuffle**）。使用的是 **Knuth shuffle** ，简单的说就是遍历数组，随机交换两个元素：

```
for i from 0 to n−2 do
     j ← random integer such that i ≤ j < n
     exchange a[i] and a[j]
```

洗牌法分为两步：

- 首先将雷按序填入游戏地图
- 再使用 Knuth shuffle 打乱顺序

为了更好的游戏体验，下面算法可以实现第一次点击时，不会碰到雷。不过这也意味着雷的生成要推迟到第一次点击时。

```java
   // ensure excludePoint is not mine
	private void genRandomMinesEnsured(int mineSize, Point excludePoint) {
    // fill mines
    int minesCnt = 0;
    for (int i = 0; i < size; i++) {
      for (int j = 0; j < size; j++) {
        if (i == excludePoint.x && j == excludePoint.y) continue;
        board[i][j] = true;
        minesCnt++;
        if (minesCnt == mineSize) break;
      }
      if (minesCnt == mineSize) break;
    }
    // shuffle array
    Random random = new Random(System.currentTimeMillis());
    for (int i = 0; i < size; i++) {
      for (int j = 0; j < size; j++) {
        int randomPoint = random.nextInt(size * size);
        Point point = oneD2point(randomPoint);
        if (i == excludePoint.x && j == excludePoint.y || point.equals(excludePoint)) continue;
        boolean temp = board[point.x][point.y];
        board[point.x][point.y] = board[i][j];
        board[i][j] = temp;
      }
    }
  }
```