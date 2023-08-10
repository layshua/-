

此样式指南适用于谷歌内部开发的 C# 代码，也是谷歌 C# 代码的默认样式。它的风格选择符合谷歌的其他语言，如谷歌 C++ 风格和谷歌 Java 风格。

## [格式化指南](#格式化指南)

### [命名规则](#命名规则)

命名规则如下 [微软的 C# 命名准则](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/naming-guidelines). 如果 Microsoft 的命名准则未指定（例如，私有变量和局部变量），则规则取自 [COREFX C# 编码指南](https://github.com/dotnet/runtime/blob/master/docs/coding-guidelines/coding-style.md)

规则摘要:

#### [代码](#代码)

*   类、方法、枚举、公共字段、公共属性的名称， namespaces: `PascalCase`.
*   局部变量的名称，参数:`camelCase`。
*   private、protected、internal 和 protected internal 字段的名称和 properties: `_camelCase`.
*   命名惯例不受 const、static、 readonly, etc.
*   对于 casing，“word” 指的是任何没有内部空格的文字，包括 acronyms. For example, `MyRpc` instead of ~`MyRPC`~.
*   接口的名称以 `I` 开头，例如 `IInterface`。

#### [档案](#档案)

*   文件名和目录名是 `PascalCase`，例如 `MyFile.cs`。
*   在可能的情况下，文件名应该与主文件名相同。 class in the file, e.g. `MyClass.cs`.
*   一般情况下，每个文件只有一个核心类。

### [组织](#组织)

*   修饰语按以下顺序排列:“公共保护的内部私人” new abstract virtual override sealed static readonly extern unsafe volatile async`.
*   名称空间 `using` 声明位于顶部，位于任何名称空间之前。`using` import order is alphabetical, apart from `System` imports which always go first.
*   类成员订购:
    *   按以下顺序分组类成员:
        *   嵌套的类，枚举，代表和事件。
        *   静态、const 和 readonly 字段。
        *   字段和属性。
        *   构造者和终结者。
        *   方法。
    *   在每个组中，元素应按以下顺序排列:
        *   公开。
        *   内部。
        *   内部保护。
        *   受保护。
        *   私人。
    *   在可能的情况下，将接口实现组合在一起。

### [空格规则](#空格规则)

从谷歌 Java 风格发展而来。

*   每行最多一条语句。
*   每个语句最多一个赋值。
*   两个空格的缩进，没有选项卡。
*   列限制: 100。
*   在打开支架之前，没有折线。
*   在闭合大括号和 `else` 之间没有折线。
*   即使在可选的情况下也要使用大括号。
*   在 `if`/`for`/`while’等之后加上空格，在逗号之后加上空格。
*   在开括号之后或闭括号之前没有空格。
*   一元运算符与其操作数之间没有空格。之间的一个空间 operator and each operand of all other operators.
*   线条包装是从谷歌 C++ 风格的指导方针发展而来的，有一些次要的 modifications for compatibility with Microsoft's C# formatting tools:
    *   一般情况下，行连续是缩进的 4 个空格。
    *   带大括号的换行符（例如，列表初始化器、lambdas、对象） initializers, etc) do not count as continuations.
    *   对于函数定义和调用，如果参数不完全符合 one line they should be broken up onto multiple lines, with each subsequent line aligned with the first argument. If there is not enough room for this, arguments may instead be placed on subsequent lines with a four space indent. The code example below illustrates this.

### [例子](#例子)

