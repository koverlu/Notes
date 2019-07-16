# 多态
C++继承可以让子类继承另基类所包含的属性和方法，有时，子类虽继承了基类，却有些方法存在自己的实现。我们看下面这样一个例子，两个类动物（Animal）和人（Human）。Human继承了Animal，Animal有呼吸方法，Human也有呼吸方法。代码如下：
```c++
#include <iostream>

using namespace std;

class Animal {
public:
    char *name;
    void breathe() {
        cout << "Animal breathe" << endl;
    }
    virtual void eat() {
        cout << "Animal eat" << endl;
    }
};

class Human: public Animal {
public:
    int race;
    void breathe() {
        cout << "Human breathe" << endl;
    }
    void eat() {
        cout << "Human eat" << endl;
    }
};

int main(void) {
    // 用实例调用
    Animal a;
    Human h;
    a.breathe();
    a.eat();
    h.breathe();
    h.eat();

    cout << endl;

    // 用基类指针调用
    Animal *aPtr = NULL;
    aPtr = &a;
    aPtr->breathe();
    aPtr->eat();
    aPtr = &h;
    aPtr->breathe();
    aPtr->eat();
    return 0;
}
```
输出的结果是:
```
Animal breathe
Animal eat
Human breathe
Human eat

Animal breathe
Animal eat
Animal breathe
Human eat
```
首先我们对一个Animal实例和一个Human实例分别调用breathe方法和eat方法，结果如我们所想要的，各自调用了各自的实现。
但我们知道，基类的指针可以指向子类，因为有时候我们为了让代码更通用，会用一个更通用的基类指针来指向不同的实例。在例子中，我们发现，对breathe方法，基类指针并没有调用具体实例所属Human类的实现，两次输出都是“Animal breathe”，而对eat方法，基类指针调用了所指向的实例所属Human类的实现，两次输出分布是“Animal eat”和“Human eat”。这就是引入虚函数的基本情况。
对于没有声明被声明成虚函数的方法，比如这里的breathe，代码中对于breathe方法的调用在编译时就已经被绑定了实现，绑定的是基类的实现，此为早绑定。对于被声明成虚函数的方法，比如这里的eat，代码中对于eat方法的调用是在程序运行时才去绑定的，而这里的基类指针指向了一个Human类的实例，它会调用Human类的eat方法实现。那么它是如何做到调用具体类的实现而非基类的实现呢？

## 虚函数表
我们来观察一下类的内存分布，大部分编译器都提供了查看C++代码中类内存分布的工具，在Visual Studio中，右击项目，在属性（Properties）-> C/C++ -> 命令行（Command Line）-> 附加选项（Additional Options）中输入/d1 reportAllClassLayout即可在输出窗口中查看类的内存分布。对于上述代码中的Animal类和Human类，内存的分布如下：
```
class Animal    size(8):
    +---
 0  | {vfptr}
 4  | name
    +---

Animal::$vftable@:
    | &Animal_meta
    |  0
 0  | &Animal::eat

class Human size(12):
    +---
 0  | +--- (base class Animal)
 0  | | {vfptr}
 4  | | name
    | +---
 8  | race
    +---

Human::$vftable@:
    | &Human_meta
    |  0
 0  | &Human::eat
 ```
 对于有虚函数的类，它在类内存的开始有一个指针指向虚函数表，虚函数表中包含了基类中以virtual修饰的所有虚函数。在基类Animal中，虚函数表中的eat指向的是Animal::eat，而在子类Human中，虚函数表中的eat指向的是Human::eat，因而在使用基类指针调用实例方法时，会调用虚函数表中的函数，也就是具体实例所属类的实现。

## 几种常见继承关系中的类内存分布
### 单继承
我们来研究如下单继承的例子，Animal类是Human类的基类，Human类是Asian类的基类。在Animal类中，breathe是一个普通方法，而eat是声明为虚函数的方法。在Human类中，breathe是声明成虚函数的方法，eat是一个普通方法。在Asian类中，breathe和eat都是普通方法。类的定义代码如下：

