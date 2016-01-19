<link rel="stylesheet" type="text/css" href="/css/GitHub2.css">
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
    jax: ["input/TeX", "output/HTML-CSS"],
    tex2jax: {
        inlineMath: [ ['$', '$'] ],
        displayMath: [ ['\[', '\]']],
        processEscapes: true,
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre', 'code']
    },
    messageStyle: "none",
    "HTML-CSS": { preferredFont: "TeX", availableFonts: ["STIX","TeX"] }
});
</script>
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

# 第四章-3D图形中的数学

## 本章我们会学到什么
- 向量是什么，以及为什么我们要关心它们
- 矩阵是什么，以及为什么我们要关心它们
- 我们如何使用矩阵和向量来移动几何图形
- OpenGL约定和坐标空间是什么

到目前为止，我们已经学习了绘制点、线以及三角形，并写过几个简单的着色器传递硬编码的顶点数据。我们还未真正地渲染过3D的内容--在一本3D图形的书中这很奇怪！好吧，要把一系列形状转换到连贯的场景中，我们必须将它们互相以及和视见者安排好。在本章，我们将开始在坐标系统中移动形状和对象。在场景中摆放并设置对象的朝向对于每个3D图形程序员都是重要的能力。我们将会看到，在原点描述好对象的尺寸然后变换到目标位置实际上是很方便的。

## 本章是可怕的数学课吗？
在大多数3D图形编程的书中本章应该就是可怕的数学课了。然而，你可以放轻松，我们会采用比较适度的方式来描述这些原理，而不是干巴巴的文字。

我们的着色器会执行的第一个基础的数学操作就是坐标转换，归结起来就是乘上矩阵和向量。OpenGL程序员用来进行对象和坐标变换的关键是两个矩阵约定。为了让我们熟悉这些矩阵，本章在两种极端的计算机图形哲学中采取折中。一方面，我们可以警告你：“请在阅读本章前去复习下线性代数课本”。另一方面，我们可以欺骗你假装消除你的疑虑，你可以“学习3D图形而丝毫不用关心那些复杂的数学公式”。但我们不赞成任何一个阵营。

实际上，就算你没有非常出色地理解3D图形中的数学，你也可以继续前进，就好像我们每天都开车但不用了解机动车修理和内燃机引擎。但我们最好对车有足够的了解，我们需要常常换油，定期为油箱加油，当轮胎秃了要进行更换。这些知识会让你成为一个负责的、安全的机动车主。如果你像成为一个负责的、有能力的OpenGL程序员，同理。你至少需要懂得基础这样你可以知道什么可以做以及哪个工具最适合工作。如果你是个初学者，你会发现通过一些实践，矩阵数学和向量会逐渐变得易于理解，而且你也会开发出更好的直觉和强大的能力以全面利用本章所介绍的概念。

所以就算你还没有在脑海中将两个矩阵相乘的能力，你也需要知道矩阵是什么，它们如何成为OpenGL 3D魔法的服务手段。不过在我们掸去《线性代数》课本的尘土之前，不要怕：本书的`sb7`库有一个称为`vmath`的组件，它包含有很多有用的类和函数，可以用来表示以及操作向量和矩阵。它们可以直接和OpenGL一起使用，在语法和外观上与GLSL(我们用来编写着色器的语言)神似。所有，我们不需要自己全部来做矩阵和向量的操作，不过知道它们是什么以及如何应用它们仍然是大有裨益的。

## 3D图形数学的一个快速课程
首先，在这里我们不会假设我们会涉及到所有重要的东西。实际上，我们甚至不会涉及到应该知道的所有东西。在本章，我们仅仅会涉及到真正需要知道的东西。如果你已是一个数学能手，你应该立刻跳到后面关于标准3D变换的章节。不仅仅是你已经知道我们会涉及到什么，而且大多数数学迷会觉得不愉快，我们不会给他们喜爱的齐次坐标空间(homogeneous coordinate spaces)功能太多的发挥空间。想象那些真实的TV秀，我们现在必须从一个充满鳄鱼的沼泽中逃出去。知道多少真正需要的3D数学才能生存下来？那就是接下来两节会讲到的--3D数学生存技巧。鳄鱼才不会关心我们是不是真的知道什么是齐次坐标空间。

### 向量(Vector)
OpenGL的主要输入为顶点，顶点有一些属性通常包括位置。最基本的，这是在xyz坐标空间的位置，而且一个给定的位置在这个坐标空间中有且只有一个xyz三元组。一个xyz三元组可以表示为一个向量(实际上，对于纯粹的数学之心来说，一个位置其实也是一个向量--看，我们丢给了你一根骨头...)。当操作3D几何时向量可能是唯一最重要的基础概念。三个值(x,y和z)的组合表达了两个重要的值：方向(direction)和程度(magnitude)。

图示4.1展示了一个坐标空间的随意选取的点，并且有一个从坐标系统原点指向它的箭头。当我们构造三角形时可以把这个点想成顶点，这个箭头可以想象成向量。首先一个向量，同时也是最简单地，它代表一个从坐标空间原点到点的方向。在OpenGL中我们总是用向量来表示方向量。比如，x轴可以表示为向量(1,0,0)。这个向量说明在x方向上前进一个正单位，在y和z方向上不动。向量还用来表示我们要去哪如何指示--比如，摄像机指向哪个方向，或者我们要逃向哪个方向可以远离鳄鱼。

![figure4.1](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure4.1.png)

向量在OpenGL操作中是如此的基础性，在GLSL中各种大小的向量是一等类型，并且被赋予名称，比如**vec3**、**vec4**(分别代表三元、四元向量)。

向量可以表达的第二个量是程度。向量的程度是向量的长度。比如x轴向量(1,0,0)，它的长度是1。一个长度为1的向量我们称为*单位向量(unit vector)*。将一个不是单位向量的向量进行缩放为单位向量的过程，我们称为正规化。正规化一个向量使它的长度变为1然后这个向量可以说是正规化过的。当我们只想表达方向而不关心程度时单位向量很重要。同时，如果向量长度出现在我们使用的等式中，那向量的长度为1将会使等式变得简单多！向量的程度也是极重要的，比如，它指示我们应该在某个方向上跑多远
--多远才能远离鳄鱼。

向量和矩阵在3D图形中是如此重要的概念，以至于它们是GLSL(我们编写着色器使用的语言)的一等公民。不过在C++中并不是如此。为了让我们在C++中使用向量和矩阵，本书提供的`vmath`库包含有与GLSL中等价的同名类用来表示向量和矩阵。比如，`vmath::vec3`可以表示一个三分量浮点向量(x,y,z)，`vmath::vec4`可以表示一个四分量浮点向量(x,y,z,w)。*w*坐标添加到向量中使得它变为齐次坐标空间向量，不过它通常设置为1.0。之后x,y和z值可能会被w除，不过当w是1.0时，就不用管了。`vmath`中的类实际上是模板类，可以传递参数实例化为单精度和双精度浮点值，以及带符号整型和非负整型变量。`vmath::vec3`和`vmath::vec4`其实是如下定义：

	typedef Tvec3<float> vec3;
	typedef Tvec4<float> vec4;

声明一个三分量向量：

	vmath::vec3 vVector;

如果我们包含`using namespace vmath;`在源代码中，我们甚至可以写成：

	vec3 vVector;

不过，在本书的例子中，我们总是会在使用`vmath`库时显示带上`vmath::`命名空间。`vmath`中的类都定义了好几个构造函数和拷贝函数，这意味我们可以如下声明和初始化：

	vmath::vec3 vVertex1(0.0f, 0.0f, 1.0f);
	vmath::vec4 vVertex2 = vec4(1.0f, 0.0f, 1.0f, 1.0f);
	vmath::vec4 vVertex3(vVertex1, 1.0f);

现在，一个三个顶点的数组，比如一个三角形，可以声明如下：

	vmath::vec3 vVerts[] = { vmath::vec3(-0.5f, 0.0f, 0.0f),
							 vmath::vec3(0.5f, 0.0f, 0.0f),
							 vmath::vec3(0.0f, 0.5f, 0.0f) };

