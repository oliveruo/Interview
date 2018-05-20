### C++相关

---

1.1  构造函数和析构函数

构造函数的作用：用于新建对象的初始化工作。

 析构函数的作用：用于在撤销对象前，完成一些清理工作，比如：释放内存等。

 每当创建对象时，需要添加初始化代码时，则需要定义自己的构造函数；而对象撤销时，需要自己添加清理工作的代码时，则需要定义自己的析构函数。

在研究C++类的继承、派生、组合时，一直没有清晰地了解构造函数与析构函数的调用过程。本章通过点-线组合类，来深入分析组合类情况下，对象的构造与析构。

1.问题的引入

源代码：

include <iostream>

using namespace std;  

class Point  {   

     private:  

      int x;    int y;  

    public:  

    int getx();  

    int gety();  

   Point()  

{ cout<<"Calling the default construction of Point!"<<endl; }  

Point(int xx, int yy)  

{ cout<<"Calling the parameter construction of Point!"<<endl; x = xx; y = yy;}  

Point(Point &pt)  

{ cout<<"Calling the copy construction of Point!"<<endl; x = pt.x;  y = pt.y;}  

~Point()  

{cout<<"Calling the destructor of Point!"<<endl;}  

};  

int Point::getx() { return x;}  

int Point::gety() { return y;}  

//==================================================================  

class Line  

{  

private:  

Point dot1,dot2;  

double Len;  

public:  

Line()  

{cout<<"Calling the default construction of Line!"<<endl;}  

Line(Point pt1, Point pt2)  

{  

cout<<"Calling the parameter construction of  Line"<<endl;  

dot1 = pt1;  dot2 = pt2;  

double Linex = abs( dot1.getx() - dot2.getx() );  

double Liney = abs( dot1.gety() - dot2.gety() );  

Len = sqrt( LinexLinex + LineyLiney);  

}  

Line(Line &l)  

{  

cout<<"Calling the copy construction of Line"<<endl;  

Len = l.Len;  

}  

~Line(){ cout<<"Calling the destructor of Line!"<<endl;}  

};  

void main()  

{  

Point mypoint1;   

Point mypoint2(1,2);  

Point mypoint3(mypoint2);  

cout<<mypoint2.getx()<<endl;  

cout<<mypoint3.gety()<<endl;  

Point mypt1(1,2),mypt2(3,4);  

Line line1(mypt1,mypt2);  

Line line2(line1);  

}

程序执行结果：

2.逐条语句进行分析 

Point mypoint1;   

 该条语句是利用Point类进行初始化Point对象，此时，编译器会自动调用程序的构造函数。由于对象定义不带有任何参数，所以此时仅调用程序的默认构造函数。

输出为“Calling the default construction of Point!”

Point mypoint2(1,2);  

语句定义一个Point类的对象，但是此时我们应该注意到对mypoint2对象进行了初始化，此时操作系统会调用含有参数的构造函数。

具体行为过程是，在内存中开辟空间定义临时变量 int xx, int yy;通过参数传递，xx = 1. yy = 2;然后在执行含有参数构造函数的函数体，完成临时变量值向Point类的属性x,y的赋值。

输出“Calling the parameter construction of Point!”

Point mypoint3(mypoint2);  

语句仍然定义了一个Point类的对象，但是应该注意到，这里利用了前一个对象mypoint2对mypoint3进行初始化 。调用了Point类复制构造函数。相当于&pt = mypoint2 。此时，通过看看内存地址，我们可以看到，编译器并没用定义一个Point类的对象临时pt。而是采用了“引用”的模式，pt仅仅是mypoint2的一个别名而已！！！

输出“Calling the copy construction of point!”

cout<<mypoint2.getx()<<endl;  

cout<<mypoint3.gety()<<endl;  

 输出当前Point类的属性x,y;输出“ 1 2”

Point mypt1(1,2),mypt2(3,4);  

定义Point类的两个带有初始化参数的对象,

输出“Calling the parameter construction of Point!”

"Calling the parameter construction of Point!"

[cpp] view plain copy

1.Line line1(mypt1,mypt2);  

注意到，此时我们在尝试利用参数传递的方式来构造Line的对象line1，也就是说，相比较而言Line是上一层的类，而mypt1/mypt2是底层类的对象。此时的程序执行过程可以分解为：

&pt = mypt2;  // “Calling the copy construction of Point!”

&pt = mypt1; // "Calling the copy construction of Point!"

//注意到，此时仍然是引用的格式，没有进行临时对象的构建

line1(mypt1,mypt2); //“Calling the parameter construction of Line!”

重点：当我们利用底层类的对象对上一层类对象进行初始化之后，要立刻销毁底层类的对象，及调用相应的析构函数。

“Calling the destructor of Point!” //销毁对象mypt1

“Calling  the destructor of Point!” //销毁对象mypt2

[cpp] view plain copy

1.Line line2(line1);  

参考上例，此时的拷贝过程可以分解为：

