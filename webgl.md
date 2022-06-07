## index
顶点着色器：计算顶点的位置


片段着色器：计算出每个像素的颜色值

语言：GLSL

着色器获取数据的4种方法：
1. 属性（Attributes）和缓冲
2. 全局变量（Uniforms）
3. 纹理（Textures）

4. 可变量（Varyings）
可变量是一种顶点着色器给片断着色器传值的方式，依照渲染的图元是点， 线还是三角形，顶点着色器中设置的可变量会在片断着色器运行中获取不同的插值



WebGL在GPU上的工作基本上分为两部分，第一部分是将顶点（或数据流）转换到裁剪空间坐标， 第二部分是基于第一部分的结果绘制像素点


数据被塞入 gl.ARRAY_BUFFER 中，并绑定到相应的 buffer  
buffer 中的数据被紧接着（代码顺序上）的 atrrbute 消费  


缓冲操作是在GPU上获取顶点和其他顶点数据的一种方式  
gl.createBuffer： 创建一个缓冲  
gl.bindBuffer：设置缓冲为当前使用缓冲  
// 这个命令是将缓冲绑定到 ARRAY_BUFFER 绑定点，它是WebGL内部的一个全局变量  
gl.bindBuffer(gl.ARRAY_BUFFER, someBuffer);  
gl.butterDate: 将数据拷贝到缓冲  


这个命令是告诉WebGL我们想从缓冲中提供数据  
gl.enableVertexAttribArray(location);  


### 处理变量  
1. 找到变量的内存位置
var positionAttributeLocation = gl.getAttribLocation(program, 'a_position');
2. 为变量填充数据
```
// 创建缓存，并指定使用缓存类型
var positionBuffer = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);

var positions = [
    0, 0,
    0, 0.5,
    0.7, 0,
  ];
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW);
```
3. 说明如何使用数据
```
// 启用变量
gl.enableVertexAttribArray(positionAttributeLocation);

 // 然后，告诉属性怎么从positionBuffer中读取数据 (ARRAY_BUFFER)
 var size = 2;           // 每次迭代运行提取两个单位数据
 var type = gl.FLOAT;    // 每个单位的数据类型是32位浮点型
 var normalize = false;  // 不需要归一化数据
 var stride = 0;         // 0 = 移动单位数量 * 每个单位占用内存（sizeof(type)）, 每次迭代运行运动多少内存到下一个数据开始点
 var offset = 0;         // 从缓冲起始位置开始读取
 gl.vertexAttribPointer(positionAttributeLocation, size, type, normalize, stride, offset);
```
4. 变量 a_position 设置完成




### 全局变量
gl.uniformXXX (xxx 表示数据维度和类型)
  var translation = [0, 0];
  var color = [Math.random(), Math.random(), Math.random(), 1];

	gl.uniform2f(resolutionLocation, gl.canvas.width, gl.canvas.height);


  gl.uniform4fv(colorLocation, color);


  gl.uniform2fv(translationLocation, translation);

#### 旋转
坐标 * 单位圆上的点的坐标
单位圆上的点的坐标，用角度表示：x = sin(a)，y = cos(a)


#### 矩形
由两个三角组成，所以数据
	-50, 50,
  50, 50,
  -50, -50,
  -50, -50,
  50, -50,
  50, 50
是六个坐标数据（两个三角形有 6个角）


#### 图元
gl 渲染的基础是图元，通过图元的拼接组成图形。因此 position 的数据是用了渲染各个图元的点。


### 为什么三维坐标需要四个维度？
三维坐标 x, y, z 用 （x, y, z, w）表示，其中 w 被称作齐次坐标。  
在透示投影时需要使用。   
坐标系的 scale 和 rotate 可以使用 mat3 的变换来实现，但坐标的平移不可以。因为前者的根坐标（0，0，0）是不变的，而后者需要改变根坐标的位置。   

使用 mat3 也可以计算平移：  

但为了使用矩阵特性，即矩阵乘法有结合性：  

