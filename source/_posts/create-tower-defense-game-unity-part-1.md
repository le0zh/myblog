title: 使用unity创建塔防游戏(译)(part1)
tags:
  - unity
categories:
  - game
date: 2016-01-21 12:40:12
---

塔防游戏非常地受欢迎，木有什么能比看着自己的防御毁灭邪恶的入侵者更爽的事了。
在这个包含两部分的教程中，你将使用Unity创建一个塔防游戏。

你将会学到如何：
+ 创建一波一波的敌人
+ 使敌人随着路标移动
+ 创建和升级防御塔，并将敌人销毁

最后，你会有一个此类型游戏的框架，之后可在此基础之上进行扩展。
![img](http://cdn4.raywenderlich.com/wp-content/uploads/2015/03/Screen-Shot-2015-03-26-at-18.29.53.png)

<!-- more --> 

### 最终效果
在本篇教程中，你将创建一个塔防游戏，敌人（小虫子）会朝着你的饼干移动，你可以在一些战略点上，使用金币放置和升级你收下的小怪兽来进行防御。

玩家必须在小虫子抵达饼干之前消灭它们，敌人会随着波数的增加而变得更加强大。 游戏将在玩家在所有波数之后存活下来（游戏胜利），或者有5个敌人抵达饼干之后结束（游戏失败）。

下面是一张完成的游戏截图:

![img](http://cdn1.raywenderlich.com/wp-content/uploads/2015/03/Screen-Shot-2015-03-26-at-18.41.11.png)

### 准备开始
如果你还没有Unity5，请从[Unity的官网](http://unity3d.com/get-unity/)下载。

同时，下载这个[starter项目](http://7xl2vf.com1.z0.glb.clouddn.com/td/TowerDefense-Part1-Starter.zip)，解压缩并且使用unity打开`TowerDefense-Part1-Starter`这个工程。
starter项目中包括了美术和声音资源，同时还有预设的动画以及一些帮助用的脚本，这些脚本跟塔防游戏没有直接的关系，所以不会再本教程中详细介绍。但是如果你想要更多的学习关于unity 2d动画的创建，请参考[Unity 2D 教程](http://www.raywenderlich.com/66345/unity-2d-tutorial-animations)。

项目中同时包含了一些 `prefab`供你稍后扩展用来创建游戏角色。最后，工程中包含背景和UI设置好的场景。

在`Scenes`文件夹中找到并打开`GameScene`, 设置Game视图的显示比例为4:3来保证labels能够正确的在背景中对齐，你在Game视图中看到的应该如下所示:

![img](http://cdn5.raywenderlich.com/wp-content/uploads/2015/05/Screen-Shot-2015-05-31-at-09.48.38.png)

>注：一开始我也没弄清楚什么意思，后来理解了是在
![img](http://7xl2vf.com1.z0.glb.clouddn.com/game-view.png)
这里所示的窗口

Starter project – check!
Assets – check!
走向征服世界的第一步(你的塔防游戏...)已经准备好了!

### 可放置点的 X 标记

小怪兽只能放在标记有X的地方。
从`Project Brower`中拖动`Images\Objects\Openspot`到场景中，目前来说放到哪个位置没有关系。

在`Hierarchy`中选中`Openspot`，在`Inspector`面板点击 `Add Component`并且选择 `Physics 2D\Box Collider 2D`。 Unity将会在场景中以绿色的线显示盒子碰撞器。你将会使用这个碰撞器来检测在某个点的鼠标点击。
![img](http://cdn5.raywenderlich.com/wp-content/uploads/2015/06/first_open_spot.png)

使用相同的步骤，添加一个`Audio\Audio Source`组件到 `Openspot`上。并且设置`Audio Source’s AudioClip` 为 `tower_place`（可以在`Audio`文件夹中找到），记住不能勾选`Play On Awake`。

你需要再创建11个放置点，每个都得重复上面的步骤，不过不用担心，Unity有一个很好的解决办法: Prefab！

将`OpenSpot`从`Hierarchy`拖动到`Project Browser`里面的`Prefabs`文件夹，它的名字在`Hierarchy`将会变成蓝色，用来标示它是和一个prefab相关联的。像下面这样:
 ![gif](http://cdn3.raywenderlich.com/wp-content/uploads/2015/07/prefab.gif)

现在，你拥有了一个prefab，你就可以创建任意多的拷贝。简单的将`Openspot`从`Prefabs`文件夹中拖拽到场景中，再重复11次，一共在场景中创建12个放置点。
下面使用`Inspector`面板来分别设置这个12个放置点的坐标如下:
+ (-5.2, 3.5, 0)
+ (-2.2, 3.5, 0)
+ (0.8, 3.5, 0)
+ (3.8, 3.5, 0)
+ (-3.8, 0.4, 0)
+ (-0.8, 0.4, 0)
+ (2.2, 0.4, 0)
+ (5.2, 0.4, 0)
+ (-5.2, -3.0, 0)
+ (-2.2, -3.0, 0)
+ (0.8, -3.0, 0)
+ (3.8, -3.0, 0) 

结束后，你的场景应该像下面这样:
![img](http://cdn5.raywenderlich.com/wp-content/uploads/2015/05/Screen-Shot-2015-05-31-at-09.49.54.png)


### 放置小怪兽(防御塔)
为了使放置的工作简单点儿，工程的Prefab文件下中包含了一个`Monster`的prefab。
![img](http://cdn1.raywenderlich.com/wp-content/uploads/2015/06/Bildschirmfoto-2015-06-06-um-14.42.08.png)

现在，它包含了一个空的游戏对象，由三种不同的精灵组成，和他们各自的射击动画。

每一个精灵代表了小怪兽不同的能力级别。`Monster`的prefab同时包含了一个`Audio Source`的组件，当怪兽发射激光的时候会触发播放声音。

现在你将创建一个脚本来在`Openspot`上放置一个小怪兽。

在`Project Browser`中 选择`Openspot`，在`Inspector`面板中，点击 `Add Component`然后选择`New Script`并重命名为`PlaceMonster`，选择C#作为脚本语言并以此点击`Create`和`Add`。因为你是向prefab添加的脚本，所以场景中所有的`Openspot`都将会被附件该脚本。

双击刚才创建的脚本，在编辑器中打开。然后添加下面的这两个变量 
```
public GameObject monsterPrefab;
private GameObject monster;
```
你将使用`monsterPrefab`中的对象实例化一个拷贝来创建一个小怪兽，然后保存在`monster`变量中，方便之后的操作。

### 一个位置一个怪兽
添加下面的方法来限制一个位置只能放置一个怪兽:
```
private bool canPlaceMonster()
{
    return monster == null;
}
```

在`canPlaceMonster`方法中，我们检查`monster`变量是否为`null`，如果是的，则表示当前没有放置小怪兽，所以是允许放置一个的。

再添加下面的代码，来执行当玩家点击鼠标之后放置一个怪兽:
```
//1
void OnMouseUp()
{
    //2
    if (canPlaceMonster())
    {
        //3
        monster = (GameObject)Instantiate(monsterPrefab, transform.position, Quaternion.identity);
        //4
        AudioSource audioSource = gameObject.GetComponent<AudioSource>();
        audioSource.PlayOneShot(audioSource.clip);

        //todo: Deduct gold
    }
}
```
上面的代码在玩家点击或者Tap的时候放置一个怪兽，那么如何工作的呢？
1.  当玩家点击一个游戏对象的碰撞器的时候，Unity会自动的调用`OnMouseUp`方法
2 . 当被调用之后，这个方法会检查是否可以放置怪兽，如果可则放置一个新的怪兽
3.  使用 `Instantiate`方法创建一个怪兽对象，这个方法会根据指定的prefab和指定的位置和旋转角度创建一个对象。在本例中，我们拷贝了`monsterPrefab` ，并指定了当前游戏对象的位置，以及没有旋转，最后将结果转换为`GameObject`类型存储在`monster`变量中。
4.  最后，调用`PlayOneShot`方法播放附加在`AudioSource`组件上的声音特效。

现在我们的`PlaceMonster`脚本已经可以放置新的怪兽了，但还需要指定prefab这个步骤。

### 使用正确的Prefab
在代码编辑器中保存，并返回Unity 。

下面给`monsterPrefab `赋值，首先在Prefabs文件夹中选中`OpenSpot`，在`Inspector`面板中，点击`PlaceMonster (Script)`组件的`Monster Prefab`属性右边的小圆圈按钮，然后在弹出来的对话框中选择`Monster`。
![gif](http://cdn2.raywenderlich.com/wp-content/uploads/2015/07/assign-prefab.gif)

现在开始游戏，在 X 标记上点击来创建一些怪物~
![img](http://cdn5.raywenderlich.com/wp-content/uploads/2015/06/build-monsters-everywhere.png)


### 升级我们的怪物
在下面的图片中，我们看到怪物在不同的等级有不同的外观
![img](http://cdn5.raywenderlich.com/wp-content/uploads/2015/03/Screen-Shot-2015-03-27-at-17.19.50.png)

我们需要一个脚本来作为怪物升级系统实现的基础，来跟踪管理怪物在各个级别的能力大小，当然还有怪物所处的当前等级。

现在来添加这个脚本 。

在` Project Browser`中选中 `Prefabs/Monster`，添加一个新的C#脚本命名为`MonsterData`，在代码编辑器中打开该脚本并添加下面的代码在`MonsterData` 类的上面:
```
[System.Serializable]
public class MonsterLevel
{
    public int cost;
    public GameObject visualization;
}
```
这里定义了一个`MonsterLevel`类型，包含了费用（金币）以及对于某个特定等级的视觉效果。

我们添加了`[System.Serializable]`这个特性来使这个类的对象可以在`inspector`面板中编辑。这可以使我们方便快速的改变`MonsterLevel`中的值，甚至在游戏运行过程中。这在调节游戏平衡性的时候特别的有用。

### 定义怪物的等级
我们将预先定义的`MonsterLevel`存储在`List<T>`中。

为什么不简单的使用数组`MonsterLevel []`呢，首先我们会经常用到某个特定`MonsterLevel`对象的下标，当然如果使用数组编写一点代码来做这件事也不是特别困难。我们可以直接使用`List`对象的`IndexOf()`方法，没有必要重新发明轮子了这次 :]
![img](http://cdn5.raywenderlich.com/wp-content/uploads/2015/06/SpecialBike.jpg)

在MonsterData.cs文件的顶部，添加下面的引用：
```
using System.Collections.Generic;
```
这将允许我们使用泛型的数据结构类型，所以可以在脚本代码中使用`List<T>`。

 接下来添加下面的变量到`MonsterData`类中，用来存储`MonsterLevel`的列表:
```
public List<MonsterLevel> levels;
```
使用泛型，可以保证`levels`只能存放`MonsterLevel`类型的对象 。

在代码编辑器中保存文件，并返回到Unity中配置每个阶段.

在`Project Browser`中选中`Prefabs/Monster`，然后在`Inspector`面板中我们可以在`MonsterData (Script)`组件看到`Levels`属性，设置`size`为3
![img](http://cdn4.raywenderlich.com/wp-content/uploads/2015/07/Screen-Shot-2015-07-24-at-11.26.28-AM.png)

接下来，设置每个等级的花费如下：
+ Element 0: 200
+ Element 1: 110
+ Element 2: 120

接下来，给`visualization`赋值。

在project browers中展开`Prefabs/Monster`来查看其子节点。拖拽子节点`Monster0`到`visualization`属性的`Element 0`。
重复上面的动作`Monster1`对`Element 1`，`Monster2`对`Element 2`，参考下面动图中的演示：

![gif](http://cdn1.raywenderlich.com/wp-content/uploads/2015/07/assign-monsters2.gif)

现在，当你选中了`Prefabs/Monster`，它应该看下像下面这样子：
![img](http://cdn2.raywenderlich.com/wp-content/uploads/2015/06/MonsterData1.png)

### 定义当前的等级
 切换回MonsterData.cs中，向`MonsterData`类中添加另外一个变量：
```
private MonsterLevel currentLevel;
```
在私有变量`currentLevel`中，我们存放怪物当前的等级信息。
现在设置`currentLevel`同时将它暴露给其他脚本使用，添加下面的代码到`MonsterData`中：
```
//1
public MonsterLevel CurrentLevel
{
    //2
    get
    {
        return currentLevel;
    }

    //3
    set
    {
        currentLevel = value;

        int currentLevelIndex = levels.IndexOf(currentLevel);

        GameObject levelVisualization = levels[currentLevelIndex].visualization;
        for(var i = 0; i< levels.Count; i++)
        {
            if(levelVisualization != null)
            {
                if(i == currentLevelIndex)
                {
                    levels[i].visualization.SetActive(true);
                }
                else
                {
                    levels[i].visualization.SetActive(false);
                }
            }
        }
    }
}
```
看起来有很多C#代码，我们慢慢来看：
1. 为私有变量`currentLevel`定义一个属性。当属性被定义之后，你就能像其他变量一样去调用，既可以`CurrentLevel`这样在类的内部调用，也可以使用`monster.CurrentLevel`这样在类的外部调用。然后还能自定义属性的getter和setter方法，通过只提供一个getter，只有一个setter或者都有，来控制一个属性是只读的、只写的或者读写的。
2. 在getter方法中，直接返回私有变量`currentLevel`的值
3. 在setter方法中，给`currentLevel`赋值。先拿到当前等级的下标，然后遍历`levels`数组依据`currentLevelIndex`的值设置外观的活动或者非活动状态，好处就在于不管什么时候谁设置了`currentLevel`，精灵会自动更新。

添加下面的`OnEnable `的一个实现：
```
void OnEnable()
{
    CurrentLevel = levels[0];
}
```
这里设置了`CurrentLevel`的默认值，确保它只显示正确的那个精灵。

#### 注意
在`OnEnable`中而不是`OnStart `中初始化属性的值，是因为当prefabs被实例化时方法的调用次序问题。
`OnEnable `会在创建prefab时立即被调用，而`OnStart`会等到对象作为场景的一部反的时候才会被调用。
这里我们需要在放置一个怪兽之前就要初始化一下，所以在`OnEnable `中初始化。

>注意OnEnable中的大小写，如果大小写不对，方法不会被调用！

保存文件返回到Unity中，运行项目并且放置一些怪兽，现在他们显示正确的，也就是最低等级的精灵，如下图：
![img](http://cdn3.raywenderlich.com/wp-content/uploads/2015/06/No-more-mushyness.png)

### 升级小怪兽
返回到代码编辑器，增加下面的方法到`MonsterData`中：
```
public MonsterLevel getNextLevel()
{
    int currentLevelIndex = levels.IndexOf(currentLevel);
    int maxLevelIndex = levels.Count - 1;
    if (currentLevelIndex < maxLevelIndex)
    {
        return levels[currentLevelIndex+1];
    }
    else
    {
        return null;
    }
}
```
在`getNextLevel`方法中，我们首先拿到当前等级的下标以及最高等级的下标，以此判断如果当前不是最高等级时，返回下个等级，否则返回null。

我们可以使用该方法判断是否可以升级到下一个等级。

添加下面的方法增加怪兽的等级：
```
public void increaseLevel()
{
    int currentLevelIndex = levels.IndexOf(currentLevel);
    if (currentLevelIndex < levels.Count - 1)
    {
        CurrentLevel = levels[currentLevelIndex+1];
    }
}
```
这里我们获取当前等级的下标，然后确保它不会大于最高等级的下标，如果不大于，则将`CurrentLevel`设置为下一个等级。

### 测试是否可以升级
保存刚才的脚本文件，返回到PlaceMonster.cs文件，增加下面的方法:
```
private bool canUpdateMonster()
{
    if (monster != null)
    {
        MonsterData monsterData = monster.GetComponent<MonsterData>();
        MonsterLevel nextLevel = monsterData.getNextLevel();
        if (nextLevel != null)
        {
            return true;
        }
    }
    return false;
}
```
首先，检查`monster`变量是否为`null`，如果为`null`则肯定不能升级了，如果不为`null`则获取其`MonsterData`组件，并检查更高的等级是否存在，如果`getNextLevel `方法返回的值不是`null`则说明更高的等级存在（返回`true`），否则不存在（返回`false`）。


### 使可以通过金币升级

为了能够升级，在`PlaceMonster`中的`OnMouseUp`中添加`else if`分支:
```
void OnMouseUp()
{
    if (canPlaceMonster())
    {
        //...这里跟原来的代码一样，这里省略了
    }else if (canUpdateMonster())
    {
        monster.GetComponent<MonsterData>().increaseLevel();
        AudioSource audioSource = gameObject.GetComponent<AudioSource>();
        audioSource.PlayOneShot(audioSource.clip);
    }
}
```
首先我们通过`canUpdateMonster`检查能否升级，如果可以升级，则通过调用`MonsterData`组件的`increaseLevel`方法升级怪物，最后播放一次声音特效。

保存文件并返回到Unity，运行游戏，试试放置和升级怪物~
![img](http://cdn2.raywenderlich.com/wp-content/uploads/2015/06/Upgrade-monsters.png)

### 支付金币 - Game Manager
目前而言，可以立即的建造和升级任意数目的怪兽，但这样一来，就木有挑战了。

我们接下来就处理金币的问题，为例维护金币的信息，你不得不要在不同的游戏对象之间共享数据信息。
下面的图片显示了所有需要金币信息的游戏对象：
![img](http://cdn4.raywenderlich.com/wp-content/uploads/2015/06/sharedData.png)

我们将使用一个其他对象都能访问的共享对象来存储这类数据。

在Hierarchy面板中，右键选择`Create Empty`，并命名为`GameManager`。
添加一个C#脚本组件`GameManagerBehavior`到刚刚创建的`GameManager`上，然后打开并编辑这个脚本。
因为我们需要使用一个`label`显示玩家所拥有的金币数，所以在文件的顶部增加下面的引用:
```
using UnityEngine.UI;
```
这将允许我们使用UI相关的类型，比如`Text`用做显示用的label，接下来在类中添加下面的变量：
```
public Text goldLabel;
```
这个变量存储了对`Text`组件的引用，将用于显示玩家所拥有的所有金币数。

现在`GameManager`已经可以操作label了，但是我们如何保证变量中存储的金币数和label显示的数量同步呢，我们创建一个属性：
```
private int gold;

public int Gold
{
    get { return gold; }
    set
    {
        gold = value;
        goldLabel.GetComponent<Text>().text = "GOLD: " + gold;
    }
}
```

是不是看起来很熟悉，这个跟我们在`Monster`中定义的`CurrentLevel`比较相像，首先我们创建一个私有的变量`gold`用来存储当前所有的金币数，然后定义一个名为`Gold`的属性，并提供getter和setter方法。

在getter方法中简单的直接返回`gold`，setter方法则比较有趣了，除了设置`gold`的值，还设置了`goldLabel `的显示。
这样就保持了金币数和显示的同步。

在`Start()`中增加下面的初始化语句，默认给玩家100金币(当然你可以给更少的)
```
Gold = 100;
```

### 给脚本中的Label对象赋值
保持脚本文件并返回到Unity中。

在Hierarcy中，选中GameManager，在Inspector面板，点击`GoldLabel`右边的圆圈按钮，在Select Text对话框中，选中Scene标签页，并选中`GoldLabel` 
![img](http://cdn2.raywenderlich.com/wp-content/uploads/2015/06/Assign-goldLabel.png)

运行游戏，可以看到金币的显示如下:
![img](http://cdn5.raywenderlich.com/wp-content/uploads/2015/06/1000-gold.png)


### 检查玩家的“钱包"
打开PlaceMonster.cs文件，增加下面这行代码:
```
private GameManagerBehavior gameManager;
```
我们将通过变量`gameManager`来访问场景中GameManager的`GameManagerBehavior`组件，并在`Start`方法中初始化:
```
void Start ()
{
    gameManager = GameObject.Find("GameManager").GetComponent<GameManagerBehavior>();
}
```
我们使用`GameObject.Find`方法，找到名为GameManager的游戏对象，然后定位到其`GameManagerBehavior`组件并存到一个私有变量中，供稍后使用。

### 收钱！
我们还没有减少金币数，所以在`OnMouseUp`方法里面，将原来的TODO注释修改为下面的代码:
```
//todo: Deduct gold
```
```
gameManager.Gold -= monster.GetComponent<MonsterData>().CurrentLevel.cost;
```
注意是两个地方，在放置和升级的逻辑里各有一处。

保存文件并返回unity，升级一些怪物并注意观察金币数目的变化。现在我们能够减少金币数了，但是。。。玩家可以一直放置怪物（只要有空位），金币数甚至可以变成负数。
![img](http://cdn3.raywenderlich.com/wp-content/uploads/2015/06/Bildschirmfoto-2015-06-06-um-16.45.32.png)

这显然是不能被允许的，只有当玩家有足够的金币时，才能放置或者升级我们的怪物。

### 
切换到PlaceMonster.cs脚本，更新`canPlaceMonster`和`canUpdateMonster`方法如下，就是加上检查剩余金币是否足够的条件。

```
private bool canPlaceMonster()
{
    int cost = monsterPrefab.GetComponent<MonsterData>().levels[0].cost;
    return monster == null && gameManager.Gold >= cost; //确保金币足够
}

private bool canUpdateMonster()
{
    if (monster != null)
    {
        MonsterData monsterData = monster.GetComponent<MonsterData>();
        MonsterLevel nextLevel = monsterData.getNextLevel();
        if (nextLevel != null)
        {
            return gameManager.Gold >= nextLevel.cost; //确保金币足够
        }
    }
    return false;
}
```
保存，并运行游戏，试试还能不能无限添加怪物。
![img](http://cdn5.raywenderlich.com/wp-content/uploads/2015/06/Monsters-cost-gold.png)

### 敌人、波数和路标
是时候给敌人“铺路”了。敌人首先在第一个路标的地方出现，然后向下一个路标移动并重复这个动作，知道他们抵达你的饼干。
我们将通过下面的手段使敌人行军起来：
1.  定义敌人移动的路线
2.  使敌人沿着路线移动
3.  旋转敌人，使他们看起来是向前方行进

### 通过路标建立路线
在Hierarcy中右键，选择Create Empty创建一个新的空的游戏对象，命名为Road，并确保其位置坐标为(0, 0, 0)
接下来，右键点击Road并创建另一个空的游戏对象，命名为Waypoint0 并将其坐标设置为（-12， 2， 0），这将是敌人开始进攻的起始点。
![img](http://cdn2.raywenderlich.com/wp-content/uploads/2015/06/Road-waypoint-hierarchy.png)

创建另外5个路标：
+ Waypoint1: (7, 2, 0)
+ Waypoint2: (7, -1, 0)
+ Waypoint3: (-7, 3, 0)
+ Waypoint4: (-7.3, -4.5, 0)
+ Waypoint5: (7, -4.5, 0)
下面的截图标示出了路标的位置以及最终的路线：
![img](http://cdn1.raywenderlich.com/wp-content/uploads/2015/07/Screen-Shot-2015-07-24-at-12.09.11-PM1.png)

### 召唤敌人

现在是时候去创建一些敌人来沿着上面的路线移动了。在Prefabs的文件夹中包含了一个Enemy的prefab。 它的位置坐标是（-20,0,0） ，所以新创建的敌人对象一开始在平面外面。

跟Monster的prefab一样，Enemy的prefab同样包含了一个AudioSource，一个精灵图片（一会儿可以旋转其方向）。
![img](http://cdn2.raywenderlich.com/wp-content/uploads/2015/06/Bug1-1.png)

### 使敌人沿着路线移动

向Prefabs\Enemy新建一个名为MoveEnemy的C#脚本，使用代码编辑器打开，并添加下面的变量定义:
```
[HideInInspector]
public GameObject[] waypoints;
private int currentWaypoint = 0;
private float lastWaypointSwitchTime;
public float speed = 1.0f;
```
`waypoints`以数组的形式存储了所有的路标，它上面的`HideInInspector`特性确保了我们不会在inspector面板中不小心修改了它的值，但是我们仍然可以在其他脚本中访问。
`currentWaypoint `记录了敌人当前所在的路标，`lastWaypointSwitchTime`记录了当敌人经过路标时用的时间，最后使用`speed`存储敌人的移动速度。

增加下面这行代码到`Start`方法中：
```
lastWaypointSwitchTime = Time.time;
```
这里将`lastWaypointSwitchTime `初始化为当前时间。

为了使敌人能沿路线移动，在`Update`方法中添加下面的代码:
```
// 1 
Vector3 startPosition = waypoints[currentWaypoint].transform.position;
Vector3 endPosition = waypoints[currentWaypoint + 1].transform.position;

// 2 
float pathLength = Vector3.Distance(startPosition, endPosition);
float totalTimeForPath = pathLength / speed;
float currentTimeOnPath = Time.time - lastWaypointSwitchTime;
gameObject.transform.position = Vector3.Lerp(startPosition, endPosition, currentTimeOnPath / totalTimeForPath);

// 3 
if (gameObject.transform.position.Equals(endPosition))
{
    if (currentWaypoint < waypoints.Length - 2)
    {
        // 3.a 
        currentWaypoint++;
        lastWaypointSwitchTime = Time.time;
        // TODO: Rotate into move direction
    }
    else
    {
        // 3.b 
        Destroy(gameObject);

        AudioSource audioSource = gameObject.GetComponent<AudioSource>();
        AudioSource.PlayClipAtPoint(audioSource.clip, transform.position);
        // TODO: deduct health
    }
}
```
让我们一步一步来看:
1. 从路标数组中，取出当前路段的开始路标和结束路标。
2. 计算出通过整个路段所需要的时间（使用 距离除以速度 的公式），使用`Vector3.Lerp`插值计算出当前时刻应该在的位置。
3. 检查敌人是否已经抵达结束路标，如果是，则有两种可能的场景:
    A. 敌人尚未抵达最终的路标，所以增加`currentWayPoint`并更新`lastWaypointSwitchTime`，稍后我们要增加旋转敌人的代码使他们朝向前进的方向。
    B. 敌人抵达了最终的路标，就销毁敌人对象，并触发声音特效，稍后我们要增加减少玩家生命值的代码。

保存文件，并返回到Unity。

### 给敌人指明方向
现在，敌人还不知道路标的次序。

在Hierarchy中选中Road，然后添加一个新的C#脚本命名为SpawnEnemy，并在代码编辑器中打开，增加下面的变量:
```
public GameObject[] waypoints;
```
我们将使用`waypoints`来存放路标的引用。
保存文件返回Unity， 在Hierarchy中选中Road，将`WayPoints`数组的大小改为6，拖拽Road的孩子节点到想用的Element位置，Waypoint0对应Element0以此类推。
![gif](http://cdn2.raywenderlich.com/wp-content/uploads/2015/07/waypoints2.gif)

现在我们已经有了路线的路标数组，注意到敌人不会退缩。。。
![img](http://cdn5.raywenderlich.com/wp-content/uploads/2012/07/certaindeath.jpg)

### 检查一切顺利

打开SpawnEnemy脚本，增加下面的变量：
```
public GameObject testEnemyPrefab;
```
这里使用了`testEnemyPrefab`保存对`Enemy`prefab的引用。
使用下面的代码，当脚本开始时添加一个敌人：
```
void Start () {
    Instantiate(testEnemyPrefab).GetComponent<MoveEnemy>().waypoints = waypoints;
}
```
上面的代码使用testEnemyPrefab实例化一个敌人对象，并将路标赋值给它。

保存文件返回Unity，在Hierarchy中选中Road并将Enemy prefab赋值给testEnemyPrefab。
 运行游戏，可以看到敌人已经能够沿着路线移动了:
![gif](http://cdn1.raywenderlich.com/wp-content/uploads/2015/06/BugFollowsRoadWithoutRotating.gif)

nice,但是有没有注意到敌人移动的时候没有朝向移动的方向。。没有关系，这个问题将在part2部分修复。


>原文地址：http://www.raywenderlich.com/107525/create-tower-defense-game-unity-part-1
