# Принципы SOLID

Принципы SOLID — это стандарт программирования, который все разработчики должны хорошо понимать,
чтобы избегать создания плохой архитектуры. Этот стандарт широко используется в ООП.
Если применять его правильно, он делает код более расширяемым, логичным и читабельным.
Когда разработчик создаёт приложение, руководствуясь плохой архитектурой, код получается негибким,
даже небольшие изменения в нём могут привести к багам. Поэтому нужно следовать принципам SOLID.
На их освоение потребуется какое-то время, но если вы будете писать код в соответствии с этими принципами,
то его качество повысится, а вы освоите создание хорошей архитектуры ПО.
Чтобы понять принципы SOLID, нужно чётко понимать, как использовать интерфейсы.
Если у вас такого понимания нет, то сначала почитайте документацию.
Я буду объяснять SOLID самым простым способом, так что новичкам легче будет разобраться.
Будем рассматривать принципы один за другим.

## Принцип единственной ответственности (Single Responsibility Principle)
Существует лишь одна причина, приводящая к изменению класса.
Один класс должен решать только какую-то одну задачу. Он может иметь несколько методов,
но они должны использоваться лишь для решения общей задачи.
Все методы и свойства должны служить одной цели. Если класс имеет несколько назначений,
его нужно разделить на отдельные классы.

**Рассмотрим пример:**

```c#
namespace Report {
  public class OrdersReport {
    public string GetOrdersInfo (DateTime startDate, DateTime endDate) {
      var orders = QueryOrders(startDate, endDate);

      return FormatOrders(orders);
    }

    // If we would update our persistence layer in the future,
    // we would have to do changes here too. <=> reason to change!
    private DataTable QueryOrders (DateTime startDate, DateTime endDate) {
      var table = new DataTable();    
      using var da = new SqlDataAdapter("SELECT id FROM orders", "connection string");
          da.Fill(table);

      return table;
    }
    
    // If we changed the way we want to format the output,
    // we would have to make changes here. <=> reason to change!
    private string FormatOrders (DataTable table) {
      var sb = new StringBuilder("<ul>");

      foreach(var row in table.Rows) {
        sb.Append($"<li>{row["id"]}</li>");
      }

      sb.Append("</ul>");

      return sb.ToString();
    }
  }
}
```

Приведённый здесь класс нарушает принцип единственной ответственности.
Почему он должен извлекать данные из базы? Это задача для уровня хранения данных,
на котором данные сохраняются и извлекаются из хранилища (например, базы данных).
Это ответственность не этого класса.
Также данный класс не должен отвечать за формат следующего метода,
потому что нам могут понадобиться данные другого формата, например, XML, JSON, HTML и т.д.

**Код после рефакторинга будет выглядеть так:**

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

## Принцип открытости/закрытости (Open-closed Principle)

Программные сущности должны быть открыты для расширения, но закрыты для модификации.
Программные сущности (классы, модули, функции и прочее) должны быть расширяемыми без изменения своего содержимого.
Если строго соблюдать этот принцип, то можно регулировать поведение кода без изменения самого исходника.

Рассмотрим пример:

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

Если нам нужно вычислить площадь квадрата, то нужно изменить метод вычисления в классе AreaCalculator.
Но это нарушит принцип открытости/закрытости, согласно которому мы можем не изменять, а только расширять.
Как же можно решить поставленную задачу?

**Смотрите:**

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

Теперь можно найти площадь круга, не меняя класс AreaCalculator.

## Принцип подстановки Барбары Лисков (Liskov Substitution Principle)

Этот принцип Барбара Лисков предложила в своём докладе “Data abstraction” в 1987-м.
А в 1994-м идея была вкратце сформулирована Барбарой и Дженнет Уинг:

>Пусть φ(x) — доказуемое свойство объекта x типа T. Тогда φ(y) должно быть верным для объектов y типа S, где S — подтип T.

В удобочитаемой версии повторяется практически всё, что говорил Бертранд Майер, но здесь в качестве базиса взята система типов:

> * Предварительные условия не могут быть усилены в подтипе
> * Постусловия не могут быть ослаблены в подтипе.
> * Инварианты супертипа могут быть сохранены в подтипе.

Роберт Мартин в 1996-м дал другое определение, более понятное:

>Функции, использующие указатели ссылок на базовые классы,
должны уметь использовать объекты производных классов, даже не зная об этом.

Попросту говоря:

>подкласс/производный класс должен быть взаимозаменяем с базовым/родительским классом.

Значит, любая реализация абстракции (интерфейса) должна быть взаимозаменяемой в любом месте,
в котором принимается эта абстракция. По сути, когда мы используем в коде интерфейсы,
то используем контракт не только по входным данным, принимаемым интерфейсом, но и по выходным данным,
возвращаемым разными классами, реализующими этот интерфейс. В обоих случаях данные должны быть одного типа.

**В этом коде нарушен обсуждаемый принцип и показан способ исправления:**

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

После замены дочернего объекта, то есть Apple, сохраняющей объект Orange, поведение также изменяется.
Это противоречит принципу LSP.

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

Абстрактный класс позволяет вам создавать функциональные возможности, которые подклассы могут реализовать или переопределить.
Интерфейс позволяет вам только определять функциональность, но не реализовывать ее.
И хотя класс может расширять только один абстрактный класс, он может использовать несколько интерфейсов.

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

## Принцип разделения интерфейса (Interface Segregation Principle)

Нельзя заставлять клиента реализовать интерфейс, которым он не пользуется.
Это означает, что нужно разбивать интерфейсы на более мелкие, лучше удовлетворяющие конкретным потребностям клиентов.

Как и в случае с принципом единственной ответственности, цель принципа разделения интерфейса заключается в минимизации побочных эффектов и повторов за счёт разделения ПО на независимые части.

**Рассмотрим пример:**

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

RobotWorker’у не нужно спать, но класс должен реализовать метод sleep, потому что все методы в интерфейсе абстрактны. Это нарушает принцип разделения. Вот как это можно исправить:

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

## Принцип инверсии зависимостей (Dependency Inversion Principle)

Высокоуровневые модули не должны зависеть от низкоуровневых. Оба вида модулей должны зависеть от абстракций.
Абстракции не должны зависеть от подробностей. Подробности должны зависеть от абстракций.
Проще говоря: зависьте от абстракций, а не от чего-то конкретного.

Применяя этот принцип, одни модули можно легко заменять другими, всего лишь меняя модуль зависимости,
и тогда никакие перемены в низкоуровневом модуле не повлияют на высокоуровневый.

**Рассмотрим пример:**

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

Есть распространённое заблуждение, что «инверсия зависимостей» является
синонимом «внедрения зависимостей».Но это разные вещи.
В приведённом коде, невзирая на то, что класс MySQLConnection был внедрён в класс PasswordReminder,
последний зависит от MySQLConnection. Более высокуровневый PasswordReminder не должен зависеть от
более низкуровневого модуля MySQLConnection.
Если нам нужно изменить подключение с MySQLConnection на MongoDBConnection,
то придётся менять прописанное в коде внедрение конструктора в класс PasswordReminder.
Класс PasswordReminder должен зависеть от абстракций, а не от чего-то конкретного. Но как это сделать?

**Рассмотрим пример:**

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

Здесь нам нужно изменить подключение с MySQLConnection на MongoDBConnection.
Нам не нужно менять внедрение конструктора в класс PasswordReminder, потому что в данном случае класс PasswordReminder зависит только от абстракции.
