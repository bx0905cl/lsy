一、Zernike多项式是在单位圆上正交的无限多多项式集合。

二、使用这些多项式作为基函数集来参数化望远镜中存在的波前误差的原因：

​        低阶多项式对应着大尺度波前误差，这些误差可以很好地通过OOF技术来约束；

​        其中一些多项式对应着常见于望远镜中的像差

三、zernike多项式的公式：

<img src="D:\workspace\大佬网站内容\Zernike Polynomials.assets\Zernike Polynomials.assets\image-20231117193134332.png" alt="image-20231117193134332" style="zoom:50%;" />

<img src="D:\workspace\大佬网站内容\Zernike Polynomials.assets\Zernike Polynomials.assets\image-20231117193150992.png" alt="image-20231117193150992" style="zoom:50%;" />

这个公式涉及到阶乘、幂运算和多项式项的求和，因此计算过程可能相对复杂。总的来说，通过调整 *n* 和 m这两个参数，可以获得不同阶数和模式的 Zernike 多项式，用于描述不同类型的波前畸变。在实际应用中，通常使用专门的数学软件或库来计算Zernike多项式。

OOF系统用于计算Zernike多项式的**代码**，从软件页面下载bnlib组件，然后查找名为“**zernikepoly.hxx/cxx**”的文件

```
#include "zernikepoly.hxx"
#include "bndebug.hxx"

namespace BNLib{
  static int factorial( int n ); 
  //这是一个静态函数，用于计算阶乘。它递归的计算了整数‘n’的阶乘，直到'n'变为小于2

  static int factorial( int n )
  { 
    return ( n < 2 ) ? 1 : n * factorial( n - 1 ); 
  }
  /*return ( n < 2 ) ? 1 : n * factorial( n - 1 );: 这是一个递归定义的阶乘函数。
   递归算法：在于将一个复杂问题分解为规模更小的相同问题直到问题规模小到可以用非常简单直接的方式来解决；在算法方面明显特征就是调用    自身；必须有一个基本结束条件。
   递归调用部分： n * factorial( n - 1 ) ；基本结束条件：(n < 2) ? 1，即当 n 小于 2 时，直接返回 1，结束递归
   ?: 是 C 语言中的条件运算符。
   condition ? expression_if_true : expression_if_false;
   condition 是一个表达式，其值为真或假。
   如果 condition 为真，那么整个表达式的值就是 expression_if_true。
   如果 condition 为假，那么整个表达式的值就是 expression_if_false。
   如果 (n < 2) 为真，整个表达式的值就是 1。
   如果 (n < 2) 为假，即 n 大于等于 2，整个表达式的值就是 n * factorial(n - 1)。
  */
  
  ZernPoly::ZernPoly( int n, int l  ): 
  /*ZernPoly(int n,int l)构造函数接受两个参数'n'，'l'，分别表示Zernike多项式的径向和角向阶数。在构造函数中进行了以下操作：
    计算'm'：'m'表示'l'的绝对值，用于确定多项式的'cos'或'sin'依赖性
    计算'nradterm'：这个变量用于确定在计算多项式时需要的项数
    初始化'radcoeffs'和'radpowers'：这两个成员变量是'valarray'类型的数组，用于存储多项式的系数和幂次
    通过循环计算多项式的系数，根据Zernike多项式的定义进行计算
  */
    n(n),
    l(l),
    m( l >= 0 ? l : -l),
    nradterm ( 1 + ( ( n - m ) / 2 ) ),
    radcoeffs(0.0, nradterm),
    radpowers(0 , nradterm )
  {

    ENFORCE( n >= 0);

    ENFORCE( m <= n );

    // Make sure n-m is not odd
    ENFORCE( ( ( n-m) & 1 ) == 0 );

  
    int sign = -1;
    //初始化一个变量 sign，用于在每次循环中交替为 -1 和 1，以改变系数的正负号
    for ( int s = 0; s < nradterm; s++ ) //循环变量 s 从 0 开始，逐步递增，直到 nradterm - 1。
      {
      sign *= -1;      
    //改变 sign 的符号，交替为 -1 和 1。 
      radpowers[s] = n - 2 * s;
    //计算幂次 radpowers[s]，其值为 n - 2 * s。
      radcoeffs[s]  = sign * factorial( n - s ) / 
	( factorial( s ) * factorial( ( n + m ) / 2 - s ) * factorial( ( n - m ) / 2 - s ) );
	/*计算系数 radcoeffs[s]。
      sign 控制正负号。
      factorial(n - s) 计算阶乘 (n - s)!。
      factorial(s) 计算阶乘 s!。
      factorial((n + m) / 2 - s) 计算阶乘 ((n + m) / 2 - s)!。
      factorial((n - m) / 2 - s) 计算阶乘 ((n - m) / 2 - s)!。
      这些阶乘项组合起来，形成 Zernike 多项式的系数。
    */

      }

  }
 
  ZernPoly::~ZernPoly()
  {
  }

  double ZernPoly::operator()( double x, double y )
 
  {
    double r= sqrt ( pow(x,2) + pow(y,2) );
    //计算极坐标系中的径向坐标 r，即点 (x, y) 到原点的距离。
    double phi = atan2(y,x);
    //计算极坐标系中的极角 phi，即点 (x, y) 对应的极坐标角度。
    double v = 0.0;
    //初始化变量 v 为 0.0，用于累加 Zernike 多项式的值。
    for ( int s = 0; s < nradterm; s++ )  //循环计算径向部分的 Zernike 多项式值。
      {
	v += radcoeffs[s] * pow( r, radpowers[s] );
      }
    //累加每一项的贡献，其中 radcoeffs[s] 是系数，radpowers[s] 是幂次。

    if ( l > 0 )
      {
	v *= cos( l * phi );
      }
    else if ( l < 0 )
      {
	v *= sin( - l * phi ); 
      }
    /*角向修正：
    如果 l > 0，则乘以 cos(l * phi)。
    如果 l < 0，则乘以 sin(-l * phi)。
   */

    return v;
  }

  
  size_t ZernIFromNL(int n, int l)
  {
    return ( n*(n+1) + n+ l)/2 ; 
  }


}

```