可以先计算 ，然后施于大量点或矢量. 

所以使用四维（x, y, z, w） 来表示三维坐标，其中 w 可以表示平移量？  

### 对于坐标的变换可以理解为如下矩阵的变换. 
WVP = World * View * Projection. 
然后 World = Scale * Rotation * Translation. 
所以 WVP = Scale * Rotation * Translation * View * Projection，其中每一个都是 4X4. 


即 gl_Position = wvp * a_position;

- world：世界矩阵
- view：视图矩阵
- projection：投影矩阵



## 视图矩阵
用于确定相机角度和位置的矩阵  

Matrix4.setLookAt(eyeX, eyeY, eyeZ, atX, atY, atZ, upX, upY, upZ). 
var viewMatrix = new Matrix4();  
viewMatrix.setLookAt(3,3,7,0,0,0,0,1,0);  

### modelView
即对相机的位置做一些变换的矩阵. 
In OpenGL 1.1, a transform that combines the modeling transform with the viewing transform. That is, it is the composition of the transformation from object coordinates to world coordinates and the transformation from world coordinates to eye coordinates. Because of the equivalence between modeling and viewing transformations, world coordinates are not really meaningful for OpenGL, and only the combined transformation is tracked.


### 投影矩阵

所有的透视矩阵的输入都是以下几个参数： 
● fov：垂直视角，即可是空间和底面间的夹角；
● aspect：近裁切面的宽高比；
● near: 近裁切面位置
● far：远裁切面位置
 //设置透视投影矩阵
  var projMatrix = new Matrix4();
  projMatrix.setPerspective(30,canvas.width/canvas.height,1,100);




## 方向光源
将方向光的方向和面的朝向点乘就可以得到两个方向的余弦值。  
将三维物体的朝向和光的方向点乘， 结果为 1 则物体朝向和光照方向相同，为 -1 则物体朝向和光照方向相反. 

#### 三维物体的朝向
面的法向量  

1. 在顶点着色器中声明法向量：varying vec3 a_normal;
2. 在片段着色器中
  a. 用 normalize 将其单位化： vec3 normal = normalize(v_normal);
  b. 将法向量与光的方向点乘：float light = dot(normal, u_reverseLightDirection);
  c. 将颜色部分（不包括 alpha）和 光照相乘：gl_FragColor.rgb *= light;
3. 在 glsl 中为变量赋值

### 如何求法向量
法向量初始应该是一个固定的值。在物体旋转的过程中需要重置法向量，方法是法向量和世界矩阵相乘。
如果世界矩阵被缩放，使用世界矩阵的转置矩阵与法向量相乘。



归一化：normalize  ===》标准立方体


纹理值代替漫反射系数


曲面细分着色器
贴图置换
几何着色器
距离函数
水平集
隐式几何/显式几何

.obj
f => face  顶点/纹理/法线  (index)



实时渲染管线


绘制多个物体
1. 给着色器变量设置不同的值, 多次调用 draw 方法
2. 有多个着色器, 多次调用 useProgram 和 draw 方法


如何绘制物体
3D 建模: 静态, 无法动态修改物体
shader 程序: 复杂
软件: blender






## glsl 语法
在 WebGL 中有两个基本的绘制函数。 gl.drawArrays 和 gl.drawElements
drawElements
数组数据渲染图元
void gl.drawElements(mode, count, type, offset);
配合 gl.ELEMENT_ARRAY_BUFFER 使用，这个 buffer 中存储复用的顶点的索引。
Uint16Array：无符号16位整型

positions 里的位置数据是 x,y,z 平铺. 即三个位置的数据代表一个空间中的点
indices 里的数据是一个数值指定空间中的点在 positions 数据里的排序
这一个数值是如何映射到空间中的点坐标(x,y,z) 三个数值在 position 中的位置?

count 指定要渲染的元素数量：不是基础元素(mode)的数量而是需要画的顶点的数量