```C++
#include <iostream>

using namespace std;

class Animal {
public:
    char *name;
    void breathe() {
        cout << "Animal breathe" << endl;
    }
    virtual void eat() {
        cout << "Animal eat" << endl;
    }
};

class Human: public Animal {
public:
    int race;
    virtual void breathe() {
        cout << "Human breathe" << endl;
    }
    void eat() {
        cout << "Human eat" << endl;
    }
};

class Asian : public Human {
public:
    int country;
    void breathe() {
        cout << "Asian breathe" << endl;
    }
    void eat() {
        cout << "Asian eat" << endl;
    }
};

int main(void) {
    Animal animal;
    Human human;
    Asian asian;

    Animal *anPtr = NULL;
    Human *hmPtr = NULL;
    Asian *asPtr = NULL;

    cout << "用Animal指针调用human和asian实例" << endl;
    anPtr = &human;
    anPtr->breathe();
    anPtr->eat();
    anPtr = &asian;
    anPtr->breathe();
    anPtr->eat();

    cout << endl;
    cout << "用Human指针调用asian实例" << endl;
    hmPtr = &asian;
    hmPtr->breathe();
    hmPtr->eat();

    return 0;
}
```
运行的结果如下：
```
用Animal指针调用human和asian实例
Animal breathe
Human eat
Animal breathe
Asian eat

用Human指针调用asian实例
Asian breathe
Asian eat
```
编译器显示的内存分布如下：
```
class Animal    size(8):
    +---
 0  | {vfptr}
 4  | name
    +---

Animal::$vftable@:
    | &Animal_meta
    |  0
 0  | &Animal::eat

class Human size(12):
    +---
 0  | +--- (base class Animal)
 0  | | {vfptr}
 4  | | name
    | +---
 8  | race
    +---

Human::$vftable@:
    | &Human_meta
    |  0
 0  | &Human::eat
 1  | &Human::breathe

class Asian size(16):
    +---
 0  | +--- (base class Human)
 0  | | +--- (base class Animal)
 0  | | | {vfptr}
 4  | | | name
    | | +---
 8  | | race
    | +---
12  | country
    +---

Asian::$vftable@:
    | &Asian_meta
    |  0
 0  | &Asian::eat
 1  | &Asian::breathe
```

有上面的内存分布可以看出：

1. 一个类中的某个方法被声明为虚函数，则它将放在虚函数表中。
2. 当一个类继承了另一个类，就会继承它的虚函数表，虚函数表中所包含的函数，如果在子类中有重写，则指向当前重写的实现，否则指向基类实现。若在子类中定义了新的虚函数，则该虚函数指针在虚函数表的后面（如Human类中的breathe，在eat的后面）。
3. 在继承或多级继承中，要用一个祖先类的指针调用一个后代类实例的方法，若想体现出多态，则必须在该祖先类中就将需要的方法声明为虚函数，否则虽然后代类的虚函数表中有这个方法在后代类中的实现，但对祖先类指针的方法调用依然是早绑定的。（如用Animal指针调用Asian实例中的breathe方法，虽然在Human类中已经将breathe声明为虚函数，依然无法调用Asian类中breathe的实现，但用Human指针调用Asian实例中的breathe方法就可以）。

### 多继承
现在假设这样一个例子，有LandAnimal（陆生动物）类和Mammal（哺乳动物）类，它们都有breathe和eat方法，都被声明成虚函数。Human类继承了LandAnimal类和Mammal类，同时Human类重写了eat方法。代码如下：
```C++
#include <iostream>

using namespace std;

class LandAnimal {
public:
    int numLegs;
    virtual void run() {
        cout << "Land animal run" << endl;
    }
};

class Mammal {
public:
    int numBreasts;
    virtual void milk() {
        cout << "Mammal milk" << endl;
    }
};

class Human: public Mammal, public LandAnimal {
public:
    int race;
    void milk() {
        cout << "Human milk" << endl;;
    }
    void run() {
        cout << "Human run" << endl;
    }
    void eat() {
        cout << "Human eat" << endl;
    }
};

int main(void) {
    Human human;

    cout << "用LandAnimal指针调用human实例的方法" << endl;
    LandAnimal *laPtr = NULL;
    laPtr = &human;
    laPtr->run();

    cout << "用Mammal指针调用human实例的方法" << endl;
    Mammal *mPtr = NULL;
    mPtr = &human;
    mPtr->milk();

    return 0;
}
```
运行的结果如下，可以看出，对于重写了的milk和run方法，通过基类指针的调用会指向实例所属类的实现：
```
用LandAnimal指针调用human实例的方法
Human run
用Mammal指针调用human实例的方法
Human milk
```
类的内存结构如下：
```
class LandAnimal    size(8):
    +---
 0  | {vfptr}
 4  | numLegs
    +---

LandAnimal::$vftable@:
    | &LandAnimal_meta
    |  0
 0  | &LandAnimal::run

class Mammal    size(8):
    +---
 0  | {vfptr}
 4  | numBreasts
    +---

Mammal::$vftable@:
    | &Mammal_meta
    |  0
 0  | &Mammal::milk

class Human size(20):
    +---
 0  | +--- (base class Mammal)
 0  | | {vfptr}
 4  | | numBreasts
    | +---
 8  | +--- (base class LandAnimal)
 8  | | {vfptr}
12  | | numLegs
    | +---
16  | race
    +---

Human::$vftable@Mammal@:
    | &Human_meta
    |  0
 0  | &Human::milk

Human::$vftable@LandAnimal@:
    | -8
 0  | &Human::run

```
可见，对于多继承的情况，子类会包含多个基类的内存结构，包括多个虚函数表，若子类中重写了基类种被定义为虚函数的方法，则虚函数表中的函数指针指向子类的实现，否则指向基类的实现。