这段代码实现了 Zernike 多项式的计算和相关功能:

**`factorial` 静态函数**：

这是一个用于计算阶乘的静态函数。它递归地计算整数 n的阶乘，直到 n 变为小于 2。

**`ZernPoly` 类**：

这是一个表示 Zernike 多项式的类。在构造函数中，它接受两个参数 n 和 l，分别表示 Zernike 多项式的径向和角向阶数。

在构造函数中进行了一系列操作：

- 计算 m：m 表示 l 的绝对值，用于确定多项式的 cos 或 sin 依赖性。
- 计算 nradterm：一个变量，用于确定在计算多项式时需要的项数。
- 初始化 radcoeffs 和 radpowers：这两个成员变量是 valarray 类型的数组，用于存储多项式的系数和幂次。
- 通过循环计算多项式的系数，根据 Zernike 多项式的定义进行计算。

- operator() 函数用于计算 Zernike 多项式在给定坐标 (x, y) 处的值。它执行以下操作：
  - 计算极坐标系的径向坐标 r和极角 phi。
  - 初始化 v为 0.0。
  - 通过循环，使用事先计算的系数和幂次，计算多项式的值 v。
  - 最后，如果角向阶数 l 大于 0，则将 v 乘以 cos(l * phi)；如果 l 小于 0，则将 v 乘以 sin(-l * phi)。

**`ZernIFromNL` 函数**：

- 这是一个用于计算 Zernike 多项式索引的函数，接受两个参数 n 和 l。它返回一个索引值，该值用于标识 Zernike 多项式系列中的特定多项式。





































































```
/**
  \file zernikepoly.hxx

  Bojan Nikolic <bn204@mrao.cam.ac.uk>, <bojan@bnikolic.co.uk>

  Zernike Polynomial functions
*/
#ifndef _BNLIB_ZERNIKEPOLY_HXX__
#define _BNLIB_ZERNIKEPOLY_HXX__

#include "binaryfn.hxx"

#include <valarray>

namespace BNLib {

  /*!
   * Defines a Zernike polynomial of radial order n and angular order
   * l.
   * 
   * Note under the definition used here that polynomials \f$ l < 0
   * \f$ have a \f$ sin\f$-like dependance on the azimuth angle and
   * polynomials with \f$l>0\f$ have a \f$ cos\f$-like dependance.
   */
  class ZernPoly :  public BinaryDD {

  public:
    /*! Radial order of the polynomial */
    const int    n;

    /*! Angular order of the polynomial */
    const int    l;

  private:
  
    const int    m;
    const int    nradterm;

    std::valarray<double>  radcoeffs;
    std::valarray<int>     radpowers;
  
  public:
    
    /* ---------- Constructors & Destructors -----------*/

    /*! 
      \param n is the radial  order
      \param l is the angular order
     */
    ZernPoly( int n, int l  );

    /*! destructor is trivial.
     */
    virtual ~ZernPoly();

    /* ---------- Inherited             ----------------*/  
    double operator()( double x, double y );

  };

  /*! Return the sequential zernike polynomil number i from the radial
   * order n and angular order l*/
  size_t ZernIFromNL(int n, int l);

}

#endif

```