如何画的？
如何取 gl_Position 和 gl_FragColor，取多少次？
通过索引来获取 postionBuffer 中点的坐标，并赋值给 gl_Position
colorBuffer 中的颜色数据，每4个一组代表一个顶点的颜色。colorBuffer 中第一个颜色（前4个数据），是 positionBuffer 中第一个坐标的颜色。

drawArrays
void gl.drawArrays(mode, first, count);
用于从向量数组中绘制图元


可以指定渲染的图元类型
● gl.POINTS: 画单独的点
● gl.LINE_STRIP: 画一条直线到下一个顶点
● gl.LINE_LOOP: 绘制一条直线到下一个顶点，并将最后一个顶点返回到第一个顶点
● gl.LINES: 在一对顶点之间画一条线.
● gl.TRIANGLE_STRIP: 画一组相连的三角形带
● gl.TRIANGLE_FAN: 画一组连接的三角形。以第一个点做原点，每个顶点都连着它的前一个点和第一个顶点（类似风扇）
● gl.TRIANGLES:  为一组三个顶点绘制一个三角形.


bufferData
void gl.bufferData(target, ArrayBufferView srcData, usage, srcOffset, length);
以下两者的区别？
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW);
 gl.bufferData(gl.ELEMENT_ARRAY_BUFFER,
      new Uint16Array(indices), gl.STATIC_DRAW);
ARRAY_BUFFER： 包含顶点属性的Buffer，如顶点坐标，纹理坐标数据或顶点颜色数据
ELEMENT_ARRAY_BUFFER：用于元素索引的Buffer


vertexAttribPointer
void gl.vertexAttribPointer(index, size, type, normalized, stride, offset);
告诉显卡从哪个(index 指定)绑定的缓冲区（bindBuffer()指定的缓冲区）中读取顶点数据
● index： 指定要修改的顶点属性的索引(对象地址)
● size：指定每个顶点属性的组成数量，必须是1，2，3或4 （？顶点的坐标由几个数表示【几维】）

enableVertexAttribArray
void gl.enableVertexAttribArray(index)
打开属性数组列表中指定索引处的通用顶点属性数组, 激活属性以便使用，不被激活的属性是不会被使用的


对 attribute(GPU 变量) 的使用
1. 绑定 buffer 数据到当前的缓冲中：gl.bindBuffer
2. 告诉 gl ，属性(attribute) 使用当前缓冲中的数据，和使用数据的方式：gl.vertexAttribPointer  （属性attribute和数据关联）
3. 启用属性(attribute)索引：gl.enableVertexAttribArray


数据填充(生成数据buffer)
1. 创建一个空 buffer：gl.createBuffer()
2. 绑定 buffer 和缓冲：gl.bindBuffer
3. 将数据填入缓冲：gl.bufferData
4. (生成了一个带数据的 buffer)


uniformMatrix4fv
为全局变量指定矩阵值

精度 precision
和 shader 渲染有关
● lowp
● mediump
● highp





attribute 和 uniform 的区别
如果一个变量表示顶点相关的数据并且需要从javascript代码中获取顶点数据， 需要使用attribute关键字声明该变量.  比如上面代码attribute关键字声明的顶点位置变量apos、顶点颜色变量a_color, 顶点法向量变量a_normal。
如果一个变量是非顶点相关的数据并且,  需要javascript传递该变量相关的数据，需要使用uniform关键字声明该变量.   比如上面代码通过uniform关键字声明的光源位置变量u_lightPosition、光源颜色变量u_lightColor。
uniform变量就像是C语言里面的常量（const ），它不能被shader程序修改。（shader只能用，不能改）attribute变量是只能在vertex shader中使用的变量。 varying变量是vertex和fragment shader之间做数据传递用的




gl.useProgram
指定要使用的着色程序 (使用哪个 shader 程序和数据的绑定是解偶的)










## 变换
平移 translate、旋转 rotate、缩放 scale。都是对 基 的操作。

基
i^、j^、k^

矩阵
1.0	0.0	0.0
0.0	1.0	0.0
tx	ty	1.0