```
using System;                                       // `using` goes at the top, outside the
                                                    // namespace.

namespace MyNamespace {                             // Namespaces are PascalCase.
                                                    // Indent after namespace.
  public interface IMyInterface {                   // Interfaces start with 'I'
    public int Calculate(float value, float exp);   // Methods are PascalCase
                                                    // ...and space after comma.
  }

  public enum MyEnum {                              // Enumerations are PascalCase.
    Yes,                                            // Enumerators are PascalCase.
    No,
  }

  public class MyClass {                            // Classes are PascalCase.
    public int Foo = 0;                             // Public member variables are
                                                    // PascalCase.
    public bool NoCounting = false;                 // Field initializers are encouraged.
    private class Results {
      public int NumNegativeResults = 0;
      public int NumPositiveResults = 0;
    }
    private Results _results;                       // Private member variables are
                                                    // _camelCase.
    public static int NumTimesCalled = 0;
    private const int _bar = 100;                   // const does not affect naming
                                                    // convention.
    private int[] _someTable = {                    // Container initializers use a 2
      2, 3, 4,                                      // space indent.
    }

    public MyClass() {
      _results = new Results {
        NumNegativeResults = 1,                     // Object initializers use a 2 space
        NumPositiveResults = 1,                     // indent.
      };
    }

    public int CalculateValue(int mulNumber) {      // No line break before opening brace.
      var resultValue = Foo * mulNumber;            // Local variables are camelCase.
      NumTimesCalled++;
      Foo += _bar;

      if (!NoCounting) {                            // No space after unary operator and
                                                    // space after 'if'.
        if (resultValue < 0) {                      // Braces used even when optional and
                                                    // spaces around comparison operator.
          _results.NumNegativeResults++;
        } else if (resultValue > 0) {               // No newline between brace and else.
          _results.NumPositiveResults++;
        }
      }

      return resultValue;
    }

    public void ExpressionBodies() {
      // For simple lambdas, fit on one line if possible, no brackets or braces required.
      Func<int, int> increment = x => x + 1;

      // Closing brace aligns with first character on line that includes the opening brace.
      Func<int, int, long> difference1 = (x, y) => {
        long diff = (long)x - y;
        return diff >= 0 ? diff : -diff;
      };

      // If defining after a continuation line break, indent the whole body.
      Func<int, int, long> difference2 =
          (x, y) => {
            long diff = (long)x - y;
            return diff >= 0 ? diff : -diff;
          };

      // Inline lambda arguments also follow these rules. Prefer a leading newline before
      // groups of arguments if they include lambdas.
      CallWithDelegate(
          (x, y) => {
            long diff = (long)x - y;
            return diff >= 0 ? diff : -diff;
          });
    }

    void DoNothing() {}                             // Empty blocks may be concise.

    // If possible, wrap arguments by aligning newlines with the first argument.
    void AVeryLongFunctionNameThatCausesLineWrappingProblems(int longArgumentName,
                                                             int p1, int p2) {}

    // If aligning argument lines with the first argument doesn't fit, or is difficult to
    // read, wrap all arguments on new lines with a 4 space indent.
    void AnotherLongFunctionNameThatCausesLineWrappingProblems( int longArgumentName, int longArgumentName2, int longArgumentName3) {}

    void CallingLongFunctionName() {
      int veryLongArgumentName = 1234;
      int shortArg = 1;
      // If possible, wrap arguments by aligning newlines with the first argument.
      AnotherLongFunctionNameThatCausesLineWrappingProblems(shortArg, shortArg,
                                                            veryLongArgumentName);
      // If aligning argument lines with the first argument doesn't fit, or is difficult to
      // read, wrap all arguments on new lines with a 4 space indent.
      AnotherLongFunctionNameThatCausesLineWrappingProblems(
          veryLongArgumentName, veryLongArgumentName, veryLongArgumentName);
    }
  }
}
```

## [C# 编码指南](#c-编码指南)

### [常数](#常数)

*   可以设置 `const` 的变量和字段应该始终设置 `const`。
*   如果 `const` 不可能，`readonly` 可以是一个合适的选择。
*   与幻数相比，更喜欢命名常数。

### [iEnumerable VS iList VS iReadOnlyList](#ienumerable-vs-ilist-vs-ireadonlylist)

*   对于输入，可以使用最严格的集合类型，例如 `IReadOnlyCollection` / `IReadOnlyList` / `IEnumerable` as inputs to methods when the inputs should be immutable.
*   对于输出，如果将返回的容器的所有权传递给所有者， prefer `IList` over `IEnumerable`. If not transferring ownership, prefer the most restrictive option.

### [发电机 VS 容器](#发电机-vs容器)

*   运用你最好的判断，记住:
    *   生成器代码的可读性通常不如填写容器。
    *   如果结果是正确的，那么生成器代码的性能会更好。 processed lazily, e.g. when not all the results are needed.
    *   通过 `ToList()` 直接转换成容器的生成器代码 will be less performant than filling in a container directly.
    *   多次调用的生成器代码将会慢得多。 than iterating over a container multiple times.

### [属性样式](#属性样式)

*   对于单行只读属性，首选表达式主体属性 (`=>`) when possible.
*   对于其他所有操作，请使用较旧的 `{ get; set; }` 语法。

### [表达式体语法](#表达式体语法)

例如:

```
int SomeProperty => _someProperty
```