### 菱形继承
在多继承的基础上，我们在考虑这样一种情况。
```C++
#include <iostream>

using namespace std;

class Animal {
public:
    int name;
    virtual void breathe() {
        cout << "Animal breathe" << endl;
    }
};

class LandAnimal: public Animal {
public:
    int numLegs;
    virtual void run() {
        cout << "Land animal run" << endl;
    }
};

class Mammal: public Animal {
public:
    int numBreasts;
    virtual void milk() {
        cout << "Mammal milk" << endl;
    }
};

class Human: public Mammal, public LandAnimal {
public:
    int race;
    void milk() {
        cout << "Human milk" << endl;
    }
    void run() {
        cout << "Human run" << endl;
    }
    void eat() {
        cout << "Human eat" << endl;
    }
};

int main(void) {
    Human human;

    cout << "用LandAnimal指针调用Human实例的方法" << endl;
    LandAnimal *laPtr = NULL;
    laPtr = &human;
    laPtr->run();

    cout << "用Mammal指针调用Human实例的方法" << endl;
    Mammal *mPtr = NULL;
    mPtr = &human;
    mPtr->milk();

    cout << "用Animal指针调用Human实例的方法" << endl;
    Animal *aPtr = NULL;
    aPtr = &human; // error: base class "Animal" is ambiguous

    return 0;
}
```
则当我们让Animal指针指向human实例时，IDE会报错。因为Human类同时继承了LandAnimal类和Mammal类。此时的内存结构如下：
```
Animal::$vftable@:
    | &Animal_meta
    |  0
 0  | &Animal::breathe

class LandAnimal    size(12):
    +---
 0  | +--- (base class Animal)
 0  | | {vfptr}
 4  | | name
    | +---
 8  | numLegs
    +---

LandAnimal::$vftable@:
    | &LandAnimal_meta
    |  0
 0  | &Animal::breathe
 1  | &LandAnimal::run

class Mammal    size(12):
    +---
 0  | +--- (base class Animal)
 0  | | {vfptr}
 4  | | name
    | +---
 8  | numBreasts
    +---

Mammal::$vftable@:
    | &Mammal_meta
    |  0
 0  | &Animal::breathe
 1  | &Mammal::milk

class Human size(28):
    +---
 0  | +--- (base class Mammal)
 0  | | +--- (base class Animal)
 0  | | | {vfptr}
 4  | | | name
    | | +---
 8  | | numBreasts
    | +---
12  | +--- (base class LandAnimal)
12  | | +--- (base class Animal)
12  | | | {vfptr}
16  | | | name
    | | +---
20  | | numLegs
    | +---
24  | race
    +---

Human::$vftable@Mammal@:
    | &Human_meta
    |  0
 0  | &Animal::breathe
 1  | &Human::milk

Human::$vftable@LandAnimal@:
    | -12
 0  | &Animal::breathe
 1  | &Human::run
 ```