这看起来很像我们在第二章-“绘制我们的第一个三角形”中的代码。`vmath`库还包含很多数学相关的函数并且覆盖了类的大多数运算子以支持向量和矩阵的加、减、乘、移项等等。

我们也不能太无视第四个分量*w*的存在。大多数时候当我们用顶点位置指定一个几何图形时，三分量的顶点就是我们想要存储和发送给OpenGL的。对很多方向性向量来讲，比如表面法线(surface normal)(一个垂直于表面的向量，用来计算光照)，三分量的向量就足够了。不过，我们马上会深入研究的世界矩阵(world of matrices)，以及变换一个3D顶点，我们就必须要用一个4x4的变换矩阵与它相乘。规则是一个四分量向量我们必须用一个4x4的矩阵来乘。如果我们试图用一个三分量向量和4x4矩阵相乘，那鳄鱼就会吃了我们。如果你想在向量上做自己的矩阵操作，大多数时候你会想要四分量的向量。

### 常规向量运算
向量支持诸多运算，比如加、减、取反，等等。这些运算对每个分量进行计算然后结果是与输入同样大小的向量。`vmath`向量类覆盖了加、减以及取反运算以及其他一些运算。这使得我们可以编写如下代码：

	vmath::vec3 a(1.0f, 2.0f, 3.0f);
	vmath::vec3 b(4.0f, 5.0f, 6.0f);
	vmath::vec3 c;

	c = a + b;
	c = a - b;
	c += b;
	c = -c;

不过向量还有很多操作，我们在接下来的几个小节会从数学角度进行解释。这些操作也同样在`vmath`库中进行了实现。

#### 点积(Dot Product)
向量可以加、减以及缩放，通过简单地分别对xyz分量进行加、减或者缩放。有一个有趣并且有用的操作能且只能应用在两个向量上，那就是*点积(dot product)*，有时候也称为*内积(inner product)*。两个三分量的向量的点积点积得出一个标量(scalar)，这个标量是两个向量夹角的余弦值(cosine)并乘上它们的长度。如果两个向量都是单位长度，则结果位于-1.0到1.0之间，也是它们夹角的余弦值。当然，如果想得到它们真正的夹角，我们还需要取结果的反余弦或者反余切。点积在光照计算中被用到，在漫反射光照计算中两个向量分别是表面法线向量和一个指向光源的向量。在第13章“光照模型”中我们会深入这种类型的着色器代码。图示4.2展示了两个向量，V1和V2，它们的夹角θ表示为。

![figure4.2](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure4.2.png)

V1和V2的点积可被计算为：

V1 x V2 = V1.<sub>x</sub> x V2.<sub>x</sub> + V1.<sub>y</sub> x V2.<sub>y</sub> + V1.<sub>z</sub> x V2.<sub>z</sub>

`vmath`库中有一些有用的函数使用到点积运算。对于新手可以使用函数`vmath::dot`来计算两个向量的点积，或者使用向量类的成员函数`dot`。

	vmath::vec3 a(...);
	vmath::vec3 b(...);

	float c = a.dot(b);
	float d = vmath::dot(a, b);

使用一个稍微高级一点的函数`vmath::angle`可以获得两个向量夹角的弧度值。

	float angle(const vmath::vec3& u, const vmath::vec3& v);

#### 叉积(Cross Product)
另一个作用于两个向量的数学运算是*叉积(cross product)*，有时又被称为*向量积(vector product)*。叉积的结果是一个新的向量，它垂直于运算的两个向量形成的平面。两个向量V1和V2的叉积可被定义为：

	V1 X V2 = ||V1|| ||V2||sin(θ)n

其中*n*是垂直于V1和V2形成的平面的单位向量。意即如果我们将叉积的结果正规化，那我们就得到了运算平面的法线(normal)。如果V1和V2都是单位长度而且互相垂直，那叉积的结果不需要正规化即为单位长度。图示4.3展示了两个向量V1和V2，以及它们的叉积结果V3。

图示4.3 叉积的结果垂直于运算向量的平面：

![figure4.3](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure4.3.png)

两个三维向量V1和V2的叉积可被如下计算：

![formula4.1](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/formula4.1.png)

再次提醒，`vmath`库中有接收两个向量计算叉积的函数：一个是三分量向量类的成员函数，一个是全局函数。

	vmath::vec3 a(...);
	vmath::vec3 b(...);
	
	vmath::vec3 c = a.cross(b);
	vmath::vec3 d = vmath::cross(a, b);

不比点积，叉积的运算向量的顺序是很重要的。在图示4.3中，V3是V1XV2的结果。如果我们把V1和V2的顺序调换，V3就会指向相反的方向。叉积有很多应用，从三角形表面法线到构造变换矩阵。

#### 向量的长度
正如我们已经讨论过的，向量有方向和程度。向量的程度即是向量的长度。一个三维向量的程度可用如下等式计算：

![formula4.2](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/formula4.2.png)

这可以被总结为向量每个分量的平方和的平方根值，向量每个分量的平方和也是一个向量与自身的点积值。在二维空间中，这就是简单的毕达哥拉斯定理：斜边的平方等于其余两边的平方和。这一定理被扩展到任意维度，并且`vmath`库也有相应计算的函数。

	template <typename T, int len>
	static inline T length(const vecN<T,len>& v) { ... }

#### 反射和折射
计算机图形中普遍的运算就是计算反射和折射向量。给定一个入射向量R<sub>in</sub>以及一个表面N的法线，我们希望知道R<sub>in</sub>的反射方向R<sub>reflect</sub>，并且给定一个折射率η的索引，得出R<sub>in</sub>的折射方向。我们在图示4.4中展示这一情况，其中不同值的η对应的折射向量表示为R<sub>refract</sub>,η1到R<sub>refract</sub>,η4。

图示4.4 反射和折射：

![figure4.4](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure4.4.png)

尽管图示4.4展示的是二维的情形，但我们感兴趣的在三维中的情况(毕竟这是一本3D图形的书)。计算R<sub>reflect</sub>的数学等式为：

R<sub>reflect</sub> = R<sub>in</sub> - (2N · R<sub>in</sub>)N

计算给定η值的R<sub>refract</sub>的数学等式为：

![formula4.3](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/formula4.3.png)

要得到预期的结果，R和N都必须是单位向量(意即，它们在被使用之前需正规化处理)。`vmath`中的`reflect()`和`refract()`函数实现了这些等式。

### 矩阵
矩阵(matrix)不仅是一部好莱坞电影三部曲，也是一个异常强大的数学工具，它可以极度简化解决一个或多个等式中变量和其他等式变量有复杂关系的情况。对于图形程序员之心来说近在眼前的这种情况的一个常用示例就是坐标变换。比如，如果我们有一个空间中的点表示为(x,y,z)，我们想知道如果将这个点绕着任意点以任意方向旋转一定角度之后这个点会在哪，我们需要使用矩阵。为什么呢？因为新的x坐标不仅依赖于旧的x坐标和其他旋转参数，还依赖于y和z坐标。这种变量和解决方案之间的依赖仅仅只是矩阵擅长的一类问题。对于电影Matrix的数学爱好者粉丝，“matrix”确实是一个极佳的标题。

在数学上，矩阵无非就是一系列数字以统一的行和列排列好--用编程的行话来说，就是二维数组。矩阵不需要是正方的，但所有行必须有相同数目的元素，所有列必须有相同数目的元素。。如下为一个矩阵，它没有什么特殊意义，这是演示矩阵的结构。

![cookie4.1](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/cookie4.1.png)

值得提及的是，一个矩阵可以单列或单行的。单行或单列的几个数字可以被更简单地称为我们前面已讨论过的向量。实际上，马上我们就会看到，很多矩阵可以被当做一张列向量的表。

“矩阵”和“向量”是两个重要的专业术语，我们在3D图形编程资料中会经常看到。当处理这些数量时，我们会看到专业术语“标量(scalar)”。一个标量只是一个普通的数字，要来表示一个程度或者一个特殊的数量(你懂的--一个正常的熟悉的、单调的、简单的数字...就像以前一样，让我们把这个行话加到我们的词汇中去)。矩阵可以和其他矩阵相乘或者相加，不过他们也可以和向量或者标量相乘。一个矩阵(表示一个变换)乘上一个点(用向量表示)产生一个变换过的点(另一个向量)。矩阵变换乍看有点可怕但实际上不难理解。因为对矩阵的理解是很多3D任务的基础，所以我们必须试图熟悉它们。幸运的是，只需要一点点的理解就足以让我们前进并且使用OpenGL做很多不可思议的事情。过一阵子，并且通过一些实践和学习，我们就会自己掌握这一数学工具的。