*   在 lambdas 和 properties 中明智地使用表达式体语法。
*   不要使用方法定义。这将在 C#7 直播时进行审查， which uses this syntax heavily.
*   与方法和其他范围代码块一样，将结束部分与 first character of the line that includes the opening brace. See sample code for examples.

### [结构和类:](#结构和类)

*   结构与类有很大的不同:
    
    *   结构总是按值传递和返回的。
    *   向返回的结构的成员赋值不会修改 original - e.g. `transform.position.x = 10` doesn’t set the transform’s position.x to 10; `position` here is a property that returns a `Vector3` by value, so this just sets the x parameter of a copy of the original.
*   几乎总是使用一个类。
    
*   当类型可以像对待其他值类型一样对待时，考虑 struct-for example, if instances of the type are small and commonly short-lived or are commonly embedded in other objects. Good examples include Vector3, Quaternion and Bounds.
    
*   请注意，这个指导可能会因团队而异，例如， performance issues might force the use of structs.
    

### [lambdas 与命名方法](#lambdas-与命名方法)

*   如果 lambda 是非平凡的（例如，多于几个语句，排除 declarations), or is reused in multiple places, it should probably be a named method.

### [字段初始化器](#字段初始化器)

*   通常鼓励使用字段初始化器。

### [扩展方法](#扩展方法)

*   只有在原始类的源不是时才使用扩展方法 available, or else when changing the source is not feasible.
*   只有在添加的功能是 “核心” 的情况下，才使用扩展方法 general feature that would be appropriate to add to the source of the original class.
    *   注意 - 如果我们有要扩展的类的源，并且 maintainer of the original class does not want to add the function, prefer not using an extension method.
*   只将扩展方法放入可用的核心库中 everywhere - extensions that are only available in some code will become a readability issue.
*   请注意，使用扩展方法总是会混淆代码，因此很容易出错。 the side of not adding them.

### [裁判和裁判](#裁判和裁判)

*   对于不是输入的返回，使用 `out`。
*   将 `out` 参数放在方法定义中所有其他参数之后。
*   当需要修改输入时，应该很少使用 `ref`。
*   不要使用 `ref` 作为传递结构的优化。
*   不要使用 `ref` 将可修改的容器传递到方法中。`ref` 仅为 required when the supplied container needs be replaced with an entirely different container instance.

### [林克](#林克)

*   一般来说，更喜欢单行 LINQ 调用和命令式代码，而不是 long chains of LINQ. Mixing imperative code and heavily chained LINQ is often hard to read.
*   与 SQL 风格的 LINQ 关键字相比，更喜欢成员扩展方法 - 例如，更喜欢 `myList.Where(x)` to `myList where x`.
*   避免 `Container.ForEach(...)` 超过一条语句的时间。

### [数组 VS 列表](#数组-vs-列表)

*   一般来说，对于公共变量、属性，更喜欢 `List<>` 而不是数组， and return types (keeping in mind the guidance on `IList` / `IEnumerable` / `IReadOnlyList` above).
*   当容器的大小可以改变时，首选 `List<>`。
*   当容器的大小是固定的并且已知在容器时，更喜欢数组 construction time.
*   对于多维数组，首选数组。
*   注:
    *   数组和 `List<>` 都表示线性的、连续的容器。
    *   与 C++ 阵列 VS`std::vector` 类似，阵列的容量是固定的， whereas `List<>` can be added to.
    *   在某些情况下，数组的性能更好，但通常 `List<>` 是 more flexible.

### [文件夹和文件位置](#文件夹和文件位置)

*   与项目保持一致。
*   在可能的情况下，最好是平面结构。

### [将 tuple 用作返回类型](#将-tuple-用作返回类型)

*   通常，比起 `Tuple<>`，更喜欢有名称的类类型，尤其是在以下情况下 returning complex types.

### [字符串插值 vs`String.Format()`vs`String.Concat`vs`operator+`](#字符串插值-vsstringformatvsstringconcatvsoperator)

*   通常，使用最容易阅读的内容，尤其是在日志和 assert messages.
*   请注意，链接 `operator+` 的连接将会更慢，并导致 significant memory churn.
*   如果性能是一个问题，`StringBuilder` 将是更快的倍数 string concatenations.

### [`using`](#using)

*   通常情况下，不要别名为 `using` 的长字体。这通常是一种征兆。 that a `Tuple<>` needs to be turned into a class.
    *   例如 `using RecordList = List<Tuple<int, float>>` 可能应该是一个 named class instead.
*   请注意，`using` 语句仅受文件范围的限制，因此使用范围有限。 Type aliases will not be available for external users.

