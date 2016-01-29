[译]使用unity创建塔防游戏_part2.md

欢迎来到使用unity创建塔防游戏的第二部分，我们将继续使用unity创建一个塔防游戏，在[第一部分](http://le0zh.github.io/2016/01/21/create-tower-defense-game-unity-part-1/)中，我们已经可以放置和升级小怪兽了。同时创建了一个向饼干进攻的敌人。

然而，这时候敌人移动时还不会转向，同时不会攻击。这部分教程中，我们将增加一波敌人，同时武装我们的小怪兽使他们能保卫我们珍贵的饼干。

![img](http://cdn4.raywenderlich.com/wp-content/uploads/2015/03/Screen-Shot-2015-03-26-at-18.29.53.png)

### 开始
在Unity中，打开在第一部分中完成的项目，如果你是从这篇教程才开始的，可以直接下载并打开[TowerDefence-Part2-Starter](http://7xl2vf.com1.z0.glb.clouddn.com/td/TowerDefense-Part2-Starter.7z)项目。

从scene文件夹中，打开GameScene。

### 旋转敌人
在第一部分教程的最后，敌人可以沿着路线移动，但是不能面向前进的方向。在代码编辑器中打开MoveEnemy.cs，添加下面的方法来修正这个问题:
```
private void RotateIntoMoveDirection()
{
    //1
    Vector3 newStartPosition = waypoints[currentWaypoint].transform.position;
    Vector3 newEndPosition = waypoints[currentWaypoint + 1].transform.position;
    Vector3 newDirection = (newEndPosition - newStartPosition);

    //2
    float x = newDirection.x;
    float y = newDirection.y;
    float rotationAngle = Mathf.Atan2(y, x)*180/Mathf.PI;

    //3
    GameObject sprite = (GameObject) gameObject.transform.FindChild("Sprite").gameObject;
    sprite.transform.rotation = Quaternion.AngleAxis(rotationAngle, Vector3.forward);
}
```
`RotateIntoMoveDirection`这个方法将会旋转敌人使其永远朝向前进的方向:
1. 通过用当前的路标位置减去下一个路标位置，得到当前敌人的移动方向。
2. 使用`Mathf.Atan2`方法算出新的移动发祥的角度(单位是弧度)，然后将结果转换为角度(使用 `180/Mathf.PI`转换)。
3. 最后，得到名为Sprite的子节点，并沿着z轴旋转`rotationAngle`，请注意，我们只是旋转了Sprite子节点，而不是整个敌人，这样我们稍后会添加的生命值条将会保持水平方向不变。

![img](http://cdn2.raywenderlich.com/wp-content/uploads/2015/04/calculate-angles.png)

在`Update()`方法中，替换原来的注释`// TODO: Rotate into move direction`为下面这行代码：
```
RotateIntoMoveDirection();
```

保存脚本文件，并返回到Unity中，运行游戏，现在敌人知道面向前进的方向了:
![gif](http://cdn5.raywenderlich.com/wp-content/uploads/2015/06/BugFollowsRoad.gif)

只有一个敌人？没什么意思，来一群敌人吧！同时像多数塔防游戏一样，成群的敌人一波一波的来袭！

### 通知玩家
在成群的敌人进攻之前，我们要通知玩家即将到来的攻击。同时，为什么不在屏幕的上方显示当前的波数呢。

不止一个的游戏对象需要知道波数的信息，所以我们将它加到GameManamger的GameManagerBehavior组件。

在代码编辑器中打开GameManagerBehavior.cs并添加下面这两个变量：
```
public Text waveLabel;
public GameObject[] nextWaveLabels;
```

其中`waveLabel`是在屏幕右上角显示当前波数的对象引用，`nextWaveLabels`中有连个游戏对象，组合来制作当新的一波敌人开始时的的动画，就像下面这样:
![gif](http://cdn1.raywenderlich.com/wp-content/uploads/2015/06/nextWaveAnimation.gif)

保存脚本文件，返回Unity，在Hierarcy面板选中GameManager，点击WaveLabel右侧的小圆形按钮，在 Select Text对话框中选中Scene标签页下的WaveLabel。
![img](http://cdn1.raywenderlich.com/wp-content/uploads/2015/06/gameManager-with-waves.png)

如果玩家游戏失败了，他不应该看到下一波的信息，为了解决这个问题，回到GameManagerBehavior.cs脚本并添加下面的另一个变量:
```
public bool gameOver = false;
```

我们将使用`gameOver`来记录玩家是否已经游戏失败。

同样，我们将使用属性来保持游戏中的元素在当前回合中的一致性，在GameManagerBehavior中添加下面的代码:
```
private int wave;

public int Wave
{
    get { return wave; }
    set
    {
        wave = value;
        if (!gameOver)
        {
            for (int i = 0; i < nextWaveLabels.Length; i++)
            {
                nextWaveLabels[i].GetComponent<Animator>().SetTrigger("nextWave");
            }
        }
        waveLabel.text = "WAVE: " + (wave + 1);
    }
}
```

上面创建私有变量、属性、getter方法都是常规的方法，但是再一次的，setter方法有些特殊：
+ 我们使用新的`value`更新`wave`
+ 然后检查游戏是已经结束，如果还没有结束，则遍历`nextWaveLabels`（包含Animatro组件的label集合），触发nextWave的动画
+ 最后，设置waveLabel的显示文本为`wave + 1`，为什么要加1，因为普通玩家不会从0开始数，你懂得。

在`Start()`方法中，设置属性的值:
```
Wave = 0;
```

我们从0开始计数`Wave`。
保存脚本文件，在Unity中运行游戏，现在波数应该能正常的显示为从1开始:
![img](http://cdn5.raywenderlich.com/wp-content/uploads/2015/06/counting-waves.png)


### 让敌人来的更猛烈些
目前，我们还不能创建更多的敌人，同时，我们不能再当前波的敌人被消灭完之前，召唤出下一波的敌人，至少现在不能。

所以，游戏必须要能区分出当前游戏场景中是否存在敌人，这里Tags是个非常好的用来区分游戏对象的方法。

#### 给敌人打上标签(Tags)
在Project Browser中选中Enemy prefab，在Inspector面板的顶部，点击Tag下拉框并选中添加Tag（Add Tag）
![img](http://cdn3.raywenderlich.com/wp-content/uploads/2015/06/Create-Tag.png)

创建一个名为Enemy的Tag
![img](http://cdn1.raywenderlich.com/wp-content/uploads/2015/06/Bildschirmfoto-2015-06-06-um-03.55.00.png)

选中Enemy prefab，在Inspector面板中设置其Tag为新创建的Enemy。

#### 定义Waves
现在我们需要定义一波的敌人了，在代码编辑器中打开SpawnEnemy.cs，在类`SpawnEnemy`之前增加下面的定义 ：
```
[System.Serializable]
public class Wave
{
    public GameObject enemyPrefab;
    public float spawnInterval = 2;
    public int maxEnemies = 20;
}
```
`Wave`中有一个`enemyPrefab`，是用来实例化敌人的基础，`spawnInterval`用来指定创建敌人的间隔，`maxEnemies`指定一波敌人的最大数量。

上面这个类是可序列化的，意味着我们可以在Inspector中改变它的值。
向`SpawnEnemy`类中添加下面的变量 ：
```
public Wave[] waves;
public int timeBetweenWaves = 5;

private GameManagerBehavior gameManager;

private float lastSpawnTime;
private int enemiesSpawned = 0;
```

这里设置了一些用来召唤敌人的变量，跟我们用来使敌人沿着路标移动的那些很相似。我们将在`waves`中定义游戏中的各个回合的敌人信息，使用`enemiesSpawned`和`lastSpawnTime`记录已经召唤出来的敌人个数和召唤的时间。

玩家需要再杀死一波的敌人之后休息一下，所以设置`timeBetweenWaves`为5秒。

在`Start()`方法中增加下面的内容:
```
lastSpawnTime = Time.time;
gameManager = GameObject.Find("GameManager").GetComponent<GameManagerBehavior>();
```
这里我们设置了`lastSpawnTime`为当前时间，也就是当场景加载完毕脚本开始的时间。
然后我们用了之前的方式拿到GameManagerBehavior的引用。

在`Update()`方法中添加下面的代码：
```
//1 拿到当前波数的索引，并检查是否是最后的回合（已经没有敌人了）
int currentWave = gameManager.Wave;
if (currentWave < waves.Length)
{
    //2 如果不是最后一波，根据当前的时间计算距离上次召唤敌人的间隔
    float timeInterval = Time.time - lastSpawnTime;
    float spawnInterval = waves[currentWave].spawnInterval;
    if (((enemiesSpawned == 0 && timeInterval > timeBetweenWaves) || timeInterval > spawnInterval) &&
        enemiesSpawned < waves[currentWave].maxEnemies)
    {
        //3 如果当前场景中没有一个敌人，并且间隔时间大于波数的间隔时间，或者已经存在敌人并且间隔时间大于了召唤的间隔
        // 在上面条件的基础上并且当前的敌人数还没有达到当前波数设定的最多敌人数，则可以进行召唤
        lastSpawnTime = Time.time;
        GameObject newEnemy = (GameObject) Instantiate(waves[currentWave].enemyPrefab);
        newEnemy.GetComponent<MoveEnemy>().waypoints = waypoints;
        enemiesSpawned++;
    }

    //4 如果已经召唤出了当前波的所有敌人，则进入下一波的回合
    if (enemiesSpawned == waves[currentWave].maxEnemies &&
        GameObject.FindGameObjectWithTag("Enemy") == null)
    {
        gameManager.Wave++;
        gameManager.Gold = Mathf.RoundToInt(gameManager.Gold*1.1f);
        enemiesSpawned = 0;
        lastSpawnTime = Time.time;
    }
}
//5 如果所有的回合都通过了，则显示游戏胜利的动画
else
{
    gameManager.gameOver = true;
    GameObject gameOverText = GameObject.FindGameObjectWithTag("GameWon");
    gameOverText.GetComponent<Animator>().SetBool("gameOver", true);
}
```
让我们逐步来看:
1. 拿到当前波数的索引，并检查是否已经是最后回合（已经没有敌人了）；
2. 如果还有敌人，计算自从上次召唤敌人到现在的时间间隔，来检查当前时间是否可以召唤敌人，这里有两种情况，首先是即将召唤的敌人是当前回合的第一个敌人，这种情况下检查时间间隔是否大于`timeBetweenWaves`(回合的间隔时间), 否则，检查时间间隔是否大于`spawnInterval`(召唤的时间间隔)。无论上面哪种情况，我们要确保还没有召唤出当前回合的所有敌人；
3. 如果需要，通过实例化一个`enemyPrefab`召唤出一个敌人，同时增加`enemiesSpawned`计数；
4. 检查当前场景中的敌人数，如果敌人已经被干掉完了，则进入下一回合，同时奖励玩家10%的金币；
5. 如果所有的回合都通过了，则显示游戏胜利的动画。





>原文地址: http://www.raywenderlich.com/107529/unity-tower-defense-tutorial-part-2