# vector
## 计算机视角
* vercots<=>lists of numbers
* 向量<=>数字列表
* 向量只不过是列表的一种说法
* 2 dimensional二维向量是因为这个列表的长度是2


## 向量加法
### 几何角度
![](images/vector-addition/1.jpg)
![](images/vector-addition/2.jpg)
![](images/vector-addition/3.jpg)
![](images/vector-addition/4.jpg)
![](images/vector-addition/5.jpg)
![](images/vector-addition/6.jpg)
![](images/vector-addition/7.jpg)
![](images/vector-addition/8.jpg)
![](images/vector-addition/9.jpg)
![](images/vector-addition/10.jpg)

### 计算机视角
向量加法就是把列表对应项相加
![](images/vector-addition/11.jpg)

## 向量数乘
### 几何角度
向量数乘就是向量缩放
![](images/scalar-multipication/1.jpg)
![](images/scalar-multipication/2.jpg)
![](images/scalar-multipication/3.jpg)
![](images/scalar-multipication/4.jpg)
![](images/scalar-multipication/5.jpg)
### 计算机角度
向量数乘就是列表中的每个元素与标量相乘
![](images/scalar-multipication/6.jpg)
![](images/scalar-multipication/7.jpg)
![](images/scalar-multipication/8.jpg)
![](images/scalar-multipication/9.jpg)

# 坐标
i帽，j帽（x,y方向单位向量）
![](images/linear-combination/1.jpg)
![](images/linear-combination/2.jpg)
![](images/linear-combination/3.jpg)
## 基向量
当我们用数字描述向量时，它都依赖于我们正在使用的基（坐标系）
![](images/linear-combination/4.jpg)
![](images/linear-combination/5.jpg)
![](images/linear-combination/6.jpg)

# 向量组合
![](images/linear-combination/7.jpg)
![](images/linear-combination/8.jpg)
![](images/linear-combination/9.jpg)
![](images/linear-combination/10.jpg)
![](images/linear-combination/11.jpg)
![](images/linear-combination/12.jpg)
![](images/linear-combination/13.jpg)
![](images/linear-combination/14.jpg)
![](images/linear-combination/15.jpg)
## 张成的空间
![](images/linear-combination/16.jpg)

* 在二维平面内的两个向量
    * 一个向量保持不动，另一个向量的标量自由变化，张成的空间是一条直线。

        证明：

        两个向量(a,b),(c,d)

        组合得到向量(a+c,b+d)

        其中一个向量的标量自由变化

        组合得到向量(a+ck,b+dk),k为变量

        由

        ![](images/linear-combination/math1.gif)

        可得，

        一个向量保持不动，另一个向量的标量自由变化，张成的空间是一条直线。

    * 若两个向量都运动，则张成的空间是整个平面（若两个向量方向相同则张成的空间还是一条直线）
* 在三维空间内的三个向量
    * 一个向量保持不动，另两个向量张成的空间是一个穿过远点的平面。
    * 若三个向量都运动，则张成的空间是整个三维空间
    ![](images/linear-combination/17.jpg)

# 线性相关和线性无关
## 线性相关
![](images/linear-combination/18.jpg)
![](images/linear-combination/19.jpg)
![](images/linear-combination/20.jpg)
## 线性无关
![](images/linear-combination/21.jpg)
![](images/linear-combination/22.jpg)
![](images/linear-combination/23.jpg)
# 基的严格定义
![](images/linear-combination/24.jpg)

# 矩阵和线性变换
## 线性变换Linear transformation

* linear transformation就是函数的一种说法
* 在线性代数中，就是输入一个向量，输出一个变换向量
### linear transformation特点
* 直线依旧是直线
* 原点保持固定
* ![](images/linear-transformation/1.jpg)
![](images/linear-transformation/2.jpg)

### 如何用数值描述线性变换
![](images/linear-transformation/3.jpg)
![](images/linear-transformation/4.jpg)
* 例如
    * 向量v=-1i+2j
    * Transformed v=-1(Transformed i)+2(Transformed j)
    * Transformed i=1i-2j
    * Transformed j=3i+0j
    * Transformed v=-1(1i-2j)+2(3i+0j)=-i+2j+6i=5i+2j
    * 对于任意向量(xi,yj),其变化后的位置就是x(1i-2j)+y(3i+0j)=xi-2xj+3yi=(x+3y)i-2xj