### [对象初始化器语法](#对象初始化器语法)

例如:

```
var x = new SomeClass {
  Property1 = value1,
  Property2 = value2,
};
```

*   对象初始化器语法适合于 “纯旧数据” 类型。
*   避免对具有构造函数的类或结构使用此语法。
*   如果在多行之间分割，则缩进一个块级别。

### [命名空间命名](#命名空间命名)

*   一般来说，命名空间的深度不应超过 2 个级别。
*   不要强迫文件 / 文件夹布局与名称空间匹配。
*   对于共享的库 / 模块代码，使用名称空间。对于叶子的 “应用程序” 代码， such as `unity_app`, namespaces are not necessary.
*   新的顶级名称空间名称必须是全局唯一且可识别的。

### [结构的默认值 /null 返回](#结构的默认值-null-返回)

*   希望返回一个 “成功” 布尔值和一个 struct`out` 值。
    
*   在这种情况下，性能并不是一个问题，其产生的代码要比其他代码多得多 readable (e.g. chained null conditional operators vs deeply nested if statements) nullable structs are acceptable.
    
*   注:
    
    *   可空结构很方便，但要加强一般的 “null is” failure’ pattern Google prefers to avoid. We will investigate a `StatusOr` equivalent in the future, if there is enough demand.

### [在迭代时从容器中删除](#在迭代时从容器中删除)

C#（像许多其他语言一样）没有提供一种明显的机制来在迭代时从容器中删除项。有几种选择:

*   如果所需要的只是删除满足某些条件的项目， `someList.RemoveAll(somePredicate)` is recommended.
*   如果需要在迭代中完成其他工作，`RemoveAll` 可能不是 sufficient. A common alternative pattern is to create a new container outside of the loop, insert items to keep in the new container, and swap the original container with the new one at the end of iteration.

### [召集代表](#召集代表)

*   调用委托时，使用 `Invoke()` 并使用空条件 operator - e.g. `SomeDelegate?.Invoke()`. This clearly marks the call at the callsite as ‘a delegate that is being called’. The null check is concise and robust against threading race conditions.

### [`var` 关键字](#var-关键字)

*   如果使用 `var` 可以通过避免类型名来提高可读性，则鼓励使用它。 that are noisy, obvious, or unimportant.
    
*   鼓励:
    
    *   当类型很明显时 - 例如 `var apple = new Apple();`，或 `var request = Factory.Create<HttpRequest>();`
    *   对于仅直接传递给其他方法的瞬态变量 - e.g. `var item = GetItem(); ProcessItem(item);`
*   不鼓励:
    
    *   使用基本类型时 - 例如 `var success = true;`
    *   当使用编译器时 - 解析的内置数字类型 - 例如 `var number = 12 * ReturnsFloat();`
    *   当用户清楚地知道类型 -- 例如 `var’--将会受益的时候 listOfItems = GetList();`

### [属性](#属性)

*   属性应该出现在字段、属性或方法上面的一行中 they are associated with, separated from the member by a newline.
*   多个属性应该用换行符分隔。这就更容易了。 adding and removing of attributes, and ensures each attribute is easy to search for.

### [参数命名](#参数命名)

源自 Google C++ 样式指南。

当函数论点的含义不明显时，考虑以下补救措施之一:

*   如果参数是一个字面常量，并且相同的常量用于 multiple function calls in a way that tacitly assumes they're the same, use a named constant to make that constraint explicit, and to guarantee that it holds.
*   考虑更改函数签名，将 `bool` 参数替换为 an `enum` argument. This will make the argument values self-describing.
*   用命名变量替换大型或复杂的嵌套表达式。
*   考虑使用 [Named Arguments](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/named-and-optional-arguments) to clarify argument meanings at the call site.
*   对于具有多个配置选项的函数，考虑定义一个 single class or struct to hold all the options and pass an instance of that. This approach has several advantages. Options are referenced by name at the call site, which clarifies their meaning. It also reduces function argument count, which makes function calls easier to read and write. As an added benefit, call sites don't need to be changed when another option is added.

考虑以下示例:

```
// Bad - what are these arguments?
DecimalNumber product = CalculateProduct(values, 7, false, null);
```

相对于:

```
// Good
ProductOptions options = new ProductOptions();
options.PrecisionDecimals = 7;
options.UseCache = CacheUsage.DontUseCache;
DecimalNumber product = CalculateProduct(values, options, completionDelegate: null);
```