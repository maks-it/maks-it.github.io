# Adapter

Adapter is a structural design pattern, which allows to adapt incompatible objects.

The Adapter is a wrapper between two objects. His objective is to catch calls for one object and transform them to recognizable format and interface by the second object.

**Complexity**: 1/3

**Popularity**: 3/3

**Usage examples**: The Adapter pattern is very popular in C# code. It’s often used in systems based on some legacy code. In these cases, Adapters make legacy code work with new classes.

**Identification**: Adapter is recognizable by a constructor which takes an instance of a different abstract/interface type. When the adapter receives a call to any of its methods, it translates parameters to the appropriate format and then directs the call to one or several methods of the wrapped object.

# Code Example

This example illustrates the structure of the Adapter design pattern. It focuses on answering these questions:

What classes does it consist of?

What roles do these classes play?

In what way the elements of the pattern are related?

**Adaptee.cs**

```c#
using System;

namespace Example.Adapter;

public class IAdaptee {
    string GetSpecificRequest();
}

// The Adaptee contains some useful behavior, but its interface is
// incompatible with the existing client code. The Adaptee needs some
// adaptation before the client code can use it.
public class Adaptee : IAdaptee {
    public string GetSpecificRequest() {
        return "Specific request.";
    }
}
```

**Adapter.cs**

```c#
using System;

namespace Example.Adapter;

// The Target defines the domain-specific interface used by the client code.
public interface IAdapter {
    string GetRequest();
}

// The Adapter makes the Adaptee's interface compatible with the Target's
// interface.
public class Adapter : IAdapter {
    private readonly IAdaptee _adaptee;

    public Adapter (
        IAdaptee adaptee
    ) {
        _adaptee = adaptee;
    }

    public string GetRequest() {
        return $"This is '{_adaptee.GetSpecificRequest()}'";
    }
}
```

**Program.cs**

```c#
using System;

namespace Example.Adapter;

class Program {
    static void Main(string[] args) {
        var adaptee = new Adaptee();
        ITarget target = new Adapter(adaptee);

        Console.WriteLine("Adaptee interface is incompatible with the client.");
        Console.WriteLine("But with adapter client can call it's method.");

        Console.WriteLine(target.GetRequest());
    }
}
```

**Console output**

```bash
Adaptee interface is incompatible with the client.
But with adapter client can call it's method.
This is 'Specific request.'
```