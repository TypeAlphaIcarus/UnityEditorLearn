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



### 5、Inspector深度扩展

#### 5.1、扩展AddComponent(添加组件)菜单

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



##### 常规扩展

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



##### 更新可序列化数据

```C#
public override void OnInspectorGUI()
{
    ...
    //更新可序列化的数据，显示效果类似在类中声明  public List<string> list
    serializedObject.Update();	//更新可序列化的数据
    SerializedProperty sp = serializedObject.FindProperty("Items");     //找到组件上的对应成员变量
    EditorGUILayout.PropertyField(sp, new GUIContent("道具信息"), true);    //绘制可序列化的数据
    serializedObject.ApplyModifiedProperties();     //将修改后的数据，写入到可序列化的原始数据中
}
```



##### 可视化扩展

```C#
public override void OnInspectorGUI()
{
    ...
    //绘制滑动条
    unit.health = EditorGUILayout.Slider(new GUIContent("单位生命值"), unit.health, 0, 100);

    //绘制提示窗口
    if (unit.health > 80)
        EditorGUILayout.HelpBox("血量太高", MessageType.Error);
    if (unit.health < 20)
        EditorGUILayout.HelpBox("血量太低", MessageType.Warning);

    //绘制按钮
    GUILayout.Button("这是一个按钮");
    if (GUILayout.Button("打印"))         //通过if来判断按钮是否被点击，按钮点击后会返回true
        Debug.Log("点击了按钮");
    
    //排列顺序
    //横向排列多个控件
    EditorGUILayout.BeginHorizontal();

    GUILayout.Button("按钮1");
    GUILayout.Button("按钮2");
    GUILayout.Button("按钮3");

    EditorGUILayout.EndHorizontal();
}
```



### 6、菜单栏扩展

新建脚本`MenuExtension`

```C#
public class MenuExtension
{
    //设置后将在菜单栏添加相应的选项，路径为工具-导出AB包，点击后将执行相应的方法
    [MenuItem("工具/导出AB包")]  
    static void BuildAssetBundle()
    {
        Debug.Log("导出AB包");
    }
}
```



### 7、窗口

新建脚本`PopWindow`

```C#
public class PopWindow : EditorWindow
{
    [MenuItem("工具/测试窗口")]
    static void OpenWindow()
    {
        //三个参数分别为：是否为工具窗口，窗口显示的名称，是否立即聚焦到窗口
        PopWindow window = GetWindow<PopWindow>(false, "窗口", true);
        window.minSize = new Vector2(400, 300);
        window.maxSize = new Vector2(800, 600);
    }

    private void OnEnable()     //打开窗口时调用
    {
        
    }
    private void OnDisable()    //关闭窗口时调用
    {
        
    }
    private void Update()       //打开窗口后每帧调用一次
    {
        
    }
    private void OnGUI()        //类似Inspector中的OnInspectorGUI()
    {
        if (GUILayout.Button("按钮"))
        {
            Debug.Log("点击了按钮");
        }
    }
    
    private void OnHierarchyChange()    //场景层级面板变化时调用一次
    {
        Debug.Log("Hierarchy Change!");
    }
    private void OnProjectChange()      //Project 资源窗口改变时调用
    {
        Debug.Log("Projecct Change!");
    }
    private void OnSelectionChange()    //场景中选中的对象改变时调用
    {
        Debug.Log(Selection.activeObject.name); //打印当前选中物体的名称
    }
}
```

在窗口中绘制的方式和在Inspector中类似



### 8、例子：节点编辑器

新建脚本`NodeManager`和`NodeManagerExtension`

```C#
[ExecuteInEditMode]
public class NodeManager : MonoBehaviour
{
    public List<GameObject> nodes;
    
    void Update()
    {
        DrawLinesBetweenNodes();
    }

    private void DrawLinesBetweenNodes()
    {
        if (nodes.Count <= 0)
            return;
        for (int i = 0; i < nodes.Count; i++)
        {
            Debug.DrawLine(nodes[i].transform.position,
                nodes[i + 1].transform.position,
                Color.red,
                Time.deltaTime);
        }
    }
}
```

```C#
namespace EditorLearn.Node
{
    [CustomEditor(typeof(NodeManager))]
    public class NodeManagerExtension : Editor
    {
        NodeManager manager;
        bool isEditor = false;      //是否开始编辑节点

        private void OnEnable()
        {
            manager = (NodeManager)target;
        }
        private void OnDisable()
        {
            manager = null;
        }
        public override void OnInspectorGUI()
        {
            //绘制序列化List<>列表
            serializedObject.Update();
            SerializedProperty nodes = serializedObject.FindProperty("nodes");
            EditorGUILayout.PropertyField(nodes, new GUIContent("路径"), true);
            serializedObject.ApplyModifiedProperties();

            if (!isEditor && GUILayout.Button("开始编辑"))  //非编辑状态，点击开始编辑按钮
            {
                NodeWindow.OpenWindow(manager.gameObject);  //调用打开窗口的方法，进入编辑模式，传入物体
                isEditor = true;
            }
            else if (isEditor && GUILayout.Button("结束编辑"))  //非编辑状态下，点击结束编辑按钮
            {
                NodeWindow.CloseWindow();
                isEditor = false;
            }
            if (GUILayout.Button("删除最后一个节点"))
            {
                RemoveLast();
            }
            else if (GUILayout.Button("清除所有节点"))
            {
                RemoveAll();
            }
        }

        RaycastHit hit;

        //在选中关联的脚本后，鼠标在Scene窗口中变化时，执行方法，如鼠标的点击和移动
        private void OnSceneGUI()
        {
            if (!isEditor)  //非编辑状态下不生成节点
                return;

            //Event.current.button判读点击了哪个按键，0是鼠标左键
            //Event.current.type判断鼠标的事件方式，EventType.MouseDown为点击事件
            if (Event.current.button == 0 && Event.current.type == EventType.MouseDown)
            {
                //从Scene视图发射射线，这里和场景中的摄像机无关，是在Scene中自由移动视角的位置
                Ray ray = HandleUtility.GUIPointToWorldRay(Event.current.mousePosition);
                if (Physics.Raycast(ray, out hit, 100))
                {
                    InstancePathNode(hit.point + Vector3.up * 0.1f);
                }
            }
        }

        private void InstancePathNode(Vector3 hitPoint)
        {
            GameObject prefab = Resources.Load<GameObject>("PathNode"); //获取节点预制体
                                                                        //实例化节点预制体，参数二为位置，参数三：旋转 参数四：父物体
            GameObject pathNode = Instantiate<GameObject>(prefab, hitPoint, Quaternion.identity, manager.transform);
            manager.nodes.Add(pathNode);    //将生成的节点添加到列表中
        }

        private void RemoveLast()
        {
            if (manager.nodes.Count > 0)
            {
                //DestroyImmediate()为立即销毁，会影响主线程运行
                //Destory()为异步销毁，一般在下一帧就销毁了，不会影响主线程运行
                DestroyImmediate(manager.nodes[manager.nodes.Count - 1]);   //摧毁最后一个节点，即最新添加的节点
                manager.nodes.RemoveAt(manager.nodes.Count - 1);
            }
        }

        private void RemoveAll()
        {
            if (manager.nodes.Count > 0)
            {
                for (int i = 0; i < manager.nodes.Count - 1; i++)
                {
                    DestroyImmediate(manager.nodes[i]);
                    manager.nodes.RemoveAt(i);
                }
            }
        }

    }
    public class NodeWindow : EditorWindow
    {
        static NodeWindow window;
        static GameObject nodeManager;

        //这个窗口有一定的作用
        //Unity在新建物体后，会自动选中新建的物体
        //这里在进入编辑模式后调用该方法打开了一个窗口，并获取了父节点的物体，即搭建了关联脚本的物体
        //打开这个窗口后会持续运行Update，始终让该物体被选中
        //这样就不会因为创建新物体后切换选中到新物体而退出编辑模式
        public static void OpenWindow(GameObject manager)
        {
            nodeManager = manager;
            window = EditorWindow.GetWindow<NodeWindow>();  //开启窗口后才能运行下面的Update()
        }
        private void Update()
        {
            Selection.activeGameObject = nodeManager;   //让被编辑的物体一直被选中
        }
        public static void CloseWindow()
        {
            window.Close();
        }
    }
}
```