与此同时，就像之前的向量一样，我们会从`vmath`库中找到很多有用的矩阵函数和功能。这个库的源代码在本书的源代码文件夹的`vmath.h`文件中。这个3D数据库极大地简化本章和接下来的任务。这个库的一个“有用的”特性是它没有很聪明的和高度优化的代码！这使得这个库高度可移植而且容易理解。我们还会发现它有恨类似GLSL的语法。

在我们使用OpenGL进行3D编程任务时会用到三种规格的矩阵：2x2，3x3，以及4x4.`vmath`库中有着与GLSL中对应的这些矩阵数据类型，比如：

	vmath::mat2 m1;
	vmath::mat3 m2;
	vmath::mat4 m3;
	
一如GLSL，`vmath`库中的矩阵类定义常用的运算，比如加、减、取反、乘、除，还有构造函数和相关的运算。再次强调，`vmath`库中的矩阵类是使用模板构造的，并且包含有单、双精度浮点数以及有、无符号整型数类型定义。

### 矩阵构造和运算
表示一个4x4的矩阵，OpenGL并不是当做一个浮点数二维数组，而是当成一个16个浮点数一维数组。缺省情况下，OpenGL对矩阵使用列优先或者说是列为主的布局。意即，对于一个4x4的矩阵，前四个元素表示矩阵的第一列，然后四个元素表示矩阵的第二列，以此类推。这种方法与很多数学库不同，大多数数学库都使用二维数组的方式。举个栗子，如下两种表示方式OpenGL会倾向于第一种：

	GLfloat matrix[16];	  // Nice OpenGL-friendly matrix
	GLfloat matrix[4][4];	// Not as convenient for OpenGL programmers
	

OpenGL可以使用第二种变体，但第一种是更高效的表示方式。我们马上就会搞明白这是为什么。16个元素表示4x4的矩阵，如下所示。

![cookie4.2](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/cookie4.2.png)

当数组中的元素一个接一个地先表示矩阵的列时，我们称这为*列优先(column-major)*矩阵顺序。在主存中，二维数组表示的4x4矩阵(前面代码中的第二种)以*行优先(row-major)*顺序存储。以数学术语来说，两者互为转置矩阵。

在主存中使用列优先表示上述矩阵产生的数组如下：

    static const float A[] = 
    {
        A00, A01, A02, A03, A10, A11, A12, A13,
        A20, A21, A22, A23, A30, A31, A32, A33
    };

相对的，在主存中使用行优先表示上述矩阵为：

    static const float A[] = 
    {
        A00, A10, A20, A30, A01, A11, A21, A31,
        A20, A21, A22, A23, A30, A31, A32, A33
    };

这16个值真正的魔法在于它们可以表示空间的一个特定的位置以及相对视见者在三个坐标轴的一个方位。解释这些数字易如反掌。四个列每个表示一个四元素向量(实际上，`vmath`库在内部用自己的向量类的数组来表示矩阵，其中每个向量表示矩阵的一列)。对于本书来说，为了让事情变得简单，我们只关注前三列的向量的前三个元素。第四列向量包含变换过的坐标系统的原点的x，y和z值。

前三列向量的前三个元素只是方向性向量，它们表示在空间中的x，y，z方位。对于大多数用途来讲，这三个向量总是单位长度(除非我们应用了缩放或者剪切)，而且互成90度。用数学术语来讲(如果你想在朋友面前装逼的话)，当这三个向量互成90度，都是单位长度时，称为标准正交；当不是单位长度时，称为正交。图示4.5展示了一个4x4变换矩阵，其中关心的部分我们高亮了。注意矩阵的最后一行都是0除了最后的元素是1。

图示4.5 一个表示旋转和平移的4x4矩阵：

![figure4.5](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure4.5.png)

图示4.5中的左上3x3的子矩阵表示一个旋转或者方位。矩阵的最后一列表示一个平移或者位置。

最神奇的事是，如果我们有一个坐标系统内包含有位置和方位的4x4矩阵，然后这个4x4矩阵乘上一个等价的坐标系统内的顶点(表示为列矩阵或向量)，结果是一个变换到新坐标系统的新顶点。总而言之，空间内的任何位置和任何方位都能使用4x4矩阵唯一定义，并且如果这个矩阵乘上一个对象的所有顶点，我们可以将整个对象变换到空间内给定的位置和方位。

另外，如果我们使用一个矩阵将一个对象的顶点从一个空间变换到另一个空间，我们之后可以再用另一个矩阵变换刚才变换过的那些顶点，将它们再次变换到另一个坐标空间。给定矩阵A和B以及向量v，我们知道

    A·(B·v)

等价于

    (A·B)·v

这一关系存在是因为矩阵乘法是*结合性(associative)*的。这里有一个魔法：将所有的变换矩阵进行乘法得到的结果矩阵可以将所有变换堆到一起，然后在最终的乘积中使用结果矩阵。

我们的场景或者对象的最终表现和模型矩阵应用的次序有着重大关系。对于平移和旋转来说尤其如此。我们可以看到矩阵乘法的结合性和*交换性(commutativity)*规则。我们可以用任何我们喜欢的方式将变换的次序群组，因为矩阵乘法是结合性的，但是矩阵在乘法中的次序不能变动，因为矩阵乘法是不具备交换性的。

图示4.6(a)展示了一个正方形先沿z轴旋转然后沿变换后的新x轴平移，图示4.6(b)展示了同样的正方形先先沿x轴平移然后沿z轴旋转。最后两个正方形的布置是不一样的，因为每一个变换都是相对于上一次的变换。在图示4.6(a)中，正方形相对于原点先进行旋转。在图示4.6(b)中，正方形平移后，旋转则是绕着平移后的原点进行旋转。

图示4.6 模型变换：

![figure4.6](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure4.6.png)

## 理解变换
如果你仔细想想，其实大多数3D图形都不是真正的3D。我们使用3D的概念和术语来描述那些看起来很像3D的东西，然后这些3D的数据被“挤扁”到2D的计算机显示屏幕上。我们将3D数据“挤扁”到2D数据的过程称之为*投影(projection)*。我们描述顶点处理中发生的变换类型(正交-orthographic或者透视-perspective)时会使用投影，不过投影只是OpenGL中发生的变换的一种类型。变换使得我们可以旋转对象、移动对象，甚至拉伸、收缩以及弯曲对象。

### OpenGL的坐标空间
一系列的变换可以使用矩阵来表示，而且与一个矩阵相乘实际上将一个向量从一个坐标空间移动到另一个坐标空间。有几个坐标空间在OpenGL编程中被广泛使用到。在指定顶点到顶点显示到显示屏幕上的这段时间可以发生任意次几何变换，但大多数情况下是模型变换、视见变换和投影。在本章节中，我们对3D计算机图形中常用的坐标空间一一介绍，以及用来将向量在它们间进行移动的变换。

    *坐标空间*             *表示什么*
    模型空间               相对局部原点的位置。有时也称对象空间(object space)。
    
    世界空间               相对全局原点的位置。意即它们在世界中的位置。
    
    视见空间               相对视见者的位置。有时也称摄像机(camera)或者眼睛空间(eye space)。
    
    修剪空间               顶点投影到一个非线性齐次坐标后的位置
    
    标准设备坐标空间        顶点的修剪空间坐标除以各自的w分量之后即为标准设备坐标
    
    窗体空间               相对于窗体原点的顶点像素的位置

将坐标从一个空间移动到另一个空间的矩阵通常用那些空间来命名。比如，将一个对象的顶点从模型空间变换到视见空间的矩阵一般叫做模型-视见矩阵。

