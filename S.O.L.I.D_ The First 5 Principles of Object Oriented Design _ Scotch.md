<!--S.O.L.I.D: The First 5 Principles of Object Oriented Design-->

# S.O.L.I.D：面向对象设计的5大原则 #

S.O.L.I.D is an acronym for the first five object-oriented design(OOD) principles by Robert C. Martin, popularly known as Uncle Bob.

S.O.L.I.D是**最早的**五大面向对象设计(OOD)原则的**缩写**，由Robert C. Martin提出，**俗称**Bob大叔。

These principles, when combined together, make it easy for a programmer to develop software that are easy to maintain and extend. They also make it easy for developers to avoid code smells, easily refactor code, and are also a part of the agile or adaptive software development.

这些原则，结合在一起，方便程序员开发易于维护和扩展的软件。他们也**方便**开发人员避免**代码异味**，更容易重构代码，也是敏捷或自适应软件开发的一部分。

Note: this is just a simple “welcome to S.O.L.I.D” article, it simply sheds light on what S.O.L.I.D is.

注意：这只是一个简单的“欢迎来到S.O.L.I.D”的文章，它只是揭示了S.O.L.I.D是什么。

<!--S.O.L.I.D stands for:-->

## S.O.L.I.D代表什么:##

When expanded the acronyms might seem complicated, but they are pretty simple to grasp.

当把缩略词展开看似复杂，但他们非常容易掌握。

<!--S – Single-responsiblity principle
O – Open-closed principle
L – Liskov substitution principle
I – Interface segregation principle
D – Dependency Inversion Principle-->

* S – 单一职责原则
* O – 开放封闭原则
* L – 里氏替换原则
* I – 接口隔离原则
* D – 依赖倒置原则

Let’s look at each principle individually to understand why S.O.L.I.D can help make us better developers.

让我们来单独看看每个原则，来理解为什么S.O.L.I.D能帮助我们成为更好的开发人员。

<!--## Single-responsibility Principle ##-->

## 单一职责原则 ##

S.R.P for short – this principle states that:

S.R.P（简称） -这个原则指出:

A class should have one and only one reason to change, meaning that a class should have only one job.

一个类应该有一个且只有一个去改变它的理由，这意味着一个类应该只有一项工作。

For example, say we have some shapes and we wanted to sum all the areas of the shapes. Well this is pretty simple right?

例如，假设我们有一些shape（形状），并且我们想求所有shape的面积的和。这很简单对吗?

```
class Circle {
    public $radius;

    public function __construct($radius) {
        $this->radius = $radius;
    }
}

class Square {
    public $length;

    public function __construct($length) {
        $this->length = $length;
    }
}
```

First, we create our shapes classes and have the constructors setup the required parameters. Next, we move on by creating the AreaCalculator class and then write up our logic to sum up the areas of all provided shapes.

首先，我们创建shape类，构造函数设置必需的参数。接下来，我们继续通过创建AreaCalculator类，然后编写求取所提供的shape的面积的和的逻辑。

```
class AreaCalculator {

    protected $shapes;

    public function __construct($shapes = array()) {
        $this->shapes = $shapes;
    }

    public function sum() {
        // logic to sum the areas
    }

    public function output() {
        return implode('', array(
            "<h1>",
                "Sum of the areas of provided shapes: ",
                $this->sum(),
            "</h1>"
        ));
    }
}
```

To use the AreaCalculator class, we simply instantiate the class and pass in an array of shapes, and display the output at the bottom of the page.

使用AreaCalculator类，我们简单地实例化类，同时传入一组shape，并在页面的底部显示输出。

```
$shapes = array(
    new Circle(2),
    new Square(5),
    new Square(6)
);

$areas = new AreaCalculator($shapes);

echo $areas->output();
```

The problem with the output method is that the AreaCalculator handles the logic to output the data. Therefore, what if the user wanted to output the data as json or something else?

输出方法的问题在于，AreaCalculator拥有了输出数据的逻辑。因此，如果用户想要以json或其他方式输出数据该怎么办?

All of that logic would be handled by the AreaCalculator class, this is what SRP frowns against; the AreaCalculator class should only sum the areas of provided shapes, it should not care whether the user wants json or HTML.

所有的逻辑将由AreaCalculator类处理，这是违反SRP的；AreaCalculator类应该只对提供的shape进行面积求和，它不应该关心用户是需要json还是HTML。

