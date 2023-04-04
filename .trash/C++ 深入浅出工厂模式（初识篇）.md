## 初识工厂模式

我们先看工厂模式的介绍

这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。  
在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。

简单来说，使用了 C++ **多态**的特性，将存在**继承**关系的类，通过一个工厂类创建对应的子类（派生类）对象。在项目复杂的情况下，可以便于子类对象的创建。

工厂模式的实现方式可分别**简单工厂模式、工厂方法模式、抽象工厂模式**，每个实现方式都存在优和劣。

最近炒鞋炒的非常的火，那么以鞋厂的形式，一一分析针对每个实现方式进行分析。

## 简单工厂模式

**具体的情形**：

鞋厂可以指定生产耐克、阿迪达斯和李宁牌子的鞋子。哪个鞋炒的火爆，老板就生产哪个，看形势生产。

**UML 图：**

![](<assets/1680608374180.png>)

**简单工厂模式的结构组成**：

1. 工厂类：工厂模式的核心类，会定义一个用于创建指定的具体实例对象的接口。

2. 抽象产品类：是具体产品类的继承的父类或实现的接口。

3. 具体产品类：工厂类所创建的对象就是此具体产品实例。

**简单工厂模式的特点：**

工厂类封装了创建具体产品对象的函数。

**简单工厂模式的缺陷：**

扩展性非常差，新增产品的时候，需要去修改工厂类。

**简单工厂模式的代码：**

Shoes 为鞋子的抽象类（基类），接口函数为 Show()，用于显示鞋子广告。

NiKeShoes、AdidasShoes、LiNingShoes 为具体鞋子的类，分别是耐克、阿迪达斯和李宁鞋牌的鞋，它们都继承于 Shoes 抽象类。

```
// 鞋子抽象类
class Shoes
{
public:
    virtual ~Shoes() {}
    virtual void Show() = 0;
};

// 耐克鞋子
class NiKeShoes : public Shoes
{
public:
    void Show() {
        std::cout << "我是耐克球鞋，我的广告语：Just do it" << std::endl;
    }
};

// 阿迪达斯鞋子
class AdidasShoes : public Shoes
{
public:
    void Show() {
        std::cout << "我是阿迪达斯球鞋，我的广告语:Impossible is nothing" << std::endl;
    }
};

// 李宁鞋子
class LiNingShoes : public Shoes
{
public:
    void Show() {
        std::cout << "我是李宁球鞋，我的广告语：Everything is possible" << std::endl;
    }
};
```

ShoesFactory 为工厂类，类里实现根据鞋子类型创建对应鞋子产品对象的 CreateShoes(SHOES_TYPE type) 函数。

```
enum SHOES_TYPE
{
    NIKE,
    LINING,
    ADIDAS
};

// 总鞋厂
class ShoesFactory
{
public:
    // 根据鞋子类型创建对应的鞋子对象
    Shoes *CreateShoes(SHOES_TYPE type) {
        switch (type)
        {
        case NIKE:
            return new NiKeShoes();
            break;
        case LINING:
            return new LiNingShoes();
            break;
        case ADIDAS:
            return new AdidasShoes();
            break;
        default:
            return NULL;
            break;
        }
    }
};
```

main 函数，先是构造了工厂对象，后创建指定类型的具体鞋子产品对象，创建了具体鞋子产品的对象便可直接打印广告。因为采用的是 `new` 的方式创建了对象，用完了要通过 `delete` 释放资源资源哦！

```
int main() {
    // 构造工厂对象
    ShoesFactory shoesFactory;

    // 从鞋工厂对象创建阿迪达斯鞋对象
    Shoes *pNikeShoes = shoesFactory.CreateShoes(NIKE);
    if (pNikeShoes != NULL)
    {
        // 耐克球鞋广告喊起
        pNikeShoes->Show();

        // 释放资源
        delete pNikeShoes;
        pNikeShoes = NULL;
    }

    // 从鞋工厂对象创建阿迪达斯鞋对象
    Shoes *pLiNingShoes = shoesFactory.CreateShoes(LINING);
    if (pLiNingShoes != NULL)
    {
        // 李宁球鞋广告喊起
        pLiNingShoes->Show();

        // 释放资源
        delete pLiNingShoes;
        pLiNingShoes = NULL;
    }

    // 从鞋工厂对象创建阿迪达斯鞋对象
    Shoes *pAdidasShoes = shoesFactory.CreateShoes(ADIDAS);
    if (pAdidasShoes != NULL)
    {
        // 阿迪达斯球鞋广告喊起
        pAdidasShoes->Show();

        // 释放资源
        delete pAdidasShoes;
        pAdidasShoes = NULL;
    }

    return 0;
}
```

