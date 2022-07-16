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
function add(...)  
local s = 0  
  for i, v in ipairs{...} do   --> {...} 表示一个由所有变长参数构成的数组  
    s = s + v  
  end  
  return s  
end  
print(add(3,4,5,6,7))  --->25
```