这样就能在添加了节点编辑组件的物体上编辑节点了

# 二、AssetBundle

## 1、概述

AssetBundle是独立于游戏主包存在的资源存储文件，可以存放在网络服务器或者本地硬盘上，使用内部资源时，需要单独存储和下载

工作流程：使用`WWW.assetBundle`获取网络AB包或`AssetBunlde.CreateFromFile`获取本地AB包，获取后镜像到内存中，然后使用`AssetBundle.Load`来解压镜像文件，之后便可引用或实例化对应的文件，最后使用`AssetBundle.Unload`来释放镜像文件占用的内存

### 1.1、AB包和Resource的区别

1、存储方式不同

​	Resource存储在游戏的发布包中

​	AB包存储在独立的文件中

2、加载方式不同

​	Resources内部资源使用`Resources.Load()`

​	AB包会先下载(或本地获取)相应文件(如从StreamingAsset文件夹中拷贝到可写目录)，然后通过AB包文件路径，加载并解压AB包，然后通过资源名称，加载内部资源



注意：AB包内部不能包含C#脚本文件，AB包可以配合Lua实现资源和游戏逻辑代码的更新



### 1.2、AB包的压缩方式

1、不压缩：AB包会较大，但加载速度很快

2、LZMA：默认压缩模式，文件大小居中，加载速度居中

3、LZ4：压缩比例高，加载慢



### 1.3、依赖关系

如果一个AB包（A包）使用了另一个AB包（B包）的资源，那么两个AB包就产生了依赖关系，即A依赖于B

如一个预制体和其使用的材质、贴图等，若预制体打为A包，材质、贴图打为B包，那么A包依赖于B包

需要先加载依赖的AB包，然后再加载自身的AB包

依赖关系保存在manifest文件中



## 2、创建AB包

准备一些资源，如果文件经常需要更新，就不要存放在Resources目录下

新建一个AB文件夹，将所需要打包的文件分类放入文件夹

选中同一类型的文件，在右下角AssetBundle选项中，选择New创建并命名即可

例子：这里选择几个UI图片，创建新的AB包，命名为ui(小写！)

<img width="396" height="108" src="data:image/png;base64,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" />

下面还有两个选项，RemoveUnusedName：删除没有使用的AB包，FilterSelectedName：过滤并显示AB包内的所有资源

注意：AB包配置或内部资源修改后，都需要重新生成

如果AB包的名称配置为`ui/packge`，那么`ui`会作为AB包存储的父目录，``packge`为AB包的名称

在导出AB包时，如果创建路径为`ui/packge`，那么会生成`ui`文件夹，然后将AB包(这里是`packge`)存放在该路径下



这里已经规划完成AB包，接下来要生成AB包

这里需要创建一个Editor工具来进行出包

在Editor文件夹中新建脚本`ExportAB`

```C#
public class ExportAB
{
    [MenuItem("Tool/Export Asset Bundle")]
    public static void Export()
    {
        string path = Application.dataPath;         //该路径为Assets文件夹目录，这里是E:\UnityProject\EditorLearn\Assets
        path = path.Substring(0, path.Length - 6);  //截取字符串长度，截取后为E:\UnityProject\EditorLearn\
        path = Path.Combine(path, "ab");            //最终路径：E:\UnityProject\EditorLearn\ab
        if (!Directory.Exists(path))     //检查存储路径是否存在
        {
            Directory.CreateDirectory(path);    //不存在就创建路径
        }
        //创建AB包 ，不同平台AB包是不同的，这里选择了windows用于测试
        BuildPipeline.BuildAssetBundles(
            path,                           //存储路径
            BuildAssetBundleOptions.None,   //导出选项
            BuildTarget.StandaloneWindows   //目标平台
            );
        Debug.Log($"导出AB包成功，导出路径：{path}");
    }
}
```

运行后在对应文件夹可以看见生成的AB包

<img src="../../../asset/image-20220715112545277.png" alt="image-20220715112545277" style="zoom: 80%;" />

<img src="../../../asset/image-20220715112556065.png" alt="image-20220715112556065" style="zoom:80%;" />

## 3、加载AB包资源

新建类`Config`用于存储加载路径

```C#
public class Config
{
    public static string ABPath = Application.dataPath.Substring(0, Application.dataPath.Length - 6) + "ab";
}
```

新建脚本`SimpleLoad`用于测试

```C#
public class SimpleLoad : MonoBehaviour
{
    void Start()
    {
        //1、加载ab包文件
        AssetBundle ab = AssetBundle.LoadFromFile(Config.ABPath + "/ui");

        //2、加载所需资源文件
        Sprite jesscia = ab.LoadAsset<Sprite>("Jesscia");

        if (jesscia != null)
        {
            if (TryGetComponent(out SpriteRenderer renderer)) renderer.sprite = jesscia;
        }
        else
        {
            Debug.Log("文件不存在");
        }
        ab.Unload(false);	//加载完成后释放内存
    }
}
```



## 4、在运行过程中替换资源

```C#
public class SimpleLoad : MonoBehaviour
{
    void Start()
    {
        LoadAB(Config.ABPath + "old/ui");   //一开始加载旧的AB包
    }
    public void ChangeAB()
    {
        LoadAB(Config.ABPath + "new/ui");   //按钮调用方法，加载更改后的AB包
    }
    private void LoadAB(string path)
    {
        //1、加载ab包文件
        AssetBundle ab = AssetBundle.LoadFromFile(path);

        //2、加载所需资源文件
        Sprite jesscia = ab.LoadAsset<Sprite>("Jesscia");

        if (jesscia != null)
        {
            if (TryGetComponent(out SpriteRenderer renderer)) renderer.sprite = jesscia;
        }
        else
        {
            Debug.Log("文件不存在");
        }
        ab.Unload(false);
    }
}
```

这样在运行过程中，点击按钮后，就能替换对应的贴图

这里两个AB包都是存放在本地的，实际可用将多个AB包存放在远程服务器上，根据需要进行下载



## 5、导出工具优化

### 5.1、打包时的选项BuildAssetBundleOptions

```C#
    [Flags]
    public enum BuildAssetBundleOptions
    {
        None = 0,

        UncompressedAssetBundle = 1,	//这个选项不会压缩AB包，将不会使用压缩模式

        CollectDependencies = 2,		//导出AB包时，会记录所有依赖关系
        //     Forces inclusion of the entire asset.
        CompleteAssets = 4,
        //     Do not include type information within the AssetBundle.
        DisableWriteTypeTree = 8,
        
        DeterministicAssetBundle = 16,	//会将AssetBundle的哈希校验值，存储在ID中(默认开启)

        ForceRebuildAssetBundle = 32,	//强制重新导出所有的AB包，会重新导出并覆盖旧的AB包
        //     Ignore the type tree changes when doing the incremental build check.
        IgnoreTypeTreeChanges = 64,
        //     Append the hash to the assetBundle name.
        AppendHashToAssetBundleName = 128,
        //     Use chunk-based LZ4 compression when creating the AssetBundle.
        ChunkBasedCompression = 256,	//使用LZ4算法进行压缩
        //     Do not allow the build to succeed if any errors are reporting during it.
        StrictMode = 512,
        //     Do a dry run build.
        DryRunBuild = 1024,
        //     Disables Asset Bundle LoadAsset by file name.
        DisableLoadAssetByFileName = 4096,
        //     Disables Asset Bundle LoadAsset by file name with extension.
        DisableLoadAssetByFileNameWithExtension = 8192,
        //     Removes the Unity Version number in the Archive File & Serialized File headers
        //     during the build.
        AssetBundleStripUnityVersion = 32768,
        //     Enable asset bundle protection.
        EnableProtection = 65536
    }