这段 C++ 代码定义了一个头文件 `zernikepoly.hxx`，其中包含了 Zernike Polynomial 相关的函数和类的声明:

**命名空间**：

命名空间的解释：

当一个班上有两个名叫 Zara 的学生时，为了明确区分他们，我们在使用名字之外，不得不使用一些额外的信息，比如他们的家庭住址，或者他们父母的名字等等。

同样的情况也出现在 C++ 应用程序中。例如，可能会写一个名为 xyz() 的函数，在另一个可用的库中也存在一个相同的函数 xyz()。这样，编译器就无法判断使用的是哪一个 xyz() 函数。

因此，引入了命名空间这个概念，专门用于解决上面的问题，它可作为附加信息来区分不同库中相同名称的函数、类、变量等。

namespace BNLib { ... }：将后续声明的类和函数放置在 `BNLib` 命名空间中，防止命名冲突。

**`ZernPoly` 类的声明**：

- `class ZernPoly : public BinaryDD { ... }`：声明了一个名为 `ZernPoly` 的类，该类继承自 `BinaryDD` 类。`BinaryDD` 类可能定义了一些二元函数的基本功能。

**成员变量**：

const int n;：表示 Zernike 多项式的径向阶数。

const int l;：表示 Zernike 多项式的角向阶数。

const int m;：表示 Zernike 多项式的 l 的绝对值，用于确定多项式的 sin 或 cos 依赖性。

const int nradterm;：用于确定在计算多项式时需要的项数。

std::valarray<double> radcoeffs;：valarray 类型的数组，用于存储多项式的系数。

std::valarray<int> radpowers;：valarray 类型的数组，用于存储多项式的幂次。

**构造函数和析构函数**：

ZernPoly(int n, int l);：构造函数，用于初始化 Zernike 多项式的对象。

virtual ~ZernPoly();：析构函数，用于清理对象的资源。这里声明为虚拟析构函数，可能是为了支持多态性。

**operator() 函数**：

double operator()(double x, double y);：用于计算 Zernike 多项式在给定坐标 (x, y) 处的值。

**ZernIFromNL函数声明**：

size_t ZernIFromNL(int n, int l);：声明了一个函数，用于计算 Zernike 多项式的索引。



四、下表显示了前二十个Zernike多项式及其对应的模型光束。这些图像实际上具有512×512的分辨率，可以以此分辨率查看，方法是将它们保存到磁盘上或者在Firefox或Mozilla中，右键单击图像，然后选择“查看图像”

​        表格的第一列显示了所显示的Zernike多项式使用的一些Label。n=,l=标签显示了多项式的径向（n）和角向（l）阶数。OOF=标签显示了OOF软件使用的标签。GBT=标签是GBT控制软件使用的标签。

​        第二列显示了与Zernike多项式相对应的波前相位的表示。它已在单位圆上计算，多项式的振幅在圆（孔径）边缘处为一弧度

​        第三、第四和第五列显示了与第二列中显示的像差相对应的在焦点内、在正焦点外和在负焦点外的光束。离焦的大小为两弧度，位于孔径边缘。

这些图形显示了在OOF全息拟合中通常使用的Zernike多项式，以及相应的在焦点内和离焦的光束。

<img src="C:\Users\刘思宇\AppData\Roaming\Typora\typora-user-images\image-20231109134451772.png" alt="image-20231109134451772" style="zoom:67%;" />

<img src="C:\Users\刘思宇\AppData\Roaming\Typora\typora-user-images\image-20231109134519579.png" alt="image-20231109134519579" style="zoom:67%;" />

<img src="C:\Users\刘思宇\AppData\Roaming\Typora\typora-user-images\image-20231109134535612.png" alt="image-20231109134535612" style="zoom:67%;" />