![](images/linear-transformation/5.jpg)
![](images/linear-transformation/6.jpg)
![](images/linear-transformation/7.jpg)
![](images/linear-transformation/8.jpg)
![](images/linear-transformation/9.jpg)
![](images/linear-transformation/10.jpg)
![](images/linear-transformation/11.jpg)
![](images/linear-transformation/12.jpg)
![](images/linear-transformation/13.jpg)
### 矩阵乘法
![](images/linear-transformation/14.jpg)
![](images/linear-transformation/15.jpg)
![](images/linear-transformation/16.jpg)
![](images/linear-transformation/17.jpg)
![](images/linear-transformation/18.jpg)
![](images/linear-transformation/19.jpg)
![](images/linear-transformation/20.jpg)
![](images/linear-transformation/21.jpg)
![](images/linear-transformation/22.jpg)

空间逆时针旋转90度

![](images/linear-transformation/23.jpg)
![](images/linear-transformation/24.jpg)

如果变化后的i帽与j帽是线性相关的，即其中一个向量是另一个向量的倍数，则这个线性变化将二维空间挤压到它们所在的一条直线上

总结，线性变换

![](images/linear-transformation/25.jpg)
![](images/linear-transformation/26.jpg)
![](images/linear-transformation/27.jpg)
![](images/linear-transformation/28.jpg)
![](images/linear-transformation/29.jpg)
![](images/linear-transformation/30.jpg)
![](images/linear-transformation/31.jpg)

# 复合变换（矩阵相乘）
对一个向量先后应用旋转与剪切变换，等同于对它应用旋转与剪切的复合变换
![](images/composition/1.jpg)
![](images/composition/2.jpg)
![](images/composition/3.jpg)
![](images/composition/4.jpg)
![](images/composition/5.jpg)
![](images/composition/6.jpg)
![](images/composition/7.jpg)
![](images/composition/8.jpg)
![](images/composition/9.jpg)
![](images/composition/10.jpg)
![](images/composition/11.jpg)
![](images/composition/12.jpg)
![](images/composition/13.jpg)
![](images/composition/14.jpg)
![](images/composition/15.jpg)
* M1*M2不等于M2*M1
    * 先进行M2变换再进行M1变换不等于先进性M1变换再进行M2变换
* (AB)C=A(BC)
    * 成立的原因是，二者都是先进行C变换再进行B变换再进行A变换

# 行列式

![](images/determinant/1.jpg)
![](images/determinant/2.jpg)
![](images/determinant/3.jpg)
![](images/determinant/4.jpg)
![](images/determinant/5.jpg)
![](images/determinant/6.jpg)
![](images/determinant/7.jpg)
![](images/determinant/8.jpg)
![](images/determinant/9.jpg)
![](images/determinant/10.jpg)
![](images/determinant/11.jpg)
![](images/determinant/12.jpg)
![](images/determinant/13.jpg)
![](images/determinant/14.jpg)
![](images/determinant/15.jpg)
![](images/determinant/16.jpg)
![](images/determinant/17.jpg)
![](images/determinant/18.jpg)
![](images/determinant/19.jpg)
![](images/determinant/20.jpg)
![](images/determinant/21.jpg)
![](images/determinant/22.jpg)
![](images/determinant/23.jpg)
![](images/determinant/24.jpg)
![](images/determinant/25.jpg)
![](images/determinant/26.jpg)
![](images/determinant/27.jpg)
![](images/determinant/28.jpg)
![](images/determinant/29.jpg)
![](images/determinant/30.jpg)
![](images/determinant/31.jpg)
![](images/determinant/32.jpg)
![](images/determinant/33.jpg)
![](images/determinant/34.jpg)
![](images/determinant/35.jpg)
![](images/determinant/36.jpg)
![](images/determinant/37.jpg)
![](images/determinant/38.jpg)
![](images/determinant/39.jpg)
![](images/determinant/40.jpg)
![](images/determinant/41.jpg)
![](images/determinant/42.jpg)
![](images/determinant/43.jpg)
![](images/determinant/44.jpg)
![](images/determinant/45.jpg)
![](images/determinant/46.jpg)
![](images/determinant/47.jpg)
![](images/determinant/48.jpg)
![](images/determinant/49.jpg)
![](images/determinant/50.jpg)
![](images/determinant/51.jpg)
![](images/determinant/52.jpg)
![](images/determinant/53.jpg)
![](images/determinant/54.jpg)
![](images/determinant/55.jpg)