其中
1.0	0.0
0.0	1.0
即 
[i^ j^]
二维向量
 i^ => (1.0, 0)
 j^ => (0, 1.0)
参考：
https://www.bilibili.com/video/BV1ys411472E  4、5 集

二维变换（transform）为什么需要 3*3 的 matrix？
因为除了表示向量 i^ , j^之外，还需要表示平移的变换。并且为了变换后的维度不变，需要加第三列。

缩放
sx	0.0	0.0
0.0	sy	0.0
0.0	0.0	1.0
旋转
c	-s	0.0
s	c	0.0
0.0	0.0	1.0


gl-matrix
create
创建单位向量
1, 0, 0, 0,
 0, 1, 0, 0,
 0, 0, 1, 0,
 0, 0, 0, 1,

平移
mat4.translate( modelview, modelview, [dx,dy,dz] );

  const modelViewMat = create();
    /**
    * 此时的 modelViewMat：
    * 1, 0, 0, 0,
    * 0, 1, 0, 0,
    * 0, 0, 1, 0,
    * 0, 0, 0, 1,
    */
  translate(modelViewMat, modelViewMat, [-0.0, 0.0, -20.0])
      /**
    * 此时的 modelViewMat：
    * 1, 0, 0, 0,
    * 0, 1, 0, 0,
    * 0, 0, 1, 0,
    * 0, 0, -20, 1,
    */
证明 translate 矩阵是：
1,     0,     0,     0,
0,     1,     0,     0,
0,     0,     1,     0,
dx,   dy,   dz,   1,

translate 方法的最后一个参数是 [dz, dy, dz]

代表对于每个坐标向量
x' = x + dx;
y' = y + dy;
z' = z + dz;

缩放
mat4.scale( modelview, modelview, [sx,sy,sz] );
scale 矩阵是：
sx,     0,     0,     0,
0,     sy,     0,     0,
0,     0,     sz,     0,
0,     0,     0,   1,

代表对于每个坐标向量
x' = sx;
y' = sy;
z' = sz;

旋转
mat4.rotate( modelview, modelview, radians, [dx,dy,dz] );
第四个参数代表：从位置 0,0,0 到位置 dx,dy,dz 的向量为旋转的轴 
低三个参数是旋转的角度
其他旋转方法：
mat4.rotateX( modelview, modelview, radians ); 
mat4.rotateY( modelview, modelview, radians ); 
mat4.rotateZ( modelview, modelview, radians );




为什么单位向量的变换可以由矩阵来表示？
已知单位向量的变换可以表示所有顶点坐标的变换，因为单位向量的变换意味着坐标系的变换
所有问题就变成了：为什么矩阵可以表示坐标是的变换？


为什么使用 matrix 来做变换
martix 并不表示空间中的物体, matrix 的含义是表示坐标系将如何变换;
matrix 的第一列代表 x 轴的单位向量 i^ 将如何变换
matrix 的第二列代表 x 轴的单位向量 j^ 将如何变换
matrix 的第三列代表 x 轴的单位向量 k^ 将如何变换