#### 对象坐标
通常情况下我们的顶点数据始于对象空间(模型空间)。在对象空间中，顶点的位置是相对于一个局部原点的。考量一个太空船模型。这个模型的原点可能是某个合乎逻辑的地方，比如太空船的鼻尖、重心或者是飞行员坐着的位置。在一个3D建模程式中，回到原点并缩得足够小之后应该将整个太空船展现给我们。一个模型的原点通常可能是我们的旋转点，基于此点旋转调整模型的方向。将原点放置在模型外很远的地方是不合理的，因为这样旋转对象时可能会伴随着很大的平移。

#### 世界坐标
下一个普遍的坐标空间是世界空间。这个空间中的坐标相对于一个固定的、全局的原点。继续拿太空船举例，这个原点可以是一个运动场的中心或者其他固定物体(比如临近的星球)。一旦处于世界空间，所有的对象都置身于一个共有的框架中。通常，光照和物理计算也在这个空间执行。

#### 视见坐标
贯穿本章的一个重要的概念是视见坐标，通常也被称作*摄相机(camera)*或者*眼睛坐标(eye coordinates)*。视见坐标相对于观察者位置(所以才有“摄相机”或者“眼睛”的术语)而不管任何可能发生的变换，我们可以把它们想成“绝对”坐标。所以，眼睛坐标表示一个虚拟的固定坐标系，当成引用的一个共有的框架。图示4.7展示了两个视见点的视见坐标系。

图示4.7 两个视见坐标的透视：

![figure4.7](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure4.7.png)

左边的视见坐标一如场景观察者所见(意即垂直于监视器)。右边的视见坐标系稍微旋转了一下，这样我们可以更好地看出z轴的关系了。从视见者的角度来看，正向x轴指向右，正向y轴指向上。正向z轴从原点指向用户，负向z轴从视见点指向显示屏幕内。显示屏幕位于z轴0处。

当我们使用OpenGL进行3D绘图时，我们使用笛卡尔坐标系。在没有任何变换的情况下，使用的坐标系与刚才描述的眼睛坐标系无异。

#### 修剪和标准设备空间(NDC)
修剪空间是OpenGL进行修剪的坐标空间。当我们的顶点着色器写入到`gl_Position`，这个坐标就被认为在修剪空间中了。这个坐标也总是一个四维的齐次坐标。在离开修剪空间时，顶点的四个分量都会除以*w*分量。显而易见的是，除完后*w*绝逼会变成1.0。如果在除之前*w*不是1.0，那*x*，*y*和*z*分量就会放大*w*的倒数。这样可以造成透视收缩以及投影的效果。除完后的结果被认为在标准设备空间(NDC)了。很显然，如果一个修剪空间坐标的*w*分量为1.0，那修剪空间和标准设备空间对于这个坐标来说就是等价的。

### 坐标变换
正如前面所述，坐标可以从空间移动到空间，通过将它们的向量表示与*变换矩阵(transformation matrices)*相乘。变换用来操作我们的模型以及它里面特定的对象。这些变换移动对象到特定空间、旋转它们，以及缩放它们。图示4.8展示了三种我们最常会用到的模型变换。图示4.8(a)展示了平移，一个对象沿着给定轴线移动。图示4.8(b)展示了旋转，一个对象绕着轴线旋转。最后图示4.8(c)展示了缩放的效果，对象的大小增长或减少一个特定的量。缩放可以是非统一的(不同的维度可以缩放不同的量)，所以我们可以使用缩放来拉伸或者收缩对象。

图示4.8 模型变换：

![figure4.8](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure4.8.png)

这些标准变换都可以表示为一个矩阵，我们可以将顶点坐标和这些矩阵相乘来计算出变换后的顶点位置。接下来的几个小节讨论这些矩阵的构造，所有数学和使用到的函数都在`vmath`库中有提供。

#### 单位矩阵
有一些重要类型的变换矩阵在我们使用它们之前先要熟悉一下。第一个就是单位矩阵。如下所示，单位矩阵除了矩阵对角线上是清一色的1.0之外其他都是0.0。4x4的单位矩阵看起来像这样：

![cookie4.3](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/cookie4.3.png)

一个向量与单位矩阵相乘等价于与1相乘，这没什么卵用。

![cookie4.4](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/cookie4.4.png)

使用单位矩阵绘制的对象是不会变换的，它们就在原点处(单位矩阵的最后一列)，*x*、*y*和*z*轴与眼睛坐标一样。显而易见的是，2x2，3x3的单位矩阵以及其他维度的单位矩阵都存在并且都是对角线上清一色的1.0。所有的单位矩阵都是正方形的。没有不是正方形的单位矩阵。所有的单位矩阵都是它自己的转置矩阵。我们可以这样用C++生成一个OpenGL可用的单位矩阵：

    // Using a raw array:
    GLfloat m1[] = { 1.0f, 0.0f, 0.0f, 0.0f,     // X Column               0.0f, 1.0f, 0.0f, 0.0f,     // Y Column
                     0.0f, 0.0f, 1.0f, 0.0f,     // Z Column
                     0.0f, 0.0f, 0.0f, 1.0f };   // W Column
                     
    // Or using the vmath::mat4 constructor:
    vmath::mat4 m2 = { 1.0f, 0.0f, 0.0f, 0.0f,     // X Column               0.0f, 1.0f, 0.0f, 0.0f,     // Y Column
                     0.0f, 0.0f, 1.0f, 0.0f,     // Z Column
                     0.0f, 0.0f, 0.0f, 1.0f };   // W Column
                     
`vmath`库中也有为我们构造单位矩阵的便利方法。每个矩阵类都有一个静态成员函数生成一个相应维度的单位矩阵：

    vmath::mat2 m2 = vmath::mat2::identity();
    vmath::mat3 m3 = vmath::mat3::identity();
    vmath::mat4 m4 = vmath::mat4::identity();
    
如果你能回想起来，在第二章的“我们的第一个OpenGL程式”中第一个顶点着色器是一个直通着色器。它不会变换顶点，只是简单地将硬编码的数据以缺省的坐标系原封不动地传递下去。我们本可以将所有顶点与单位矩阵相乘，但这肯定是一个浪费时间、精力毫无意义的操作。

#### 平移矩阵
平移矩阵就是简单地将顶点沿着一个或多个轴线进行移动。图示4.9展示了将一个立方体沿着*y*轴向上移动10个单位。

图示4.9 一个立方体在*y*正方向平移10个单位：

![figure4.9](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure4.9.png)

4x4平移矩阵的公式如下：

![cookie4.5](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/cookie4.5.png)

其中，t<sub>x</sub>、t<sub>y</sub>和t<sub>z</sub>分别代表x、y和z轴上的平移。

考查平移矩阵的结构显示出在3D图形中为什么我们要用四维齐次坐标表示位置的一个原因。一个*w*分量为1.0的向量*v*，与上面的平移矩阵相乘得出：

![cookie4.6](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/cookie4.6.png)

我们可以看到，t<sub>x</sub>、t<sub>y</sub>和t<sub>z</sub>都添加到了v的对应分量上，这样就产生了平移。如果向量*v*的*w*分量不曾为1.0，那使用这个矩阵进行平移就会导致t<sub>x</sub>、t<sub>y</sub>和t<sub>z</sub>被那个值所缩放，最终影响变换的输出结果。在实际应用中，位置向量几乎总是使用*w*分量为1.0的四分量向量来表示，不过方向向量可以用三分量向量或者*w*分量为0的四分量向量。所以一个四分量的方向向量与一个平移矩阵相乘是没什么卵用的。`vmath`库中有两个函数用以创建一个4x4的平移矩阵，可以使用三个分开的参数或者一个3D向量：

    template <typename T>
    static inline Tmat4<T> translate(T x, T y, T z) { ... }
    
    template <typename T>
    static inline Tmat4<T> translate(const vecN<T,3>& v) { ... }
    
#### 旋转矩阵
要将一个对象绕着三个坐标轴或者任意向量进行旋转，那我们需要使用一个旋转矩阵。一个旋转矩阵的形式依赖于我们想要绕着哪个轴旋转。想要绕着x轴旋转，我们使用：

![cookie4.7](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/cookie4.7.png)

其中，R<sub>x</sub>(θ)表示绕着x轴旋转θ角度。同样，要绕着y或z轴，我们可以使用：

