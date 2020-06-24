# 设计模式

[TOC]

## 简介
设计模式是软件设计中常见问题的典型解决方案，它们就像能根据需求进行调整的预制蓝图，可用于解决代码中反复出现的设计问题

[参考文档](https://refactoringguru.cn/design-patterns)

设计模式根据功能分为一下类型：
- 创建型模式：提供创建对象的机制，增加已有代码的灵活性和可复用性
- 结构型模式：介绍如何将对象和类组装成较大的结构，并同时保持结构的灵活和高效
- 行为型模式：负责对象间的高效沟通和职责委派

## 创建型
### 工厂方法模式
解决：
- 客户端代码可以不关心具体子类，在不指定具体产品类的情况下，通过调用子类们的统一方法来创建产品对象

意图：
- 在父类中定义一个用于创建产品对象的抽象方法，称为工厂方法，并允许子类对该方法进行不同的实现，返回不同类型的产品对象
- 创建产品对象时，通过调用具体子类的工厂方法来代替直接调用产品类

缺点：
- 每新增的一个产品，需要新增一个产品类，也必须新增相应的具体子类，代码可能会因此变得更复杂

实现：
``` py
from __future__ import annotations
from abc import ABC, abstractmethod


class Product(ABC):
    """
    抽象产品类，声明抽象方法
    """

    @abstractmethod
    def operation(self) -> str:
        pass


"""
具体产品类，实现已声明的抽象方法
"""


class ConcreteProduct1(Product):
    def operation(self) -> str:
        return "result of ConcreteProduct1"


class ConcreteProduct2(Product):
    def operation(self) -> str:
        return "result of ConcreteProduct2"


class Creator(ABC):
    """
    抽象创建者类，声明抽象工厂方法，用于创建具体产品对象
    """

    @abstractmethod
    def factory_method(self):
        pass


"""
具体创建者类，实现已声明的抽象工厂方法
"""


class ConcreteCreator1(Creator):
    def factory_method(self) -> ConcreteProduct1:
        return ConcreteProduct1()


class ConcreteCreator2(Creator):
    def factory_method(self) -> ConcreteProduct2:
        return ConcreteProduct2()


def client_code(creator: Creator) -> None:
    """
    客户端代码，通过调用创建者的工厂方法得到产品对象，不需要关心接收的创建者对象具体是什么
    """
    
    product = creator.factory_method()
    print(f"Client: I'm not aware of the creator's class, but it still works.\n"
          f"{product.operation()}", end="")


if __name__ == "__main__":
    print("App: Launched with the ConcreteCreator1.")
    client_code(ConcreteCreator1())
    print("\n")

    print("App: Launched with the ConcreteCreator2.")
    client_code(ConcreteCreator2())
```

结果：
> App: Launched with the ConcreteCreator1.
Client: I'm not aware of the creator's class, but it still works.
result of ConcreteProduct1
> 
> App: Launched with the ConcreteCreator2.
Client: I'm not aware of the creator's class, but it still works.
result of ConcreteProduct2

### 抽象工厂模式
解决：
- 客户端代码可以不关心具体工厂类，在不指定具体产品类的情况下，可以通过调用一个具体工厂类的各个方法来创建一簇产品对象，也可以通过调用不同具体工厂类的一个方法来创建一簇产品对象

意图：
- 在抽象工厂类中定义了用于创建不同产品的抽象方法，但这些方法的实现留给了具体工厂类
- 创建产品对象时，通过调用具体工厂类的不同创建方法来代替直接调用产品类
- 每个具体工厂类都对应一簇产品，每个创建方法也对应一簇产品对象
- 每簇产品对象通常拥有相同的属性和方法

缺点：
- 每新增一簇产品，需要新增对应的产品类和簇产品类，也必须新增一个具体工厂类，或者为每个现有的所有具体工厂类新增一个创建方法，代码可能会比之前更加复杂

实现：
``` py
from __future__ import annotations
from abc import ABC, abstractmethod


class AbstractProductA(ABC):
    """
    A簇的抽象产品类，声明抽象方法
    """

    @abstractmethod
    def useful_function_a(self) -> str:
        pass


"""
A簇的具体产品类，实现已声明的抽象方法
"""


class ConcreteProductA1(AbstractProductA):
    def useful_function_a(self) -> str:
        return "The result of the product A1."


class ConcreteProductA2(AbstractProductA):
    def useful_function_a(self) -> str:
        return "The result of the product A2."


class AbstractProductB(ABC):
    """
    B簇的抽象产品类，声明抽象方法
    """

    @abstractmethod
    def useful_function_b(self) -> None:
        pass

    @abstractmethod
    def another_useful_function_b(self, collaborator: AbstractProductA) -> None:
        """
        可以调用所有的的簇族具体产品对象，因为簇族的抽象产品类定义了统一的方法
        """
        pass


"""
B簇的具体产品类，实现已声明的抽象方法
"""


class ConcreteProductB1(AbstractProductB):
    def useful_function_b(self) -> str:
        return "result of product B1."

    def another_useful_function_b(self, collaborator: AbstractProductA) -> str:
        result = collaborator.useful_function_a()
        return f"result of product B1 collaborating with the {result}"


class ConcreteProductB2(AbstractProductB):
    def useful_function_b(self) -> str:
        return "result of product B2."

    def another_useful_function_b(self, collaborator: AbstractProductA):
        result = collaborator.useful_function_a()
        return f"result of product B2 collaborating with the {result}"


class AbstractFactory(ABC):
    """
    抽象工厂类，声明抽象方法，用于创建不同簇的抽象产品对象，同一簇产品中也分为很多类，但通常拥有相同的属性和方法
    """

    @abstractmethod
    def create_product_a(self) -> AbstractProductA:
        pass

    @abstractmethod
    def create_product_b(self) -> AbstractProductB:
        pass


"""
具体工厂类，实现抽象方法，用于创建一类具体产品对象
"""


class ConcreteFactory1(AbstractFactory):
    def create_product_a(self) -> ConcreteProductA1:
        return ConcreteProductA1()

    def create_product_b(self) -> ConcreteProductB1:
        return ConcreteProductB1()


class ConcreteFactory2(AbstractFactory):
    def create_product_a(self) -> ConcreteProductA2:
        return ConcreteProductA2()

    def create_product_b(self) -> ConcreteProductB2:
        return ConcreteProductB2()


def client_code(factory: AbstractFactory) -> None:
    """
    客户端代码只能通过抽象来使用工厂和产品，并不关心具体的工厂或子类对象是什么
    """

    product_a = factory.create_product_a()
    product_b = factory.create_product_b()

    print(f"{product_b.useful_function_b()}")
    print(f"{product_b.another_useful_function_b(product_a)}", end="")


if __name__ == "__main__":
    print("Client: Testing client code with the first factory type:")
    client_code(ConcreteFactory1())

    print("\n")

    print("Client: Testing the same client code with the second factory type:")
    client_code(ConcreteFactory2())
```

结果：
> Client: Testing client code with the first factory type:
result of product B1.
result of product B1 collaborating with the The result of the product A1.
>
> Client: Testing the same client code with the second factory type:
result of product B2.
result of product B2 collaborating with the The result of the product A2.

### 建造者模式
解决：
- 客户端代码创建一个复杂的对象，可以不依赖一个拥有大量参数的复杂构造函数，可以先创建一个基本的对象，然后一步步补充对象的细节，最终得到一个复杂的对象

意图：
- 定义一个用于管理结果对象创建过程的建造类，它拥有一个构建方法和多个配置结果对象的方法
- 调用不同的具体建造类对象的方法，可以分步骤地创建出不同类型的复杂对象
- 可以直接调用建造类对象的方法，也可以通过指导者对象定义调用的顺序进行间接调用
- 建造类的方法通常支持方法链

缺点：
- 由于该模式需要新增多个类，因此代码整体复杂程度会有所增加

实现：
``` py
from __future__ import annotations
from abc import ABC, abstractmethod
from typing import Any


class Product:
    """
    产品类，一般是非常复杂并且需要大量配置的
    """

    def __init__(self) -> None:
        self.parts = []

    def add(self, part: Any) -> None:
        self.parts.append(part)

    def list_parts(self) -> None:
        print(f"Product parts: {', '.join(self.parts)}", end="")


class Builder(ABC):
    """
    抽象建造者类，声明用于创建产品的不同部分的抽象方法
    """

    @property
    @abstractmethod
    def product(self) -> None:
        pass

    @abstractmethod
    def reset(self) -> None:
        pass

    @abstractmethod
    def produce_part_a(self) -> None:
        pass

    @abstractmethod
    def produce_part_b(self) -> None:
        pass

    @abstractmethod
    def produce_part_c(self) -> None:
        pass


class ConcreteBuilder1(Builder):
    """
    具体建造者类，实现声明的用于产品的不同部分的抽象方法
    """

    def __init__(self) -> None:
        """
        初始化一个空产品对象
        """
        self._product = Product()

    def reset(self) -> None:
        self._product = Product()

    @property
    def product(self) -> Product:
        product = self._product
        self.reset()
        return product

    def produce_part_a(self) -> Product:
        self._product.add("PartA1")
        return self._product

    def produce_part_b(self) -> Product:
        self._product.add("PartB1")
        return self._product

    def produce_part_c(self) -> Product:
        self._product.add("PartC1")
        return self._product


class Director:
    """
    指导者类，关联了建造者对象后，通过方法建造产品对象
    也可以不需要指导者，客户端直接使用建造者对象来建造产品对象
    """

    def __init__(self) -> None:
        self._builder = None

    @property
    def builder(self) -> Builder:
        return self._builder

    @builder.setter
    def builder(self, b: Builder) -> None:
        self._builder = b

    def build_minimal_viable_product(self) -> None:
        self.builder.produce_part_a()

    def build_full_featured_product(self) -> None:
        self.builder.produce_part_a()
        self.builder.produce_part_b()
        self.builder.produce_part_c()


if __name__ == "__main__":
    builder = ConcreteBuilder1()
    director = Director()
    Director.builder = builder

    print("minimal basic product: ")
    director.build_minimal_viable_product()
    builder.product.list_parts()

    print("\n")

    print("full featured product: ")
    director.build_full_featured_product()
    builder.product.list_parts()

    print("\n")

    print("Custom product: ")
    builder.produce_part_a()
    builder.produce_part_b()
    builder.product.list_parts()
```

结果：
> minimal basic product: 
> Product parts: PartA1

> full featured product: 
> Product parts: PartA1, PartB1, PartC1

> Custom product: 
> Product parts: PartA1, PartB1

### 原型模式
解决：
- 客户端代码可以直接通过对象本身的方法，来得到对象的复制，即使是复杂对象，而又无需使用其所属类

意图：
- 在原型类中定义一个通用的接口，返回对象的复制
- 在对象本身的所属类未知的情况下，也能得到对象的复制
- 原型对象通过自身的方法进行复制，由于和复制对象是同一个类型，因此两者可以互相访问和复制私有属性

缺点：
- 克隆包含循环引用的复杂对象可能会非常麻烦

实现：
``` py
from __future__ import annotations
from copy import deepcopy


"""
Python 的 Cloneable 抽象类，对其实现就能得到具体原型类
"""


class Prototype:
    """
    原型类，调用对象的 clone 方法可以得到复制体对象
    """

    def __init__(self, mutable:int, immutable:list) -> None:
        self.mutable = mutable
        self.immutable = immutable
        self.circular_reference = ComponentWithBackReference(self)

    def clone(self) -> Prototype:
        """
        deepcopy 对于不同属性的复制操作不一样，对不可变对象会直接引用，对可变对象会复制一份并引用，
        对循环引用对象会记录已复制过的对象，防止无限迭代
        """
        return deepcopy(self)


class ComponentWithBackReference:
    """
    和原型类有循环调用关系
    """
    def __init__(self, prototype: Prototype):
        self._prototype = prototype

    @property
    def prototype(self) -> Prototype:
        return self._prototype


if __name__ == "__main__":
    p1 = Prototype(1, [])
    p2 = p1.clone()

    print('the different of p1 and p2:')
    print('var | self | mutable | immutable | circular_reference | circular_reference.prototype')
    print(f'p1 | {id(p1)} | {id(p1.mutable)} | {id(p1.immutable)} | {id(p1.circular_reference)}'
          f' | {id(p1.circular_reference.prototype)}')
    print(f'p1 | {id(p2)} | {id(p2.mutable)} | {id(p2.immutable)} | {id(p2.circular_reference)}'
          f' | {id(p2.circular_reference.prototype)}')
```

结果：
> the different of p1 and p2:
> var | self | mutable | immutable | circular_reference |circular_reference.prototype
> p1 | 4505885648 | 4503272224 | 4505970480 | 4505885712 | 4505885648 |
> p1 | 4506002192 | 4503272224 | 4505970320 | 4506032208 | 4506002192 |

### 单例模式
解决：
- 保证一个类只有一个实例，客户端代码可以把这个类可作为一个访问该实例的全局节点，可用该实例进行全局变量存储

意图：
- 通过使用单例元类或者继承单例基类，来得到单例类
- 仅在首次进行单例类实例化时，完成单例对象的创建，后续进行实例化均返回该单例对象

缺点：
- 在多线程环境下需要进行特殊处理，避免多个线程多次创建单例对象

实现：
``` py
from __future__ import annotations
from typing import Optional


class SingletonMeta(type):
    """
    实现单例元类
    """

    _instance: Optional[Singleton1] = None

    def __call__(cls, *args, **kwargs) -> Singleton1:
        if cls._instance is None:
            cls._instance = super().__call__(*args, **kwargs)
        return cls._instance


class Singleton1(metaclass=SingletonMeta):
    """
    使用单例元类，使类的对象仅存在一个
    """
    pass


class SingletonObject:
    """
    实现单例基类
    """

    _instance: Optional[Singleton2] = None

    def __new__(cls, *args, **kwargs) -> Singleton2:
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance


class Singleton2(SingletonObject):
    """
    继承单例基类，使类的对象仅存在一个
    """
    pass


if __name__ == "__main__":
    s1 = Singleton1()
    s2 = Singleton1()
    print(f's1 id is {id(s1)} and s2 id is {id(s2)}, singleton1 works')

    s3 = Singleton2(1, ok=1)
    s4 = Singleton2(2, ok=2)
    print(f's3 id is {id(s3)} and s4 id is {id(s4)}, singleton2 works')
```

结果：
> s1 id is 4468969616 and s2 id is 4468969616, singleton1 works
> s3 id is 4468934480 and s4 id is 4468934480, singleton2 works

## 结构型
### 适配器模式
解决：
- 客户端代码可以通过封装了不兼容的对象的适配器对象，来对该不兼容的对象进行访问

意图：
- 定义一个适配器类，通过继承目标抽象类，并实现其方法为调用被适配对象的方法，并将返回结果转化可被识别的结果
- 通过指定被适配对象实例化得到适配器对象，统一按照目标对象的使用方式来使用适配器对象即可

缺点：
- 代码整体复杂度增加，因为需要新增一系列类，有时直接更改适被配器类使其与其他代码兼容会更简单

实现：
``` py
from __future__ import annotations


class Target():
    """
    目标类，客户端通过目标对象获取结果
    """

    def request(self) -> str:
        return "target result"


class Adaptee:
    """
    被适配类，其调用方式和目标类不同，且结果倒序
    """

    def specific_request(self) -> str:
        return "tluser laiceps eetpada"


class Adapter(Target):
    """
    适配器类，用于使被适配类符合目标类的调用方式，且将结果还原
    """

    def __init__(self, adaptee: Adaptee) -> None:
        self.adaptee = adaptee

    def request(self) -> str:
        return f"{self.adaptee.specific_request()[::-1]}"


def client_code(target: Target) -> None:
    print(target.request())


if __name__ == "__main__":
    print("Client: I can work just fine with the Target objects:")
    target = Target()
    client_code(target)
    print("")

    print(f"Client: I can't work with Adaptee, But I can work with it via the Adapter:")
    adaptee = Adaptee()
    adapter = Adapter(adaptee)
    client_code(adapter)
```

结果：
> Client: I can work just fine with the Target objects:
> target result
>
> Client: I can't work with Adaptee, But I can work with it via the Adapter:
> adaptee special result

### 桥接模式
解决：
- 客户端代码通过为具体的抽象层对象指定实现层对象，调用抽象层对象的方法可以执行相同处理逻辑不同处理细节的操作

意图：
- 将一个大类或一系列紧密相关的类拆分为抽象和实现两个独立的层次结构
- 抽象层对象的方法将包含对实现层对象的方法的调用
- 两个层对象都有都有一系列通用抽象方法，因此抽象层能被统一调用，且实现层能够在抽象层的内部相互替换

缺点：
- 对高内聚的类，即方法之间的联系程度很高的类，使用该模式可能会让代码更加复杂

实现：
``` py
from __future__ import annotations
from abc import ABC, abstractmethod


class Abstraction:
    """
    抽象层类，定义了用于控制实现过程的方法，并将所有实现通过调用实现层对象完成
    """

    def __init__(self, implementation: Implementation) -> None:
        self.implementation = implementation

    def operation(self) -> str:
        return (f"Abstraction: Base operation with:\n"
                f"{self.implementation.operation_implementation()}")


class ExtendedAbstraction(Abstraction):
    """
    可以扩展抽象层类的方法，而不必关心实现层对象
    """

    def operation(self) -> str:
        return (f"ExtendedAbstraction: Extended operation with:\n"
                f"{self.implementation.operation_implementation()}")


class Implementation(ABC):
    """
    抽象实现层类，声明了所有的实现抽象方法
    """

    @abstractmethod
    def operation_implementation(self) -> str:
        pass


"""
具体实现层类，实现了声明的实现抽象方法
"""


class ConcreteImplementationA(Implementation):
    def operation_implementation(self) -> str:
        return "ConcreteImplementationA: Here's the result on the platform A."


class ConcreteImplementationB(Implementation):
    def operation_implementation(self) -> str:
        return "ConcreteImplementationB: Here's the result on the platform B."


def client_code(abstraction: Abstraction) -> None:
    """
    客户端代码只依赖于抽象层类，
    """

    print(abstraction.operation(), end="")


if __name__ == "__main__":
    """
    在初始化阶段，实现层对象关联到抽象层对象关上
    """

    implementation = ConcreteImplementationA()
    abstraction = Abstraction(implementation)
    client_code(abstraction)

    print("\n")

    implementation = ConcreteImplementationB()
    abstraction = ExtendedAbstraction(implementation)
    client_code(abstraction)
```

结果：
> Abstraction: Base operation with:
> ConcreteImplementationA: Here's the result on the platform A.
>
> ExtendedAbstraction: Extended operation with:
> ConcreteImplementationB: Here's the result on the platform B.

### 组合模式
解决：
- 客户端代码可以把独立组件按树状结构组合成组合对象，并且可以像使用独立对象一样使用它们

意图：
- 定义一个组件抽象类，声明了统一的抽象方法，独立组件类和组合类都实现了这些方法
- 对于绝大多数需要生成树状结构的问题，生成组合对象都是非常有效的方法 
- 组合对象最主要的功能是能在整个树状结构上递归调用子级的方法并对结果进行汇总

缺点：
- 对于功能差异较大的独立类，提供公共接口或许会有困难

实现：
``` py
from __future__ import annotations
from abc import ABC, abstractmethod
from typing import List


class Component(ABC):
    """
    组件抽象类，声明了通用的方法
    """

    def __init__(self) -> None:
        self._parent = None

    @property
    def parent(self) -> Component:
        return self._parent

    @parent.setter
    def parent(self, parent: Component):
        """
        用于设置当前组件的父级组件
        """
        self._parent = parent

    """
    管理子级的方法，对于叶级组件，这些方法为空
    """

    def add(self, component: Component) -> None:
        pass

    def remove(self, component: Component) -> None:
        pass

    def is_composite(self) -> bool:
        """
        用于确定该组件是否能拥有子级
        """

        return False

    @abstractmethod
    def operation(self) -> str:
        """
        组件进行操作的抽象方法
        """
        pass


class Leaf(Component):
    """
    叶级类，即独立组件类，无子级的具体组件类，且通常是用于进行操作的
    """

    def operation(self) -> str:
        return "Leaf"


class Composite(Component):
    """
    组合类，可有子级的具体组件类
    """

    def __init__(self) -> None:
        self._children: List[Component] = []

    """
    组合对象可以对子级列表中的子级组件进行管理
    """

    def add(self, component: Component) -> None:
        self._children.append(component)
        component.parent = self

    def remove(self, component: Component) -> None:
        self._children.remove(component)
        component.parent = None

    def is_composite(self) -> bool:
        return True

    def operation(self) -> str:
        """
        组合对象以特定方式执行操作，一般会遍历所有子级，并收集和汇总其结果，子级依此类推，将遍历整个对象树
        """

        results = []
        for child in self._children:
            results.append(child.operation())
        return f"Branch({'+'.join(results)})"


def client_code(component1: Component, component2: Component) -> None:
    """
    客户端代码通过组件对象的基础方法进行操作
    """
    print("Client: Now I've got a composite tree:")
    print(f"RESULT: {component1.operation()}\n")

    if component1.is_composite():
        component1.add(component2)

    print("Client: I don't need to check the components classes even when managing the tree:")
    print(f"RESULT: {component1.operation()}")


if __name__ == "__main__":
    tree = Composite()

    branch1 = Composite()
    branch1.add(Leaf())
    branch1.add(Leaf())

    tree.add(branch1)

    branch2 = Composite()
    branch2.add(Leaf())

    client_code(tree, branch2)
```

结果：
> Client: Now I've got a composite tree:
> RESULT: Branch(Branch(Leaf+Leaf))

> Client: I don't need to check the components classes even when managing the tree:
> RESULT: Branch(Branch(Leaf+Leaf)+Branch(Leaf))

### 装饰模式
解决：
- 可以无数次地将原对象进行装饰得到一个装饰对象，为原对象的方法的新增行为，客户端代码可以按照原对象的方式使用装饰对象

意图：
- 定义一个组件抽象类，声明了统一的抽象方法，具体组件类和装饰类都实现了这些方法
- 通过指定被装饰的原对象进行装饰类实例化，得到装饰对象，装饰对象的方法包含对原对象的同名方法的引用
- 可以将原对象通过多个装饰器类进行多次封装，得到的装饰对象将获得所有装饰器类叠加而来的行为

缺点：
- 在对象的装饰栈中删除指定的装饰器比较困难
- 实现行为不受装饰栈顺序影响的装饰比较困难

实现：
``` py
from __future__ import annotations
from abc import ABC, abstractmethod


class Component(ABC):
    """
    组件抽象类，声明抽象方法，可被装饰器进行装饰
    """

    @abstractmethod
    def operation(self) -> str:
        pass


class ConcreteComponent(Component):
    """
    组件具体类，实现声明的抽象方法
    """

    def operation(self) -> str:
        return "ConcreteComponent"


class Decorator(Component):
    """
    基础装饰器类，对于组件抽象类声明的方法，根据装饰的组件对象提供默认实现类
    """

    _component: Component = None

    def __init__(self, component: Component) -> None:
        self._component = component

    @property
    def component(self) -> Component:
        return self._component

    def operation(self) -> str:
        return self._component.operation()


"""
具体装饰器类，具体对被装饰组件的方法进行封装
"""

class ConcreteDecoratorA(Decorator):

    def operation(self) -> str:
        """
        调用被装饰组件的方法，也可以通过父类进行调用
        """
        return f"ConcreteDecoratorA({self.component.operation()})"


class ConcreteDecoratorB(Decorator):

    def operation(self) -> str:
        return f"ConcreteDecoratorB({self.component.operation()})"


def client_code(component: Component) -> None:
    """
    根据组件抽象类的声明方法进行操作，不需要关心装饰器的存在
    """

    print(f"RESULT: {component.operation()}", end="")


if __name__ == "__main__":
    simple = ConcreteComponent()
    print("Client: I've got a simple component:")
    client_code(simple)
    print("\n")

    decorator1 = ConcreteDecoratorA(simple)
    decorator2 = ConcreteDecoratorB(decorator1)
    print("Client: Now I've got a decorated component:")
    client_code(decorator2)
```

结果：
> Client: I've got a simple component:
> RESULT: ConcreteComponent
> 
> Client: Now I've got a decorated component:
> RESULT: ConcreteDecoratorB(ConcreteDecoratorA(ConcreteComponent))

### 外观模式
解决：
- 客户端代码能够通过简单方法调用，来联合使用多个复杂的子系统或者框架，不需要关心其内部的交互细节

意图：
- 定义一个外观类，其实例化时需要设置其子系统对象，通过外观对象的方法，可以简化地完成对所有子系统的联合调用
- 外观模式降低了程序的整体复杂度，但它同时也有助于将调用方不需要关注的依赖移动到同一个位置
- 每个子系统也可以被单独地调用，外观对象只是它的另一个客户端

缺点：
- 外观类可能成为与程序中所有对象都依赖的类

实现：
``` py
from __future__ import annotations


class Facade:
    """
    外观类，提供封装了使用多个子系统的复杂过程的简单方法
    """

    def __init__(self, subsystem1: Subsystem1, subsystem2: Subsystem2) -> None:

        self._subsystem1 = subsystem1 or Subsystem1()
        self._subsystem2 = subsystem2 or Subsystem2()

    def operation(self) -> str:
        """
        对多个子系统复杂逻辑进行封装操作
        """

        results = []
        results.append("Facade initializes subsystems:")
        results.append(self._subsystem1.operation1())
        results.append(self._subsystem2.operation1())
        results.append("Facade orders subsystems to perform the action:")
        results.append(self._subsystem1.operation_n())
        results.append(self._subsystem2.operation_z())
        return "\n".join(results)


"""
子系统类，可以直接接受来自外观类或客户端的调用，一些外观类可以并发调用多个子系统
外观类是子系统的另一个客户端，不是子系统的一部分
"""


class Subsystem1:

    def operation1(self) -> str:
        return "Subsystem1: Ready!"

    def operation_n(self) -> str:
        return "Subsystem1: Go!"


class Subsystem2:

    def operation1(self) -> str:
        return "Subsystem2: Ready!"

    def operation_z(self) -> str:
        return "Subsystem2: Go!"


def client_code(facade: Facade) -> None:
    """
    客户端代码通过外观类提供的简单方法，来完成使用多个子系统的复杂过程，不关心子系统的存在
    """

    print(facade.operation(), end="")


if __name__ == "__main__":
    subsystem1 = Subsystem1()
    subsystem2 = Subsystem2()
    facade = Facade(subsystem1, subsystem2)
    client_code(facade)
```

结果：
> Facade initializes subsystems:
> Subsystem1: Ready!
> Subsystem2: Ready!
> Facade orders subsystems to perform the action:
> Subsystem1: Go!
> Subsystem2: Go!

### 享元模式
解决：
- 允许消耗少量内存的情况下支持大量对象，将每个对象公共的数据通过享元对象来保存起来，然后独立的数据再保存在各自的内存中

意图：
- 摒弃了在每个对象中保存所有数据的方式，而是通过共享多个对象所共有的相同状态，让你能在有限的内存容量中载入更多对象
- 每个享元对象保存一份公共数据，通过享元工厂来对享元对象进行获取和创建
- 通过传入独立数据来调用享元对象的方法，来执行对于完整数据的操作

缺点：
- 可能需要牺牲执行速度来换取内存，每次调用享元方法时都需要重新计算独立部分的数据
- 代码会变得更加复杂，因为涉及对于对象状态的拆分

实现：
``` py
import json
from typing import Dict, List


class Flyweight:
    """
    享元类，用于存储状态的公共部分，或称为固有状态，这些状态属于多个实体
    """

    def __init__(self, shared_state: List) -> None:
        self._shared_state = shared_state

    def operation(self, unique_state: List) -> None:
        """
        方法通过参数接受状态的其余部分，又称为外部状态，对于每个实体都是唯一的
        """
        s = json.dumps(self._shared_state)
        u = json.dumps(unique_state)
        print(f"Flyweight: Displaying shared ({s}) and unique ({u}) state.", end="")


class FlyweightFactory:
    """
    享元工厂类，创建并管理享元对象，确保享元对象正确地共享状态
    当客户端请求一个享元对象时，将
    """

    _flyweights: Dict[str, Flyweight] = {}

    def __init__(self, initial_flyweights: List[List]) -> None:
        for state in initial_flyweights:
            self._flyweights[self.get_key(state)] = Flyweight(state)

    @staticmethod
    def get_key(shared_state: List) -> str:
        """
        返回为状态生成的键
        """
        return "_".join(sorted(shared_state))

    def get_flyweight(self, shared_state: List) -> Flyweight:
        """
        返回一个现有或新建一个享元对象
        """
        key = self.get_key(shared_state)

        if not self._flyweights.get(key):
            print("FlyweightFactory: Can't find a flyweight, creating new one.")
            self._flyweights[key] = Flyweight(shared_state)
        else:
            print("FlyweightFactory: Reusing existing flyweight.")

        return self._flyweights[key]

    def list_flyweights(self) -> None:
        count = len(self._flyweights)
        print(f"FlyweightFactory: I have {count} flyweights:")
        print("\n".join(map(str, self._flyweights.keys())), end="")


def client_code(factory: FlyweightFactory, plates: str, owner: str,
                brand: str, model: str, color: str) -> None:
    """
    客户端代码通过享元工厂得到享元对象，再调用享元对象的方法并传入外部状态来对所有状态来进行操作
    """
    print("\n\nClient: Adding a car to database.")
    flyweight = factory.get_flyweight([brand, model, color])
    flyweight.operation([plates, owner])


if __name__ == "__main__":
    """
    The client code usually creates a bunch of pre-populated flyweights in the
    initialization stage of the application.
    """

    factory = FlyweightFactory([
        ["Chevrolet", "Camaro2018", "pink"],
        ["Mercedes Benz", "C300", "black"],
        ["Mercedes Benz", "C500", "red"],
        ["BMW", "M5", "red"],
        ["BMW", "X6", "white"],
    ])

    factory.list_flyweights()

    client_code(
        factory, "CL234IR", "James Doe", "BMW", "M5", "red")

    client_code(
        factory, "CL234IR", "James Doe", "BMW", "X1", "red")

    print("\n")
    factory.list_flyweights()
```

结果：
> FlyweightFactory: I have 5 flyweights:
Camaro2018_Chevrolet_pink
C300_Mercedes Benz_black
C500_Mercedes Benz_red
BMW_M5_red
BMW_X6_white
>
> Client: Adding a car to database.
FlyweightFactory: Reusing existing flyweight.
Flyweight: Displaying shared (["BMW", "M5", "red"]) and unique (["CL234IR", "James Doe"]) state.
>
> Client: Adding a car to database.
FlyweightFactory: Can't find a flyweight, creating new one.
Flyweight: Displaying shared (["BMW", "X1", "red"]) and unique (["CL234IR", "James Doe"]) state.
>
> FlyweightFactory: I have 6 flyweights:
Camaro2018_Chevrolet_pink
C300_Mercedes Benz_black
C500_Mercedes Benz_red
BMW_M5_red
BMW_X6_white
BMW_X1_red

### 代理模式
解决：
- 代理对象可以控制着对原对象的方法的调用，在其调用前后进行额外的处理，客户端代码按照原对象的使用方式使用代理对象

意图：
- 在主题抽象类中定义一些公共的方法，真实主题类和代理类实现了这些方法，并且代理类的方法中包含了对真实主题类的方法调用
- 代理类在调用真实主题类的方法的前后进行额外的处理，如延迟加载，缓存，控制访问，日志记录等
- 装饰模式和代理模式能为实现和原对象相同的方法，且实现方式相同，但装饰模式是为加强方法的行为，代理模式则是控制方法的调用，两者的意图不同

缺点：
- 代码可能会变得复杂，因为需要新建许多类
- 程序对于请求的响应可能会延迟

实现：
``` py
from abc import ABC, abstractmethod


class Subject(ABC):
    """
    主题抽象类，声明了真实主题和代理的公共抽象方法
    """

    @abstractmethod
    def request(self) -> None:
        pass


class RealSubject(Subject):
    """
    真实主题类，用于业务处理
    """

    def request(self) -> None:
        print("RealSubject: Handling request.")


class Proxy(Subject):
    """
    代理类，初始化需要绑定真实主题对象，也可以通过继承真实主题类来实现代理类
    """

    def __init__(self, real_subject: RealSubject) -> None:
        self._real_subject = real_subject

    def request(self) -> None:
        """
        代理业务处理方法，最常见是延迟加载，缓存，控制访问，日志记录等
        """

        if self.check_access():
            self._real_subject.request()
            self.log_access()

    @staticmethod
    def check_access() -> bool:
        print("Proxy: Checking access prior to firing a real request.")
        return True

    @staticmethod
    def log_access() -> None:
        print("Proxy: Logging the time of request.", end="")


def client_code(subject: Subject) -> None:
    """
    客户端代码通过主题声明的方法，可同时支持真实主题对象和代理对象
    """

    subject.request()


if __name__ == "__main__":
    print("Client: Executing the client code with a real subject:")
    real_subject = RealSubject()
    client_code(real_subject)

    print("")

    print("Client: Executing the same client code with a proxy:")
    proxy = Proxy(real_subject)
    client_code(proxy)
```

结果：
> Client: Executing the client code with a real subject:
RealSubject: Handling request.
>
> Client: Executing the same client code with a proxy:
Proxy: Checking access prior to firing a real request.
RealSubject: Handling request.
Proxy: Logging the time of request.

## 行为型
### 责任链模式
解决：
- 客户端代码将请求交给处理者链中的一个对象，可以允许该对象在处理者链后的多个对象收到请求，进行递交、修改或处理，而无需调用具体的处理者对象

意图：
- 将请求按照处理者链进行发送，收到请求的每个处理者均可对请求进行递交、修改或处理，直到其中一个处理者对请求进行了处理
- 每个具体处理者类都实现了进行设置下个处理者对象和处理的公共方法，处理者链可在运行时由调用任意处理者对象的方法动态生成
- 发送者对象将请求发给任意处理者，都可以该对象在处理者链后的多个对象对请求进行接收，无需与具体接收者对象相耦合

缺点：
- 如果处理者链最后没有一个接收所有请求的处理者，部分请求可能未被任何处理者处理

实现：
``` py
from __future__ import annotations
from abc import ABC, abstractmethod
from typing import Any, Optional


class Handler(ABC):
    """
    处理抽象类，声明抽象方法，用于处理请求
    """

    @abstractmethod
    def set_next(self, handler: Handler) -> Handler:
        pass

    @abstractmethod
    def handle(self, request) -> Optional[str]:
        pass


class BaseHandler(Handler):
    """
    基础处理类，为处理抽象类声明的部分抽象方法提供基本实现，但处理方法还是抽象的，需要被子类继续实现
    """

    _next_handler: Handler = None

    def set_next(self, handler: Handler) -> Handler:
        """
        设置在处理链上，该处理对象的下个处理对象
        可以链式调用设置：monkey.set_next(squirrel).set_next(dog)
        """
        self._next_handler = handler
        return handler

    @abstractmethod
    def handle(self, request: Any) -> str:
        """
        若在处理链上，存在下个处理对象，则递交请求给下个处理对象
        """
        if self._next_handler:
            return self._next_handler.handle(request)


"""
具体处理类，其处理方法接收到请求后，会处理或者递交一个请求给下个处理对象
"""


class MonkeyHandler(BaseHandler):
    def handle(self, request: Any) -> str:
        if request == "Banana":
            return f"Monkey: I'll eat the {request}"
        else:
            return super().handle(request)


class SquirrelHandler(BaseHandler):
    def handle(self, request: Any) -> str:
        if request == "Nut":
            return f"Squirrel: I'll eat the {request}"
        else:
            return super().handle(request)


class DogHandler(BaseHandler):
    def handle(self, request: Any) -> str:
        if request == "MeatBall":
            return f"Dog: I'll eat the {request}"
        else:
            return super().handle(request)


def client_code(handler: Handler) -> None:
    """
    客户端代码通常使用单个处理对象一起使用，不关心处理对象是处理链的一部分
    """

    for food in ["Nut", "Banana", "Cup of coffee"]:
        print(f"\nClient: Who wants a {food}?")
        result = handler.handle(food)
        if result:
            print(f"  {result}", end="")
        else:
            print(f"  {food} was left untouched.", end="")


if __name__ == "__main__":
    monkey = MonkeyHandler()
    squirrel = SquirrelHandler()
    dog = DogHandler()

    monkey.set_next(squirrel).set_next(dog)
    print("Chain: Monkey > Squirrel > Dog")

    # 允许传递任意处理对象到客户端，不仅仅是处理链的首个处理对象
    client_code(monkey)
    print("\n")

    print("Subchain: Squirrel > Dog")
    client_code(squirrel)
```

结果：
> Chain: Monkey > Squirrel > Dog
>
> Client: Who wants a Nut?
Squirrel: I'll eat the Nut
Client: Who wants a Banana?
Monkey: I'll eat the Banana
Client: Who wants a Cup of coffee?
Cup of coffee was left untouched.
>
> Subchain: Squirrel > Dog
>
> Client: Who wants a Nut?
Squirrel: I'll eat the Nut
Client: Who wants a Banana?
Banana was left untouched.
Client: Who wants a Cup of coffee?
Cup of coffee was left untouched.

### 命令模式
解决：
- 客户端代码可以将多个操作转换分别命令对象，以属性的形式保存在触发对象中，通过调用触发对象的方法来控制这些命令的执行

意图：
- 在命令抽象类中定义统一的执行方法，具体命令类是对这些方法的实现，触发对象用于保存各个具体命令对象
- 定义触发对象不同的方法，对应着不同命令对象的不同方法的使用，可以实现命令组合、命令撤销，命令记录
- 命令对象初始化时可以传入命令参数，对于复杂命令对象初始化时还需要传入接收者对象和相关命令参数

缺点：
- 代码可能会变得更加复杂，因为调用方和执行方之间增加了一个命令的层次

实现：
``` py
from __future__ import annotations
from abc import ABC, abstractmethod


class Command(ABC):
    """
    命令抽象类，声明执行命令的抽象方法
    """

    @abstractmethod
    def execute(self) -> None:
        pass


class SimpleCommand(Command):
    """
    简单命令类，实现只通过自身的简单执行方法
    """

    def __init__(self, payload: str) -> None:
        self._payload = payload

    def execute(self) -> None:
        print(f"SimpleCommand: See, I can do simple things like printing"
              f"({self._payload})")


class ComplexCommand(Command):
    """
    复杂命令类，实现需要调用接收器对象的复杂执行方法
    """

    def __init__(self, receiver: Receiver, a: str, b: str) -> None:
        self._receiver = receiver
        self._a = a
        self._b = b

    def execute(self) -> None:
        print("ComplexCommand: Complex stuff should be done by a receiver object", end="")
        self._receiver.do_something(self._a)
        self._receiver.do_something_else(self._b)


class Receiver:
    """
    接收器类，包含了一些完成重要操作的方法
    """

    def do_something(self, a: str) -> None:
        print(f"\nReceiver: Working on ({a}.)", end="")

    def do_something_else(self, b: str) -> None:
        print(f"\nReceiver: Also working on ({b}.)", end="")


class Invoker:
    """
    触发类，用于保存一个或多个命令，然后按照顺序发送执行请求到命令
    """

    _on_start = None
    _on_finish = None

    def set_on_start(self, command: Command):
        self._on_start = command

    def set_on_finish(self, command: Command):
        self._on_finish = command

    def do_something_important(self) -> None:
        """
        触发类的执行方法不关心具体命令对象
        """

        print("Invoker: Does anybody want something done before I begin?")
        self._on_start.execute()

        print("Invoker: ...doing something really important...")

        print("Invoker: Does anybody want something done after I finish?")
        self._on_finish.execute()


if __name__ == "__main__":
    invoker = Invoker()
    invoker.set_on_start(SimpleCommand("Say Hi!"))
    receiver = Receiver()
    invoker.set_on_finish(ComplexCommand(
        receiver, "Send email", "Save report"))

    invoker.do_something_important()
```

结果：
> Invoker: Does anybody want something done before I begin?
SimpleCommand: See, I can do simple things like printing(Say Hi!)
Invoker: ...doing something really important...
Invoker: Does anybody want something done after I finish?
ComplexCommand: Complex stuff should be done by a receiver object
Receiver: Working on (Send email.)
Receiver: Also working on (Save report.)

### 迭代器模式
解决：
- 客户端代码为包含多个元素结构生成一个可迭代对象，通过迭代该对象可以遍历该结构的所有元素，不需要关心这个结果的底层表现形式

意图：
- 实现一个可迭代类和迭代器类，可迭代类在实例化时接收包含多个元素结构
- 可迭代对象实现了被 `iter()` 操作时会返回一个迭代器对象，迭代器对象被 `next()` 操作时会返回下一个元素
- 在不暴露集合底层表现形式（列表、 栈和树等）的情况下，通过可迭代对象提供遍历所有元素的方式

缺点：
- 只与简单的集合进行交互，应用该模式可能会使得更加复杂
- 对于某些特殊集合，使用迭代器可能比直接遍历的效率低

实现：
``` py
from __future__ import annotations
from collections.abc import Iterable, Iterator
from typing import Any, List


"""
collections 标准库中有两个和迭代器相关的抽象类 Iterable、Iterator
通过实现 Iterable 中 __iter __（）方法来得到可迭代类，实现为 Iterator 中的 __next__（）方法来得到迭代器类
"""


class AlphabeticalIterator(Iterator):
    """
    字母迭代器类，以下可设置用于记录迭代状态信息的属性，可以根据迭代的复杂度设置很多这样的属性
    分别表示迭代器的当前位置信息、是否倒序迭代
    """
    _position: int = None
    _reverse: bool = False

    def __init__(self, collection: List[Any], reverse: bool = False) -> None:
        self._collection = collection
        self._reverse = reverse
        self._position = -1 if reverse else 0

    def __next__(self):
        """
        被调用 next() 迭代元素时时自动调用，返回下一个元素
        """
        try:
            value = self._collection[self._position]
            self._position += -1 if self._reverse else 1
        except IndexError:
            raise StopIteration()

        return value


class WordsCollection(Iterable):
    """
    文字收集类，是可迭代对象，收集文字内容后用于生成迭代器对象
    """

    def __init__(self, collection: List[Any] = list()) -> None:
        self._collection = collection

    def __iter__(self) -> AlphabeticalIterator:
        """
        被通过 iter() 获取迭代器时自动调用，返回由内容生成的迭代器对象
        """
        return AlphabeticalIterator(self._collection)

    def get_reverse_iterator(self) -> AlphabeticalIterator:
        return AlphabeticalIterator(self._collection, True)

    def add_item(self, item: Any):
        self._collection.append(item)


if __name__ == "__main__":
    # 客户端代码不需要关心具体的迭代器对象
    collection = WordsCollection()
    collection.add_item("First")
    collection.add_item("Second")
    collection.add_item("Third")

    print("Straight traversal:")
    print("\n".join(collection))
    print("")

    print("Reverse traversal:")
    print("\n".join(collection.get_reverse_iterator()), end="")
```

结果：
> Straight traversal:
First
Second
Third
>
> Reverse traversal:
Third
Second
First

### 中介者模式
解决：
- 客户端代码、各个对象之间的相互访问，都必须通过中介者对象来进行，减少对象之间混乱无序的依赖关系

意图：
- 该模式会限制对象之间的直接交互，迫使它们通过一个中介者对象进行合作，达到减少组件之间依赖关系的目的
- 中介者初始化时会保存所有关联的组件对象到属性中，并且调用组件对象的方法时，会传入中介者对象本身，以便组件对象的方法通过中介者对象调用其他组件
- 中介者能使得程序更易于修改和扩展，而且能更方便地对独立的组件进行复用，因为它们不再依赖于很多其他的类

缺点：
- 和外观对象类似，中介者类也可能会成为程序中所有对象都耦合的类

实现：
``` py
from __future__ import annotations
from abc import ABC


class Mediator(ABC):
    """
    中介者抽象类，定义了抽象方法用于被其他组件通知，然后中介者作出反应
    """

    def notify(self, sender: object, event: str) -> None:
        pass


"""
具体中介者类，实现定义的抽象方法
"""


class ConcreteMediator(Mediator):
    def __init__(self, component1: Component1, component2: Component2) -> None:
        self._component1 = component1
        self._component1.mediator = self
        self._component2 = component2
        self._component2.mediator = self

    def notify(self, sender: object, event: str) -> None:
        if event == "A":
            print("Mediator reacts on A and triggers following operations:")
            self._component2.do_c()
        elif event == "D":
            print("Mediator reacts on D and triggers following operations:")
            self._component1.do_b()
            self._component2.do_c()


class BaseComponent:
    """
    基础组件类，定义管理中介者的属性方法
    """

    def __init__(self, mediator: Mediator = None) -> None:
        self._mediator = mediator

    @property
    def mediator(self) -> Mediator:
        return self._mediator

    @mediator.setter
    def mediator(self, mediator: Mediator) -> None:
        self._mediator = mediator


"""
具体组件类，实现了中介者作出反应时需要调用的操作方法，这些方法组件间不互相依赖
"""


class Component1(BaseComponent):
    def do_a(self) -> None:
        print("Component 1 does A.")
        self.mediator.notify(self, "A")

    def do_b(self) -> None:
        print("Component 1 does B.")
        self.mediator.notify(self, "B")


class Component2(BaseComponent):
    def do_c(self) -> None:
        print("Component 2 does C.")
        self.mediator.notify(self, "C")

    def do_d(self) -> None:
        print("Component 2 does D.")
        self.mediator.notify(self, "D")


if __name__ == "__main__":
    c1 = Component1()
    c2 = Component2()
    mediator = ConcreteMediator(c1, c2)

    print("Client triggers operation A.")
    c1.do_a()

    print("\n", end="")

    print("Client triggers operation D.")
    c2.do_d()
```

结果：
> Client triggers operation A.
Component 1 does A.
Mediator reacts on A and triggers following operations:
Component 2 does C.
> 
> Client triggers operation D.
Component 2 does D.
Mediator reacts on D and triggers following operations:
Component 1 does B.
Component 2 does C.

### 备忘录模式
解决：
- 客户端代码可以通过管理者对象，对发起者对象的状态进行备份、恢复等操作，而不需要关心其实现细节

意图：
- 通过备忘录对象作为发起者对象的状态的存储介质，通过管理者对象对存储介质进行管理，以及发起者对象对存储介质的应用进行管理
- 在不暴露发起者对象实现细节的情况下，完成了对发起者对象的保存、恢复和历史记录
- 备忘录对象不会影响应用它的对象的内部结构，也不会影响管理者对象中的历史数据

缺点：
- 如果客户端代码过于频繁地备份，程序将消耗大量内存
- 管理者对象必须完整跟踪发起者对象的生命周期，这样才能销毁弃用的备忘录
- 绝大部分动态编程语言（如 Python）不能确保备忘录对象中的状态不被修改

实现：
``` py
from __future__ import annotations
from abc import ABC, abstractmethod
from datetime import datetime
from random import sample
from string import ascii_letters


class Originator():
    """
    发起类，存在一些会被操作所变化的状态，并可以对状态通过备忘录对象进行保存和恢复
    """

    _state = None

    def __init__(self, state: str) -> None:
        self._state = state
        print(f"Originator: My initial state is: {self._state}")

    def do_something(self) -> None:
        """
        发起类的业务逻辑方法，会修改状态，因此调用之前应该先保存状态
        """
        print("Originator: I'm doing something important.")
        self._state = "".join(sample(ascii_letters, 30))
        print(f"Originator: and my state has changed to: {self._state}")

    def save(self) -> Memento:
        """
        将状态保存在备忘录，并返回
        """

        return ConcreteMemento(self._state)

    def restore(self, memento: Memento) -> None:
        """
        从备忘录恢复状态
        """

        self._state = memento.get_state()
        print(f"Originator: My state has changed to: {self._state}")


class Memento(ABC):
    """
    备忘录抽象类，定义抽象方法
    """

    @abstractmethod
    def get_state(self) -> str:
        pass

    @abstractmethod
    def get_name(self) -> str:
        pass

    @abstractmethod
    def get_date(self) -> str:
        pass


class ConcreteMemento(Memento):
    """
    具体备忘录类，实现了定义的抽象方法
    """

    def __init__(self, state: str) -> None:
        self._state = state
        self._date = str(datetime.now())[:19]

    def get_state(self) -> str:
        return self._state

    def get_name(self) -> str:
        return f"{self._date} / ({self._state[0:9]}...)"

    def get_date(self) -> str:
        return self._date


class Caretaker():
    """
    管理者类，不依赖具体的备忘录类，管理发起者对备忘录的应用
    """

    def __init__(self, originator: Originator) -> None:
        self._mementos = []
        self._originator = originator

    def backup(self) -> None:
        print("\nCaretaker: Saving Originator's state...")
        self._mementos.append(self._originator.save())

    def undo(self) -> None:
        if not len(self._mementos):
            return

        memento = self._mementos.pop()
        print(f"Caretaker: Restoring state to: {memento.get_name()}")
        try:
            self._originator.restore(memento)
        except Exception:
            self.undo()

    def show_history(self) -> None:
        print("Caretaker: Here's the list of mementos:")
        for memento in self._mementos:
            print(memento.get_name())


if __name__ == "__main__":
    originator = Originator("Super-duper-super-puper-super.")
    caretaker = Caretaker(originator)

    caretaker.backup()
    originator.do_something()

    caretaker.backup()
    originator.do_something()

    print("")
    caretaker.show_history()

    print("\nClient: Now, let's rollback!\n")
    caretaker.undo()
```

结果：
> Originator: My initial state is: Super-duper-super-puper-super.
> 
> Caretaker: Saving Originator's state...
Originator: I'm doing something important.
Originator: and my state has changed to: fBKTCtloLAaMQwvmXZrOxcHpFeuqsS
>
> Caretaker: Saving Originator's state...
Originator: I'm doing something important.
Originator: and my state has changed to: scGkJeyEKBdfSCTFitWmwjRqDUOrQV
>
> Caretaker: Here's the list of mementos:
2019-12-05 11:35:54 / (Super-dup...)
2019-12-05 11:35:54 / (fBKTCtloL...)
>
> Client: Now, let's rollback!
>
> Caretaker: Restoring state to: 2019-12-05 11:35:54 / (fBKTCtloL...)
Originator: My state has changed to: fBKTCtloLAaMQwvmXZrOxcHpFeuqsS

### 观察者模式
解决：
- 客户端代码可以充当事件的发布者，通过主题对象的方法触发事件，并通知到其关联的所有观察者对象，这些观察者对象分别会对该事件进行处理

意图：
- 定义一个主题类，可以通过方法关联和取消关联观察者对象，以及通知事件或者状态到所有关联的观察者对象
- 观察者抽象类定义了统一的接口，供所有主题对象进行通知事件时进行调用
- 调用方只需要对主题对象进行管理即可

缺点：
- 观察者对象的通知顺序是无序的

实现：
``` py
from __future__ import annotations
from abc import ABC, abstractmethod
from random import randrange
from typing import List


class Subject(ABC):
    """
    抽象主题类，定义了一些抽象方法
    """

    @abstractmethod
    def attach(self, observer: Observer) -> None:
        """
        关联观察者
        """
        pass

    @abstractmethod
    def detach(self, observer: Observer) -> None:
        """
        取消关联观察者
        """
        pass

    @abstractmethod
    def notify(self) -> None:
        """
        通知该主题下的相关观察者
        """
        pass


class ConcreteSubject(Subject):
    """
    具体主题类，实现声明的方法，定义一些属性用于保存信息
    """

    """
    状态信息和观察者列表
    """
    _state: int = None
    _observers: List[Observer] = []

    def attach(self, observer: Observer) -> None:
        print("Subject: Attached an observer.")
        self._observers.append(observer)

    def detach(self, observer: Observer) -> None:
        self._observers.remove(observer)

    def notify(self) -> None:
        for observer in self._observers:
            observer.update(self._state)

    def some_business_logic(self) -> None:
        """
        业务逻辑，修改状态后通过主题对象通知到所有观察者对象
        """

        print("\nSubject: I'm doing something important.")
        self._state = randrange(0, 10)

        print(f"Subject: My state has just changed to: {self._state}")
        self.notify()


class Observer(ABC):
    """
    观察者抽象类，声明统一的方法来被主题对象调用
    """

    @abstractmethod
    def update(self, subject: Subject) -> None:
        pass


"""
具体观察者类，实现了声明的方法
"""


class ConcreteObserverA(Observer):
    def update(self, state: int) -> None:
        if state < 5:
            print("ConcreteObserverA: Reacted to the event")


class ConcreteObserverB(Observer):
    def update(self, state: int) -> None:
        if state == 0 or state >= 5:
            print("ConcreteObserverB: Reacted to the event")


if __name__ == "__main__":
    subject = ConcreteSubject()

    observer_a = ConcreteObserverA()
    subject.attach(observer_a)

    observer_b = ConcreteObserverB()
    subject.attach(observer_b)

    subject.some_business_logic()
    subject.detach(observer_a)
    subject.some_business_logic()
```

结果：
> Subject: Attached an observer.
Subject: Attached an observer.
>
> Subject: I'm doing something important.
Subject: My state has just changed to: 0
ConcreteObserverA: Reacted to the event
ConcreteObserverB: Reacted to the event
> 
> Subject: I'm doing something important.
> Subject: My state has just changed to: 8
> ConcreteObserverB: Reacted to the event

### 状态模式
解决：
- 客户端代码可以通过改变上下文对象的状态来调用相同的方法但得到不同的行为，也可以通过其行为再对其状态进行改变

意图：
- 根据有限状态机的概念，实现多个具体状态对象，具体统一的方法
- 实现具体上下文类，其包含改变其状态对象，以及调用其关联的状态对象的方法，从而能其内部状态变化时改变其行为，使其看上去就像改变了自身的类型一样
- 该模式将与状态相关的行为抽取到独立的状态类中，让原对象将工作委派给这些状态对象，而不是自行进行处理

缺点：
- 如果状态机只有很少的几个状态，或者很少发生改变，那么应用该模式可能会显得小题大作

实现：
``` py
from __future__ import annotations
from abc import ABC, abstractmethod


class Context(ABC):
    """
    抽象上下文类，定义了抽象方法，提供给客户端使用，来调用状态对象
    """

    """
    上下文的当前状态
    """
    _state = None

    def __init__(self, state: State) -> None:
        self.transition_to(state)

    def transition_to(self, state: State):
        """
        更改状态对象
        """

        print(f"Context: Transition to {type(state).__name__}")
        self._state = state
        self._state.context = self

    """
    调用状态对象的处理方法
    """

    def request1(self):
        self._state.handle1()

    def request2(self):
        self._state.handle2()


class State(ABC):
    """
    抽象状态对象，定义了抽象方法，用于被上下文对象调用，同时也可以反向调用上下文对象
    """

    @property
    def context(self) -> Context:
        return self._context

    @context.setter
    def context(self, context: Context) -> None:
        self._context = context

    @abstractmethod
    def handle1(self) -> None:
        pass

    @abstractmethod
    def handle2(self) -> None:
        pass


"""
具体状态类，实现声明的方法
"""


class ConcreteStateA(State):
    def handle1(self) -> None:
        print("ConcreteStateA handles request1.")
        print("ConcreteStateA wants to change the state of the context.")
        self.context.transition_to(ConcreteStateB())

    def handle2(self) -> None:
        print("ConcreteStateA handles request2.")


class ConcreteStateB(State):
    def handle1(self) -> None:
        print("ConcreteStateB handles request1.")

    def handle2(self) -> None:
        print("ConcreteStateB handles request2.")
        print("ConcreteStateB wants to change the state of the context.")
        self.context.transition_to(ConcreteStateA())


if __name__ == "__main__":
    context = Context(ConcreteStateA())
    context.request1()
    context.request2()
```

结果：
> Context: Transition to ConcreteStateA
ConcreteStateA handles request1.
ConcreteStateA wants to change the state of the context.
Context: Transition to ConcreteStateB
ConcreteStateB handles request2.
ConcreteStateB wants to change the state of the context.
Context: Transition to ConcreteStateA

### 策略模式
解决：
- 客户端代码通过改变下文对象关联的策略对象，调用相同的方法，但执行不同的算法

意图：
- 实现一系列具体策略类，并将每种算法分别放入独立的策略类中，以使策略对象在上下文对象内部能够相互替换
- 上下文对象包含其关联的策略对象，以及将行为分派给策略对象
- 为了改变上下文完成其工作的方式，可以使用另一个策略对象来替换当前关联的策略对象

缺点：
- 如果你的算法极少发生改变，那么没有任何理由引入新的类和接口，使用该模式只会让程序过于复杂
- 客户端代码需要选择合适的策略，必须知道不同策略对象的区别
- 许多编程语言支持允许你在一组函数中实现不同版本的算法，这样使用这些函数的方式就和使用策略对象时完全相同，无需借助额外的类和接口来保持代码简洁

实现：
``` python
from __future__ import annotations
from abc import ABC, abstractmethod
from typing import List


class Context():
    """
    上下文类，定义了客户端使用的方法
    """

    def __init__(self, strategy: Strategy) -> None:
        """
        初始化和运行阶段都可以设置策略
        """
        self._strategy = strategy

    @property
    def strategy(self) -> Strategy:
        return self._strategy

    @strategy.setter
    def strategy(self, strategy: Strategy) -> None:
        self._strategy = strategy

    def do_some_business_logic(self) -> None:
        """
        业务逻辑，通过调用策略对象的方法完成
        """
        print("Context: Sorting data using the strategy (not sure how it'll do it)")
        result = self._strategy.do_algorithm(["a", "b", "c", "d", "e"])
        print(",".join(result))


class Strategy(ABC):
    """
    策略抽象类，定义了抽象方法，来完成业务逻辑
    """

    @abstractmethod
    def do_algorithm(self, data: List) -> List:
        pass


"""
具体策略类，实现了抽象方法
"""


class ConcreteStrategyA(Strategy):
    def do_algorithm(self, data: List) -> List:
        return sorted(data)


class ConcreteStrategyB(Strategy):
    def do_algorithm(self, data: List) -> List:
        return list(reversed(sorted(data)))


if __name__ == "__main__":
    # 通过更换策略可以改变业务逻辑的行为
    context = Context(ConcreteStrategyA())
    print("Client: Strategy is set to normal sorting.")
    context.do_some_business_logic()
    print()

    print("Client: Strategy is set to reverse sorting.")
    context.strategy = ConcreteStrategyB()
    context.do_some_business_logic()

```

结果：
> Client: Strategy is set to normal sorting.
Context: Sorting data using the strategy (not sure how it'll do it)
a,b,c,d,e
> 
> Client: Strategy is set to reverse sorting.
Context: Sorting data using the strategy (not sure how it'll do it)
e,d,c,b,a

### 模版方法模式
解决：
- 客户端可以通过基础定义执行框架的父类，来实现执行框架中的各个步骤的方法，来改变执行框架的具体行为

意图：
- 在抽象类中定义好模版方法，即执行框架，其中包含对于其他步骤方法的调用
- 在其子类中实现各个步骤方法，在不修改模版方法的情况下改变了执行框架的行为

缺点：
- 客户端代码可能会受到执行框架的限制
- 模板方法中调用的步骤方法越多，其维护工作就可能会越困难

实现：
``` py
from abc import ABC, abstractmethod


class AbstractClass(ABC):
    """
    抽象类，定义了一个模版方法，包含调用多个操作的框架
    """

    def template_method(self) -> None:

        self.base_operation1()
        self.required_operations1()
        self.base_operation2()
        self.hook1()
        self.required_operations2()
        self.base_operation3()
        self.hook2()

    # 已实现的方法

    def base_operation1(self) -> None:
        print("AbstractClass says: I am doing the bulk of the work")

    def base_operation2(self) -> None:
        print("AbstractClass says: But I let subclasses override some operations")

    def base_operation3(self) -> None:
        print("AbstractClass says: But I am doing the bulk of the work anyway")

    # 具体子类需要实现的方法

    @abstractmethod
    def required_operations1(self) -> None:
        pass

    @abstractmethod
    def required_operations2(self) -> None:
        pass

    # 钩子方法，子类可以利用钩子，在模版方法执行过程的关键点进行扩展

    def hook1(self) -> None:
        pass

    def hook2(self) -> None:
        pass


"""
具体子类应该实现抽象类的模版方法中调用的操作，但是保持模板方法本身的完整性
"""


class ConcreteClass1(AbstractClass):

    def required_operations1(self) -> None:
        print("ConcreteClass1 says: Implemented Operation1")

    def required_operations2(self) -> None:
        print("ConcreteClass1 says: Implemented Operation2")


class ConcreteClass2(AbstractClass):

    def required_operations1(self) -> None:
        print("ConcreteClass2 says: Implemented Operation1")

    def required_operations2(self) -> None:
        print("ConcreteClass2 says: Implemented Operation2")

    def hook1(self) -> None:
        print("ConcreteClass2 says: Overridden Hook1")


def client_code(abstract_class: AbstractClass) -> None:
    """
    客户端通过调用模版方法来执行
    """

    abstract_class.template_method()


if __name__ == "__main__":
    print("Same client code can work with different subclasses:")
    client_code(ConcreteClass1())
    print("")

    print("Same client code can work with different subclasses:")
    client_code(ConcreteClass2())
``` 

结果：
> Client: Strategy is set to normal sorting.
Context: Sorting data using the strategy (not sure how it'll do it)
a,b,c,d,e
> 
> Client: Strategy is set to reverse sorting.
Context: Sorting data using the strategy (not sure how it'll do it)
e,d,c,b,a

### 访问者模式
解决：
- 客户端代码通过对指定组件对象的方法传入不同的访问者对象，来调用不同访问者对象的指定方法

意图：
- 定义用于执行指定方法的组件对象，其方法是接收访问者对象并对其指定方法进行调用
- 访问者对象实现了所有组件对象分别要调用的方法，因此调用方可以不关心访问者对象的具体类型，将其传入组件对象的方法中即可
- 该模式将要执行的方法和该方法的所属访问者对象隔离开
- 组件方法在调用访问者对象的方法时也传入了组件对象本身，因此可以进行反向调用

缺点：
- 每次在组件层次结构中添加或移除一个类时，都要更新所有的访问者类
- 在访问者同某个组件对象进行交互时，它们可能没有访问组件对象的私有变量

实现：
``` py
from __future__ import annotations
from abc import ABC, abstractmethod
from typing import List


class Component(ABC):
    """
    组件抽象类，声明了一个接收访问者对象的抽象方法
    """

    @abstractmethod
    def accept(self, visitor: Visitor) -> None:
        pass


"""
具体组件类，实现抽象方法，对访问者调用时同时传入本身作为参数，也可以提供方法让访问者对象调用
"""

class ConcreteComponentA(Component):

    def accept(self, visitor: Visitor) -> None:
        visitor.visit_concrete_component_a(self)

    def exclusive_method_of_concrete_component_a(self) -> str:
        return "A"


class ConcreteComponentB(Component):

    def accept(self, visitor: Visitor):
        visitor.visit_concrete_component_b(self)

    def special_method_of_concrete_component_b(self) -> str:
        return "B"


class Visitor(ABC):
    """
    访问者抽象类，声明在具体组件类对访问者对象调用的抽象方法
    """

    @abstractmethod
    def visit_concrete_component_a(self, element: ConcreteComponentA) -> None:
        pass

    @abstractmethod
    def visit_concrete_component_b(self, element: ConcreteComponentB) -> None:
        pass


"""
具体访问者类，实现了抽象方法
"""


class ConcreteVisitor1(Visitor):
    def visit_concrete_component_a(self, element) -> None:
        print(f"{element.exclusive_method_of_concrete_component_a()} + ConcreteVisitor1")

    def visit_concrete_component_b(self, element) -> None:
        print(f"{element.special_method_of_concrete_component_b()} + ConcreteVisitor1")


class ConcreteVisitor2(Visitor):
    def visit_concrete_component_a(self, element) -> None:
        print(f"{element.exclusive_method_of_concrete_component_a()} + ConcreteVisitor2")

    def visit_concrete_component_b(self, element) -> None:
        print(f"{element.special_method_of_concrete_component_b()} + ConcreteVisitor2")


def client_code(components: List[Component], visitor: Visitor) -> None:
    """
    客户端代码可以在多个组件对象中，对访问者对象进行操作，如对于复杂的对象结构（如树结构），可在结构的各种对象上对访问者对象进行操作
    """

    for component in components:
        component.accept(visitor)


if __name__ == "__main__":
    components = [ConcreteComponentA(), ConcreteComponentB()]

    print("The client code works with all visitors via the base Visitor interface:")
    visitor1 = ConcreteVisitor1()
    client_code(components, visitor1)

    print("It allows the same client code to work with different types of visitors:")
    visitor2 = ConcreteVisitor2()
    client_code(components, visitor2)
```

结果：
> The client code works with all visitors via the base Visitor interface:
A + ConcreteVisitor1
B + ConcreteVisitor1
It allows the same client code to work with different types of visitors:
A + ConcreteVisitor2
B + ConcreteVisitor2