例如当 i^ 是任意不为 (1, 0, 0) 的 (x', y', z')值时, 表示 x 轴已经做了变化, 同时整个坐标系也做了这样的变化.


view mat
设置 view mat 的变化, 看到的就是 model mat 在进行这种变换的逆变换


转置

行列式





## 纹理
纹理的应用
gl_FragColor = texture Map

法线贴图
凹凸贴图
阴影贴图

纹理处理的原理
sampler 采样器的工作原理？


uv 图的样子？


模型如何制作？


一张图片纹理的原点是?
左下角


纹理坐标是怎么对应的?
纹理坐标和顶点一一对应, 即和 position 里的顺序一一对应






## 光照
方向光

光照应用于颜色（相乘）
gl_FragColor = vColor * vLighting
光照由环境光和漫反射光组成（相加）
vLighting = ambient + diffuse
环境光由光照强度和颜色决定（相乘）
ambient = ambientColor * ambientStrength 
漫反射光由光照强度、颜色、漫反射因子决定（相乘）
diffuse =  diffuseColor * diffuseStrength * diffuseFactor
漫反射因子由转置(transform)之后的法向量(点的法向量？面的法向量？)与光照方向决定（反射因子不能小于0）
diffuseFactor = max(dot(transformedNormal.xyz, diffuseDir), 0)
diffuseDir 为向量，光照的方向与向量的方向相反


为什么
物体表面接受的到光照为光照的 cosθ  即 点乘


从各个方向看到的反射光都一样

怎么获取法向量?
面的法向量可以由面上的两个向量 x乘 得到


法向量为什么需要转置?
当物体进行位置变换时, 光的位置没有变换, 当物体各个面收到的光是应该变换的. 所以这个时候应该对法向量进行变换.
vec4 transformedNormal = normalMat * vertexNormal;
normalMat 的值是世界矩阵(modelViewMat) 的逆矩阵的转置

转置的作用是
当世界矩阵进行运动, 法向量就会变化. 这时对世界矩阵的逆矩阵进行转置, 再和法向量进行相乘, 就会得到正确的法向量
invert(normalMat, modelViewMat);
transpose(normalMat, normalMat);



phong 模型
光的类型
● 环境光 ambient
● 漫反射 diffuse
● 点光源光


法向量
● 面的法向量
● 顶点的法向量









## opengl
标准化设备坐标


告诉 openGL 如何使用缓冲数据, 属性解释
● Stride(步长): 它告诉我们在连续的顶点属性组之间的间隔



Vertex Array Object ( VAO) 模式 ( webgl 中没有)
● glEnableVertexAttribArray和glDisableVertexAttribArray
● glBindVertexArray 


索引缓冲对象 (IBO, indices) 
● GL_ELEMENT_ARRAY_BUFFER 当作缓冲目标


uniform
● 它可以被着色器程序的任意着色器在任意阶段访问
● 但是更新一个uniform之前你必须先使用程序（调用glUseProgram)，因为它是在当前激活的着色器程序中设置uniform的

纹理
glTexParameter*函数对单独的一个(纹理)坐标轴设置

纹理像素: 纹理坐标是你给模型顶点设置的那个数组，OpenGL以这个顶点的纹理坐标数据去查找纹理图像上的像素

纹理过滤

glBindTexture: 把纹理赋值给片段着色器的采样器

采样器 samper 需要获得纹理数据和纹理坐标, 从这两者最后得到纹理像素

glUniform1i: 片段着色器可以使用多个纹理, 这个函数告诉 gl 使用哪一个纹理


多级渐远纹理(mipmap)

主要是使用在纹理被缩小的情况



### 变换
向量的点乘, 叉乘
单位矩阵: 是一个除了对角线以外都是0的N×N矩阵


齐次坐标有几点好处: 
● D向量上进行位移
● 创建3D视觉效果
如果一个向量的齐次坐标是0，这个坐标就是方向向量(Direction Vector)，因为w坐标是0，这个向量就不能位移



### 旋转
万向节死锁（Gimbal Lock)
对于3D空间中的旋转，一个更好的模型是沿着任意的一个轴，比如单位向量$(0.662, 0.2, 0.7222)$旋转，而不是对一系列旋转矩阵进行复合
避免万向节死锁的真正解决方案是使用四元数(Quaternion)


坐标系统
● 局部空间(Local Space，或者称为物体空间(Object Space))
● 世界空间(World Space)
● 观察空间(View Space，或者称为视觉空间(Eye Space))
● 裁剪空间(Clip Space)
● 屏幕空间(Screen Space)









## 学习资料
● mdn：入门，api
● doodlewind  WebGL Seminar：代码范式
● webglfundamentals：入门
● 3Blue1Brown：线性代数
● https://math.hws.edu/graphicsbook/c7/s1.html#webgl3d.1.2 ： gl matrix 函数
●  闫令琪 game 101：图形学原理，入门
● 虎书