![cookie4.8](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/cookie4.8.png)

将这三个矩阵相乘得出一个复合变换矩阵是可行的，然后在一次矩阵-向量乘法中分别对三个轴进行给定量的旋转。这样的矩阵像这样：

![cookie4.9](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/cookie4.9.png)

其中，S<sub>ψ</sub>、S<sub>θ</sub>和S<sub>φ</sub>分别表示ψ、θ和φ的正弦值，C<sub>ψ</sub>、C<sub>θ</sub>和C<sub>φ</sub>分别表示ψ、θ和φ的余弦值。如果这看起来太数学化了，别担心--`vmath`来帮忙：

    template <typename T>
    static inline Tmat4<T> rotate(T angle_x, T angle_y, T angle_z);
    
我们也可以绕着任意轴进行旋转，只要我们将轴向量的x,y,z值指定好。要想看到旋转轴，我们只需要绘制一条从原点到(x,y,z)点的一条线就可以了。`vmath`库也包含有生成角度-轴表示法的旋转矩阵：

    template <typename T>
    static inline Tmat4<T> rotate(T angle, T x, T y, T z);
    
    template <typename T>
    static inline Tmat4<T> rotate(T angle, const vecN<T,3>& axis);
    
这两种`vmath::rotate`函数的重载生成一个旋转矩阵，第一种表示绕着x,y,z指定的轴旋转`angle`度，第二种表示绕着向量*v*旋转`angle`度。其中，我们绕着由x,y,z参数指定的向量进行了旋转。旋转角度是逆时针方向以度进行度量的且由`angle`参数指定。在最简单的情况下，旋转只会绕着坐标系的一个主轴(x,y或者z)进行。

作为示例，如下的代码创建一个旋转矩阵，它将顶点绕着(1,1,1)的轴旋转45°，一如图示4.10刻画的一样。

    vmath::mat4 rotation_matrix = vmath::rotate(45.0, 1.0, 1.0, 1.0);
    
图示4.10：

![figure4.10](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure4.10.png)

值得注意的是本例中度的应用。这个函数在内部实际上会将度转为弧度，之所以这样定义接口，是因为很多程序员更愿意使用度。

#### 欧拉角(Euler Angles)
欧拉角是用来表示空间中方向的三个角的集合。每个角表示一个旋转，每个旋转绕着我们框架的三个正交向量(比如x,y,z轴)之一进行。我们已经了解到进行矩阵变换时顺序是很重要的，以不同的顺序进行某些变换(比如旋转)会产生不同的结果。这是因为矩阵乘法的不可交换性。所以给定一个欧拉角，我们应该先绕着x轴旋转，然后绕着y，绕后再绕着z还是应该与此相反又或者先绕y轴旋转？好吧，到目前为止我们大可不必烦忧，这不是太要紧。

将方向表示为三个角的集合有一些好处。比如，这种表示法较为直观，这在我们想要将角与用户界面关联起来时很重要。另一个好处就是这样会很容易进行角的插值计算，在一个点上构造一个旋转矩阵，最终的动画就可以看到一个平滑的、连续的运动。不过欧拉角也有一个很严重的缺陷--万向节死锁(gimbal lock)。

当一个角的旋转导致一个轴和另一个轴平行时万向节死锁就发生了。之后绕着现在这两个共轴的任一轴进行旋转都会对模型产生同样的变换，这样便丢失了一个维度的自由。所以欧拉角不适合连接变换或者累积旋转。

