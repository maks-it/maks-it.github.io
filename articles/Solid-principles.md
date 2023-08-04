 
# SOLID Principles

The SOLID principles are a programming standard that all developers must understand well,
to avoid creating bad architecture. This standard is widely applied in OOP.
Used correctly, it makes your code more extensible, logical, and readable.
When a developer builds a poor architecture application, the code is inflexible.
even small changes in it can lead to bugs. Therefore, you need to follow the SOLID principles.
It will take some time to master them if you write code in accordance with these principles,
it will improve and you will learn how to create a good software architecture.
To understand the SOLID principles, you need to have a clear understanding of how to use Interfaces.
If you do not have such understanding, then first read the documentation.
I will explain SOLID in the simplest way, so it will be easier for beginners to figure it out.
Let's look at the principles one by one.

## Single Responsibility Principle
There is only one reason which leads to the class change.
One class should only solve one problem. It can have several methods,
but all have to solve a common problem.
All methods and properties should serve the same purpose. If the class has multiple purposes,
it have to be splited into separate classes.

**Consider an example:**

```c#
namespace Report {
  public class OrdersReport {
    public string GetOrdersInfo (DateTime startDate, DateTime endDate) {
      var orders = QueryOrders (startDate, endDate);

      return FormatOrders (orders);
    }

    // If we update our persistence level in the future,
    // and here we have to make changes. <=> reason to change!
    private DataTable QueryOrders (DateTime startDate, DateTime endDate) {
      var table = new DataTable ();
      using var da = new SqlDataAdapter ("SELECT ID FROM orders", "connection string");
          da.Fill (table);

      return table;
    }
    
    // If we changed the way we format the output,
    // we need to make changes here. <=> reason to change!
    private string FormatOrders (DataTable) {
      var sb = new StringBuilder ("<ul>");

      foreach (var row in table.Rows) {
        sb.Append ($ "<li> {row [" id "]} </li>");
      }

      sb.Append ("</ul>");

      return sb.ToString ();
    }
  }
}
```

The class shown here violates the single responsibility principle.
Why should he fetch data from the database? This is a task for the storage layer,
on which data is saved and retrieved from storage (e.g. a database).
This is not the responsibility of this class.
Also, the class should not be responsible for the format of this method,
because we may need data in a different format like XML, JSON, HTML, etc.

**The code after refactoring will look like this:**

```c#
namespace Report {
  public class OrdersReport {

    private readonly IOrdersRepository _repository;
    private readonly IOrdersFormatter _formatter;
    public OrdersReport(IOrdersRepository repository, IOrdersFormatter formatter) {
      _repository = repository;
      _formatter = formatter;
    }

    public string GetOrdersInfo (DateTime startDate, DateTime endDate) {
      var orders = _repository.QueryOrders(startDate, endDate);
      return _formatter.Output(orders);
    }    
  }
}

namespace Report {
  public interface IOrdersFormatter {
    string Output(DataTable orders);
  }

  public class HtmlFormatter : IOrdersFormatter {
    private string Output (DataTable table) {
      var sb = new StringBuilder("<ul>");

      foreach(var row in table.Rows) {
        sb.Append($"<li>{row["id"]}</li>");
      }

      sb.Append("</ul>");

      return sb.ToString();
    }
  }
}

namespace Report {
  public interface IOrdersRepository {
    DataTable QueryOrders (DateTime startDate, DateTime endDate);
  }

  public class OrdersRepository : IOrdersRepository {
    private DataTable QueryOrders (DateTime startDate, DateTime endDate) {
      var table = new DataTable();    
      using var da = new SqlDataAdapter("SELECT id FROM orders", "connection string");
          da.Fill(table);

      return table;
    }
  }
}
```

## Open-closed Principle

Software entities must be open for extension, but closed for modification.
Software entities (classes, modules, functions, etc.) must be extensible without changing their content.
If you strictly adhere to this principle, then you can regulate the behavior of the code without changing the source itself.

Let's consider an example:

