# Octoller.LineCommander
> Небольшая библиотека написанная (и находящаяся в доработке) в качестве проекта для обучения.

> README находится в процессе написания

Библиотека позволяет, реализуя специальные интерфейсы, создавать обработчики строковых команд, перекладывая парсинг строки и запуск обработчиков на внутренние классы. 

### На текущий момент поддерживаются следующие возможности:
* Передача любого количества аргументов в команду через символ разделения `,`

      CL> команда:аргумент1,аргумент2,аргумент3... аргументN
      

* Выполнение цепочки команд с использованием `команд-переходов`, определяющих условие перехода к следующей команде. 
 
      CL> команда:аргумент,аргумент,аргумент & команда:аргумент && команда:аргумент,аргумент || команда  
       
  **&**  — используется для разделения нескольких команд в одной командной строке. Команды выполняются последовательно.

  **&&** — запускает команду, стоящую за символом &&, только если команда, стоящая перед этим символом была выполнена успешно.

  **||** — запускает команду, стоящую за символом ||, только если команда, стоящая перед символом || не была выполнена.
  
  
* Пропуск через обработчики команд объекта реализующего интерфейс `IChContext`, что позволяет отследить успешное выполнение команд, получить результат выполнения или сообщения об ошибках

* Содержит встроенную команду `help` позволяющую вывести список всех загруженных команд с указанием ключа и описанием

### Использование
Для создания собственной команды необходимо реализовать интерфейс `IOrderHandler` и шаблонный класс `BaseHeader<TOrderHandler>`

`BaseHeader<TOrderHandler>` - отвечает за строковое имя команды и её описание для `help`.

Для использования необходимо переопределить свойства `Key` и `Description`, а в параметре наследуемого класса указать тип класса обработчика команды

```C#
using Octoller.OrderLineHandler.ServiceObjects;

public sealed class ContainerMore : BaseHeader<OrderMore> {

  public override string Key {
      get => "more";
  }

  public override string Description {
      get => "выводит большее число из полученного массива";
  }
}
```

`IOrderHandler` - отвечает за результат выполнения команды и обработку передаваемого контекста.

Класс обработчика должен обязательно содержать пустой конструктор по умолчанию!

```C#
using Octoller.OrderLineHandler.ServiceObjects;
using Octoller.OrderLineHandler.Processor;

public sealed class OrderMore : IOrderHandler {

  private int[] numbers = null;
  private bool isError = false;

  //метод обработки
  public bool Invoke(IChContext context) {
      if (isError) {
          context.Complite = false;
          context.SetError("Некорректный аргумент.");
          return false;
      } else {
          context.Complite = true;
          int max = numbers.Max();
          System.Console.WriteLine(max);
          return true;
      }
  }

  //парсим и проверяем на корректность полученные аргументы из командной строки
  public void SetArgument(params string[] arg) {
      if (arg != null && arg.Length > 0) {
          List<int> temp = new List<int>();
          foreach (var a in arg) {
              if (int.TryParse(a, out int i)) {
                  temp.Add(i);
              } else {
                  isError = true;
                  break;
              }
          }
          numbers = temp.ToArray();
      } else {
          isError = true;
      }
  }
}
```

     In the process of writing...
