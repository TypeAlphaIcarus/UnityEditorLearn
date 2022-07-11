# AB包-异步加载-热更新(一)

# 一、编辑器基础

编辑器的特殊目录为Editor，不同文件夹下的Editor中的脚本都会合并，视为在同一个Editor文件夹中

Editor文件夹中的内容在打包时不会生成到项目中，仅在编辑模式下才会运行和使用

Unity代码的逻辑命名空间：UnityEngine

Unity编辑器命名空间：UnityEditor，该命名空间，不要出现在游戏被发布的逻辑代码中，会导致项目打包失败



### 1、基本的Inspector(检视器)属性

新建脚本`SimpleInspector`

```C#
public class SimpleInspector : MonoBehaviour
{
    [HideInInspector] public int a;     //添加了HideInInspector后，公开的可序列化数据将不会在Inspector窗口中显示
                                        //可以在保持变量公开的情况下，放置在Inspector对其进行修改

    [SerializeField] private int b;     //添加了SerializeField后，私有的可序列化数据将显示在Inspector窗口中
                                        //可以在保证变量私有的情况下，能够在Inspector窗口中对其进行修改
                                        //注意：public类型默认进行序列化

    //在检视面板显示对象,这里创建一个Unit类型用于测试
    //需要将类声明为Serializable后才能序列化显示在检视面板
    public Unit unit;

    [Space(50)]              //添加Space后，Inspector窗口中将空出一段距离

    [Header("这里填数字")]   //添加Header后，将在Inspector面板上显示一行文字，可以进行分割和提示
    [Tooltip("不要填写大于100的数字")]   //添加Tooltip后，鼠标悬浮在Inspector窗口的变量上时，将进行悬浮文字提示
    [Range(0, 100)]
    public int num;     //添加Range后，该变量将限制在一定范围内，且在检视窗口中显示为滑动条

    [Multiline(5)]          //添加Multiline后，可以输入多行数据，这里5是在Inspector中显示5行，可以输入多于5行的数据
    public string names;    //只能使用键盘上下移动

    [TextArea(5, 10)]              //添加TextArea后，可以输入多行字符串
    public string dialogue;        //这里在检视窗口中最少显示5行，最大显示10行，窗口大小会进行变化，超出最大行数后可以使用滚轮进行滑动

    //其它常用的Unity的可序列化类型
    public Color color;

    public Texture texture;

    public Sprite sprite;

    public List<string> list;       //列表将根据元素数量自动进行缩放

    public Wether wether;           //枚举类型将以下拉菜单的形式显示
}

[Serializable]
public class Unit
{
    [SerializeField] private int id;
    [SerializeField] private int health;
}

public enum Wether
{
    Sunny,
    Raining,
    Windy,
    Cloudy
}
```

显示效果：

<img src="../asset/image-20220707230711482.png" alt="image-20220707230711482" style="zoom:80%;" />

### 2、在Inspector窗口中运行方法

#### 2.1、通过脚本菜单运行方法

```C#
[ContextMenu("运行Print方法")]
public void Print()
{
    Debug.Log("运行了Print方法");
}
```

添加`ContextMenu`后，可以在脚本窗口的右上角(有Reset选项的窗口)点击，将运行自定义的方法

<img src="../asset/image-20220707231240544.png" alt="image-20220707231240544" style="zoom:80%;" />

#### 2.2、通过右键点击对象运行方法

```C#
[ContextMenuItem("打印名称", ("PrintName"))]    //参数二为会调用的方法名称
public string MyName;
public void PrintName()
{
    Debug.Log(MyName);
}
```

添加了`ContextMenuItem`后，右键点击相应的对象，将弹出菜单，可以选择并运行对应的方法

<img src="../asset/image-20220707231910632.png" alt="image-20220707231910632" style="zoom:80%;" />

<img src="../asset/image-20220707231916753.png" alt="image-20220707231916753" style="zoom:80%;" />

### 3、脚本间的依赖

新建脚本`Player`

```C#
[RequireComponent(typeof(BoxCollider))]     //让当前脚本组件依赖于某个组件，当添加该脚本组件时，将自动添加依赖的组件
public class Player : MonoBehaviour
{
    
}
```

<img src="../asset/image-20220707232627909.png" alt="image-20220707232627909" style="zoom:80%;" />

同时，不能单独删除被依赖的组件

注意：这里`BoxCollider`只是类名，`typeof(BoxCollider)`才是具体的类型



### 4、ExecuteInEditMode

```C#
[ExecuteInEditMode]
public class Player : MonoBehaviour
{
    void Update()
    {
		Debug.Log("Update");
    }
}
```

