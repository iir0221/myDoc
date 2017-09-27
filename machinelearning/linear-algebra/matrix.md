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
### 矩阵向量乘法
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

## 复合变换（矩阵相乘）
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