```c#
namespace Shapes {
  public abstract class Shape { }

  public class Rectangle : Shape {
    private int _width;
    private int _height;

    public double Width { get => _width; }
    public double Height { get => _height; }

    public Rectangle(double width, double height) {
      _width = width;
      _height = height;
    }
  }

  public class Circle : Shape {
    private double _radius;
    public double Radius { get => _radius; }

    public Circle (double radius) {
        _radius = radius;
    }
  }

  public class AreaCalculator {
    public double Calculate(Shape shape) {
      if (shape is Rectangle rect) {
          return rect.Width * rect.Height;
      }

      if (shape is Circle circ)
          return circ.Radius * 2 * pi();
      }
    }
  }

  public class Program {
    public static void Main(string[] args) {
      var circle = new Circle(5);
      var rect = new Rectangle(8,5);
      var areaCalc = new AreaCalculator();
      
      Console.WriteLine(areaCalc.Calculate(circle));
      Console.WriteLine(areaCalc.Calculate(rect));        
    }
  }
```

If we need to calculate the square area, then we need to change the calculation method in the AreaCalculator class.
But this will violate the openness / closedness principle, according to which we cannot change, but only expand.
How can you solve the problem?

**See:**

```c#
namespace Shapes {
  public interface IShape {
    double CalculateArea();
  }

  public class Rectangle : IShape {
    private int _width;
    private int _height;

    public double Width { get => _width; }
    public double Height { get => _height; }

    public Rectangle(double width, double height) {
      _width = width;
      _height = height;
    }

    public double CalculateArea() {
      return _height * _width;
    }
  }

  public class Circle : IShape {
    private double _radius;
    public double Radius { get => _radius; }

    public Circle (double radius) {
        _radius = radius;
    }

    public double CalculateArea() {
      return _radius * 2 * pi();
    }
  }

  public class AreaCalculator {
    public double Calculate(IShape shape) {
      return shape.CalculateArea() ?? 0;
    }
  }
  
  public class Program {
    public static void Main(string[] args) {
      var circle = new Circle(5);
      var rect = new Rectangle(8,5);
      var areaCalc = new AreaCalculator();
      
      Console.WriteLine(areaCalc.Calculate(circle));
      Console.WriteLine(areaCalc.Calculate(rect));        
    }
  }
}
```

Now you can find the area of ​​a circle without changing the AreaCalculator class.

## Liskov Substitution Principle

This principle was proposed by Barbara Liskov in her paper "Data abstraction" in 1987.
And in 1994, the idea was summarized by Barbara and Jennette Wing:

> Let φ (x) be a provable property of an object x of type T. Then φ (y) must be true for objects y of type S, where S is a subtype of T.

The human readable version repeats almost everything Bertrand Meyer said, but here the type system is taken as a basis:

> * Preconditions cannot be strengthened in a subtype
> * Postconditions cannot be relaxed in a subtype.
> * Supertype invariants can be stored in a subtype.

Robert Martin in 1996 gave a different definition, more understandable:

> Functions using reference pointers to base classes,
should be able to use objects of derived classes without even knowing it.

Simply put:

> subclass / derived class must be interchangeable with base / parent class.

This means that any implementation of an abstraction (interface) must be interchangeable anywhere,
in which this abstraction is adopted. Basically, when we use interfaces in our code,
then we use the contract not only for the input data received by the interface, but also for the output data,
returned by different classes that implement this interface. In both cases, the data must be of the same type.

**This code violates this principle and let's see a way to fix it:**

```c#
namespace SOLID_PRINCIPLES.LSP {
    public class Apple {
        public virtual string GetColor() {
            return "Red";
        }
    }

    public class Orange : Apple {
        public override string GetColor() {
            return "Orange";
        }
    }

    public class Program {
        public static void Main(string[] args) {
            Apple apple = new Orange();
            Console.WriteLine(apple.GetColor());
        }
    }
}
```

After replacing the child object, that is, Apple retaining the Orange object, the behavior also changes.
This is contrary to the LSP principle.

```c#
namespace SOLID_PRINCIPLES.LSP {
  public abstract class Fruit {
    public abstract string GetColor();
  }

  public class Apple : Fruit {
    public override string GetColor() {
        return "Red";
    }
  }

  public class Orange : Fruit {
    public override string GetColor() {
        return "Orange";
    }
  }

  class Program {
    static void Main(string[] args) {
        Fruit fruit = new Orange();
        Console.WriteLine(fruit.GetColor());
        fruit = new Apple();
        Console.WriteLine(fruit.GetColor());
    }
  }
}
```