**输出结果：**

```
[root@lincoding factory]# ./simpleFactory 
我是耐克球鞋，我的广告语：Just do it
我是阿迪达斯球鞋，我的广告语:Impossible is nothing
我是李宁球鞋，我的广告语：Everything is possible
```

## 工厂方法模式

**具体情形：**

现各类鞋子抄的非常火热，于是为了大量生产每种类型的鞋子，则要针对不同品牌的鞋子开设独立的生产线，那么每个生产线就只能生产同类型品牌的鞋。

**UML 图：**

![](<assets/1680608374238.png>)

**工厂方法模式的结构组成：**

1. 抽象工厂类：工厂方法模式的核心类，提供创建具体产品的接口，由具体工厂类实现。

2. 具体工厂类：继承于抽象工厂，实现创建对应具体产品对象的方式。

3. 抽象产品类：它是具体产品继承的父类（基类）。

4. 具体产品类：具体工厂所创建的对象，就是此类。

**工厂方法模式的特点：**

*   工厂方法模式抽象出了工厂类，提供创建具体产品的接口，交由子类去实现。
*   工厂方法模式的应用并不只是为了封装具体产品对象的创建，而是要把具体产品对象的创建放到具体工厂类实现。

**工厂方法模式的缺陷：**

*   每新增一个产品，就需要增加一个对应的产品的具体工厂类。相比简单工厂模式而言，工厂方法模式需要更多的类定义。
*   一条生产线只能一个产品。

**工厂方法模式的代码：**

ShoesFactory 抽象工厂类，提供了创建具体鞋子产品的纯虚函数。

NiKeProducer、AdidasProducer、LiNingProducer` 具体工厂类，继承持续工厂类，实现对应具体鞋子产品对象的创建。

```
// 总鞋厂
class ShoesFactory
{
public:
    virtual Shoes *CreateShoes() = 0;
    virtual ~ShoesFactory() {}
};

// 耐克生产者/生产链
class NiKeProducer : public ShoesFactory
{
public:
    Shoes *CreateShoes() {
        return new NiKeShoes();
    }
};

// 阿迪达斯生产者/生产链
class AdidasProducer : public ShoesFactory
{
public:
    Shoes *CreateShoes() {
        return new AdidasShoes();
    }
};

// 李宁生产者/生产链
class LiNingProducer : public ShoesFactory
{
public:
    Shoes *CreateShoes() {
        return new LiNingShoes();
    }
};
```

main 函数针对每种类型的鞋子，构造了每种类型的生产线，再由每个生产线生产出对应的鞋子。需注意的是具体工厂对象和具体产品对象，用完了需要通过 delete 释放资源。

```
int main() {
    // ================ 生产耐克流程 ==================== //
    // 鞋厂开设耐克生产线
    ShoesFactory *niKeProducer = new NiKeProducer();
    // 耐克生产线产出球鞋
    Shoes *nikeShoes = niKeProducer->CreateShoes();
    // 耐克球鞋广告喊起
    nikeShoes->Show();
    // 释放资源
    delete nikeShoes;
    delete niKeProducer;

    // ================ 生产阿迪达斯流程 ==================== //
    // 鞋厂开设阿迪达斯生产者
    ShoesFactory *adidasProducer = new AdidasProducer();
    // 阿迪达斯生产线产出球鞋
    Shoes *adidasShoes = adidasProducer->CreateShoes();
    // 阿迪达斯球鞋广喊起
    adidasShoes->Show();
    // 释放资源
    delete adidasShoes;
    delete adidasProducer;

    return 0;
}
```

**输出结果：**

```
[root@lincoding factory]# ./methodFactory 
我是耐克球鞋，我的广告语：Just do it
我是阿迪达斯球鞋，我的广告语:Impossible is nothing
```

## 抽象工厂模式

**具体情形：**

鞋厂为了扩大了业务，不仅只生产鞋子，把运动品牌的衣服也一起生产了。

**UML 图：**

![](<assets/1680608374284.png>)

**抽象工厂模式的结构组成（和工厂方法模式一样）：**

1. 抽象工厂类：工厂方法模式的核心类，提供创建具体产品的接口，由具体工厂类实现。

2. 具体工厂类：继承于抽象工厂，实现创建对应具体产品对象的方式。

3. 抽象产品类：它是具体产品继承的父类（基类）。

4. 具体产品类：具体工厂所创建的对象，就是此类。

**抽象工厂模式的特点：**

提供一个接口，可以创建多个产品族中的产品对象。如创建耐克工厂，则可以创建耐克鞋子产品、衣服产品、裤子产品等。

**抽象工厂模式的缺陷：**

同工厂方法模式一样，新增产品时，都需要增加一个对应的产品的具体工厂类。

**抽象工厂模式的代码：**

Clothe 和 Shoes，分别为衣服和鞋子的抽象产品类。

NiKeClothe 和 NiKeShoes，分别是耐克衣服和耐克衣服的具体产品类。

```
// 基类 衣服
class Clothe
{
public:
    virtual void Show() = 0;
    virtual ~Clothe() {}
};

