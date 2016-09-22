# C# Best practices

# Prefer 'using' keyword when accessing native resources (file, DB connection, socket, mutex and so on)

:-1:__Bad practice__
```C#
var fs = new FileStream("Data.bin", FileMode.Create);
fs.Write(...);
fs.Write(...);
fs.Dispose(); <== 'fs' may not close if an exception occurs in above statements.
```
:+1:__Better practice__
```C#
using (var fs = new FileStream("Data.bin", FileMode.Create))
{
    fs.Write(...);
    fs.Write(...);
}

// Copy first 10 lines from one file to another file
using (var reader = new StreamReader("source file"))
using (var writer = new StreamWriter("dest file"))
{
    int lineNo = 0;
    while (lineNo++ < 10 && reader.Peek() > -1)
    {
        writer.WriteLine(reader.ReadLine());
    }
}
```

# Prefer 'as' operator when casting *reference* type

:-1:__Bad practice__
```C#
object obj = GetPeopleById(id);
Student s = (Student)obj;  <== throw exception if 'obj' cannot cast to Student type.
Console.Write(s.ClassName);
```
:+1:__Better practice__
```C#
object obj = GetPeopleById(id);
Student s = obj as Student;  <== return 'null' if 'obj' cannot cast to Student type.
if (s != null)
{
    Console.Write(s.ClassName);
}
```

The implementation of 'as' operator:
```C#
Student s = obj as Student;

is equivalent to:

Student s = obj is Student ? (Student)obj : (Student)null;
```

# Prefer LINQ over for/foreach
With LINQ, source code will be:
- Shorter, readable, 
- Focus on what you want than how to do
- Prevent unexpected side-effects to source collection

:-1:__Bad practice__
```C#
// Copy from Pronexus.IFRS project
public IfrsLinkDataInfo GetLinkData(int id)
{
    var bindingRangeInfo = ifrsBindingRangeInfoRepository.GetById(id);
    ....
    ....
    var linkInfo = new IfrsLinkDataInfo();

    List<string> lstRuleId = new List<string>();
    foreach (var bindingInfo in bindingRangeInfo.IFRS_BINDING_INFO)
    {
        lstRuleId.Add(bindingInfo.RULE_ID);
    }
    linkInfo.RULE_IDS = lstRuleId;

    List<string> lstSubRuleId = new List<string>();
    foreach (var bindingSubInfo in bindingRangeInfo.IFRS_BINDING_SUB_INFO)
    {
        lstSubRuleId.Add(bindingSubInfo.RULE_ID);
    }
    linkInfo.SUB_RULE_IDS = lstSubRuleId.Count > 0 ? lstSubRuleId : null;  <== This statement is different from 2 similar ones.

    List<int> lstIndexValue = new List<int>();
    foreach (var bindingGroup in bindingRangeInfo.IFRS_BINDING_GROUP)
    {
        lstIndexValue.Add(bindingGroup.PNX_INDEX_VALUE);
    }
    linkInfo.PNX_INDEX_VALUE = lstIndexValue;

    return linkInfo;
}

```
:+1:__Better practice__

```C#
// Source code after refactoring
public IfrsLinkDataInfo GetLinkData(int id)
{
    var bindingRangeInfo = ifrsBindingRangeInfoRepository.GetById(id);
    ....
    ....
    var linkInfo = new IfrsLinkDataInfo();
    linkInfo.RULE_IDS = bindingRangeInfo.IFRS_BINDING_INFO.Select(i => i.RULE_ID).ToList();
    linkInfo.SUB_RULE_IDS = bindingRangeInfo.IFRS_BINDING_SUB_INFO.Select(i => i.RULE_ID).ToList();
    linkInfo.PNX_INDEX_VALUE = bindingRangeInfo.IFRS_BINDING_GROUP.Select(i => i.PNX_INDEX_VALUE).ToList();

    return linkInfo;
}

```

# Const vs static readonly
__Const:__
 - Must be initialized
 - Initialization must be at compile time
 - Faster
 - Client assembly must re-compile if const value changes

When use:
 - Value never changes: Zero (0), Number days of a week (7), Int32.MaxValue (0x7FFFFFFF)
 - Modifier is: private, internal

__Static readonly:__
 - Can use default value, without initializing
 - Initialization can be at run time

 When use:
  - Condition to use const does not meet.

```C#
// In .NET Framework
public class string
{
    public static readonly string Empty;
    ....
}

```