译者注：[更多关于万向节死锁的资料](http://www.cnblogs.com/soroman/archive/2008/03/24/1118996.html)。

为避免这种情况的发生，`vmath::roate`函数可以接收一个旋转轴和一个旋转角作为参数。当然，如果我们非要用欧拉角的话，一一指定x,y,z将三个旋转堆叠在一起也是行的，但是使用角-轴的表示方式来进行旋转是更可取的，或者使用*四元组(quaternions)*表示变换并且在必要的时候将它们转换成矩阵。

#### 缩放矩阵
我们最后的“标准”变换矩阵是缩放矩阵。缩放矩阵改变一个对象的大小，这通过扩张或收缩三个轴上的所有点以指定的因子来达成。一个缩放矩阵有如下的形式：

![cookie4.10](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/cookie4.10.png)

其中，S<sub>x</sub>、S<sub>y</sub>、S<sub>z</sub>分别表示x,y,z维度的缩放因子。使用`vmath`库来创建一个缩放矩阵与创建一个平移或者旋转矩阵的方法类似。有三个函数用来构造缩放矩阵：

    template <typename T>
    static inline Tmat4<T> scale(T x, T y, T z) { ... }
    
    template <typename T>
    static inline Tmat4<T> scale(const Tvec3<T>& v) { ... }
    
    template <typename T>
    static inline Tmat4<T>scale(T x) { ... }
    
第一个函数通过给定的x,y,z参数分别在x,y,z轴上独立地进行缩放。第二个函数和前面的功能一致，只是使用一个三分量向量而不是三个分开的参数来表示缩放因子。第三个函数在三个维度上都进行等量`x`的缩放。缩放并不需要三个维度上统一，我们可以在不同的方向上应用拉伸或者挤压。比如图示4.11中一个10x10x10的立方体可以在x和z方向上缩放。

图示4.11 非统一缩放的立方体：

![figure4.11](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure4.11.png)

### 连接变换
我们已经了解到，坐标变换可以表示为矩阵，并且将一个向量从一个空间变换到另一个空间涉及到一次简单的矩阵-向量乘法运算。被一系列矩阵相乘可以应用一系列的变换。没有必要将每次矩阵-向量乘法得出的中间向量保存起来。相比较而言，通常更倾向于先将相关变换的矩阵相乘得出一个可以表示整个变换序列的矩阵。这个矩阵之后可以用来将向量直接从源坐标空间变换到目标坐标空间。记住，顺序是很重要的。当使用`vmath`或者在GLSL中编写代码时，我们总是应该用一个矩阵乘上一个向量并且以相反的顺序解读变换序列。比如，看看下面的代码：

    vmath::mat4 translation_matrix = vmath::translate(4.0f, 10.0f, -20.0f);
    vmath::mat4 rotation_matrix = vmath::rotate(45.0f, vmath::vec3(0.0f, 1.0f, 0.0f));
    vmath::vec4 input_vertex = vmath::vec4(...);
    
    vmath::vec4 transformed_vertex = translation_matrix * 
                                     rotation_matrix *
                                     input_vertex;
                                     
这段代码首先将模型绕y轴旋转45°(因为`rotation_matrix`)自后沿着x轴平移4个单位，沿着y轴平移10个单位，沿着z轴负方向平移20个单位(因为`translation_matrix`)。这些变换将这个模型置于一个特定的方位然后移动到某一位置。反向解读变换序列给出运算的顺序(旋转，然后平移)。我们可以将上面的代码重写如下：

    vmath::mat4 translation_matrix = vmath::translate(4.0f, 10.0f, -20.0f);
    vmath::mat4 rotation_matrix = vmath::rotate(45.0f, vmath::vec3(0.0f, 1.0f, 0.0f));
    vmath::mat4 composite_matrix = translation_matrix * rotation_matrix;
    vmath::vec4 input_vertex = vmath::vec4(...);
    
    vmath::vec4 transformed_vertex = composite_matrix *
                                     input_vertex;
                                     
其中，`composite_matrix`是由平移矩阵乘上旋转矩阵形成的，这样形成了表示旋转然后平移的复合矩阵。这个矩阵之后可以用来变换任意数量的顶点或者其他向量。如果我们有很多顶点要变换，这样可以大大加快计算。每个顶点现在只需要进行一次矩阵-向量乘法运算而不是二次。

这里需要引起注意。很容易与我们编码一样从左到右解读(或写出)变换序列。如果我们按照那样的顺序将平移矩阵和旋转矩阵相乘，那我们就会先移动模型的原点，之后发生的旋转操作就会绕着新的原点，很可能将我们的模型玩坏。

### 四元组(Quaternions)
一个四元组是一个四维量，在某些方面和复数类似。它有一个实体部分和三个虚体部分(类比于复数的一个虚体部分)。一如一个复数有一个虚体部分*i*，一个四元组有三个虚体部分：*i*，*j*，*k*。在数学上一个四元组*q*表示为：

    q = (x + yi + zj + wk)
    
四元组的虚体部分和复数的虚体部分有着类似的性质，特别是：

i<sup>2</sup> = j<sup>2</sup> = k<sup>2</sup> = ikj = -1

同样，i,j,k任两数之积等于另外一数。所以：

    i = jk
    j = ik
    k = ji
    
基于以上我们可以推断出两个四元组乘积如下：

![cookie4.11](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/cookie4.11.png)

和复数一样，四元组的乘法也是不可交换性的(译者注：复数乘法的交换律在高维度中不再成立)。四元组的加法和减法与简单的向量加减无二，对应的分量加减即可。其他诸如取反和程度(magnitude)和四分量向量表现一致。尽管四元组是四分量的，但在实际中通常用一个标量实体部分和一个三分量向量虚体部分来表示。这种表示法通常写作：

![cookie4.12](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/cookie4.12.png)

好啦，很好--但本章不是可怕的数学课不是吗？这里只应该有计算机图形、OpenGL以及其他有趣的东西。对于四元组的准备工作我们就到此为此了。想想我们那接收一个角度和一个旋转轴的旋转函数。我们可以将那两个量表示为一个四元组，把角度填到实体部分，旋转轴填到向量部分，这样便能生成一个四元组表示一个绕任意轴的旋转。

一系列的旋转可以表示为一系列的四元组相乘，生成一个最终的四元组表示所有的旋转。虽然可以制定一串矩阵表示绕着各个笛卡尔轴进行旋转，然后将它们乘在一起，但这种方法容易产生万向节死锁。如果我们用一系列四元组来做同样的事情，将不会发生万向节死锁。为了便于我们编码，`vmath`库包含`vmath::quaternion`类实现了这里描述的大部分功能。

### 模型-视见变换
在一个简单的OpenGL应用中最常用的一个变换是将模型从模型空间弄到视见空间以便渲染它。实际上我们是先将模型移动到世界空间(意即相对于世界的原点进行摆放)然后再从世界空间变换到视见空间(相对于视见者摆放)。这个过程建立场景的视野据点。缺省情况下，在透视投影中观察点位于原点(0,0,0)处，俯视z轴负方向(进入监视器或显示屏幕里面)。这个观察点相对于视见坐标系进行移动从而提供一个特殊的视野据点。当观察点位于原点时，在透视投影中绘制于z轴正方向的对象将位于观察者之后。不过在正射投影(orthographic)中视见者被假设在z轴正方向无限远处并且可以看见所有在视见体(viewing volume)中的事物。

因为这一变换将顶点从模型空间(有时亦称对象空间)直接变换到视见空间而且实际上跳过了世界空间，它常被引用为模型-视见变换，表达这一变换的矩阵称为模型-视见矩阵。

模型变换本质上是将对象放置到世界空间。每个对象都可能有它自己的模型变换，通常由一系列缩放、旋转和平移运算组成。将模型空间中顶点的位置进行模型变换后的结果是世界空间的位置。这一变换有时被称为模型-世界变换。

视见变换使得我们将观察点置于任何地方以及看向任何方向。测定视见变换就像在场景放置摄像机并设置朝向。超级重要的是，我们必须在其他任何模型变换前应用视见变换。因为相对于眼睛坐标系就像是移动了当前的工作坐标系。接下来发生的变换都会基于新的修改的坐标系。将坐标从世界空间变换到视见空间的变换有时被称为世界-视见变换。

通过将模型-世界变换和世界-视见变换相乘连接起来得出模型-视见矩阵(意即这个矩阵将坐标从模型空间变换到视见空间)。这么做有几个好处。首先我们的场景可能有很多模型而且每个模型有很多顶点。使用一个单一的复合变换将模型变换到视见空间比先变换到世界空间再变换到视见空间更有效率。第二个好处就是单精度浮点数的数值精确性：世界可以很大并且世界空间中执行的计算依据顶点离世界原点的远近有不同的精度。然而，如果我们在视见空间执行同样的计算，那精度就是依据顶点到视见者的远近，这可能才是我们想要的--对于近的对象应用比较高的精度，对于远的对象应用比较低得精度。

#### 视向矩阵(Lookat Matrix)
如果我们在一个已知位置有一个视野据点并且有一个想要看向的事物，我们会希望把我们的虚拟摄像机放置到那个位置并指向正确的方向。为了正确调整摄像机，我们还需要知道哪个方向向上，否则摄像机会绕着正向轴(forward axis)旋转，即使从技术上来讲摄像机仍然会指向正确的方向，但这多半情况下肯定不是我们想要的。所以给定一个原点、一个兴趣点、一个我们认为是向上的方向，然后我们想要构造一系列变换(理想的情况将它们合并为一个)矩阵将摄像机旋转到正确地方向并且将原点平移到摄像机的中心。这个矩阵被称为*视向矩阵(lookat matrix)*，它可以依靠目前为止我们在本章学习过的数学来构造。

首先我们知道将两个位置相减会得到从第一个位置到第二个位置的向量，如果将这个向量标准化会得出它的方向性。所以如果我们将兴趣点的坐标减去摄像机的坐标，然后标准化最终结果的向量，我们就会得到一个表示从摄像机到兴趣点的视见方向。我们将这个向量称为*正向向量(forward vector)*。

然后我们知道取两个向量的叉积，会得到一个垂直于原先两个向量的第三个向量。好，我们有了两个向量--我们刚才计算出的正向向量和向上向量(up vector)，向上向量表示我们认为是向上的方向。取这两个向量的叉积得出第三个垂直于这两个向量的向量并且指向摄像机的旁边。我们称这个为*侧向量(sideways vector)*(也有称右向量*right vector*)。不过正向向量和向上向量并不需要互相垂直而且我们需要第三个正交向量来构造旋转矩阵。要得到这个向量，我可以简单地重复刚才的计算--取正向向量和侧向量的叉积，得到表示摄像机向上的正交向量。

这三个向量都是单位长度并且互相垂直，所以它们形成一个标准正交基准向量并且表示我们的视见框架。给定这三个向量，我们可以构造一个旋转矩阵将标准笛卡尔基准里的点变换到我们的摄像机基准中。在如下的数学计算中，*e*是眼睛(或者说摄像机)的位置，*p*是兴趣点的位置，*u*是向上向量。让我们一步一步看看：

首先构造我们的正向向量*f*：

![formula4.4](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/formula4.4.png)

然后取*f*和*u*的叉积构造侧向量*s*：

    s = f x u
    
现在构造摄像机的向上向量*u'*：

    u' = s x f
    
最后构造一个旋转矩阵表示重新定位到新构造的标准正交基准：

![formula4.5](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/formula4.5.png)

事情到这还没完。要将对象变换到摄像机框架中，我们不仅要将事物的方向调正，还需要将对象的原点移动到摄像机位置中。我们简单地将最终结果向量向摄像机负向的位置平移。还记得平移矩阵是怎么构造的？简单地把偏移量放到矩阵的最右列就好。我们来看看：

![formula4.6](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/formula4.6.png)

最终，我们得到视向矩阵T。

如果这看起来太繁冗了，正好`vmath`库中有一个函数替我们构造这个矩阵：

    template <typename T>
    static inline Tmat4<T> lookat(const vecN<T,3>& eye,
                                  const vecN<T,3>& center,
                                  const vecN<T,3>& up) { ... }
                                  
`vmath::lookat`函数生成的矩阵可被当做我们摄像机矩阵的基准--这个矩阵摄像机的位置和方向。换言之，这个就是我们的视见矩阵。

### 投影变换
投影变换应用在模型-视见变换之后的顶点上。投影实际上定义了视见体并建立修剪平面。修剪平面是3D空间中的平面方程，OpenGL用它来判断几何图形是否可被视见者看见。更确切地说，投影变换指定一个完成的场景(所有的建模都已完成)如何投影成显示屏幕上的最终图像。我们对两种投影进行更多的了解--正射(orthographic)投影和透视(perspective)投影。

在正射投影或者平行投影中，所有的多边形都以指定尺寸成比例的精确绘制在显示屏幕上。线段和多边形都直接用平行线映射到2D显示屏幕上，这意味着无论事物的距离有多远，它都以同样多小绘制，就只是压扁到显示屏幕上。这种投影经常用在渲染二维图像，比如设计图中的前视图、俯视图、侧视图或者文本、显示屏幕上的菜单这样的二维图形。

透视投影将一个场景展示得更像是现实生活中的情况，而不是设计图那样。透视投影的特点就是透视缩短，这使得同样大小的对象在远处看起来小一点而在近处看起来大一点。3D空间中平行的线对于视见者来说并不总是平行的。就拿铁轨来说，两侧轨道是平行的，但在透视投影中，它们看起来就像在远处的一个点相交了。透视投影的好处是我们不必指定线段在何处相交或者远处的对象应该有多小。我们所需要做的就是用模型-视见变换指定场景然后应用透视投影矩阵。线性代数会为我们搞定其他的事。

图示4.12比较了两个不同场景的正射投影和透视投影。左边的正射投影中可以看到，随着这些立方体离视见者愈加远它并没有变得愈加小。然后在右边的透视投影中，随着立方体离视见者愈加远它变得愈加小。

图示4.12 正射投影与透视投影的比较：

![figure4.12](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure4.12.png)

正射投影通常用于2D绘制用途，我们希望绘制单元和像素之间有精确的一致。我们可能会在示意图、文本或者2D图形应用中使用它。当然我们也可以在3D渲染中使用正射投影，当渲染深度与视点距离只有很小的差别时。透视投影用于渲染包含完全开放空间或者对象的场景，它们需要应用透视缩短。在大多数情况下，3D图形都是使用透视投影。实际上使用正射投影观察一个3D对象会看起来怪怪的。

#### 透视矩阵
一旦我们的顶点处于视见空间，我们需要将它们变换到修剪空间，我们通过应用透视投影矩阵或者正射投影矩阵(或者其他投影矩阵)来达成。通常使用的透视矩阵是一个*平截头体矩阵(frustum matrix)*。平截头体矩阵是一个透视投影矩阵，它的修剪空间是一个矩形平截头体(截掉头部的矩形金字塔)。它的参数有远、近平面的距离以及世界空间内左、右、上、下修剪平面的坐标。一个平截头体矩阵看起来像这样：

![cookie4.13](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/cookie4.13.png)

`vmath`库中的`vmath::frustum`有这个功能：

    static inline mat4 frustum(float left,
                               float right,
                               float bottom,
                               float top,
                               float n,
                               float f) { ... }
                               
另一个常用的构造透视矩阵的方法是直接用一个角度来指定可视区域，一个长宽比(通常由窗体的宽除以高得出)，以及视见空间内远、近平面。`vmath`中这个功能函数为`vmath::perspective`：

    static inline mat4 perspective(float fovy /* in degrees*/,
                                   float aspect,
                                   float n,
                                   float f) { ... }

#### 正射矩阵
如果我们希望在场景中使用正射投影，那我们构造一个正射投影矩阵。正射投影矩阵就是一个简单的缩放矩阵，将视见空间的坐标线性映射到修剪空间坐标。构造正射投影矩阵的参数有视见空间内左、右、上、下场景边界坐标，以及远、近平面位置，这个矩阵看起来像这样：

![cookie4.14](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/cookie4.14.png)

在`vmath`中有一个函数为我们做这件事`vmath::ortho`：

    static inline mat4 ortho(float left,
                             float right,
                             float bottom,
                             float top,
                             float near,
                             float far) { ... }

## 插值，直线，曲线和样条
寻找一系列已知点之间的值的过程叫做*插值(Interpolation)*。考虑穿过点A和B的直线的公式：

\[P = A + t\vec{D}]

其中P是直线上的任意点，$\vec{D}$是从A到B的向量：

\[\vec{D} = (B - A)\]

因此我们可以得出如下等式：

\[P = A + t(B - A)\]

或者

\[P = (1-t)A + tB\]

显而易见的是当t为0时，P和A一样，当t为1时，P等于A+B-A，也就是B。这个直线如图示4.13。

图示4.13 找到一条直线上的点：

![figure4.13](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure4.13.png)

如果t处于0.0和1.0之间，那P终究会处在A和B之间的某处。若t的值超出这个范围，那将使得P落在直线两端之外。我们看以观察到平滑地改变t，会将P点从A移到B或者相反移动。这被称作*线性插值(linear interpolation)*。A和B(以及由此得到的P)可以是任意维度的值。比如它们可以是标量；图表上的二维值点；3D空间的坐标值顶点、颜色等等；甚至更高维度的量，比如矩阵、数组甚至一幅图像。在很多情况下线性插值没什么用处(比如在两个矩阵间做线性插值通常不会得出有意义的结果)，但角度、位置以及其他坐标值一般能很好地使用插值计算。

线性插值在图形是如此常用的一个操作，以至于GLSL包含有一个内置函数专门做这件事，`mix`：

    vec4 mix(vec4 A, vec4 B, float t);
    
`mix`函数有多个版本，A和B接收不同维度的向量或者标量，t接收标量或者相应向量。

### 曲线
如果我们想做的是把任何东西都沿着两个点的一条直线移动，那前述就够了。然而在真实世界中，对象以平滑的曲线移动并且平滑地加速、减速。一个曲线可用三个或更多*控制点(control points)*所表示。对大多数曲线来说，有超过三个控制点，其中两个是端点，其他的点定义了曲线的形状。考虑图示4.14展示的简单曲线：

图示4.14 一个简单的贝塞尔曲线：

![figure4.14](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure4.14.png)

图示4.14的曲线有三个控制点：A、B、C。A和C是曲线的端点，B定义了曲线的形状。如果我们将A和B用一根直线连接起来，将B和C用另一根直线连接起来，然后我们在两条直线上用简单的线性插值寻找一对新的点：D和E。现在，给定这两个点，我们同样可以用一根直线将它们连接起来，然后沿着它做插值计算寻找一个新的点：P。随着我们改变插值计算参数t，点P将会从A到D以一个平滑的曲线路径移动。以数学地方式来表达就是：

\[D = A + t(B - A)\]
\[E = B + t(C - B)\]
\[P = D + t(E - D)\]

将D和E进行代换并做一些小的计算，我们得到如下式子：

\[P = A + t(B - A) + t((B + (t(C - B))) - (A + t(B - A))))\]
\[P = A + t(B - A) + tB + t^2(C - B) - tA - t^2(B - A)\]
\[P = A + t(B - A + B - A) + t^2(C - B - B + A)\]
\[P = A + 2t(B - A) + t^2(C - 2B + A)\]

