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



>原文地址: http://www.raywenderlich.com/107529/unity-tower-defense-tutorial-part-2