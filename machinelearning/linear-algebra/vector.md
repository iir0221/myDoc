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
    * 一个向量保持不动，另一个向量张成的空间是一条直线。
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
