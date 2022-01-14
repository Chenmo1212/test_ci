
# 基于Babylon的三维模型旋转指南

## 前言

在做基于Babylon的三维橱柜项目时，需要用到旋转的功能，探究了一下，发现三维旋转有着多种方法，且多种方法有着自己的特性，因此在此记录一下。



## 正文

### 旋转理论

三维旋转常见的有三种表示方式：`矩阵旋转`，[欧拉旋转](http://zh.wikipedia.org/wiki/欧拉角)与`四元数`。

- 矩阵旋转使用了一个4*4大小的矩阵来表示绕任意轴旋转的变换矩阵;
- 欧拉旋转则是按照一定的坐标轴顺序（例如先x、再y、最后z）、每个轴旋转一定角度来变换坐标或向量，它实际上是一系列坐标轴旋转的组合;
- 四元数则是使用四维的四元数就可以表示任意方向.

#### 1.**矩阵旋转**

简单来说，一个点在二维或三维里旋转一定角度后新的坐标，是可以通过一个矩阵计算求解的，具体的证明过程可参见：[旋转矩阵（Rotation Matrix）的推导及其应用](https://www.cnblogs.com/meteoric_cry/p/7987548.html)

##### 优缺点

- 优点：
  - 旋转轴可以是任意向量.
- 缺点：
  - 旋转其实只需要知道一个**向量** + 一个**角度**，一共4个值的信息，但矩阵法却使用了16个元素；
  - 而且在做乘法操作时也会增加计算量，造成了空间和时间上的一些浪费.

#### 2.**欧拉旋转**

欧拉角是用来确定定点转动刚体位置的3个一组独立角参量，由[章动](https://baike.baidu.com/item/章动/695762)角θ(or *β* ) 、[旋进](https://baike.baidu.com/item/旋进/8795453)角（即进动角）ψ(or *γ* )和自转角φ(or *α* )组成。任何一个旋转可以表示为依次绕着三个旋转轴旋三个角度的组合。这三个角度称为`欧拉角`。
三个轴可以指固定的世界坐标系轴，也可以指被旋转的物体坐标系的轴。三个旋转轴次序不同，会导致结果不同。

用一句话说，欧拉角就是物体绕坐标系三个坐标轴（x,y,z轴）的旋转角度。

![img](https://blog-img-1300024309.cos.ap-nanjing.myqcloud.com/img/Eulerangles.png)

由于欧拉角并没有一个统一的标准，这样就会导致不同人描述欧拉角的**旋转轴**和**旋转**的顺序可能是不同的，因此使用其他人的欧拉角时，应当搞清楚对方的约定。具体的`欧拉角`还挺复杂的，更多详细内容可参见：[欧拉角](https://blog.csdn.net/csxiaoshui/article/details/65437633)

由于Babylon最常用的方法（`Rotation`）是基于`欧拉角`来决定模型的旋转的，因此简单介绍一下`欧拉角`。

- 文档：[Mesh Rotation](https://doc.babylonjs.com/divingDeeper/mesh/transforms/center_origin/rotation)

在Babylon中，需要记住：

- 旋转角度是左手系；
- 旋转的中心默认是模型的本地中心点；
- 参照系可以是本地坐标系，也可以是世界坐标系，也可以是指定的旋转轴；
- Babylon的**旋转**的顺序是YXZ.

##### 优缺点

- 优点：

  - 很容易理解，形象直观；

  - 表示更方便，只需要3个值（分别对应x、y、z轴的旋转角度）.

    > 但按我的理解，它还是转换到了3个3*3的矩阵做变换，效率不如四元数

- 缺点：

  - 之前提到过这种方法是要按照一个固定的坐标轴的顺序旋转的，因此不同的顺序会造成不同的结果；

  - 会造成万向节锁（Gimbal Lock）的现象;

    > 这种现象的发生就是由于上述固定坐标轴旋转顺序造成的。理论上，欧拉旋转可以靠这种顺序让一个物体指到任何一个想要的方向，但如果在旋转中不幸让某些坐标轴重合了就会发生万向节锁，这时就会丢失一个方向上的旋转能力，也就是说在这种状态下我们无论怎么旋转（当然还是要原先的顺序）都不可能得到某些想要的旋转效果，除非我们打破原先的旋转顺序或者同时旋转3个坐标轴。
    >
    > 这里有个视频可以直观的理解下：[欧拉旋转](https://v.youku.com/v_show/id_XNzkyOTIyMTI=.html)

  - 由于万向节锁的存在，欧拉旋转无法实现球面平滑插值.

#### 3.**四元数旋转**

四元数是一个四维向量（x，y，z，w），要成为一个旋转四元数，它必须是一个单位向量，即x²+y²+z²+w²=1。举个例子，如果使用一个四元数q=((x,y,z)sinθ/2, cosθ/2) 来执行一个旋转，即把空间的一个点P绕着单位向量轴u = (x, y, z)表示的旋转轴旋转θ角度。具体的证明方式可参见：[四元数](https://www.cnblogs.com/yiyezhai/p/3176725.html)

##### 优缺点

- 优点：
  - 可以避免万向节锁现象；
  - 只需要一个4维的四元数就可以执行绕任意过原点的向量的旋转，方便快捷，在某些实现下比旋转矩阵效率更高；
  - 可以提供平滑插值.
- 缺点：
  - 比欧拉旋转稍微复杂了一点点，因为多了一个维度；
  - 理解更困难，不直观.

### 实例

#### 模型本地旋转

##### 1.Rotation

- 文档：[Babylon官网——Mesh Rotation](https://doc.babylonjs.com/divingDeeper/mesh/transforms/center_origin/rotation)

**更改网格方向的最直接方法是“旋转（rotation）”特性。**

```js
// 应用旋转
mesh.rotation = new BABYLON.Vector3(alpha, beta, gamma);

// or

// 应用旋转
mesh.rotation.x  =  alpha; //rotation around x axis
mesh.rotation.y  =  beta;  //rotation around y axis
mesh.rotation.z  =  gamma; //rotation around z axis
```

Demo: [Rotation](https://playground.babylonjs.com/#HPKH80#34)



##### 2.AddRotation

- 文档：[Babylon官网——addRotation](https://doc.babylonjs.com/divingDeeper/mesh/transforms/center_origin/add_rotations)

若想实现首先围绕y轴，然后围绕x轴，最后围绕z轴，在模型本地坐标系中应用旋转，然后围绕自定义轴序列旋转。使用上面的rotation属性显然有些余力不足。这里其实涉及隐式或显式旋转四元数，但是可以使用`addRotation`方法——使用其中两个参数为0的addRotation函数。

```js
// 应用旋转
mesh.rotation.addRotation(Math.PI / 2, 0, 0);
mesh.rotation.addRotation(0, 0, Math.PI / 3);
mesh.rotation.addRotation(0, Math.PI / 8, 0);

// or

// 应用旋转
mesh.rotation.addRotation(Math.PI / 2, 0, 0).addRotation(0, 0, Math.PI / 3).addRotation(0, Math.PI / 8, 0)
```

上述代码是将模型先按照x轴旋转 π/2， 然后按照 z轴旋转π/3，

该方法的本质是将欧拉角转换成了四元数，然后再转回来。

Demo: [AddRotation](https://playground.babylonjs.com/#HPKH80#34)



##### 3.Rotate

- 文档：[Babylon官网——Rotate](https://doc.babylonjs.com/divingDeeper/mesh/transforms/center_origin/add_rotations)

想象一个圆盘的中心有一个轴,圆盘能够绕轴旋转。下图显示了光盘围绕轴的几个不同旋转点。

![disc rotate](https://doc.babylonjs.com/_next/image?url=%2Fimg%2Fhow_to%2FMesh%2Fquat1.jpg&w=3840&q=75)

当轴也可以旋转时：

![disc rotate and axle tilt](https://doc.babylonjs.com/_next/image?url=%2Fimg%2Fhow_to%2FMesh%2Fquat2.jpg&w=3840&q=75)

可以发现，当为轴指定旋转方向，和为圆盘指定旋转方向，是两种围绕不同坐标系的旋转。圆盘是本地坐标系，轴是世界坐标系。

```js
// 应用旋转
mesh.rotate(new BABYLON.Vector3(1, 0 -1), Math.PI / 3, BABYLON.Space.WORLD);
mesh.rotate(new BABYLON.Vector3(1, 0 -1), Math.PI / 3, BABYLON.Space.LOCAL);
```

`rotate`方法也是可以累加的，如下：

```js
// 应用旋转
mesh.rotate(new BABYLON.Vector3(2, -3, 7), Math.PI / 3, BABYLON.Space.LOCAL);  
mesh.rotate(BABYLON.Axis.Y, -Math.PI / 2, BABYLON.Space.WORLD);
mesh.rotate(new BABYLON.Vector3(5.6, 7.8, - 3.4), 1.5 * Math.PI, BABYLON.Space.WORLD);
mesh.rotate(BABYLON.Axis.Z, -Math.PI, BABYLON.Space.LOCAL);
```

将从模型的当前方向开始，令其按照给定本地坐标轴(2, -3, 7)旋转π/3，然后关于世界y轴旋转 -π/2，关于给定世界坐标轴(5.6, 7.8, - 3.4)旋转1.5π，最后关于局部z轴旋转-π。

Demo: [Rotate](https://playground.babylonjs.com/#LLNE9E#72)



##### 4. Rotation Quaternions （旋转四元数）

- 文档：[Babylon官网——Rotation Quaternions ](https://doc.babylonjs.com/divingDeeper/mesh/transforms/center_origin/rotation_quaternions)

我们已经使用了`rotate`来设置网格的旋转四元数。

```js
mesh.rotate(new BABYLON.Vector3(1, 0 -1), Math.PI / 3, BABYLON.Space.WORLD);
console.log(mesh.rotationQuaternion.x);
console.log(mesh.rotationQuaternion.y);
console.log(mesh.rotationQuaternion.z);
console.log(mesh.rotationQuaternion.w);
```

与`rotation`属性相同，“旋转四元数”属性设置以本地坐标系原点为旋转中心的模型方向。

除了旋转，还可以使用直接获取旋转四元数，通过`rotationQuaternion`属性设置模型的旋转，例如：

```js
// 设置旋转
let quaternion = new BABYLON.Quaternion.RotationAxis(new BABYLON.Vector3(1, 0 -1), Math.PI / 3);

// 应用旋转
mesh.rotationQuaternion = quaternion
```

旋转四元数的参数为：**轴方向，角度**。轴方向是基于世界坐标系的。

旋转四元数可以转换为欧拉角：

```js
// 设置旋转
let quaternion = new BABYLON.Quaternion.RotationAxis(new BABYLON.Vector3(1, 0 -1), Math.PI / 3);

// 转换四元数为欧拉角
const euler = quaternion.toEulerAngles();
// euler.x, euler.y, euler.z

// 应用旋转
mesh.rotation = euler
```

**注意：不能在同一模型上使用旋转四元数后跟旋转。应用旋转四元数后，任何后续使用旋转都将产生错误的方向，除非首先将旋转四元数设置为null。请注意，这在导入模型时通常适用，因为这些模型中有许多已经具有旋转四元数集。**



#### 模型外部旋转

- 文档：[Rotating Around Axis ](https://doc.babylonjs.com/divingDeeper/mesh/transforms/parent_pivot/pivot)

模型的旋转通常由两部分构成：`轴`和`旋转中心`，且`轴`穿过`旋转中心`。`轴`由**三维方向向量**定义，`旋转中心`由**位置向量**定义。在`Babylon.js`创建模型时，旋转中心默认为模型的本地坐标系的中心原点，即模型的位置。使用旋转通过Euler角度alpha、beta、gamma指定轴，使用[rotationQuaternion](https://doc.babylonjs.com/divingDeeper/mesh/transforms) 和 [rotate](https://doc.babylonjs.com/divingDeeper/mesh/transforms#rotate) 明确指定轴（方向与角度）。

但除此之外，也可以更改旋转中心，主要也三种方法：

- 变换节点 [TransformNode](https://doc.babylonjs.com/divingDeeper/mesh/transforms/parent_pivot/transform_node)
- 添加父级关系（parent）
- [使用轴作为旋转中心](https://doc.babylonjs.com/divingDeeper/mesh/transforms/parent_pivot/pivots)

##### 1. TransformNode

`TransformNode`是未渲染的对象，但可以用作旋转中心（实际上是任何变换的中心）。这可以减少内存使用并提高渲染速度。TransformNode本质是通过将其作为pilot的子对象并旋转来用作轴心点。

```js
// 旋转中心
var CoR_At = new BABYLON.Vector3(1, 3, 2);
// 旋转轴
var axis = new BABYLON.Vector3(1, 3, 2);

// 1. 设置父轴位置
var pivot = new BABYLON.TransformNode("root");
pivot.position = CoR_At;

// 2. 模型绑定父轴
pilot.parent = pivot;
pilot.position = pilotStart;

// 3. 旋转父轴
pivot.rotate(axis, Math.PI / 3, BABYLON.Space.WORLD);
```

模型会根据旋转中心的位置和旋转轴来旋转，且当旋转中心和旋转轴变化的时候，模型也会跟着变化：

Demo：

- [外部旋转轴旋转](https://playground.babylonjs.com/#1JLGFP#36)
- [旋转中心变化式旋转](https://playground.babylonjs.com/#C12LH3#3)
- [旋转中心与旋转轴变化式旋转](https://playground.babylonjs.com/#C12LH3#4)



##### 2. Parent

若模型存在父级，当父级旋转时，模型会随父级旋转而旋转：

```js
// 旋转中心
var CoR_At = new BABYLON.Vector3(1, 3, 2);
// 旋转轴
var axis = new BABYLON.Vector3(1, 3, 2);

// 设置父级位置
sphere.position = CoR_At;

// 绑定父级与设置子级位置
pilot.parent = sphere;
pilot.position = pilotStart;

// 旋转父级
sphere.rotate(axis, Math.PI / 3, BABYLON.Space.WORLD);
```

Demo: [Parent](https://playground.babylonjs.com/#1JLGFP#31)



##### 3. Pivot

有时希望考虑通过更改父级位置而不是模型位置来实现父级的定位。此时可以通过矩阵设置相对于父级的模型位置、旋转。

```js
// 旋转中心
var CoR_At = new BABYLON.Vector3(1, 3, 2);
// 旋转轴
var axis = new BABYLON.Vector3(1, 3, 2);

// 设置父级位置
sphere.position = CoR_At;

// 绑定父级与设置子级位置
pilot.parent = sphere;
pilot.setPivotMatrix(BABYLON.Matrix.Translation(pilotTranslate.x, pilotTranslate.y, pilotTranslate.z));

// 旋转父级
pilot.rotate(axis, angle, BABYLON.Space.WORLD);
```

Demo: [Pivot](https://playground.babylonjs.com/#1JLGFP#77)



## 参考文献

- [【Unity技巧】四元数（Quaternion）和旋转_candycat-CSDN博客_quaternion.euler](https://blog.csdn.net/candycat1992/article/details/41254799)
- [旋转变换（二）欧拉角_Frank的专栏-CSDN博客_欧拉旋转](https://blog.csdn.net/csxiaoshui/article/details/65437633)
- [游戏动画中欧拉角与万向锁的理解_huazai434的专栏-CSDN博客_欧拉角和万向锁](https://blog.csdn.net/huazai434/article/details/6458257)
- [三维旋转：旋转矩阵，欧拉角，四元数](https://www.cnblogs.com/yiyezhai/p/3176725.html)
- [旋转矩阵、欧拉角、四元数理论及其转换关系](https://blog.csdn.net/lql0716/article/details/72597719)
- [旋转矩阵（Rotation Matrix）的推导及其应用](https://www.cnblogs.com/meteoric_cry/p/7987548.html)









