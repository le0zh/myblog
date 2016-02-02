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


#### 设置召唤间隔

保存脚本文件，并返回Unity。在Hierarchy中选中Road，在Inspector面板中，将Waves的Size属性设置为4，然后，使用Enemy Prefab分别设置所有的四个元素的Enemy属性，同时像下面这样设置
Spawn Interval和Max Enemies：
+ Element 0: Spawn Interval: 2.5, Max Enemies: 5
+ Element 1: Spawn Interval: 2, Max Enemies: 10
+ Element 2: Spawn Interval: 2, Max Enemies: 15
+ Element 3: Spawn Interval: 1, Max Enemies: 5

最终的设置应该像下面这张截图一样：
![img](http://cdn1.raywenderlich.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-01-at-18.46.16.png)

当然，你也可以自己随意增加或减少其中的一些设置哈。
运行游戏，不错。成队的虫子向我们的饼干进发了。。
![gif](http://cdn3.raywenderlich.com/wp-content/uploads/2015/07/bugs.gif)

####　可选的：增加不同的敌人类型
现在我们的塔防游戏只有一种类型的敌人，幸运的，在Prefabs文件夹下，有另外一种类型的敌人Enemy2。Speed为3，Tag 
选中Prefabs\Enemy2，并在Inspector面板中增加 MoveEnemy 脚本组件，设置Speed为3，设置Tag为Enemy。现在我们有了这种移动速度很快的虫子敌人啦。


### 更新玩家的生命值（慢慢的杀死我。。）

现在，尽管虫子大军可以向饼干移动，但是玩家丝毫不会受到伤害。不能这样下去啦，玩家应该在敌人到达饼干时，受到伤害。

![img](http://cdn2.raywenderlich.com/wp-content/uploads/2015/04/LivesStolen2.png)
打开GameManagerBehavior.cs脚本文件，增加下面的变量:
```
public Text healthLabel;
public GameObject[] healthIndicator;
```

我们将使用`healthLabel`控制玩家的生命值显示，使用`healthIndicator`来控制代表玩家生命值的5个小的绿色的饼干（这比简单的使用一个标签显示生命值更有趣）。

#### 管理生命值
接下来，在GameManagerBehavior中增加一个属性来维护玩家的生命值：
```
private int health;
public int Health
{
    get { return health; }
    set
    {
        //1 如果收到攻击，则摇晃一下摄像头（屏幕会跟着晃动，提醒玩家）
        if(value < health)
        {
            Camera.main.GetComponent<CameraShake>().Shake();
        }

        //2 更新生命值和显示的文本
        health = value;
        healthLabel.text = "HEALTH: " + health;

        //3 玩家挂了的逻辑
        if(health <= 0 && !gameOver)
        {
            gameOver = true;
            GameObject gameOverText = GameObject.FindGameObjectWithTag("GameOver");
            gameOverText.GetComponent<Animator>().SetBool("gameOver", true);
        }

        //4 更新代表生命值的饼干
        for(int i = 0;i<healthIndicator.Length; i++)
        {
            if(i< health)
            {
                healthIndicator[i].SetActive(true);
            }
            else
            {
                healthIndicator[i].SetActive(false);
            }
        }
    }
}
```
这个用来管理玩家的生命值，再一次的，关键的逻辑代码在setter方法中：
1. 如果要减少玩家的生命值，则使用CameraShake组件来创造一个晃动的效果。这个脚本（CameraShake）被包含在工程项目中，但不会在本教程中介绍。
2. 更新私有变量和生命值的文本标签。
3. 如果生命值将为了0并且游戏尚未结束，则设置gameOver为true，并触发GameOver动画。
4. 移除被敌人吃掉的饼干，这里简单的使用了SetActive(false)，当生命值恢复时，可以再使能它使饼干回来。

在`Start()`中初始化`Health`:
```
Health = 5;
```

这里在游戏开始时，玩家有5点的生命值。

有了这个属性，就可以在敌人抵达饼干的位置时，更新玩家的生命值。保存这个脚本，切换到MoveEnemy.cs。

#### 更新生命值

为了更新玩家的生命值，在`Update`方法中找到`// TODO: deduct health`的注释，并用下面的代码替换:
```
GameManagerBehavior gameManager = GameObject.Find("GameManager").GetComponent<GameManagerBehavior>();
gameManager.Health -= 1;
```
这里拿到`GameManagerBehavior`并将`Health`减1。
保存脚本文件，返回Unity。

在Hierarchy中选中GameManager并将Health Label设置为HealthLabel。

Hierarchy中，展开Cookie并拖拽它的五个子节点HealthIndicator，到GameManager的 Health Indicator数组。

开始游戏，并等到敌人到达饼干的位置，啥也不干，等到你被打死（。。。）

![gif](http://cdn3.raywenderlich.com/wp-content/uploads/2015/07/cookie-attack.gif)


### 怪兽之战，怪兽的复仇
是时候开战啦，让守卫饼干的怪兽和试图吃饼干的虫子开战。

还需要以下这些工作：
+ 一个生命值条，让玩家清楚的知道敌人孰强孰弱
+ 检测怪兽攻击范围内的虫子
+ 向哪个虫子开火
+ 许多的子弹

#### 敌人的生命值条

 我们将使用两张图片来实现生命值条，一个深色的背景和一个稍微浅绿色的条，根据敌人的生命值来调整浅绿色条的比例。

从Project Browser中拖拽Prefabs\Enemy到场景中。
然后拖拽Images\Objects\HealthBarBackground到Enemy上，使其成为Enemy的子节点。
在Inspector中, 设置HealthBarBackground的位置为(0, 1, 0)。

然后，在 Project Browser中选中Images\Objects\HealthBar，并确保他的Pivot被设置为Left，然后在Hierarchy中添加它为Enemy的子节点，并设置它的位置为(-0.63, 1, -1)，设置X Scale为125。

新建一个新的C#脚本，重命名为HealthBar，然后挂到HealthBar上，稍后，我们将调整生命条的长度。

在Hierarchy中选中Enemy，确保它的位置为(20, 0, 0)。

在Inspector面板的顶部，点击Apply按钮保存刚刚的修改。最后将Enemy从Hierarchy上删除。

![img](http://cdn1.raywenderlich.com/wp-content/uploads/2015/04/Screen-Shot-2015-04-07-at-11.37.12.png)

重复上面的步骤，给Prefabs\Enemy2加上生命值条。

#### 调整生命值条（血条）的长度
在代码编辑器中打开HealthBar.cs，添加下面的变量：
```
public float maxHealth = 100; //敌人的最大生命值
public float currentHealth = 100; //敌人的当前生命值
private float originalScale; //记录生命值条的初始大小
```
`maxHealth`记录敌人的最大生命值，`currentHealth`记录敌人当前的生命值，最后使用`originalScale`记录生命条的初始大小。

在`Start()`方法中，给`originalScale`一个初始值：
```
void Start ()
{
    originalScale = gameObject.transform.localScale.x;
}
```
我们保存了`localScale.x`的值。

在`Update()`方法中，通过下面的代码设置血条的大小：
```
void Update()
{
    Vector3 tmpScale = gameObject.transform.localScale;
    tmpScale.x = currentHealth / maxHealth * originalScale;
    gameObject.transform.localScale = tmpScale;
}
```

首先，我们把`localScale`拷贝到了一个临时变量，因为我们将紧紧修改它的x的值，然后根据敌人现在的生命值计算新的x的值，最后将前面的临时变量赋值回`localScale`。
保存脚本文件，在Unity中运行游戏，我们将看到敌人的血条了，像下面这样：
![img](http://cdn4.raywenderlich.com/wp-content/uploads/2015/04/Screen-Shot-2015-04-06-at-10.24.48.png)

在游戏运行时, 在Hierarchy面板中展开Enemy(Clone)对象，然后选中它的HealthBar子节点， 改变当前生命的数值，检查血条的变化：
![gif](http://cdn4.raywenderlich.com/wp-content/uploads/2015/06/AdjustHealthbar.gif)

#### 追踪攻击范围内的敌人
现在我们的怪兽需要知道瞄准那个敌人。在我们实现之前，需要对怪兽和敌人做一些前期工作。

在Project Browser中选中Prefabs\Monster，然后在Inspector面板中添加一个Circle Collider 2D组件。
设置刚刚添加的碰撞器的半径为2.5，这将是怪兽的攻击范围。

勾选上Is Trigger属性，这样的话其他对象就能穿过这片区域，而不会是直接撞上。
最后，在Inspector面板的顶部, 设置Monster的层为Ignore Raycast，在弹出来的对话框中，选择" Yes, change children"(将更改应用到子节点)，如果我们不忽略光线投射(raycast)，那么碰撞器将会响应点击事件，而这将带来怪兽会挡住下面的空位(openspot)的事件.
![img](http://cdn4.raywenderlich.com/wp-content/uploads/2015/06/Bildschirmfoto-2015-06-05-um-14.47.15.png)

为了能够在碰撞器区域范围内检测到敌人，也需要给敌人添加一个碰撞器和刚体(rigid body)，因为Unity只有当一个碰撞器被附加的刚体时，才回触发`trigger`事件。

在Project Browser中选中Prefabs\Enemy， 添加一个Rigid Body 2D组件，同时选中Is Kinematic，这样该物体不应该受到物理的影响。
再添加一个Circle Collider 2D组件，设置Radius为1， 重复上面的步骤修改Prefabs\Enemy 2。

现在触发器已经设置完毕了, 怪兽可以检测敌人是否进入攻击范围内了。
我们还需要一个东西: 当敌人被消灭时，一个脚本来通知我们怪兽停止攻击。
新建一个名为EnemyDestructionDelegate的C#脚本，并挂载到Enemy和Enemy2 prefab上。
打开EnemyDestructionDelegate.cs，增加下面的委托声明：
```
public delegate void EnemyDelegate(GameObject enemy);
public EnemyDelegate enemyDelegate;
```
这里我们创建了一个委托（一个方法的容器，可以被当做变量传递）。

增加下面的方法：
```
void OnDestroy()
{
    if (enemyDelegate != null)
    {
        enemyDelegate(gameObject);
    }
}
```
在销毁一个游戏对象时，Unity会自动调用上面的方法，在这个方法里检查委托是否为空。如果不为空，则用`gameObject`作为参数调用这个委托的方法。这样一来，所有的注册了的监听者都能知道敌人已经被销毁了。
保存脚本文件，返回Unity。

#### 怪兽们开火吧（Give Monsters a License to Kill）
现在怪兽们可以检测攻击范围内的敌人啦，新建一个名为ShootEnemies的C#脚本，挂载到Monster prefab上。

打开ShootEnemies.cs，添加下面的using语句来使用泛型：
```
using System.Collections.Generic;
```

添加一个变量来保存攻击范围内的敌人:
```
public List<GameObject> enemiesInRange;
```

我们将在`enemiesInRange`中保存所有在攻击范围内的敌人的引用.
在`Start()`方法中初始化这个变量：
```
enemiesInRange = new List<GameObject>(); //初始化为空的列表
```
在开始，没有任何敌人是在攻击范围内的，所以用空的列表初始化。
添加下面的代码来填充`enemiesInRange`列表：
```
//1 当敌人被销毁时，从enemiesInRange列表中移除
void onEnemyDestroy(GameObject enemy)
{
    enemiesInRange.Remove(enemy);
}

void OnTriggerEnter2D(Collider2D other)
{
    //2 如果碰撞检测的另一个对象是敌人，则加入到列表，同时注册enemyDelegate
    if (other.gameObject.tag.Equals("Enemy"))
    {
        enemiesInRange.Add(other.gameObject);
        EnemyDestructionDelegate del = other.gameObject.GetComponent<EnemyDestructionDelegate>();
        del.enemyDelegate += onEnemyDestroy;
    }
}

void OnTriggerExit2D(Collider2D other)
{
    //3 如果超出了范围，且另一个对象是敌人， 则移除列表，并撤销enemyDelegate的注册
    if (other.gameObject.tag.Equals("Enemy"))
    {
        enemiesInRange.Remove(other.gameObject);
        EnemyDestructionDelegate del = other.gameObject.GetComponent<EnemyDestructionDelegate>();
        del.enemyDelegate -= onEnemyDestroy;
    }
}
```
1. 在`OnEnemyDestroy`，我们将敌人从`enemiesInRange`移除，当敌人进入到怪物的触发区域内时，`OnTriggerEnter2D`会被调用。
2. 然后我们将敌人加入到`enemiesInRange`，同时注册`EnemyDestructionDelegate`，这确保了当敌人被销毁时，`OnEnemyDestroy`会被自动调用，我们当然不喜欢在已经挂了的敌人上浪费弹药。
3. 在`OnTriggerExit2D`我们将敌人从`enemiesInRange`移除，同时撤销监听`EnemyDestructionDelegate`。现在我们可以搞清楚哪些敌人在攻击范围内了。

保存脚本，在Unity中运行游戏，为了检测能否正常工作，放置一个怪兽，选中它，并在Inspector中观察` enemiesInRange`列表的变化！

#### 选一个目标
当多个敌人出现在攻击范围之内时，选中哪个作为目标呢？
当然，是攻击离饼干最近的那个。

打开MoveEnemy.cs，添加下面的方法：
```
public float distanceToGoal()
{
    float distance = 0;
    distance += Vector3.Distance(gameObject.transform.position, waypoints[currentWaypoint + 1].transform.position);

    for (var i = currentWaypoint + 1; i < waypoints.Length - 1; i++)
    {
        Vector3 startPosition = waypoints[i].transform.position;
        Vector3 endPosition = waypoints[i + 1].transform.position;
        distance += Vector3.Distance(startPosition, endPosition);
    }

    return distance;
}
```
上面的代码计算了敌人剩下路段的距离，通过使用`Distance`方法计算两个`Vector3`实例之间的距离。

我们稍后将使用这个方法来找到攻击的目标，但是现在我们的怪兽还是没有装备武器呢，先解决这个。

![img](http://cdn3.raywenderlich.com/wp-content/uploads/2012/07/yunofire.jpg)

保存脚本文件，返回Unity准备开始准备子弹。

#### 给怪兽分配子弹---很多的子弹！
在Project Browser中拖拽Images/Objects/Bullet1到场景中，设置z值为-2，x和y的值随意，因为我们每次在实例化子弹时都将设置。

新建一个名为BulletBehavior的脚本，添加下面的变量 ：
```
public float speed = 10;
public int damage;
public GameObject target;
public Vector3 startPosition;
public Vector3 targetPosition;

private float distance;
private float startTime;

private GameManagerBehavior gameManager;
```
`speed`定义了子弹飞行的速度，`damage`则是伤害值。
`target`, `startPosition`, 和`targetPosition`决定了子弹的方向。
`distance`和`startTime`记录子弹的当前位置，当玩家消灭一个敌人时，用`gameManager`来奖励玩家。

在`Start()`方法中初始化:
```
startTime = Time.time;
distance = Vector3.Distance(startPosition, targetPosition);
GameObject gm = GameObject.Find("GameManager");
gameManager = gm.GetComponent<GameManagerBehavior>();
```
将`startTime`设置为当前的时间，并计算开始位置和目标位置之间的距离，同时像之前一样拿到`GameManagerBehavior`的引用。
在`Update()`方法中，添加下面的代码来控制子弹的移动：
```
//1 
float timeInterval = Time.time - startTime;
gameObject.transform.position = Vector3.Lerp(startPosition, targetPosition, timeInterval*speed/distance);

//2
if (gameObject.transform.position.Equals(targetPosition))
{
    if (target != null)
    {
        //3
        Transform healthBarTransform = target.transform.FindChild("HealthBar");
        HealthBar healthBar = healthBarTransform.gameObject.GetComponent<HealthBar>();
        healthBar.currentHealth -= Mathf.Max(damage, 0);

        //4
        if (healthBar.currentHealth <= 0)
        {
            Destroy(target);
            AudioSource audioSource = target.GetComponent<AudioSource>();
            AudioSource.PlayClipAtPoint(audioSource.clip, transform.position);

            gameManager.Gold += 50;
        }
    }

    Destroy(gameObject);
}
```
1. 我们通过使用`Vector3.Lerp`来在开始和结束位置做插值来计算子弹新的子弹。
2. 如果子弹抵达了目标位置，则验证目标是否还存在。
3. 获取目标的HealthBar组件，并用子弹的伤害值减少相应的生命值。
4. 如果敌人的生命值将为了0，则销毁敌人，播放一个声音特效并给玩家奖励相应的金币。

保存脚本文件，并返回Unity。

#### 获得更大的子弹
如果我们的怪兽在升级后，能获取更大的子弹是不是很酷。当然是的哈。幸运的是，这也很好实现。

在Hierarchy中，拖拽Bullet1到Project标签页里面来创建一个子弹的prefab。
删除场景中的Bullet1对象，我们不再需要他啦。

复制Bullet1 prefab两次（快捷键Ctrl+ D）,分别命名为Bullet2和Bullet3。
选中Bullet2，在Inspector面板中，设置它的Sprite Renderer组件的Sprite属性为Images/Objects/Bullet2，这将使Bullet2看起来别Bullet1更大一些。
重复上面的步骤设置Bullet3的sprite为Images/Objects/Bullet3。
接下来，设置子弹的伤害值。

在Project标签页中选中Bullet1 prefab，在Inspector面板汇总，找到Bullet Behavior组件，并分别设置Bullet1的Damage为10，Bullet2的Damage为15，Bullet1的Damage为20，或者其他的你喜欢的任意值。

![img](http://cdn2.raywenderlich.com/wp-content/uploads/2015/06/Bildschirmfoto-2015-06-06-um-12.39.32.png)

#### 给子弹分级别
将不同的子弹分配给不同的怪兽级别，使高等级的怪兽更快的消灭敌人。
打开MonsterData.cs脚本，在类`MonsterLevel`增加下面的变量:
```
public GameObject bullet;
public float fireRate;
```
这将为每个怪兽级别设置子弹的prefab和发射频率，保存脚本并返回Unity去完成设置工作。
在Project Browser中选中Monster prefab，在Inspector中，展开Monster Data组件的Levels属性。设置每个元素的Fire Rate为1，然后分别为每个元素的Bullet属性设置为Bullet1, Bullet2 和Bullet3。

你的怪兽级别设置应该像下面这样：
![img](http://cdn4.raywenderlich.com/wp-content/uploads/2015/06/MonsterData-with-bullets.png)

#### 开火
打开ShootEnemies.cs，添加下面的变量:
```
private float lastShotTime;
private MonsterData monsterData;
```
这项变零记录怪兽最近一次开火的时间，同时包含怪兽子弹类型、发送频率的MonsterData。

As their names suggest, these variables keep track of when this monster last fired, as well the MonsterData structure that includes information about this monster’s bullet type, fire rate, etc.
Assign values to those fields in Start():
lastShotTime = Time.time;
monsterData = gameObject.GetComponentInChildren<MonsterData> ();
Here you set lastShotTime to the current time and get access to this object’s MonsterData component.
Add the following method to implement shooting:

>原文地址: http://www.raywenderlich.com/107529/unity-tower-defense-tutorial-part-2