So, to fix this you can create an SumCalculatorOutputter class and use this to handle whatever logic you need to handle how the sum areas of all provided shapes are displayed.

因此，为了解决这个问题，你可以创建一个SumCalculatorOutputter类，使用这个来处理你所需要的逻辑，即对所提供的shape进行面积求和后如何显示。

The SumCalculatorOutputter class would work like this:

SumCalculatorOutputter类按如下方式工作:

```
$shapes = array(
    new Circle(2),
    new Square(5),
    new Square(6)
);

$areas = new AreaCalculator($shapes);
$output = new SumCalculatorOutputter($areas);

echo $output->JSON();
echo $output->HAML();
echo $output->HTML();
echo $output->JADE(); 
```

Now, whatever logic you need to output the data to the user is now handled by the SumCalculatorOutputter class.

现在，不管你需要何种逻辑来输出数据给用户，由SumCalculatorOutputter类处理。

<!--## Open-closed Principle ##-->

## 开放封闭原则 ##

Objects or entities should be open for extension, but closed for modification.

对象或实体应该对扩展开放，对修改封闭。

This simply means that a class should be easily extendable without modifying the class itself. Let’s take a look at the AreaCalculator class, especially it’s sum method.

这就意味着一个类应该在无需修改类本身就很容易扩展。让我们看看AreaCalculator类，尤其是它的sum方法。

```
public function sum() {
    foreach($this->shapes as $shape) {
        if(is_a($shape, 'Square')) {
            $area[] = pow($shape->length, 2);
        } else if(is_a($shape, 'Circle')) {
            $area[] = pi() * pow($shape->radius, 2);
        }
    }

    return array_sum($area);
}   
```

If we wanted the sum method to be able to sum the areas of more shapes, we would have to add more if/else blocks and that goes against the Open-closed principle.

如果我们希望sum方法能够对更多的shape进行面积求和，我们会添加更多的If / else块，这违背了开放封闭原则。

A way we can make this sum method better is to remove the logic to calculate the area of each shape out of the sum method and attach it to the shape’s class.

能让这个sum方法做的更好的一种方式是，将计算每个shape的面积的逻辑从sum方法中移出，将它附加到shape类上。

```
class Square {
    public $length;

    public function __construct($length) {
        $this->length = $length;
    }

    public function area() {
        return pow($this->length, 2);
    }
}   
```

The same thing should be done for the Circle class, an area method should be added. Now, to calculate the sum of any shape provided should be as simple as:

对Circle类应该做同样的事情，area方法应该添加。现在，计算任何所提的shape的面积的和的方法应该和如下简单：

```
public function sum() {
    foreach($this->shapes as $shape) {
        $area[] = $shape->area;
    }

    return array_sum($area);
}
```

Now we can create another shape class and pass it in when calculating the sum without breaking our code. However, now another problem arises, how do we know that the object passed into the AreaCalculator is actually a shape or if the shape has a method named area?

现在我们可以创建另一个shape类，并在计算和时将其传递进来，这不破坏我们的代码。然而，现在另一个问题出现了，我们怎么知道传递到AreaCalculator上的对象确实是一个shape，或者这个shape具有一个叫做area的方法?

Coding to an interface is an integral part of S.O.L.I.D, a quick example is we create an interface, that every shape implements:

面向接口编程是S.O.L.I.D不可或缺的一部分，一个快速的例子是我们创建一个接口，每个shape实现它:

```
interface ShapeInterface {
    public function area();
}

class Circle implements ShapeInterface {
    public $radius;

    public function __construct($radius) {
        $this->radius = $radius;
    }

    public function area() {
        return pi() * pow($this->radius, 2);
    }
}  
```

In our AreaCalculator sum method we can check if the shapes provided are actually instances of the ShapeInterface, otherwise we throw an exception:

在我们AreaCalculator的求和中，我们可以检查所提供的shape确实是ShapeInterface的实例，否则我们抛出一个异常:

```
public function sum() {
    foreach($this->shapes as $shape) {
        if(is_a($shape, 'ShapeInterface')) {
            $area[] = $shape->area();
            continue;
        }

        throw new AreaCalculatorInvalidShapeException;
    }

    return array_sum($area);
}    
```

<!--## Liskov substitution principle ##-->

## 里氏替换原则 ##

Let q(x) be a property provable about objects of x of type T. Then q(y) should be provable for objects y of type S where S is a subtype of T.