# 逆矩阵：通过逆矩阵求解线性方程组

![](images/inverse-transformation/1.jpg)
![](images/inverse-transformation/2.jpg)
![](images/inverse-transformation/3.jpg)
![](images/inverse-transformation/4.jpg)
![](images/inverse-transformation/5.jpg)
![](images/inverse-transformation/6.jpg)
![](images/inverse-transformation/7.jpg)
![](images/inverse-transformation/8.jpg)
![](images/inverse-transformation/9.jpg)
![](images/inverse-transformation/10.jpg)
![](images/inverse-transformation/11.jpg)
![](images/inverse-transformation/12.jpg)
![](images/inverse-transformation/13.jpg)
![](images/inverse-transformation/14.jpg)
![](images/inverse-transformation/15.jpg)
![](images/inverse-transformation/16.jpg)
![](images/inverse-transformation/17.jpg)

所以A逆的核心性质在于：A逆乘以A等于一个什么都不做的矩阵，这个什么都不做的矩阵被称为恒等变换
![](images/inverse-transformation/18.jpg)
![](images/inverse-transformation/19.jpg)
![](images/inverse-transformation/20.jpg)
![](images/inverse-transformation/21.jpg)
![](images/inverse-transformation/22.jpg)
![](images/inverse-transformation/23.jpg)
![](images/inverse-transformation/24.jpg)
![](images/inverse-transformation/25.jpg)
![](images/inverse-transformation/26.jpg)
![](images/inverse-transformation/27.jpg)
![](images/inverse-transformation/28.jpg)
![](images/inverse-transformation/29.jpg)
![](images/inverse-transformation/30.jpg)
![](images/inverse-transformation/31.jpg)
![](images/inverse-transformation/32.jpg)
![](images/inverse-transformation/33.jpg)


# 秩rank:代表变换后空间的维度
![](images/rank/1.jpg)
![](images/rank/2.jpg)
![](images/rank/3.jpg)
![](images/rank/4.jpg)

# 列空间（向量空间）
![](images/column-space/1.jpg)
![](images/column-space/2.jpg)
![](images/column-space/3.jpg)
![](images/column-space/4.jpg)
![](images/column-space/5.jpg)
![](images/column-space/6.jpg)
![](images/column-space/7.jpg)
# 零空间
![](images/null-space/1.jpg)
![](images/null-space/2.jpg)
![](images/null-space/3.jpg)
![](images/null-space/4.jpg)
![](images/null-space/5.jpg)
![](images/null-space/6.jpg)
![](images/null-space/7.jpg)
![](images/null-space/8.jpg)
![](images/null-space/9.jpg)
![](images/null-space/10.jpg)
![](images/null-space/11.jpg)
![](images/null-space/12.jpg)
![](images/null-space/13.jpg)
![](images/null-space/14.jpg)
![](images/null-space/15.jpg)
![](images/null-space/16.jpg)

# 非方阵的矩阵
![](images/nonsquare/1.jpg)
![](images/nonsquare/2.jpg)
![](images/nonsquare/3.jpg)
![](images/nonsquare/4.jpg)
![](images/nonsquare/5.jpg)
![](images/nonsquare/6.jpg)
![](images/nonsquare/7.jpg)
![](images/nonsquare/8.jpg)
![](images/nonsquare/9.jpg)
![](images/nonsquare/10.jpg)
![](images/nonsquare/11.jpg)
![](images/nonsquare/12.jpg)
![](images/nonsquare/13.jpg)
![](images/nonsquare/14.jpg)
![](images/nonsquare/15.jpg)
![](images/nonsquare/16.jpg)
![](images/nonsquare/17.jpg)
![](images/nonsquare/18.jpg)
![](images/nonsquare/19.jpg)
![](images/nonsquare/20.jpg)
![](images/nonsquare/21.jpg)
![](images/nonsquare/22.jpg)
![](images/nonsquare/23.jpg)
![](images/nonsquare/24.jpg)