An abstract class allows you to create functionality that subclasses can implement or override.
An interface only allows you to define functionality, not implement it.
Although a class can only extend one abstract class, it can use multiple interfaces.

```c#
namespace SOLID_PRINCIPLES.LSP {
  public interface IFruit {
    string GetColor();
  }

  public class Apple : IFruit {
    public string GetColor() {
        return "Red";
    }
  }

  public class Orange : IFruit {
    public string GetColor() {
        return "Orange";
    }
  }

  public class Program {
    public static void Main(string[] args) {
        IFruit fruit = new Orange();
        Console.WriteLine(fruit.GetColor());
        fruit = new Apple();
        Console.WriteLine(fruit.GetColor());
    }
  }
}
```

Now, run the application and it should give the output as expected.
Here we are following the LSP as we are now able to change the object with its subtype.

## Interface Segregation Principle

You cannot force a client to implement an interface that it does not use.
This means that you need to break down the interfaces into smaller ones that better suit the specific needs of your customers.

As with the single responsibility principle, the purpose of the interface separation principle is to minimize side effects and repetition by dividing the software into independent parts.

**Consider an example:**

```c#
namespace Worker {
  public interface IWorker {
      void Work();
      void Sleep();
  }


  public class HumanWorker : IWorker {
    public void Work() {
      Console.WriteLine("work");
    }

    public void Sleep() {
      Console.WriteLine("sleep");
    }
  }

  public class RobotWorker : IWorker {
    public void Work() {
      Console.WriteLine("work");
    }

    public void Sleep() {
      // no neeed
    }
  }
}
```

RobotWorker doesn't need to sleep, but the class must implement the sleep method, because all methods in the interface are abstract. This violates the principle of separation. Here's how to fix it:

```c#
namespace Worker {
  public interface IWork {
      void Work();
  }

  public interface ISleep {
    void Sleep();
  }

  public class HumanWorker : IWork, ISleep {
    public void Work() {
      Console.WriteLine("work");
    }

    public void Sleep() {
      Console.WriteLine("sleep");
    }
  }

  public class RobotWorker : IWork {
    public void Work() {
      Console.WriteLine("work");
    }
  }
}
```

## Dependency Inversion Principle

High-level modules should not depend on low-level ones. Both kinds of modules must depend on abstractions.
Abstractions should not depend on details. The details should depend on the abstraction.
Simply put: depend on abstractions, not on something specific.

Applying this principle, some modules can be easily replaced with others, just changing the dependency module,
and then no changes in the low-level module will affect the high-level one.

**Consider an example:**

```c#
namespace Connections {
  public class MySqlConnection {
    public void Connect() {
      Console.WriteLine("MySql Connection");
    }
  }

  public class PassworReminder {
    private readonly MySqlConnection _mySqlConnection;

    public PasswordReminder(MySqlConnection mySqlConnection) {
      _mySqlConnection = mySqlConnection;
    }
  }
}
```

There is a common misconception that Dependency Inversion is
synonymous with dependency injection. But these are different things.
In the above code, despite the fact that the MySQLConnection class was injected into the PasswordReminder class,
the latter depends on MySQLConnection. The higher-level PasswordReminder should not depend on
lower-level module MySQLConnection.
If we need to change the connection from MySQLConnection to MongoDBConnection,
then you will have to change the implementation of the constructor written in the code into the PasswordReminder class.
The PasswordReminder class should depend on abstractions, not something specific. But how to do that?

**Consider an example:**

```c#
namespace Connections {

  public interface IConnection {
    void Connect();
  }
  public class MySqlConnection : IConnection {
    public void Connect() {
      Console.WriteLine("MySql Connection");
    }
  }

  public class PassworReminder {
    private readonly IConnection _dbConnection;

    public PasswordReminder(IConnection dbConnection) {
      _dbConnection = dbConnection;
    }
  }
}
```

Here we need to change the connection from MySQLConnection to MongoDBConnection.
We don't need to change the constructor injection in the PasswordReminder class, because in this case, the PasswordReminder class only depends on the abstraction.
