title: Unity中技能冷却效果
tags:
  - unity
categories:
  - game
date: 2016-04-13 12:31:12
---

#### 目标
1. 游戏中常见的技能冷却效果
2. 简单的UI布局
![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/unity/skillcooldown/1bcb894f-bd6a-4c70-bb47-2ae253106a4e.jpg)

<!-- more -->

### 最终效果
![gif](http://7xl2vf.com1.z0.glb.clouddn.com/blog/unity/skillcooldown/finall.gif)

### 步骤
+ 打开Unity新建一个工程，这里我们用2D的项目类型就可以。

+ 设置游戏界面比例为16:9，这个是当手机屏幕比较流行的横框比。
![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/unity/skillcooldown/8b0ea6d2-0485-4a36-8f00-5fd7c585301a.png)

+ 给我们的技能一个容器，选择新建->UI->Panel，将默认生成的Canvas的设置修改如下:
![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/unity/skillcooldown/0a8a7041-073c-4d16-a207-ed4ac1643ad5.png)

+ 由于我们希望技能是栅格布局，所以给Panel增加一个GridLayoutGroup组件
![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/unity/skillcooldown/0f3134df-25ff-4c4c-acf3-e13308763c66.png)

+ 技能栏放在屏幕下方中间，所以设置Panel的锚点，依次点击下面的按钮设置，在选择锚点时，同时按下shift和alt键。
![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/unity/skillcooldown/82e3aad4-4161-4262-93e0-5d53c59143f5.png)

+ 修改pane的高度为120，宽度为 230

+ 将下载好的技能icon图片导入到工程中，注意要检查图片的Texture Type为 Sprite，如果不是，改为Sprite
![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/unity/skillcooldown/dd116ca6-380c-45ff-9e53-168d40fddf7c.png)

+ 向Panel中添加两个Button，删除默认的Text子节点，设置其Source Image，分别重名为SkillDot和SkillFlash，并修改其Color为灰色（就是冷却时想显示的颜色）完成后如下：
![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/unity/skillcooldown/169318cb-4216-48e5-b0ff-200c1d9d6d07.png)

+ 两个技能在panel的排列好像不太好看。。。如果每错的话，应该如下所示：
![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/unity/skillcooldown/f2275dff-46cf-4199-98ce-6b644bd0b26e.jpg)
（注： 我把Camera的背景颜色调成了黑色。。）
	没关系，修改Panel中GridLayoutGroup的设置如下：
	![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/unity/skillcooldown/a6777018-bd84-4316-b5ce-bc9f5ba56460.png)

现在再来看：
![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/unity/skillcooldown/41690570-4e83-4915-a229-ab2e637e5475.jpg)

Ok， 好多了。

+ 分别给每个技能按钮添加一个Image子节点，然后设置其SourceImage属性为各自的技能图片，**这两个图片将成为冷却效果的关键！！**
设置其属性如下：
![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/unity/skillcooldown/19aeaa74-e7b2-4f1a-953d-f8ccfe178586.png)

	这时候，拖动FillAmount感受下，看技能的icon，是不是有冷却的效果啦！！聪明的你可能已经知道怎么做了，下面还是说下脚本的实现吧。

+ 给SkillDot添加一个新的脚本 Skill，编辑如下：

```
using UnityEngine;
using UnityEngine.UI;
using System.Collections;

public class Skill : MonoBehaviour
{
    // 技能的图标
    public Image icon;

    // 技能的冷却时间
    public float coolDown;

    // 技能名称，用于区分使用了哪个技能的
    public string skillName;

    // 保存当前技能的冷却时间
    private float currentCoolDown;

    // 技能的按钮
    private Button skillButton;

    public void UseSkill(string skillName)
    {
        if (currentCoolDown >= coolDown)
        {
            // 使用技能，这里只是简单的打印了。
            Debug.LogFormat("使用 【{0}】", skillName);

            // 重置冷却时间
            currentCoolDown = 0;
        }
    }

    void Start()
    {
        // 获得技能按钮，然后绑定点击事件
        this.skillButton = this.GetComponent<Button>();
        skillButton.onClick.AddListener(() => this.UseSkill(skillName));

        // 一开始冷却时满的，可以立即使用技能
        // 如果不想让玩家一开始能立即使用技能，这里设置成别的小于技能冷却的值
        currentCoolDown = coolDown;
    }

    void Update()
    {
        if (currentCoolDown < coolDown)
        {
            // 更新冷却
            currentCoolDown += Time.deltaTime;

            // 显示冷却动画
            this.icon.fillAmount = currentCoolDown / coolDown;
        }
    }
}
```

build通过后，返回Unity，选中Skill，设置参数：
![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/unity/skillcooldown/1606d801-2c72-4caf-b3b9-46c8d60e8739.png)

运行游戏，使用点燃！冷却效果出来了。。不错，但是闪现还没有效果，再做一遍操作？当然可以，但是如果技能比较多呢。
如果爱偷懒的你肯定想到了一个东西，对，就是Prefab（而且后期如果是代码创建技能的话，这货更不能少）

在工程面板，创建一个Prefabs的文件夹，将SkillDot重名为Skill，并拖动到Prefabs文件夹，然后在Hierarchy面板上，删除SkillDot和SkillFlash。
将Skill Prefab重新拖动到Hierarchy面板中的Panel里面，拖动两次，并重命名、修改Icon和SkillName，CoolDown也一并改了吧，数值任意。

最终如下：
![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/unity/skillcooldown/2d3891ff-a849-4cce-877e-008318a08d4b.png)

**done**

#### 完整的代码：
https://github.com/le0zh/unity_skill_cooldown

>转载请注明出处。