在对象x为类型T时q(x)成立，那么当S是T的子类时，对象y为类型S时q(y)成立。（即对父类的调用同样适用于子类）

All this is stating is that every subclass/derived class should be substitutable for their base/parent class.

这一切说明的是，每一个子类或派生类应该可以替换它们基类或父类。

Still making use of out AreaCalculator class, say we have a VolumeCalculator class that extends the AreaCalculator class:

还利用AreaCalculator类，我们有一个VolumeCalculator类，它扩展了AreaCalculator类:

```
class VolumeCalculator extends AreaCalulator {
    public function __construct($shapes = array()) {
        parent::__construct($shapes);
    }

    public function sum() {
        // logic to calculate the volumes and then return and array of output
        return array($summedData);
    }
}    
```

In the SumCalculatorOutputter class:

在SumCalculatorOutputter类:

```
class SumCalculatorOutputter {
    protected $calculator;

    public function __constructor(AreaCalculator $calculator) {
        $this->calculator = $calculator;
    }

    public function JSON() {
        $data = array(
            'sum' => $this->calculator->sum();
        );

        return json_encode($data);
    }

    public function HTML() {
        return implode('', array(
            '<h1>',
                'Sum of the areas of provided shapes: ',
                $this->calculator->sum(),
            '</h1>'
        ));
    }
}    
```
        
If we tried to run an example like this:

如果我们试图这样来运行一个例子:

```
$areas = new AreaCalculator($shapes);
$volumes = new AreaCalculator($solidShapes);

$output = new SumCalculatorOutputter($areas);
$output2 = new SumCalculatorOutputter($volumes);
```

The program does not squawk, but when we call the HTML method on the $output2 object we get an E_NOTICE error informing us of an array to string conversion.

程序可以运行，但是当我们在$output2对象调用HTML方法，我们得到一个E_NOTICE错误，提示数组到字符串的转换。

To fix this, instead of returning an array from the VolumeCalculator class sum method, you should simply:

为了解决这个问题，不要从VolumeCalculator类的sum方法返回一个数组，你应该:

```
public function sum() {
    // logic to calculate the volumes and then return and array of output
    return $summedData;
}
```

The summed data as a float, double or integer.

求和的结果作为一个浮点数，双精度或整数。

<!--## Interface segregation principle ##-->

## 接口隔离原则 ##

A client should never be forced to implement an interface that it doesn’t use or clients shouldn’t be forced to depend on methods they do not use.

客户端不应该被迫实现一个接口，它不使用或客户不应该被迫依赖他们不使用方法。

Still using our shapes example, we know that we also have solid shapes, so since we would also want to calculate the volume of the shape, we can add another contract to the ShapeInterface:

我们仍然使用shape的例子，我们知道也有立体shape，因为我们也想计算shape的体积，我们可以添加另一个合约到ShapeInterface:

```
interface ShapeInterface {
    public function area();
    public function volume();
}   
```

Any shape we create must implement the volume method, but we know that squares are flat shapes and that they do not have volumes, so this interface would force the Square class to implement a method that it has no use of.

任何我们创建的shape必须实现volume的方法，但是我们知道平面shape，他们没有体积，所以这个接口将迫使shape类实现一个它没有使用的方法。

ISP says no to this, instead you could create another interface called SolidShapeInterface that has the volume contract and solid shapes like cubes e.t.c can implement this interface:

ISP对它说不，你可以创建另一个名为SolidShapeInterface的接口，它有一个volume合约，对于立体shape比如cube等等，可以实现这个接口:

```
interface ShapeInterface {
    public function area();
}

interface SolidShapeInterface {
    public function volume();
}

class Cuboid implements ShapeInterface, SolidShapeInterface {
    public function area() {
        // calculate the surface area of the cuboid
    }

    public function volume() {
        // calculate the volume of the cuboid
    }
}
```

This is a much better approach, but a pitfall to watch out for is when type-hinting these interfaces, instead of using a ShapeInterface or a SolidShapeInterface.

这是一个更好的方法，但小心一个缺陷，当这些接口类型提示时，使用ShapeInterface或SolidShapeInterface。

You can create another interface, maybe ManageShapeInterface, and implement it on both the flat and solid shapes, this way you can easily see that it has a single API for managing the shapes. For example:

