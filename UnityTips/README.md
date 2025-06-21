# Unity 开发技巧
## 命名
```csharp
// 不建议使用拼音命名
public void GongJi();

// 使用英文命名，且具有描述性
public void Attack();

// 不建议使用单个字母等模糊的参数名
public void Attack(int i);

// 参数名称应具有描述性
public void Attack(int damage);
```
```csharp
// 私有字段加前缀，提高可读性
private ulong _data; // .NET
private ulong m_Data; // Unity

// 避免与参数或局部变量同名，防止混淆
public void UpdateData(ulong data) => _data = data;
```
## 访问修饰符
```csharp
// 将需要公共访问的成员设为 public
public bool TryMove(Vector3 position)
{
    if (!ValidatePosition(position))
        return false;

    MoveTo(position);
    return true;
}

// 不需要公共访问的成员设为 protected
protected void MoveTo(Vector3 position);

protected bool ValidatePosition(Vector3 position);
```
```csharp
// 在 Unity 中使用 SerializeField 特性来标记需要序列化的字段
[SerializeField]
private Sprite _sprite;

// 使用公共属性访问私有字段，避免直接暴露字段
public Sprite Sprite => _sprite;
```
## 方法
```csharp
public Result Attack(int damage)
{
    // 过深的嵌套会降低代码可读性
    if (damage >= 0)
    {
        if (damage > 10)
        {
            if (damage > 50)
            {
                // Process high damage
                return Result.HighDamage;
            }
            else
            {
                // Process medium damage
                return Result.MedimuDamage;
            }
        }
        else
        {
            // Process low damage
            return Result.LowDamage;
        }
    }
    else
        throw new("Damage cannot be negative.");
}
```
```csharp
public Result Attack(int damage)
{
    // 尽早返回，减少嵌套
    if (damage < 0)
        throw new("Damage cannot be negative.");

    // 拆分提取方法，提高可读性
    if (damage < 10)
        return ProcessLowDamage(damage);

    if (damage < 50)
        return ProcessMediumDamage(damage);
    
    return ProcessHighDamage(damage);
}

protected Result ProcessLowDamage(int damage);

protected Result ProcessMediumDamage(int damage);

protected Result ProcessHighDamage(int damage);
```
## 类
```csharp
// 一个类不负责过多职务，避免出现上帝类
public abstract class GameManager
{
    public abstract Game LoadGame();

    public abstract bool CheckGame(Game game);
}
```
```csharp
public class GameManager
{
    // 使用依赖注入来解耦
    private IGameLoader _gameLoader;
    // 通过接口调用，无需关注具体实现（例如特殊模式，策略等）
    private IGameChecker _gameChecker;

    public GameManager(IGameLoader gameLoader, IGameChecker gameChecker)
    {
        _gameLoader = gameLoader;
        _gameChecker = gameChecker;
    }
}

public interface IGameLoader
{ 
    public Game Load();
}

public interface IGameChecker
{
    public bool Check(Game game);
}
```
## 注释
```csharp
/// <summary>
/// 加载指定索引的关卡
/// </summary>
/// <param name="levelIndex">关卡索引，从 1 开始计数</param>
/// <returns>关卡实例</returns>
public Level LoadLevel(int levelIndex);
```
![图片1](Images/Image_1.png)
![图片2](Images/Image_2.png)
## 可为 null 的引用类型
```csharp
// 在引用类型后加 ? 表示可空引用类型，允许值为 null
public bool Check(Game? game);
```
![图片3](Images/Image_3.png)
```csharp
// 无 ? 的引用类型不允许值为 null
public bool Check(Game game);
```
![图片4](Images/Image_4.png)
```xml
<!-- 在 Unity 中，创建 Directory.Build.props 文件 -->
<Project>
  <PropertyGroup>
    <Nullable>enable</Nullable>
  </PropertyGroup>
</Project>
```
```csharp
// 或在需要的 cs 文件中添加编译指令
#nullable enable
```
## Unity 中的 null 判断
Unity 重载了 == 和 != 运算符，native 对象被销毁后，托管的对象可能仍然存在并不为 null。因此不能使用 null 条件运算符进行 null 判断。
```csharp
public static bool operator ==(Object x, Object y) => Object.CompareBaseObjects(x, y);

public static bool operator !=(Object x, Object y) => !Object.CompareBaseObjects(x, y);
```
![图片5](Images/Image_5.png)
## 格式化字符串
```csharp
// 不同的区域信息会影响数字和日期的格式化
CultureInfo.CurrentCulture = new("fr-FR");
var str = number.ToString();
Console.WriteLine($"str: {str}");

CultureInfo.CurrentCulture = new("en-US");
Console.WriteLine($"number: {double.Parse(str)}");
```
```
str: 123,4  
number: 1234
```
```csharp
CultureInfo.CurrentCulture = new("fr-FR");
// 使用 InvariantCulture 来确保格式化的一致性
var str = number.ToString(CultureInfo.InvariantCulture);
Console.WriteLine($"str: {str}");

CultureInfo.CurrentCulture = new("en-US");
Console.WriteLine($"number: {double.Parse(str, CultureInfo.InvariantCulture)}");
```
```
str: 123.4  
number: 123.4
```
## 魔法数字
在编程中，硬编码到代码文件中的数字通常被称为“魔法数字”（Magic Numbers）。

这些是直接写在代码中的具体数值，没有用具名的常量或变量表示。虽然它们在短期内可能看起来方便，但在长期维护中往往会带来问题，因为数字的意义可能对阅读代码的人来说不清晰。例如：
```csharp
if (damage <= 10)
{
    // Process
}
```
在这里，10 就是一个“魔法数字”。更好的做法是使用具名常量来提高可读性和可维护性：
```csharp
public const int LowDamageLimit = 10;
```
```csharp
if (damage <= LowDamageLimit)
{
    // Process
}
```
通过这种方式，如果将来业务规则变动（比如低伤害上限为 20），你只需要改一个地方。
## 条件编译
条件编译能让一份代码灵活地适配不同配置（调试/发布、不同框架、功能开关）、在发布版剔除调试日志和断言、减小体积并消除无谓性能损耗，也可打造灰度发布、多目标构建等复杂场景。

用预处理指令的“条件编译”相比代码里写一个 `const bool`，表面上都能在不同配置下控制“这段代码要不要跑”，但条件编译可以控制例如引用程序集、添加特性等。

项目通常会有测试版和正式版，可以使用例如 `TEST_MODE` 编译符号定义只在测试模式下有效的方法
```csharp
#if TEST_MODE
public void TestMethod() { }
#endif
```
在 Unity 中，使用编辑器脚本调用 `PlayerSettings.SetScriptingDefineSymbols` 自动设置编译符号。
## 包管理
openupm

nuget
## 编辑器拓展
odin
## 资源加载
addressables
## 异步编程
unitask