在添加`ExecuteInEditMode`后，在编辑模式下，对Hierarchy、Project和Inspector面板中，添加、修改或则删除任何数据，甚至点击或切换窗口，都会运行一次

在点击Play进入游戏模式后，将会正常运行



### 5、编辑器深度扩展

#### 5.1、扩展AddComponent菜单

```C#
[AddComponentMenu("自定义脚本/Player", 0)]	//第二个参数将对同一层级的组件进行排序
public class Unit : MonoBehaviour
{
    
}
```

添加`AddComponentMenu`后，可以通过AddComponent菜单，在自定义脚本下点击Player添加该脚本组件

如果为`[AddComponentMenu("一级/二级/三级/Player", 0)]`，将在点击对应深度的菜单后进行选择，该菜单为树状菜单

<img src="../asset/image-20220707234937090.png" alt="image-20220707234937090" style="zoom:80%;" />

最后一个一级将为对应的组件名称，可以任意修改



#### 5.2、多选下拉菜单

枚举类型默认为单选，可以实现多选



#### 5.3、组件扩展(外挂)

注意：组件的扩展(外挂)应当放置在Editor文件夹中

新建脚本`PlayerExtend`

```C#
using UnityEditor;

[CustomEditor(typeof(Unit))]		//指定扩展的组件类型
public class UnitExtend : Editor
{
    private Unit unit;
    private void OnEnable()
    {
        unit = target as Unit;	//也可写为 unit = (Unit)target, target为关联的对象
        Debug.Log($"{unit} OnEnable");
    }
    private void OnDisable()
    {
        Debug.Log($"{unit} OnDisable");
        player = null;
    }
}
```

`OnEnable()`会在**选中**或**添加**被关联的对象时运行

`OnDisable()`会在**取消选中**或**移除**被关联对象时运行

这里的两个方法实际指的是激活和取消指定对象的Inspector面板

若对象没有搭载被扩展的组件(这里位Unit脚本)，那么扩展脚本(UnitExtend)不会运行



继续扩展

```C#
[CustomEditor(typeof(Unit))]
public class UnitExtend : Editor
{
    private Unit unit;
    private void OnEnable()
    {
        unit = (Unit)target;
        Debug.Log($"{unit.name} OnEnable");
    }
    private void OnDisable()
    {
        Debug.Log($"{unit.name} OnDisable");
        unit = null;
    }
    
    public override void OnInspectorGUI()
    {
        EditorGUILayout.LabelField("单位属性");     //显示一行文字，类似Label

        EditorGUILayout.IntField("单位Id(只读)", unit.id);//显示关联对象的id，这样写是只读的，不能实时修改对象的值，因为没有将修改后的值进行赋值

        unit.id = EditorGUILayout.IntField("单位Id", unit.id);    //这样写就能实时修改

        //其他类似的还有，根据数据类型有不同的方法来显示
        unit.name = EditorGUILayout.TextField("单位名称", unit.name);            //string
        unit.health = EditorGUILayout.FloatField("单位生命值", unit.health);     //float
        unit.isMale = EditorGUILayout.Toggle("是否为男性", unit.isMale);         //bool
        unit.headDir = EditorGUILayout.Vector3Field("单位朝向", unit.headDir);   //Vector3
        unit.color = EditorGUILayout.ColorField("单位颜色", unit.color);         //Color
        
        //以上为一般的常用类型
        //但如果是GameObject对象，那么需要特殊处理

        unit.Weapon = EditorGUILayout.ObjectField("武器", unit.Weapon, typeof(GameObject), true) as GameObject;
        //四个参数分别为：显示的名称、获取的值的对象，值的类型，是否可以用场景的对象进行赋值

        unit.Texture = EditorGUILayout.ObjectField("贴图", unit.Texture, typeof(Texture), false) as Texture;
        //贴图不能直接用场景中的对象进行赋值，这里值为false，而是应当用资源文件进行赋值
        
        //接下来实现单选枚举和多选枚举

        //单选枚举          BattleArea为枚举类型，需要值转换为对应的数据类型后才能实时修改并赋值
        unit.BattleArea = (BattleArea)EditorGUILayout.EnumPopup("作战区域", unit.BattleArea);

        //多选枚举
        unit.Weapons = (Weapon)EditorGUILayout.EnumFlagsField("使用武器类型", unit.Weapons);
        //注意：多选枚举中每个类型按照二进制位来判断是否选中，需要在枚举类型中对其进行赋值
    }
}
```

多选枚举中的赋值：

```C#
public enum Weapon	//根据二进制为的状态来判断对应的元素是否选中
{
    machineGun = 1,
    rocketLancher = 2,
    pistol = 4
}
```



终极拓展：更新可序列化数据