你可以创建另一个接口，可以是ManageShapeInterface，平面和立体shape都可用，这样你可以很容易地看到它有一个管理shape的单一API。例如:

```
interface ManageShapeInterface {
    public function calculate();
}

class Square implements ShapeInterface, ManageShapeInterface {
    public function area() { /*Do stuff here*/ }

    public function calculate() {
        return $this->area();
    }
}

class Cuboid implements ShapeInterface, SolidShapeInterface, ManageShapeInterface {
    public function area() { /*Do stuff here*/ }
    public function volume() { /*Do stuff here*/ }

    public function calculate() {
        return $this->area() + $this->volume();
    }
}   
```

Now in AreaCalculator class, we can easily replace the call to the area method with calculate and also check if the object is an instance of the ManageShapeInterface and not the ShapeInterface.

现在AreaCalculator类中，我们可以轻易用calculate替代area调用，同时可以检查一个对象是ManageShapeInterface而不是ShapeInterface的实例。

<!--## Dependency Inversion principle ##-->

## 依赖倒置原则 ##

The last, but definitely not the least states that:

最后，但肯定不是最少的声明：

Entities must depend on abstractions not on concretions. It states that the high level module must not depend on the low level module, but they should depend on abstractions.

实体必须依靠抽象而不是具体实现。它表示高层次的模块不应该依赖于低层次的模块，他们都应该依赖于抽象。

This might sound bloated, but it is really easy to understand. This principle allows for decoupling, an example that seems like the best way to explain this principle:

这听起来可能臃肿，但它很容易理解。这一原则允许解耦，举个例子，它似乎是用来解释这一原则最好的方式：

```
class PasswordReminder {
    private $dbConnection;

    public function __construct(MySQLConnection $dbConnection) {
        $this->dbConnection = $dbConnection;
    }
}
```

First the MySQLConnection is the low level module while the PasswordReminder is high level, but according to the definition of D in S.O.L.I.D. which states that Depend on Abstraction not on concretions, this snippet above violates this principle as the PasswordReminder class is being forced to depend on the MySQLConnection class.

首先MySQLConnection是低层次模块，而PasswordReminder是高层次，但根据S.O.L.I.D.中D的定义，即依赖抽象而不是具体实现，上面这段代码违反这一原则，PasswordReminder类被迫依赖于MySQLConnection类。

Later if you were to change the database engine, you would also have to edit the PasswordReminder class and thus violates Open-close principle.

以后如果你改变数据库引擎，你还必须编辑PasswordReminder类，因此违反了开闭原则。

The PasswordReminder class should not care what database your application uses, to fix this again we “code to an interface”, since high level and low level modules should depend on abstraction, we can create an interface:

PasswordReminder类不应该关心你的应用程序使用什么数据库，为了解决这个问题我们又一次“面向接口编程”，因为高层次和低层次模块应该依赖于抽象，我们可以创建一个接口:

```
interface DBConnectionInterface {
    public function connect();
}    
```

The interface has a connect method and the MySQLConnection class implements this interface, also instead of directly type-hinting MySQLConnection class in the constructor of the PasswordReminder, we instead type-hint the interface and no matter the type of database your application uses, the PasswordReminder class can easily connect to the database without any problems and OCP is not violated.

接口有一个connect方法，MySQLConnection类实现该接口，在PasswordReminder类的构造函数不使用MySQLConnection类，而是使用接口替换，不用管你的应用程序使用的数据库类型，PasswordReminder类可以在没有任何问题下很容易地连接到数据库，且不违反OCP。

```
class MySQLConnection implements DBConnectionInterface {
    public function connect() {
        return "Database connection";
    }
}

class PasswordReminder {
    private $dbConnection;

    public function __construct(DBConnectionInterface $dbConnection) {
        $this->dbConnection = $dbConnection;
    }
}  
```

According to the little snippet above, you can now see that both the high level and low level modules depend on abstraction.

根据上面的代码片段，你现在可以看到，高层次和低层次模块依赖于抽象。

<!--## Conclusion ##-->

## 结论 ##

Honestly, S.O.L.I.D might seem to be a handful at first, but with continuous usage and adherence to its guidelines, it becomes a part of you and your code which can easily be extended, modified, tested, and refactored without any problems.

老实说，S.O.L.I.D最初可能似乎没有分量，但通过连续使用和遵守其指导方针，它变成了你的一部分，你的代码可以在没有任何问题下很容易地扩展、修改、测试和重构。