我们可以看到，Human类包含了Mammal类和LandAnimal类的内存结构，而Mammal类和LandAnimal类都继承自Animal类，它们的一些成员变量和方法是相同的。如果用Animal指针指向Human类的实例，则对于共同的成员变量和方法，编译器无法判断是要使用Mammal类中的还是使用LandAnimal类中的。于是报上面的错误。
这时，我们需要用到虚继承。我们在继承的时候，加上virutal关键字，使LandAnimal类和Mammal类虚继承Animal类，代码如下:
```C++
#include <iostream>

using namespace std;

class Animal {
public:
    int name;
    virtual void breathe() {
        cout << "Animal breathe" << endl;
    }
};

class LandAnimal: virtual public Animal {
public:
    int numLegs;
    virtual void run() {
        cout << "Land animal run" << endl;
    }
};

class Mammal: virtual public Animal {
public:
    int numBreasts;
    virtual void milk() {
        cout << "Mammal milk" << endl;
    }
};

class Human: public Mammal, public LandAnimal {
public:
    int race;
    void breathe() {
        cout << "Human breathe" << endl;
    }
    void milk() {
        cout << "Human milk" << endl;
    }
    void run() {
        cout << "Human run" << endl;
    }
    void eat() {
        cout << "Human eat" << endl;
    }
};

int main(void) {
    Human human;

    cout << "用LandAnimal指针调用Human实例的方法" << endl;
    LandAnimal *laPtr = NULL;
    laPtr = &human;
    laPtr->run();

    cout << "用Mammal指针调用Human实例的方法" << endl;
    Mammal *mPtr = NULL;
    mPtr = &human;
    mPtr->milk();

    cout << "用Animal指针调用Human实例的方法" << endl;
    Animal *aPtr = NULL;
    aPtr = &human;
    aPtr->breathe();

    return 0;
}
```
运行结果如下：
```
用LandAnimal指针调用Human实例的方法
Human run
用Mammal指针调用Human实例的方法
Human milk
用Animal指针调用Human实例的方法
Human breathe
```
此时，Animal指针可以指向Human类的实例，并调用Human类中breathe方法的实现。我们查看此时的内存结构，如下：
```
class Animal    size(8):
    +---
 0  | {vfptr}
 4  | name
    +---

Animal::$vftable@:
    | &Animal_meta
    |  0
 0  | &Animal::breathe

class LandAnimal    size(20):
    +---
 0  | {vfptr}
 4  | {vbptr}
 8  | numLegs
    +---
    +--- (virtual base Animal)
12  | {vfptr}
16  | name
    +---

LandAnimal::$vftable@LandAnimal@:
    | &LandAnimal_meta
    |  0
 0  | &LandAnimal::run

LandAnimal::$vbtable@:
 0  | -4
 1  | 8 (LandAnimald(LandAnimal+4)Animal)

LandAnimal::$vftable@Animal@:
    | -12
 0  | &Animal::breathe

class Mammal    size(20):
    +---
 0  | {vfptr}
 4  | {vbptr}
 8  | numBreasts
    +---
    +--- (virtual base Animal)
12  | {vfptr}
16  | name
    +---

Mammal::$vftable@Mammal@:
    | &Mammal_meta
    |  0
 0  | &Mammal::milk

Mammal::$vbtable@:
 0  | -4
 1  | 8 (Mammald(Mammal+4)Animal)

Mammal::$vftable@Animal@:
    | -12
 0  | &Animal::breathe

class Human size(36):
    +---
 0  | +--- (base class Mammal)
 0  | | {vfptr}
 4  | | {vbptr}
 8  | | numBreasts
    | +---
12  | +--- (base class LandAnimal)
12  | | {vfptr}
16  | | {vbptr}
20  | | numLegs
    | +---
24  | race
    +---
    +--- (virtual base Animal)
28  | {vfptr}
32  | name
    +---

Human::$vftable@Mammal@:
    | &Human_meta
    |  0
 0  | &Human::milk

Human::$vftable@LandAnimal@:
    | -12
 0  | &Human::run

Human::$vbtable@Mammal@:
 0  | -4
 1  | 24 (Humand(Mammal+4)Animal)

Human::$vbtable@LandAnimal@:
 0  | -4
 1  | 12 (Humand(LandAnimal+4)Animal)

Human::$vftable@Animal@:
    | -28
 0  | &Human::breathe
```
我们可以观察到，一个子类虚继承自另一个基类，它不再像普通继承那样直接拥有一份基类的内存结构，而是加了一个虚表指针vbptr指向虚基类，这个虚基类在msvc中被放在的类的内存空间的最后。这样，当出现类似这里的菱形继承时，基类Animal在子类Human中出现一次，子类Human所包含的Mammal类和LandAnimal类各有一个虚基类指向虚基类。从而避免了菱形继承时的冲突。
## 总结
总之，C++多态的核心，就是用一个更通用的基类指针指向不同的子类实例，为了能调用正确的方法，我们需要用到虚函数和虚继承。在内存中，通过虚函数表来实现子类方法的正确调用，通过虚基类指针，仅保留一份基类的内存结构，避免冲突。
所谓虚，就是把“直接”的东西变“间接”。成员函数原先是由静态的成员函数指针来定义的，而虚函数则是由一个虚函数表来指向真正的函数指针，从而达到在运行时，间接地确定想要的函数实现。继承原先是直接将基类的内存空间拷贝一份来实现的，而虚继承则用一个虚基类指针来指向虚基类，避免基类的重复。