# 点积
点积：两个维数相同的向量，将相应坐标配对，求出每一对坐标的乘积，然后将结果相加
![](images/dot-products/1.jpg)

几何解释
![](images/dot-products/2.jpg)
![](images/dot-products/3.jpg)
![](images/dot-products/4.jpg)
![](images/dot-products/5.jpg)
![](images/dot-products/6.jpg)
![](images/dot-products/7.jpg)
![](images/dot-products/8.jpg)
![](images/dot-products/9.jpg)
![](images/dot-products/10.jpg)

点积v.w=w.v原因：

![](images/dot-products/11.jpg)
![](images/dot-products/12.jpg)
![](images/dot-products/13.jpg)
![](images/dot-products/14.jpg)
![](images/dot-products/15.jpg)
![](images/dot-products/16.jpg)
![](images/dot-products/17.jpg)
![](images/dot-products/18.jpg)
![](images/dot-products/19.jpg)
![](images/dot-products/20.jpg)
![](images/dot-products/21.jpg)
![](images/dot-products/22.jpg)
![](images/dot-products/23.jpg)
![](images/dot-products/24.jpg)
![](images/dot-products/25.jpg)
![](images/dot-products/26.jpg)
![](images/dot-products/27.jpg)
## 对偶性
![](images/dot-products/28.jpg)
![](images/dot-products/29.jpg)
### 多维空间到一维空间线性变换
![](images/dot-products/30.jpg)
![](images/dot-products/31.jpg)
![](images/dot-products/32.jpg)
![](images/dot-products/33.jpg)
![](images/dot-products/34.jpg)
![](images/dot-products/35.jpg)
![](images/dot-products/36.jpg)
![](images/dot-products/37.jpg)
![](images/dot-products/38.jpg)
![](images/dot-products/39.jpg)
![](images/dot-products/40.jpg)
![](images/dot-products/41.jpg)
![](images/dot-products/42.jpg)
![](images/dot-products/43.jpg)
![](images/dot-products/44.jpg)
![](images/dot-products/45.jpg)
![](images/dot-products/46.jpg)
![](images/dot-products/47.jpg)
![](images/dot-products/48.jpg)
![](images/dot-products/49.jpg)
![](images/dot-products/50.jpg)
![](images/dot-products/51.jpg)
![](images/dot-products/52.jpg)
![](images/dot-products/53.jpg)
![](images/dot-products/54.jpg)
![](images/dot-products/55.jpg)
![](images/dot-products/56.jpg)

几何上的意义

![](images/dot-products/57.jpg)
![](images/dot-products/58.jpg)


![](images/dot-products/59.jpg)
![](images/dot-products/60.jpg)
![](images/dot-products/61.jpg)
![](images/dot-products/62.jpg)
![](images/dot-products/63.jpg)
![](images/dot-products/64.jpg)
![](images/dot-products/65.jpg)
![](images/dot-products/66.jpg)
![](images/dot-products/67.jpg)
![](images/dot-products/68.jpg)
![](images/dot-products/69.jpg)
![](images/dot-products/70.jpg)
![](images/dot-products/71.jpg)
![](images/dot-products/72.jpg)
![](images/dot-products/73.jpg)
![](images/dot-products/74.jpg)
![](images/dot-products/75.jpg)
![](images/dot-products/76.jpg)
![](images/dot-products/77.jpg)
![](images/dot-products/78.jpg)
![](images/dot-products/79.jpg)
![](images/dot-products/80.jpg)
![](images/dot-products/81.jpg)
![](images/dot-products/82.jpg)
![](images/dot-products/83.jpg)
![](images/dot-products/84.jpg)
![](images/dot-products/85.jpg)

任意向量

![](images/dot-products/86.jpg)
![](images/dot-products/87.jpg)
![](images/dot-products/88.jpg)
![](images/dot-products/89.jpg)
![](images/dot-products/90.jpg)
![](images/dot-products/91.jpg)

非单位向量

![](images/dot-products/92.jpg)
![](images/dot-products/93.jpg)
![](images/dot-products/94.jpg)
![](images/dot-products/95.jpg)
![](images/dot-products/96.jpg)
![](images/dot-products/97.jpg)
![](images/dot-products/98.jpg)
![](images/dot-products/99.jpg)