可以看出这是关于t的二次方程。它所描述的曲线即*二次贝塞尔曲线(quadratic Bezier curve)*。实际上我们在GLSL中用`mix`函数可以轻易实现它，我们所需要做的就是将之前的两个插值计算结果再次做线性插值。

	vec4 quadratic_bezier(vec4 A, vec4 B, vec4 C, float t)
    {
        vec4 D = mix(A, B, t);    // D = A + t(B - A)
        vec4 E = mix(B, C, t);    // E = B + t(C - B)

        vec4 P = mix(D, E, t);   // P = D + t(E - D)

        return P;
    }
	
若如图示4.15中添加第四个控制点后，我们将方程提高一个阶层并得到一个*三次贝塞尔曲线(cubic Bezier curve)*。

图示4.15 一个三次贝塞尔曲线：

![figure4.15](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure4.15.png)

现在我们有了四个控制点：A、B、C、D。构造曲线的过程与二次贝塞尔曲线类似。我们从A到B连一条直线，从B到C连一条直线，从C到D连一条直线。在这三条直线上做插值计算得到三个新的点：E、F、G。使用这三个点，我们从E到F连一条直线，从F到G连一条直线，然后对这两条直线做插值计算得出点H和I，然后在直线HI上我们便可插值计算找到最终的点P。于是乎我们得到：

\[E = A + t(B - A)\]
\[F = B + t(C - B)\]
\[G = C + t(D - C)\]
\[H = E + t(F - E)\]
\[I = F + t(G - F)\]
\[P = H + t(I - H)\]