// 耐克衣服
class NiKeClothe : public Clothe
{
public:
    void Show() {
        std::cout << "我是耐克衣服，时尚我最在行！" << std::endl;
    }
};

// 基类 鞋子
class Shoes
{
public:
    virtual void Show() = 0;
    virtual ~Shoes() {}
};

// 耐克鞋子
class NiKeShoes : public Shoes
{
public:
    void Show() {
        std::cout << "我是耐克球鞋，让你酷起来！" << std::endl;
    }
};
```

Factory 为抽象工厂，提供了创建鞋子 CreateShoes() 和衣服产品 CreateClothe() 对象的接口。

NiKeProducer 为具体工厂，实现了创建耐克鞋子和耐克衣服的方式。

```
// 总厂
class Factory
{
public:
    virtual Shoes *CreateShoes() = 0;
	virtual Clothe *CreateClothe() = 0;
    virtual ~Factory() {}
};

// 耐克生产者/生产链
class NiKeProducer : public Factory
{
public:
    Shoes *CreateShoes() {
        return new NiKeShoes();
    }
	
	Clothe *CreateClothe() {
        return new NiKeClothe();
    }
};
```

main 函数，构造耐克工厂对象，通过耐克工厂对象再创建耐克产品族的衣服和鞋子对象。同样，对象不再使用时，需要手动释放资源。

```
int main() {
    // ================ 生产耐克流程 ==================== //
    // 鞋厂开设耐克生产线
    Factory *niKeProducer = new NiKeProducer();
    
	// 耐克生产线产出球鞋
    Shoes *nikeShoes = niKeProducer->CreateShoes();
	// 耐克生产线产出衣服
    Clothe *nikeClothe = niKeProducer->CreateClothe();
    
	// 耐克球鞋广告喊起
    nikeShoes->Show();
	// 耐克衣服广告喊起
    nikeClothe->Show();
	
    // 释放资源
    delete nikeShoes;
	delete nikeClothe;
    delete niKeProducer;


    return 0;
}
```

**输出结果：**

```
[root@lincoding factory]# ./abstractFactory 
我是耐克球鞋，让你酷起来！
我是耐克衣服，时尚我最在行！
```

## 总结

以上三种工厂模式，在新增产品时，都存在一定的缺陷。

*   简单工厂模式，，需要去修改工厂类，这违背了开闭法则。
*   工厂方式模式和抽象工厂模式，都需要增加一个对应的产品的具体工厂类，这就会增大了代码的编写量。

那么有什么好的方法，在新增产品时，即不用修改工厂类，也不用新增具体的工厂类？

笔者在实际项目中看到一个封装性非常强的工厂类，在扩展新产品时，不需要修改工厂类，也不需要新增具体的工厂类，详细内容可以跳转至

[小林 coding：C++ 深入浅出工厂模式（进阶篇）](https://zhuanlan.zhihu.com/p/83537599)