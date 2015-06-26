# UnityEngine.Reflection

A set of Unity classes and their inspector drawers that provide easy reflection and decoupling.

This editor extension provides 2 new classes, `UnityVariable` and `UnityMethod`, along with their custom drawers, that behave like Unity's built-in `UnityEvent` for fields, properties and methods reflection.

With them, you can easily refer to members of `Unity.Object` classes directly in the inspector and use them in scripting later. This allows for quick prototyping and decoupling for complex games and applications (albeit at a small performance cost).

### Say what?

Ugh. Here's a picture:

![Explanation](http://i.imgur.com/Tltom7f.png)

### Features

- Inspect fields, properties or methods
- Serialized for persistency across reloads
- Works on GameObjects and ScriptableObjects
- Multi-object editing
- Undo / Redo support
- Looks and feels like a built-in Unity component

### Limitations

- Doesn't support method overloading (yet)

## Installation

Import the `Assets/Reflection` folder in your project and you're good to go!

## Usage

1. Create a behaviour script
2. Add `using UnityEngine.Reflection;` to your namespaces.
2. Use `UnityVariable` or `UnityMethod` as public behaviour members
3. Set your bindings in the inspector
4. Access the variables directly from your script!

Here's a simple example that will display the value of any method and any variable on start:
```csharp
using UnityEngine;
using UnityEngine.Reflection;

public class BasicExample : MonoBehaviour
{
	public UnityMethod method;

	public UnityVariable variable;

	void Start()
	{
		Debug.LogFormat("Method return: {0}", method.Invoke());
		Debug.LogFormat("Variable value: {0}", variable.Get());
	}
}
```

### Basic Usage
There are 3 commonly used methods to deal with reflected members. Each are described below.

`Get` and `Invoke` also have typed generic equivalents that will attempt a cast.

#### UnityVariable.Get

```csharp
object UnityVariable.Get()
```

Retrieves the value of the variable.

#### UnityVariable.Set

```csharp
void UnityVariable.Set(object value)
```

Assigns a new value to the variable.

#### UnityMethod.Invoke

```csharp
object UnityMethod.Invoke(params object[] args)
```

Invokes the method with any number of arguments of any type and returns its return value, or null if there isn't any (void).

---

You can also get the type of the reflected member using the following shortcuts:

#### UnityVariable.variableType

```csharp
Type UnityVariable.variableType { get; }
```

The field or property type of the reflected variable.

#### UnityMethod.returnType

```csharp
Type UnityMethod.returnType { get; }
```

The return type of the reflected method.

### Advanced Usage

#### Self-Targeting

You can tell the inspector to look on the current object instead of manually specifying one by adding the `[SelfTargeted]` attribute. For example:

```csharp
using UnityEngine;
using UnityEngine.Reflection;

public class AdvancedExample : MonoBehaviour
{
	[SelfTargeted]
	public UnityVariable selfVariable;
}
```

#### Member Filtering

You can specify which members will appear in the inspector using the `Filter` attribute. You can combine a number of options to display only the members you want. For example:

```csharp
using UnityEngine;
using UnityEngine.Reflection;

public class AdvancedExample : MonoBehaviour
{
    // Only show variables of type Transform
    [Filter(typeof(Transform))]
    public UnityVariable transformVariable;

    // Only show methods that return an integer or a float
    [Filter(typeof(int), typeof(float))]
    public UnityMethod numericMethod;

    // Only show methods that return primitives or enums
    [Filter(TypeFamilies = TypeFamily.Primitive | TypeFamily.Enum)]
    public UnityMethod primitiveOrEnumMethod;

	// Only show static methods
	[Filter(Static = true, Instance = false)]
	public UnityMethod staticMethod;

	// Include non-public variables
    [Filter(NonPublic = true)]
	public UnityVariable hiddenVariable;

    // Exclude readonly properties
    [Filter(ReadOnly = false)]
    public UnityVariable writableVariable;

    // Only show methods that are on the defined on the object itself
    [Filter(Inherited = false)]
    public UnityMethod definedMethod;

    // Combine any of the above options
    [Filter(typeof(Collider), Static = true, ReadOnly = false)]
    public UnityVariable colliderVariable;
}
```

The available options are:

Option		| Description | Default
------------|-------------|--------
Inherited	|Display members defined in the types's ancestors|true
Instance	|Display instance members|true
Static		|Display static members|false
Public		|Display public members|true
NonPublic	|Display private and protected members|false
ReadOnly	|Display read-only properties and fields|true
WriteOnly	|Display write-only properties and fields|true
TypeFamilies|Determines which member type families are displayed|TypeFamily.All
Types		|Determines which member types are displayed|*(Any)*


The `TypeFamilies` enumeration is a [bitwise flag set](http://stackoverflow.com/questions/8447/what-does-the-flags-enum-attribute-mean-in-c) with the following options:

Flag		|Description
------------|-----------
*None*		|No type allowed
*All*		|Any type allowed
Value		|Value types only
Reference	|Reference types only
Primitive	|Primitive types only
Array		|Arrays only
Enum		|Enumerations only
Class		|Classes only
Interface	|Interfaces only

You can combine them with the bitwise or operator:

```csharp
TypeFamily enumsOrInterfaces = TypeFamily.Enum | TypeFamily.Interface;
```

#### Overriding Defaults

You can override the defaults by editing the inspector drawer classes and modifying the `DefaultFilter()` method.

- For variables: `Reflection/Editor/UnityVariableDrawer.cs`
- For methods: `Reflection/Editor/UnityMethodDrawer.cs`

For example, if you wanted to make non-public variables show up by default (without having to specify it with a `Reflection` attribute), you could add the following line:

```csharp
using System.Reflection;
using UnityEditor;

namespace UnityEngine.Reflection
{
	[CustomPropertyDrawer(typeof(UnityVariable))]
	public class UnityVariableDrawer : UnityMemberDrawer
	{
		protected override FilterAttribute DefaultFilter()
		{
			FilterAttribute filter = base.DefaultFilter();

			// Override defaults here
            filter.NonPublic = true;

			return reflection;
		}

        ...
    }
}
```

### Direct Access

If you want to directly access the `System.Reflection` objects, you can do so using the following properties. Note that you must previously have reflected the member, either manually via `UnityMember.Reflect()`, or automatically by accessing / invoking it.

#### UnityVariable.fieldInfo

```csharp
FieldInfo UnityVariable.fieldInfo { get; }
```

The underlying reflected field, or null if the variable is a property.

#### UnityVariable.propertyInfo

```csharp
PropertyInfo UnityVariable.propertyInfo { get; }
```

The underlying reflected property, or null if the variable is a field.

#### UnityMethod.methodInfo

```csharp
MethodInfo UnityVariable.methodInfo { get; }
```

The underlying reflected method.

## Contributing

I'll happily accept pull requests if you have improvements or fixes to suggest.

### To-do

- Method overloading (requires method signature distinction and serialization)
- Figure out a way to make the member dropdown less verbose (like `UnityEvent`'s)

##  License

The whole source is under MIT License, which basically means you can freely use and redistribute it in your commercial and non-commercial projects. See [the license file](LICENSE) for the boring details.

If you use it in a plugin that you redistribute, please the namespaces to avoid version conflicts with your users. For example, change `UnityEngine.Reflection` to `MyPlugin.UnityEngine.Reflection`.