如果你觉得这些等式开起来很眼熟，那你的感觉是对的：点E、F和G构成了一个二次贝塞尔曲线，我们正是用它来插值计算最终点P。如果我们将E、F、G代入到H和I的等式中，然后将H和I代入到P的等式中，经过一些展开和计算之后我们会得到一个包含$t^3$的三次方程--因此得名*三次贝塞尔曲线(cubic Bezier curve)*。我们可以再次使用GLSL中的线性插值函数`mix`来轻易地实现这个等式：

    vec4 cubic_bezier(vec4 A, vec4 B, vec4 C, vec4 D, float t)
    {
        vec4 E = mix(A, B, t);  // E = A + t(B - A)
        vec4 F = mix(B, C, t);  // F = B + t(C - B)
        vec4 G = mix(C, D, t);  // G = C + t(D - C)

        vec4 H = mix(E, F, t);  // H = E + t(F - E)
        vec4 I = mix(F, G, t);  // I = F + t(G - F)
        
        vec4 P = mix(H, I, t);  // P = H + t(I - H)
        
        return P;
    }
    
三次贝塞尔曲线的等式看起来就像包含了二次曲线的等式，所以代码也会像这样来实现。事实上我们可以基于其他曲线来堆叠新的曲线，使用其他曲线的代码来构建下一个。

    vec4 cubic_bezier(vec4 A, vec4 B, vec4 C, vec4 D, float t)
    {
        vec4 E = mix(A, B, t);  // E = A + t(B - A)
        vec4 F = mix(B, C, t);  // F = B + t(C - B)
        vec4 G = mix(C, D, t);  // G = C + t(D - C)
        
        return quadratic_bezier(E, F, G, t);
    }
    
现在我们明白了这一模式，我们可以进一步拓展它用它来生成更高阶的曲线。比如五次贝塞尔曲线(拥有五个控制点)可以如下实现：

    vec4 quintic_bezier(vec4 A, vec4 B, vec4 C, vec4 D, vec4 E, float t)
    {
        vec4 F = mix(A, B, t);  // F = A + t(B - A)
        vec4 G = mix(B, C, t);  // G = B + t(C - B)
        vec4 H = mix(C, D, t);  // H = C + t(D - C)
        vec4 I = mix(D, E, t);  // I = D + t(E - D)
        
        return cubic_bezier(F, G, H, I, t);
    }
    
这种堆叠理论上可以无限应用，即可以计算任意数目的控制点。然而在实际中，通常不会使用大于四个控制点的曲线，相反我们会使用*线条(splines)*。

### 样条
一个样条实际上就是一个长的曲线，它由几个小的曲线(比如贝塞尔)组成，每个小的曲线都在局部范围内定义它的形状。其中表示各个小曲线端点的控制点至少是被各个线段共享的(就是这些端点将小曲线组成一个样条。这样的控制点被称为*焊接点 welds*，它们之间的控制点被称为*绳结 knots*)，并且通常内部控制点的一个或多个也在邻近线段间以某种方式共享或联系起来。任意数目的曲线都可以这种方式连接起来，可以形成任意长的路径。看看图示4.16中的曲线。

图示4.16 一个三次贝塞尔样条：

![figure4.16](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible7th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure4.16.png)

在图示4.16中的样条由十个控制点所定义，从A到J，分别组成三个三次贝塞尔曲线。第一个由A、B、C、D所定义；第二个与第一个共享D，然后使用了E、F、G；第三个与第二个共享G，并添加了H、I、J。这种类型的样条被称为三次贝塞尔样条，因为它由一系列三次贝塞尔曲线所构成。这也被称作*三次B-样条(cubic B-spline)*--对于读过很多图形方面资料的人来说应该是很熟悉的一个术语。

沿着样条插值计算点P，我们简单地将样条分割为三个区域，使得t的范围从0.0到3.0。在0.0到1.0之间，我们沿着第一个曲线进行插值计算，从A移动到D。在1.0到2.0之间，我们沿着第二个曲线进行插值计算，从D移动到G。在2.0到3.0之间，我们沿着最后G到J之间的曲线进行插值计算。所以t的整数部分决定了插值的曲线段，t的小数部分用以在对应的曲线段上做插值计算。当然我们可以将t恣意地缩放。

如下的代码会沿着一个有十个控制点的三次贝塞尔样条(有三段)进行一个向量的插值计算：

    vec4 cubic_bspline_10(vec4 CP[10], float t)
    {
        float f = t * 3.0;
        int i = int(floor(f));
        float s = fract(t);
        
        if (t <= 0.0)
            return CP[0];
            
        if (t >= 1.0)
            return CP[9];
            
        vec4 A = CP[i * 3];
        vec4 B = CP[i * 3 + 1];
        vec4 C = CP[i * 3 + 2];
        vec4 D = CP[i * 3 + 3];
        
        return cubic_bezier(A, B, C, D, s);
    }
    
如果我们使用样本来决定一个对象的位置和方向，我们会发现为了使动作平滑且流畅，对于控制点位置的选择必须非常小心。插值点P的值的变化速率(即P的速度)随着关于t的曲线方程而不同。如果这个函数是非连续的，那P就会突然变向，然后我们的对象就会到处乱跳。此外P的速度变换速率(它的加速度)是关于t的曲线方程的第二衍生物。如果加速度不平滑，那P就会突然加速或者减速。

有连续的第一衍生物的函数被称为C<sup>1</sup>连续性，同理，有连续第二衍生物的曲线被称为C<sup>2</sup>连续性。贝塞尔曲线段同时满足C<sup>1</sup>和C<sup>2</sup>连续性，但为了确保样条焊接点处也是连续的，我们需要确保每个段以前面段的位置、移动方向、变化速率为开始。在某个特定位置的行进速率即为速度。所以与其为样条设置控制点，不如为每个焊接点设置速度。如果在曲线段焊接点的任一侧进行计算都使用同样的速度，那我就会得到一个样本函数同时满足C<sup>1</sup>和C<sup>2</sup>连续性。

如果我们再看看图示4.16就会发现--没有扭结并且曲线漂亮地、平滑地穿过焊接点(点D和G)。现在看看焊接点每一侧的控制点。比如点D旁边的点C和E。C和E构成一条直线并且D处于中间。我们可以将线段DE在D处的速度称为$\vec{D}$。给定点D(焊接点)的位置以及D处的曲线速度$\vec{D}$，然后C和E可被如下计算：

\[C = D - \vec{D}\]
\[E = D + \vec{D}\]

同理，如果A处的速度表示为$\vec{A}$，B可被如下计算：

\[B = A + \vec{A}\]

所以，给出一个三次B-样条焊接点的位置和速度，我们可以省掉所有其他的控制点，然后在我们对每个控制点求值时动态计算。通过这种方式(用一系列焊接点位置和速度)表示的三次B-样条被称为三次诶尔米特样条或者有时简称为cspline。在构造平滑、自然的动画时cspline是一个炒鸡有用的工具。

## 总结
在本章我们学习到了用OpenGL构造3D场景的一些重要的数学概念。就算我们还不能在脑海中玩转矩阵，但现在我们至少知道矩阵是什么以及它们如何用以各种变换。我们还学到如何构造并操作表示视见者和视口属性的矩阵。我们现在应该明白如何在场景中放置我们的对象并判断它们如何在显示屏幕上被看到。本章还介绍了引用框架这一强大的概念，我们还看到操作框架以及将它们转换为变换是如此的简单。

最后，我们介绍了`vmath`库的使用。这个库是完全由可移植的C++代码编写的，并提供了可以和OpenGL一起使用的各种数学和辅助的便利工具包。

出乎意料的是本章我们没有引入任何新的OpenGL函数调用。是的，本章是数学章节，当然如果你认为数学就是公式和计算的话你可能并没有注意到这一点。向量和矩阵，以及它们的应用，对于能使用OpenGL渲染3D对象和世界绝对是很重要的。

值得注意的是OpenGL并不对我们强加任何数学约定，它自己也并不提供任何数学功能。如果你使用其他的3D数学库，或者自己来解决，你会发现操作几何图形和3D世界与本章的套路如出一辙。现在，继续前进并开始做点啥吧！