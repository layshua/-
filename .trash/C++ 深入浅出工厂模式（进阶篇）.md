## 介绍

前文初始篇 [C++ 深入浅出工厂模式（初识篇）](https://zhuanlan.zhihu.com/p/83535678)，主要阐述了简单工厂模式、工厂方法模式和抽象工厂模式的结构、特点和缺陷等。

以上三种方式，在新增产品时，要么修改工厂类，要么需新增具体的工厂类，说明工厂类的封装性还不够好。

本文进阶篇，主要是将工厂类的封装性提高，达到新增产品时，也不需要修改工厂类，不需要新增具体的工厂类。封装性高的工厂类特点是扩展性高、复用性也高。

## 模板工厂

针对工厂方法模式封装成模板工厂类，那么这样在新增产品时，是不需要新增具体的工厂类，减少了代码的编写量。

**UML 图：**

![](<assets/1680608390362.png>)

**模板工厂代码：**

Shoes 和 Clothe，分别为鞋子和衣服的抽象类（基类）

NiKeShoes 和 UniqloClothe，分别为耐克鞋子和优衣库衣服具体产品类。

```
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
        std::cout << "我是耐克球鞋，我的广告语：Just do it" << std::endl;
    }
};

// 基类 衣服
class Clothe
{
public:
    virtual void Show() = 0;
    virtual ~Clothe() {}
};

// 优衣库衣服
class UniqloClothe : public Clothe
{
public:
    void Show() {
        std::cout << "我是优衣库衣服，我的广告语：I am Uniqlo" << std::endl;
    }
};
```

AbstractFactory 为抽象模板工厂类，其中模板参数：AbstractProduct_t` 产品抽象类，如 Shoes、Clothe

ConcreteFactory 为具体模板工厂类，其中模板参数：AbstractProduct_t 产品抽象类（如 Shoes、Clothe），ConcreteProduct_t 产品具体类（如 NiKeShoes、UniqloClothe）

```
// 抽象模板工厂类
// 模板参数：AbstractProduct_t 产品抽象类
template <class AbstractProduct_t>
class AbstractFactory
{
public:
    virtual AbstractProduct_t *CreateProduct() = 0;
    virtual ~AbstractFactory() {}
};

// 具体模板工厂类
// 模板参数：AbstractProduct_t 产品抽象类，ConcreteProduct_t 产品具体类
template <class AbstractProduct_t, class ConcreteProduct_t>
class ConcreteFactory : public AbstractFactory<AbstractProduct_t>
{
public:
    AbstractProduct_t *CreateProduct() {
        return new ConcreteProduct_t();
    }
};
```

main 函数，根据不同类型的产品，构造对应的产品的工厂对象，便可通过对应产品的工厂对象创建具体的产品对象。

```
int main() {
    // 构造耐克鞋的工厂对象
    ConcreteFactory<Shoes, NiKeShoes> nikeFactory;
    // 创建耐克鞋对象
    Shoes *pNiKeShoes = nikeFactory.CreateProduct();
    // 打印耐克鞋广告语
    pNiKeShoes->Show();

    // 构造优衣库衣服的工厂对象
    ConcreteFactory<Clothe, UniqloClothe> uniqloFactory;
    // 创建优衣库衣服对象
    Clothe *pUniqloClothe = uniqloFactory.CreateProduct();
    // 打印优衣库广告语
    pUniqloClothe->Show();

    // 释放资源
    delete pNiKeShoes;
    pNiKeShoes = NULL;

    delete pUniqloClothe;
    pUniqloClothe = NULL;

    return 0;
}
```

**输出结果：**

```
[root@lincoding factory]# ./templateFactory 
我是耐克球鞋，我的广告语：Just do it
我是优衣库衣服，我的广告语：I am Uniqlo
```

## 产品注册模板类 + 单例工厂模板类

前面的模板工厂虽然在新增产品的时候，不需要新增具体的工厂类，但是缺少一个可以统一随时随地获取指定的产品对象的类。

还有改进的空间，我们可以把产品注册的对象用 **std::map** 的方式保存，通过 **key-valve** 的方式可以轻松简单的获取对应的产品对象实例。

实现大致思路：

*   把产品注册的功能封装成产品注册模板类。注册的产品对象保存在工厂模板类的 **std::map**，便于产品对象的获取。
*   把获取产品对象的功能封装成工厂模板类。为了能随时随地获取指定产品对象，则把工厂设计成单例模式。

**UML 图：**

![](<assets/1680608390404.png>)

**产品注册模板类 + 单例工厂模板类的代码：**

IProductRegistrar 为产品注册抽象类，模板参数 ProductType_t 表示的类是产品抽象类（如 Shoes、Clothe）。提供了产品对象创建的纯虚函数 CreateProduct。

ProductFactory 为工厂模板类，模板参数 ProductType_t 表示的类是产品抽象类（如 Shoes、Clothe）。用于保存注册产品对象到 std::map 中和获取对应的产品对象。

ProductRegistrar 为产品注册模板类，模板参数 ProductType_t 表示的类是产品抽象类（如 Shoes、Clothe），ProductImpl_t 表示的类是具体产品（如 NikeShoes、UniqloClothe）。用于注册产品到工厂类和创建产品实例对象。

```
// 基类，产品注册模板接口类
// 模板参数 ProductType_t 表示的类是产品抽象类
template <class ProductType_t>
class IProductRegistrar
{
public:
   // 获取产品对象抽象接口
   virtual ProductType_t *CreateProduct() = 0;

protected:
   // 禁止外部构造和虚构, 子类的"内部"的其他函数可以调用
   IProductRegistrar() {}
   virtual ~IProductRegistrar() {}

private:
   // 禁止外部拷贝和赋值操作
   IProductRegistrar(const IProductRegistrar &);
   const IProductRegistrar &operator=(const IProductRegistrar &);
};

// 工厂模板类，用于获取和注册产品对象
// 模板参数 ProductType_t 表示的类是产品抽象类
template <class ProductType_t>
class ProductFactory
{
public:
   // 获取工厂单例，工厂的实例是唯一的
   static ProductFactory<ProductType_t> &Instance()
   {
      static ProductFactory<ProductType_t> instance;
      return instance;
   }

   // 产品注册
   void RegisterProduct(IProductRegistrar<ProductType_t> *registrar, std::string name) {
      m_ProductRegistry[name] = registrar;
   }

   // 根据名字name，获取对应具体的产品对象
   ProductType_t *GetProduct(std::string name) {
      // 从map找到已经注册过的产品，并返回产品对象
      if (m_ProductRegistry.find(name) != m_ProductRegistry.end())
      {
         return m_ProductRegistry[name]->CreateProduct();
      }

      // 未注册的产品，则报错未找到
      std::cout << "No product found for " << name << std::endl;

      return NULL;
   }

private:
   // 禁止外部构造和虚构
   ProductFactory() {}
   ~ProductFactory() {}

   // 禁止外部拷贝和赋值操作
   ProductFactory(const ProductFactory &);
   const ProductFactory &operator=(const ProductFactory &);

   // 保存注册过的产品，key:产品名字 , value:产品类型
   std::map<std::string, IProductRegistrar<ProductType_t> *> m_ProductRegistry;
};

// 产品注册模板类，用于创建具体产品和从工厂里注册产品
// 模板参数 ProductType_t 表示的类是产品抽象类（基类），ProductImpl_t 表示的类是具体产品（产品种类的子类）
template <class ProductType_t, class ProductImpl_t>
class ProductRegistrar : public IProductRegistrar<ProductType_t>
{
public:
   // 构造函数，用于注册产品到工厂，只能显示调用
   explicit ProductRegistrar(std::string name) {
      // 通过工厂单例把产品注册到工厂
      ProductFactory<ProductType_t>::Instance().RegisterProduct(this, name);
   }

   // 创建具体产品对象指针
   ProductType_t *CreateProduct() {
      return new ProductImpl_t();
   }
};
```

main 函数，通过 ProductRegistrar 注册各种不同类型产品，在统一由 ProductFactory 单例工厂获取指定的产品对象。

```
int main() {
   // ========================== 生产耐克球鞋过程 ===========================//
   // 注册产品种类为Shoes（基类），产品为NiKe（子类）到工厂，产品名为nike
   ProductRegistrar<Shoes, NiKeShoes> nikeShoes("nike");
   // 从工厂获取产品种类为Shoes，名称为nike的产品对象
   Shoes *pNiKeShoes = ProductFactory<Shoes>::Instance().GetProduct("nike");
   // 显示产品的广告语
   pNiKeShoes->Show();
   // 释放资源
   if (pNiKeShoes)
   {
      delete pNiKeShoes;
   }

   // ========================== 生产优衣库衣服过程 ===========================//
   // 注册产品种类为Clothe（基类），产品为UniqloClothe（子类）到工厂，产品名为uniqlo
   ProductRegistrar<Clothe, UniqloClothe> adidasShoes("uniqlo");
   // 从工厂获取产品种类为Shoes，名称为adidas的产品对象
   Clothe *pUniqloClothe = ProductFactory<Clothe>::Instance().GetProduct("uniqlo");
   // 显示产品的广告语
   pUniqloClothe->Show();
   // 释放资源
   if (pUniqloClothe)
   {
      delete pUniqloClothe;
   }

   return 0;
}
```

**输出结果：**

```
[root@lincoding factory]# ./singleFactory 
我是耐克球鞋，我的广告语：Just do it
我是优衣库衣服，我的广告语：I am Uniqlo
```

## 总结

将工厂方法模式改良成模板工厂，虽然可以解决产品新增时，不需要新增具体工厂类，但是缺少一个可以随时随地获取产品对象的方式，说明还有改进的空间。

将模板工厂改良成产品注册模板类 + 单例工厂模板类，产品注册模板类用于注册不同类型的产品，单例工厂模板类用于获取指定已注册的产品对象。这种方式，可以把工厂模式中产品的注册和获取的主要功能很好的抽象成两个类，并且使用单例模式使得工厂类可以随时随地获取已注册的产品对象。

所以产品注册模板类 + 单例工厂模板类的工厂模式，达到了开闭法则，并且扩展性高和封装度高。

PS：想学习更多单例模式，可以参考

[小林 coding：C++ 线程安全的单例模式总结](https://zhuanlan.zhihu.com/p/83539039)