```

这里为多选，每个选项用二进制位来表示，使用`|`来进行多选

例子：

```C#
BuildAssetBundleOptions.UncompressedAssetBundle | BuildAssetBundleOptionsCollectDependencies
```



## 6、AB包加载深入

### 6.1、完整的加载流程

1、加载主AB包

​	根据主AB包配置文件，获取依赖的AB包，因为依赖关系存储在主AB包的manifest文件中

​	加载所有依赖的AB包

2、加载AB包文件

​	`AssetBundle ab = AssetBundle.LoadFromFile("路径")`或`AssetBundle ab = AssetBundle.LoadFromFileSync("路径")`

3、加载AB包内的资源

​	资源对象 = `ab.LoadAsset<资源类型>("资源名称")`或 资源对象 = `ab.LoadAssetSync<资源类型>("资源名称")`

注意：AB包不能重复加载

这里将图片放置在一个空物体上，设为预制体，然后分别将预制体和图片设为两个AB包

新建脚本`LoadTest`

```C#
public class LoadTest : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        //加载主AB包，主AB包名称与导出路径的文件夹名称相同，会自动创建
        AssetBundle mainAB = AssetBundle.LoadFromFile(Config.ABPath + "/ab");

        //获取配置文件，这里为固定的写法
        AssetBundleManifest manifest = mainAB.LoadAsset<AssetBundleManifest>("AssetBundleManifest");

        //分析预制体所在AB包依赖哪些AB包
        string[] dependencies = manifest.GetAllDependencies("prefabs");     //这里参数为预制体所在的AB包名称，返回所有引用的AB包名称

        //加载所有依赖的AB包
        for (int i = 0; i < dependencies.Length; i++)
        {
            AssetBundle.LoadFromFile(Config.ABPath + "/" + dependencies[i]);    //根据名称加载所有AB包
        }

        //加载预制体所在的AB包
        AssetBundle prefab = AssetBundle.LoadFromFile(Config.ABPath + "/prefabs");

        //加载预制体
        Object obj = prefab.LoadAsset("TestObj");

        Instantiate(obj);
    }
}
```

运行后便会加载预制体了

AB包管理器内部可以使用**字典**来实现，可以放置同名的AB包被加载，即可以放置AB包被加载多次，然后可以根据名称去卸载AB包



### 6.2、异步加载

这里介绍异步加载

新建脚本`AsyncLoadTest`

```C#
public class AsyncLoadTest : MonoBehaviour
{
    void Start()
    {
        StartCoroutine(LoadAB());
    }
    IEnumerator LoadAB()
    {
        AssetBundleCreateRequest abcr = AssetBundle.LoadFromFileAsync(Config.ABPath + "/old/ui");
        yield return abcr;

        Sprite sprite = abcr.assetBundle.LoadAsset<Sprite>("Jesscia");

        if (TryGetComponent(out SpriteRenderer renderer)) 
            renderer.sprite = sprite;
    }
}
```



## 8、内存分析

文件加载不会占用内存空间(那么卸载也不会释放内存)，真正加载资源时才会占用内存

使用`Unload(true)`时，会卸载掉AB包和其正在使用的资源

使用`Unload(false)`时，只会卸载掉AB包，如果再次加载AB包和同一个资源，那么之前的资源将不会被引用，成为游离资源

因为Unity内存为懒回收，不会自动回收没有使用的内存，需要进行手动释放

```C#
Resources.UnloadUnusedAssets();
```

尽量减少使用频率，GC会消耗大量资源



# 三、Lua

## 1、概述

特点：

​	轻量级：可以方便嵌入其它程序

​	可扩展：提供了易于使用的接口和机制，由宿主语言(C/C++/C#等)提供一部分功能，Lua可以直接使用

使用方式：需要高性能的部分，使用C/C++等，需要快速实现的部分，使用Lua



## 2、数据类型

### 2.1、注释

```lua
--注释--
--[[
	多行注释
  ]]
--[[
	多行注释
--]]
```



### 2.2、数据类型

Lua中有8种基本的数据类型

```lua
nil			--表示无效值
boolean		--布尔类型
number		--双精度浮点实数
string		--字符串，可以用单引号或双引号表示
function	--由C或Lua编写的函数
userdata	--表示任意存储在C中的数据结构
thread		--表示执行的独立线路，用于执行协同程序
table		--表，实际为一个关联数组，数组的索引可以是数字、字符串或表类型
    		--在Lua中，table的创建是通过构造表达式来完成的，最简单的是 {} ，表示一个空表