&l = line1; //引用传递

“Calling the default construction of Point!”

"Calling the default construction of Point!"

//执行Line复制构造函数的函数体

“Calling the copy construction of Line”

退出程序后，对所有对象进行销毁，其顺序为“与构造函数调用顺序相反”

1.2、为什么不要在构造器中调用虚函数

基类部分在派生类部分之前被构造，当基类构造函数执行时派生类中的数据成员还没被初始化。如果基类构造函数中的虚函数调用被解析成调用派生类的虚函数，而派生类的虚函数中又访问到未初始化的派生类数据，将导致程序出现一些未定义行为和bug

include <iostream>

class Base

{

public:

    Base() { Foo(); }   ///< 打印 1

    virtual void Foo()
    {
        std::cout << 1 << std::endl;
    }

};

class Derive : public Base

{

public:

    Derive() : Base(), m_pData(new int(2)) {}

    ~Derive() { delete m_pData; }

    virtual void Foo()
    {
        std::cout << *m_pData << std::endl;
    }

private:

    int* m_pData;

};

int main()

{

    Base* p = new Derive();

    delete p;

    return 0;

}

这里的结果将打印：1。

　　这表明第6行执行的的是Base::Foo()而不是Derive::Foo()，也就是说：虚函数在构造函数中“不起作用”。为什么？

　　当实例化一个派生类对象时，首先进行基类部分的构造，然后再进行派生类部分的构造。即创建Derive对象时，会先调用Base的构造函数，再调用Derive的构造函数。

　　当在构造基类部分时，派生类还没被完全创建，从某种意义上讲此时它只是个基类对象。即当Base::Base()执行时Derive对象还没被完全创建，此时它被当成一个Base对象，而不是Derive对象，因此Foo绑定的是Base的Foo。

　　C++之所以这样设计是为了减少错误和Bug的出现。假设在构造函数中虚函数仍然“生效”，即Base::Base()中的Foo();所调用的是Derive::Foo()。当Base::Base()被调用时派生类中的数据m_pData还未被正确初始化，这时执行Derive::Foo()将导致程序对一个未初始化的地址解引用，得到的结果是不可预料的，甚至是程序崩溃（访问非法内存）。

　　总结来说：基类部分在派生类部分之前被构造，当基类构造函数执行时派生类中的数据成员还没被初始化。如果基类构造函数中的虚函数调用被解析成调用派生类的虚函数，而派生类的虚函数中又访问到未初始化的派生类数据，将导致程序出现一些未定义行为和bug。

1.3为什么不要在析构函数中抛出异常

C++ 里面当构造函数抛出异常时，其会调用构造函数里面已经创建对象的析构函数，但是对以自己的析构函数没有调用，就可能产生内存泄漏，比如自己new 出来的内存没有释放。

有两个办法。在Catch 块里面释放已经申请的资源 或者 用智能指针把资源当做对象处理。

class Base

{

public:

    void fun()     {   throw 1;    }

    ~Base()        {   throw 2;    }

};

int main()

{

   try

   {

       Base base;

       //base.fun();

   }

   catch (...)

   {

        //cout <<"get the catch"<<endl;

   }

}

运行没有问题。

   实验2: 打开上面注释掉的第13行代码（ //base.fun(); ），再试运行，结果呢？ 在debug 模式下弹出对话框

为什么呢？

因为SEH 是一个链表，链表头地址存在FS:[0] 的寄存器里面。 在实验2，函数base.fun先抛出异常，从FS:[0]开始向上遍历 SHL 节点，匹配到catch 块。 找到代码里面为一个catch块，再去展开栈，调用base 的析构函数，然而析构又抛出异常。 如果系统再去从SEL链表匹配，会改变FS:[0]值，这时候程序迷失了，不知道下面该怎么什么？ 因为他已经丢掉了上一次异常链那个节点。

如果析构函数的异常被处理呢， 程序还会正常运行吗？

class Base

{

public:

    void fun()     {   throw 1;    }

    ~Base()       

    {

        try

        {

              throw 2;   

        }

        catch (int e)

        {

         // do something    

        }    

    }

};

int main()

{

   try

   {

       Base base;

       //base.fun();

   }

   catch (...)

   {

        //cout <<"get the catch"<<endl;

   }

}

的确可以运行。

因为析构抛出来的异常，在到达上一层析构节点之前已经被别的catch 块给处理掉。那么当回到上一层异常函数时， 其SEH 没有变，程序可以继续执行。

这也许就是为什么C++不支持异常中抛的异常


#### 参考资料
            https://www.cnblogs.com/carter2000/archive/2012/04/28/2474960.html

           https://www.cnblogs.com/zhyg6516/archive/2011/03/08/1977007.html

           https://blog.csdn.net/melody_1016/article/details/52966355

           https://blog.csdn.net/zyq0335/article/details/7657465


>Contributes: xxx
>
>Reviewers : Hollis, Kevin Lee