```



在使用`type()`进行类型比较时应当加上" "，因为其返回值是一个string类型

```lua
x = 1
print(type(x) == number)	--这里number其实是为nil的变量
print(type(x) == "number")	--输出true
```



#### 布尔值

在Lua中，false和nil为false，其它均为true，0和1都为true



#### number

Lua默认只有一种number类型，即double类型，一下数据均为number类型

```lua
print(type(2))
print(type(2.2))
print(type(0.2))
print(type(2e+1))
print(type(0.2e-1))
print(type(7.8263692594256e-06))
```



#### string

字符串使用" "或' '来表示

```lua
string1 = "this is string1"
string2 = 'this is string2'
```



也可以使用[[ ]]来表示多行字符串

```lua
html = [[
<html>
<head></head>
<body>
    <a href="http://www.runoob.com/">菜鸟教程</a>
</body>
</html>
]]
print(html)
```



在对字符串进行算数运算时，Lua会尝试将该字符串转为数字

```lua
print("2" + 6)
--输出8
print("2" + "6")
--输出8
```



Lua中字符串通过 ..来连接，类似C#中的+

```lua
print("Hello" .. "World")
--输出HelloWorld
```



#用于计算字符串的长度

```lua
s = "123456"
print(#s)
--输出6
```



#### table表

Lua中，表是通过构造表达式来完成的，最简单的是 { }，创建了一个空表

```lua
local table1 = {}						--创建了一个空表
local table2 = {"apple","orange","pear"}  --初始化创建表
```

Lua中的表为关联数组，数组索引可以是数组或字符串

```lua
a = {}
key = 10
a["key"] = "value"
a[key] = 22
a[key] = a[key] + 11
for k, v in pairs(a) do
    print(k .. ":" .. v)
end
--输出10 ：33 和 key:value
--即a[10] = 33 a["kye"] = "value"
```

注意：

Lua的表从索引1开始

table长度和大小不固定，有新数据时table的长度会自动边长，没有初始化的table值为nil



#### function函数

在Lua中，函数被看作是"第一类值"，可以存放在变量里

```lua
function factorial(n)
    if n == 0 then
        return 1
    else
        return n * factorial(n - 1)
    end
end
print(factorial(5))
factorial2 = factorial
print(factorial2(5))
--两个都输出120
```

function可以以匿名函数的方式传递参数

```lua
function test(table, fun)
    for k, v in pairs(tab) do
        print(fun(k,v))
    end
end

tab = {key1 = "val1", key2 = "val2"}
test(tab, 
    function(key,val)	--匿名函数
        return key.."="..val
    end
)
--输出
--key1 = val1
--key2 = val2
```



#### thread线程

Lua中，最主要的线程是协同程序(coroutine)，与线程类似，拥有独立的栈、局部变量和指令指针，可以和其它协程共享局部变量和其它大部分东西

线程和协程的区别：线程可以同时多个运行，协程任意时刻只能运行一个，并且只有处于运行状态的协程被挂起时(suspend)才会暂停



#### userdata（自定义类型）

userdata为用户自定义的类型，用于表示由应用程序或C/C++语言库所创建的类型，可以将任意C/C++类型存储到Lua变量中调用



## 3、变量

Lua 变量有三种类型：全局变量、局部变量、表中的域。变量的默认值均为 nil。

Lua 中的变量都是全局变量，哪怕是语句块或是函数里，除非用 local 显式声明为局部变量。



### 3.1、赋值语句

Lua可以同时对多个变量进行赋值，赋值语句右侧的值会依次匹配左侧的变量

```lua
a, b = 10, "Hello"
--a为10，b为"Hello"
```



遇到赋值语句Lua会先计算右边所有的值然后再执行赋值操作，所以可以进行交换变量的值

```lua
x, y = y, x
a[i], a[j] = a[j], a[i]
```

还可以将函数返回值返回给变量

```lua
a, b = f()
```



注意：应尽量使用局部变量，可以避免命名冲突，而且局部变量访问更快



### 3.2、索引

table的索引除了[ ]之外还有.方式

```lua
t[i]
t.i	--当索引为字符串类型时可使用
```



## 4、循环

### 4.1、while循环

```lua
while(condition)
do
   statements
end

--例子
a = 10
while( a < 20 )
do
   print("a 的值为:", a)
   a = a + 1
end
```



### 4.2、for循环

#### 数值for循环

```lua
for var=exp1,exp2,exp3 do	--var 从 exp1 变化到 exp2，每次变化以 exp3 为步长递增 var，并执行一次 "执行体"。exp3 是可选的，如果不指定，默认为1。
    <执行体>  
end  

--例子：
for i = 1,f(x) do
    print(i)
end
 
for i = 10, 1, -1 do
    print(i)
end
```



#### 泛型for循环

泛型 for 循环通过一个迭代器函数来遍历所有值，类似 foreach 语句。

```lua
--打印数组a的所有值  
a = {"one", "two", "three"}
for i, v in ipairs(a) do
    print(i, v)
end 
--i是数组索引值，v是对应索引的数组元素值。ipairs是Lua提供的一个迭代器函数，用来迭代数组。
```



### 4.3、repeat...until循环

```lua
repeat
   statements
until( condition )

--例子
a = 10
repeat	--[ 执行循环 --]
   print("a的值为:", a)
   a = a + 1
until( a > 15 )
```



### 4.4、循环控制语句

#### break

用于退出当前循环或语句，并开始脚本执行紧接着的语句

如果使用循环嵌套，break语句将停止最内层循环的执行，并开始执行的外层的循环语句。



#### goto语句

 goto 语句允许将控制流程无条件地转到被标记的语句处。

```lua
local a = 1
::label:: print("--- goto label ---")

a = a+1
if a < 3 then
    goto label   -- a 小于 3 的时候跳转到标签 label
end
```



## 5、流程控制

### 5.1、if

```lua
if(布尔表达式)
then
   --[ 在布尔表达式为 true 时执行的语句 --]
end
```



### 5.2、if...else

```lua
if(布尔表达式)
then
   --[ 布尔表达式为 true 时执行该语句块 --]
else
   --[ 布尔表达式为 false 时执行该语句块 --]
end
```

```lua
if( 布尔表达式 1)
then
   --[ 在布尔表达式 1 为 true 时执行该语句块 --]

elseif( 布尔表达式 2)
then
   --[ 在布尔表达式 2 为 true 时执行该语句块 --]

elseif( 布尔表达式 3)
then
   --[ 在布尔表达式 3 为 true 时执行该语句块 --]
else 
   --[ 如果以上布尔表达式都不为 true 则执行该语句块 --]
end
```



## 6、函数

### 6.1、格式

```lua
--定义格式如下
optional_function_scope function function_name( argument1, argument2, argument3..., argumentn)
    function_body
    return result_params_comma_separated
end
```

- **optional_function_scope:** 该参数是可选的制定函数是**全局函数**还是**局部函数**，未设置该参数**默认为全局函数**，如果你需要设置函数为局部函数需要使用关键字 **local**。
- **function_name:** 指定函数名称。
- **argument1, argument2, argument3..., argumentn:** 函数参数，多个参数以逗号隔开，函数也可以不带参数。
- **function_body:** 函数体，函数中需要执行的代码语句块。
- **result_params_comma_separated:** 函数返回值，Lua语言函数可以返回多个值，每个值以逗号隔开。

例子：

```lua
--[[ 函数返回两个值的最大值 --]]
function max(num1, num2)

   if (num1 > num2) then
      result = num1;
   else
      result = num2;
   end

   return result;
end
-- 调用函数
print("两值比较最大值为 ",max(10,4))
print("两值比较最大值为 ",max(5,6))
```



### 6.2、函数参数

Lua 中可以将函数作为参数传递给函数，如下实例：

```lua
myprint = function(param)
   print("这是打印函数 -   ##",param,"##")
end

function add(num1, num2, functionPrint)
   result = num1 + num2
   functionPrint(result)	-- 调用传递的函数参数
end

myprint(10)		--直接调用函数
add(2,5,myprint)-- myprint 函数作为参数传递
--输出
--这是打印函数 -   ##    10    ##
--这是打印函数 -   ##    7    ##
```



### 6.3、多返回值

Lua函数可以返回多个结果值，比如string.find，其返回匹配串"开始和结束的下标"（如果不存在匹配串返回nil）。

```lua
s, e = string.find("www.runoob.com", "runoob") 
print(s, e)
--输出
--5    10	匹配的字符串下标为 5 - 10
```



### 6.4、可变参数

Lua 函数可以接受可变数目的参数，和 C 语言类似，在函数参数列表中使用三点 **...** 表示函数有可变的参数。

```lua
function add(...)  --这里类似C#的parmas，可以输入任意个参数
local s = 0  
  for i, v in ipairs{...} do   --> {...} 表示一个由所有变长参数构成的数组  
    s = s + v  
  end  
  return s  
end  
print(add(3,4,5,6,7))  --->25

--例子
function average(...)
   result = 0
   local arg={...}    --> arg 为一个表，局部变量
   for i,v in ipairs(arg) do
      result = result + v
   end
   print("总共传入 " .. #arg .. " 个数")
   return result/#arg
end

print("平均值为",average(10,5,3,4,5,6))
```

可以通过 select("#",...) 来获取可变参数的数量

```lua
function average(...)
   result = 0
   local arg={...}
   for i,v in ipairs(arg) do
      result = result + v
   end
   print("总共传入 " .. select("#",...) .. " 个数")
   return result/select("#",...)
end

print("平均值为",average(10,5,3,4,5,6))
```

有时候可能需要几个固定参数加上可变参数，固定参数必须放在变长参数之前:

```lua
function fwrite(fmt, ...)  ---> 固定的参数fmt
    return io.write(string.format(fmt, ...))    
end

fwrite("runoob\n")       --->fmt = "runoob", 没有变长参数。  
fwrite("%d%d\n", 1, 2)   --->fmt = "%d%d", 变长参数为 1 和 2
```



## 7、运算符

基本和C相同，但需要记住的：

/为除法运算，计算结果包含小数部分

//为整除运算，结果不包含小数部分

逻辑运算符

and，逻辑&，若A为false，则返回A，否则返回B，两个都为true，返回B

or，逻辑 |，若A为true，则返回A，否则返回B，两个都为false，返回B

not，逻辑 !，结果取反



## 8、字符串

常用的字符串操作

```lua
string.upper( s )	--全部转为大写
string.lower( s )	--全部转为小写

string.gsub( mainString, findString, replaceString, num )	
--替换字符串，mainString为要操作的字符，将findString替换为replaceString，num为替换次数，不写则全部替换
--string.gsub("aaaa","a","z",3);	结果为zzza

string.find(str, substr, [init,[plain]])
--在一个指定的目标字符串 str 中搜索指定的内容 substr，如果找到了一个匹配的子串，就会返回这个子串的起始索引和结束索引，不存在则返回 nil。
--init 指定了搜索的起始位置，默认为 1，可以一个负数，表示从后往前数的字符个数。
--plain 表示是否以正则表达式匹配。
--以下实例查找字符串 "Lua" 的起始索引和结束索引位置：
string.find("Hello Lua user", "Lua", 1) --结果为7 9

string.reverse() --反转字符串
string.format()  --返回格式化字符串
string.char()	--将整型数字转成字符并连接 string.char(97,98,99,100)	abcd
string.byte()	--转换字符为整数值(可以指定某个字符，默认第一个字符) string.byte("ABCD") 65，转换了A
string.len()	--计算字符串长度
string.rep(sting, n)	--返回字符串string的n个拷贝 65	string.rep("abcd",2)  abcdabcd 

string.gmatch(str, pattern)	
--返回一个迭代器函数，每一次调用这个函数，返回一个在字符串 str 找到的下一个符合 pattern 描述的子串。如果参数 pattern 描述的字符串没有找到，迭代函数返回nil。

string.match(str, pattern, init)
--string.match()只寻找源字串str中的第一个配对. 参数init可选, 指定搜寻过程的起点, 默认为1。
--在成功配对时, 函数将返回配对表达式中的所有捕获结果; 如果没有设置捕获标记, 则返回整个配对字符串. 当没有成功的配对时, 返回nil。

```

匹配模式

```lua
--匹配日期
s = "Deadline is 30/05/1999, firm"
date = "%d%d/%d%d/%d%d%d%d"
print(string.sub(s, string.find(s, date)))    --> 30/05/1999
```



## 9、数组

### 9.1、一维数组

```lua
array = {"Lua", "Tutorial"}

for i= 0, 2 do
   print(array[i])
end
```

Lua的索引可以是负数

```lua
array = {}

for i= -2, 2 do
   array[i] = i *2
end

for i = -2,2 do
   print(array[i])
end
```



### 9.2、多维数组

```lua
-- 初始化数组
array = {}
for i=1,3 do
   array[i] = {}
      for j=1,3 do
         array[i][j] = i*j
      end
end

-- 访问数组
for i=1,3 do
   for j=1,3 do
      print(array[i][j])
   end
end
```

## 10、迭代器

迭代器（iterator）是一种对象，它能够用来遍历标准模板库容器中的部分或全部元素，每个迭代器对象代表容器中的确定的地址。

在 Lua 中迭代器是一种支持指针类型的结构，它可以遍历集合的每一个元素。

泛型 for 在自己内部保存迭代函数，实际上它保存三个值：迭代函数、状态常量、控制变量。

泛型 for 迭代器提供了集合的 key/value 对，语法格式如下：

```lua
for k, v in pairs(t) do
    print(k, v)
end

--例子
array = {"Google", "Runoob"}

for key,value in ipairs(array)
do
   print(key, value)
end
```



无状态迭代器

无状态的迭代器是指不保留任何状态的迭代器，因此在循环中我们可以利用无状态迭代器避免创建闭包花费额外的代价。

每一次迭代，迭代函数都是用两个变量（状态常量和控制变量）的值作为参数被调用，一个无状态的迭代器只利用这两个值可以获取下一个元素。

这种无状态迭代器的典型的简单的例子是 ipairs，它遍历数组的每一个元素，元素的索引需要是数值。

以下实例我们使用了一个简单的函数来实现迭代器，实现 数字 n 的平方：

```lua
function square(iteratorMaxCount,currentNumber)
   if currentNumber<iteratorMaxCount
   then
      currentNumber = currentNumber+1
   return currentNumber, currentNumber*currentNumber
   end
end

for i,n in square,3,0
do
   print(i,n)
end

--例子
function iter (a, i)
    i = i + 1
    local v = a[i]
    if v then
       return i, v
    end
end
 
function ipairs (a)
    return iter, a, 0
end
```



多状态迭代器

很多情况下，迭代器需要保存多个状态信息而不是简单的状态常量和控制变量，最简单的方法是使用闭包，还有一种方法就是将所有的状态信息封装到 table 内，将 table 作为迭代器的状态常量，因为这种情况下可以将所有的信息存放在 table 内，所以迭代函数通常不需要第二个参数。

以下实例我们创建了自己的迭代器：

```lua
array = {"Google", "Runoob"}

function elementIterator (collection)
   local index = 0
   local count = #collection
   -- 闭包函数
   return function ()
      index = index + 1
      if index <= count
      then
         --  返回迭代器的当前元素
         return collection[index]
      end
   end
end

for element in elementIterator(array)
do
   print(element)
end
```



## 11、表

构造器是创建和初始化表的表达式。表是Lua特有的功能强大的东西。最简单的构造函数是{}，用来创建一个空表。可以直接初始化数组:

```lua
-- 初始化表
mytable = {}

-- 指定值
mytable[1]= "Lua"

-- 移除引用
mytable = nil
-- lua 垃圾回收会释放内存
```



### table的操作

```lua
table.concat(table,[,sep[,start[,end]]]):
--concat是concatenate(连锁, 连接)的缩写. table.concat()函数列出参数中指定table的数组部分从start位置到end位置的所有元素, 元素间以指定的分隔符(sep)隔开。

table.insert(table,[pos,] value):
--在table的数组部分指定位置(pos)插入值为value的一个元素. pos参数可选, 默认为数组部分末尾.

table.maxn(table)
--指定table中所有正数key值中最大的key值. 如果不存在key值为正数的元素, 则返回0。(Lua5.2之后该方法已经不存在了,本文使用了自定义函数实现)

table.remove(table,[,pos])
--返回table数组部分位于pos位置的元素. 其后的元素会被前移. pos参数可选, 默认为table长度, 即从最后一个元素删起。

table.sort(table[,comp])
--对给定的table进行升序排序。
```



## 12、模块与包

模块类似于一个封装库，从 Lua 5.1 开始，Lua 加入了标准的模块管理机制，可以把一些公用的代码放在一个文件里，以 API 接口的形式在其他地方调用，有利于代码的重用和降低代码耦合度。

Lua 的模块是由变量、函数等已知元素组成的 table，因此创建一个模块很简单，就是创建一个 table，然后把需要导出的常量、函数放入其中，最后返回这个 table 就行。以下为创建自定义模块 module.lua，文件代码格式如下：

```lua
-- 文件名为 module.lua
-- 定义一个名为 module 的模块
module = {}
 
-- 定义一个常量
module.constant = "这是一个常量"
 
-- 定义一个函数
function module.func1()
    io.write("这是一个公有函数！\n")
end
 
local function func2()
    print("这是一个私有函数！")
end
 
function module.func3()
    func2()
end
 
return module
```

模块的结构就是一个 table 的结构，因此可以像操作调用 table 里的元素那样来操作调用模块里的常量或函数。

上面的 func2 声明为程序块的局部变量，即表示一个私有函数，因此是不能从外部访问模块里的这个私有函数，必须通过模块里的公有函数来调用.



### require函数

Lua提供了一个名为require的函数用来加载模块。要加载一个模块，只需要简单地调用就可以了。例如：

```lua
require("<模块名>")
require "<模块名>"
--执行 require 后会返回一个由模块常量或函数组成的 table，并且还会定义一个包含该 table 的全局变量。

--例子
-- test_module.lua 文件
-- module 模块为上文提到到 module.lua
require("module")
 
print(module.constant)
 
module.func3()
```



### 加载机制

对于自定义的模块，模块文件不是放在哪个文件目录都行，函数 require 有它自己的文件路径加载策略，它会尝试从 Lua 文件或 C 程序库中加载模块。

require 用于搜索 Lua 文件的路径是存放在全局变量 package.path 中，当 Lua 启动后，会以环境变量 LUA_PATH 的值来初始这个环境变量。如果没有找到该环境变量，则使用一个编译时定义的默认路径来初始化。

当然，如果没有 LUA_PATH 这个环境变量，也可以自定义设置，在当前用户根目录下打开 .profile 文件（没有则创建，打开 .bashrc 文件也可以），例如把 "~/lua/" 路径加入 LUA_PATH 环境变量里：

```lua
#LUA_PATH
export LUA_PATH="~/lua/?.lua;;"
```

文件路径以 ";" 号分隔，最后的 2 个 ";;" 表示新加的路径后面加上原来的默认路径。

接着，更新环境变量参数，使之立即生效。

```lua
source ~/.profile
```

这时假设 package.path 的值是：

```lua
/Users/dengjoe/lua/?.lua;./?.lua;/usr/local/share/lua/5.1/?.lua;/usr/local/share/lua/5.1/?/init.lua;/usr/local/lib/lua/5.1/?.lua;/usr/local/lib/lua/5.1/?/init.lua
```

那么调用 require("module") 时就会尝试打开以下文件目录去搜索目标。

```lua
/Users/dengjoe/lua/module.lua;
./module.lua
/usr/local/share/lua/5.1/module.lua
/usr/local/share/lua/5.1/module/init.lua
/usr/local/lib/lua/5.1/module.lua
/usr/local/lib/lua/5.1/module/init.lua
```

如果找过目标文件，则会调用 package.loadfile 来加载模块。否则，就会去找 C 程序库。

搜索的文件路径是从全局变量 package.cpath 获取，而这个变量则是通过环境变量 LUA_CPATH 来初始。

搜索的策略跟上面的一样，只不过现在换成搜索的是 so 或 dll 类型的文件。如果找得到，那么 require 就会通过 package.loadlib 来加载它。



### C包

Lua和C是很容易结合的，使用 C 为 Lua 写包。

与Lua中写包不同，C包在使用以前必须首先加载并连接，在大多数系统中最容易的实现方式是通过动态连接库机制。

Lua在一个叫loadlib的函数内提供了所有的动态连接的功能。这个函数有两个参数:库的绝对路径和初始化函数。所以典型的调用的例子如下:

```lua
local path = "/usr/local/lua/lib/libluasocket.so"
local f = loadlib(path, "luaopen_socket")
```

loadlib 函数加载指定的库并且连接到 Lua，然而它并不打开库（也就是说没有调用初始化函数），反之他返回初始化函数作为 Lua 的一个函数，这样我们就可以直接在Lua中调用他。

如果加载动态库或者查找初始化函数时出错，loadlib 将返回 nil 和错误信息。我们可以修改前面一段代码，使其检测错误然后调用初始化函数：

```lua
local path = "/usr/local/lua/lib/libluasocket.so"
-- 或者 path = "C:\\windows\\luasocket.dll"，这是 Window 平台下*
local f = assert(loadlib(path, "luaopen_socket"))
f() -- 真正打开库*
```

一般情况下我们期望二进制的发布库包含一个与前面代码段相似的 stub 文件，安装二进制库的时候可以随便放在某个目录，只需要修改 stub 文件对应二进制库的实际路径即可。

将 stub 文件所在的目录加入到 LUA_PATH，这样设定后就可以使用 require 函数加载 C 库了。



## 13、元表metatable

在 Lua table 中我们可以访问对应的 key 来得到 value 值，但是却无法对两个 table 进行操作(比如相加)。

因此 Lua 提供了元表(Metatable)，允许我们改变 table 的行为，每个行为关联了对应的元方法。

例如，使用元表我们可以定义 Lua 如何计算两个 table 的相加操作 a+b。

当 Lua 试图对两个表进行相加时，先检查两者之一是否有元表，之后检查是否有一个叫 **__add** 的字段，若找到，则调用对应的值。 **__add** 等即时字段，其对应的值（往往是一个函数或是 table）就是"元方法"。

有两个很重要的函数来处理元表：

- **setmetatable(table,metatable):** 对指定 table 设置元表(metatable)，如果元表(metatable)中存在 __metatable 键值，setmetatable 会失败。
- **getmetatable(table):** 返回对象的元表(metatable)。

```lua
mytable = {}                          -- 普通表
mymetatable = {}                      -- 元表
setmetatable(mytable,mymetatable)     -- 把 mymetatable 设为 mytable 的元表
```

也可直接写为一行

```lua
mytable = setmetatable({},{})
```

然后是返回对象的原表

```lua
getmetatable(mytable)                 -- 这会返回 mymetatable
```

### 13.1、_index元方法

这是 metatable 最常用的键。

当你通过键来访问 table 的时候，如果这个键没有值，那么Lua就会寻找该table的metatable（假定有metatable）中的`_index `键。如果`_index`包含一个表格，Lua会在表格中查找相应的键。

我们可以在使用 lua 命令进入交互模式查看：

```lua
$ lua
Lua 5.3.0  Copyright (C) 1994-2015 Lua.org, PUC-Rio
\> other = { foo = 3 }
\> t = setmetatable({}, { __index = other })
\> t.foo
3
\> t.bar
nil
```

如果__index包含一个函数的话，Lua就会调用那个函数，table和键会作为参数传递给函数。

_index 元方法查看表中元素是否存在，如果不存在，返回结果为 nil；如果存在则由 _index 返回结果。

```lua
mytable = setmetatable({key1 = "value1"}, {
  __index = function(mytable, key)
    if key == "key2" then
      return "metatablevalue"
    else
      return nil
    end
  end
})

print(mytable.key1,mytable.key2)
```

可以简写为

```lua
mytable = setmetatable({key1 = "value1"}, { __index = { key2 = "metatablevalue" } })
print(mytable.key1,mytable.key2)
```



### 13.2、_newindex元方法



### 13.3、为表添加操作符



### 13.4、_call元方法



### 13.5、_tostring元方法



## 14、协程coroutine

Lua 协同程序(coroutine)与线程比较类似：拥有独立的堆栈，独立的局部变量，独立的指令指针，同时又与其它协同程序共享全局变量和其它大部分东西。

### 线程和协同程序区别

线程与协同程序的主要区别在于，一个具有多个线程的程序可以同时运行几个线程，而协同程序却需要彼此协作的运行。

在任一指定时刻只有一个协同程序在运行，并且这个正在运行的协同程序只有在明确的被要求挂起的时候才会被挂起。

协同程序有点类似同步的多线程，在等待同一个线程锁的几个线程有点类似协同。

基本语法

```lua
coroutine.create()	--创建协程，返回corotine，参数为一个函数，和resume配合使用时唤醒函数的调用
coroutine.resume()	--重启协程，和create配合使用
coroutine.yield()	--挂起协程，将coroutine设置为挂起状态，可以和resume配合使用
coroutine.status()	--查看协程状态，共有三种：dead、suspended、running
coroutine.wrap()	--创建协程，返回函数，一旦调用该函数，就进入coroutine，和create功能重复
coroutine.running()	--返回正在运行的协程，一个coroutine就是一个线程，当running时，就是返回一个coroutine的线程号
```

例子：

```lua
-- coroutine_test.lua 文件
co = coroutine.create(
    function(i)
        print(i);
    end
)
 
coroutine.resume(co, 1)   -- 1
print(coroutine.status(co))  -- dead
 
print("----------")
 
co = coroutine.wrap(
    function(i)
        print(i);
    end
)
 
co(1)
 
print("----------")
 
co2 = coroutine.create(
    function()
        for i=1,10 do
            print(i)
            if i == 3 then
                print(coroutine.status(co2))  --running
                print(coroutine.running()) --thread:XXXXXX
            end
            coroutine.yield()
        end
    end
)
 
coroutine.resume(co2) --1
coroutine.resume(co2) --2
coroutine.resume(co2) --3
 
print(coroutine.status(co2))   -- suspended
print(coroutine.running())
 
print("----------")
```

输出

```lua
1
dead
----------
1
----------
1
2
3
running
thread: 0x7fb801c05868    false
suspended
thread: 0x7fb801c04c88    true
----------
```

coroutine.running就可以看出来,coroutine在底层实现就是一个线程。

当create一个coroutine的时候就是在新线程中注册了一个事件。

当使用resume触发事件的时候，create的coroutine函数就被执行了，当遇到yield的时候就代表挂起当前线程，等候再次resume触发事件。

接下来我们分析一个更详细的实例：

```lua
function foo (a)
    print("foo 函数输出", a)
    return coroutine.yield(2 * a) -- 返回  2*a 的值
end
 
co = coroutine.create(function (a , b)
    print("第一次协同程序执行输出", a, b) -- co-body 1 10
    local r = foo(a + 1)
     
    print("第二次协同程序执行输出", r)
    local r, s = coroutine.yield(a + b, a - b)  -- a，b的值为第一次调用协同程序时传入
     
    print("第三次协同程序执行输出", r, s)
    return b, "结束协同程序"                   -- b的值为第二次调用协同程序时传入
end)
       
print("main", coroutine.resume(co, 1, 10)) -- true, 4
print("--分割线----")
print("main", coroutine.resume(co, "r")) -- true 11 -9
print("---分割线---")
print("main", coroutine.resume(co, "x", "y")) -- true 10 end
print("---分割线---")
print("main", coroutine.resume(co, "x", "y")) -- cannot resume dead coroutine
print("---分割线---")
```

输出

```lua
第一次协同程序执行输出    1    10
foo 函数输出    2
main    true    4
--分割线----
第二次协同程序执行输出    r
main    true    11    -9
---分割线---
第三次协同程序执行输出    x    y
main    true    10    结束协同程序
---分割线---
main    false    cannot resume dead coroutine
---分割线---
```

以上实例接下如下：

- 调用resume，将协同程序唤醒,resume操作成功返回true，否则返回false；
- 协同程序运行；
- 运行到yield语句；
- yield挂起协同程序，第一次resume返回；（注意：此处yield返回，参数是resume的参数）
- 第二次resume，再次唤醒协同程序；（注意：此处resume的参数中，除了第一个参数，剩下的参数将作为yield的参数）
- yield返回；
- 协同程序继续运行；
- 如果使用的协同程序继续运行完成后继续调用 resume方法则输出：cannot resume dead coroutine

resume和yield的配合强大之处在于，resume处于主程中，它将外部状态（数据）传入到协同程序内部；而yield则将内部的状态（数据）返回到主程中。



### 生产者消费者问题

现在使用Lua的协同程序来完成生产者-消费者这一经典问题。

```lua
local newProductor

function productor()
     local i = 0
     while true do
          i = i + 1
          send(i)     -- 将生产的物品发送给消费者
     end
end

function consumer()
     while true do
          local i = receive()     -- 从生产者那里得到物品
          print(i)
     end
end

function receive()
     local status, value = coroutine.resume(newProductor)
     return value
end

function send(x)
     coroutine.yield(x)     -- x表示需要发送的值，值返回以后，就挂起该协同程序
end

-- 启动程序
newProductor = coroutine.create(productor)
consumer()
```

输出：

```lua
1
2
3
4
5
6
7
8
9
10
11
12
13
……
```



## 15、文件IO

Lua I/O 库用于读取和处理文件。分为简单模式（和C一样）、完全模式。

- 简单模式（simple model）拥有一个当前输入文件和一个当前输出文件，并且提供针对这些文件相关的操作。
- 完全模式（complete model） 使用外部的文件句柄来实现。它以一种面对对象的形式，将所有的文件操作定义为文件句柄的方法

简单模式在做一些简单的文件操作时较为合适。但是在进行一些高级的文件操作的时候，简单模式就显得力不从心。例如同时读取多个文件这样的操作，使用完全模式则较为合适。

打开文件操作语句如下：

```lua
file = io.open (filename [, mode])
```

mode的值有

| 模式 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| r    | 只读模式打开，文件必须存在                                   |
| w    | 打开只写文件，若文件存在则长度清0，该文件内容会消失。若文件不存在则创建文件 |
| a    | 以附加的方式打开只写文件。若文件不存在，则会建立该文件，如果文件存在，写入的数据会被加到文件尾，即文件原先的内容会被保留。（EOF符保留） |
| r+   | 以可读可写方式打开文件，该文件必须存在。                     |
| w+   | 打开可读写文件，若文件存在则文件长度清为零，即该文件内容会消失。若文件不存在则建立该文件。 |
| a+   | 与a类似，但此文件可读可写                                    |
| b    | 二进制模式，如果文件是二进制文件，可以加上b                  |
| +    | 表示对文件既可以读也可以写                                   |



### 15.1、简单模式

简单模式使用标准的 I/O 或使用一个当前输入文件和一个当前输出文件。

```lua
-- 以只读方式打开文件
file = io.open("test.lua", "r")

-- 设置默认输入文件为 test.lua
io.input(file)

-- 输出文件第一行
print(io.read())

-- 关闭打开的文件
io.close(file)

-- 以附加的方式打开只写文件
file = io.open("test.lua", "a")

-- 设置默认输出文件为 test.lua
io.output(file)

-- 在文件最后一行添加 Lua 注释
io.write("--  test.lua 文件末尾注释")

-- 关闭打开的文件
io.close(file)
```

在以上实例中我们使用了 io."x" 方法，其中 io.read() 中我们没有带参数，参数可以是下表中的一个：

| 模式         | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| "*n"         | 读取一个数字并返回它。例：`file.read("*n")`                  |
| "*a"         | 从当前位置读取整个文件。例：`file.read("*a")`                |
| "*l"（默认） | 读取下一行，在文件尾 (EOF) 处返回 nil。例：`file.read("*l")` |
| number       | 返回一个指定字符个数的字符串，或在 EOF 时返回 nil。例：`file.read(5)` |

其他的 io 方法有：

- **io.tmpfile():**返回一个临时文件句柄，该文件以更新模式打开，程序结束时自动删除
- **io.type(file):** 检测obj是否一个可用的文件句柄
- **io.flush():** 向文件写入缓冲中的所有数据
- **io.lines(optional file name):** 返回一个迭代函数，每次调用将获得文件中的一行内容，当到文件尾时，将返回 nil，但不关闭文件。



### 15.2、完全模式

通常我们需要在同一时间处理多个文件。我们需要使用 file:function_name 来代替 io.function_name 方法。以下实例演示了如何同时处理同一个文件:

```lua
-- 以只读方式打开文件
file = io.open("test.lua", "r")

-- 输出文件第一行
print(file:read())

-- 关闭打开的文件
file:close()

-- 以附加的方式打开只写文件
file = io.open("test.lua", "a")

-- 在文件最后一行添加 Lua 注释
file:write("--test")

-- 关闭打开的文件
file:close()
```

read 的参数与简单模式一致。

其他方法:

- **file:seek(optional whence, optional offset):** 设置和获取当前文件位置,成功则返回最终的文件位置(按字节),失败则返回nil加错误信息。参数 whence 值可以是:

  - "set": 从文件头开始
  - "cur": 从当前位置开始[默认]
  - "end": 从文件尾开始
  - offset:默认为0

  不带参数file:seek()则返回当前位置,file:seek("set")则定位到文件头,file:seek("end")则定位到文件尾并返回文件大小

- **file:flush():** 向文件写入缓冲中的所有数据

- **io.lines(optional file name):** 打开指定的文件 filename 为读模式并返回一个迭代函数，每次调用将获得文件中的一行内容，当到文件尾时，将返回 nil，并自动关闭文件。
  若不带参数时io.lines() <=> io.input():lines(); 读取默认输入设备的内容，但结束时不关闭文件，如：

  ```lua
  for line in io.lines("main.lua") do
  
  　　print(line)
  
  　　end
  ```

  以下实例使用了 seek 方法，定位到文件倒数第 25 个位置并使用 read 方法的 *a 参数，即从当前位置(倒数第 25 个位置)读取整个文件。

  ```lua
  -- 以只读方式打开文件
  file = io.open("test.lua", "r")
  
  file:seek("end",-25)
  print(file:read("*a"))
  
  -- 关闭打开的文件
  file:close()
  ```

  

## 16、错误处理





## 17、Debug



## 18、垃圾回收



## 19、面向对象



## 20、数据库访问
