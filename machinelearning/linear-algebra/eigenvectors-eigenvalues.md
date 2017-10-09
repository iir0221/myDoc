# 特征向量和特征值
## 特征向量和特征值的几何意义
![](images/eigenvectors-eigenvalues/1.jpg)
![](images/eigenvectors-eigenvalues/2.jpg)
![](images/eigenvectors-eigenvalues/3.jpg)
![](images/eigenvectors-eigenvalues/4.jpg)
![](images/eigenvectors-eigenvalues/5.jpg)
![](images/eigenvectors-eigenvalues/6.jpg)
![](images/eigenvectors-eigenvalues/7.jpg)
![](images/eigenvectors-eigenvalues/8.jpg)
![](images/eigenvectors-eigenvalues/9.jpg)
![](images/eigenvectors-eigenvalues/10.jpg)
![](images/eigenvectors-eigenvalues/11.jpg)
![](images/eigenvectors-eigenvalues/12.jpg)
![](images/eigenvectors-eigenvalues/13.jpg)
![](images/eigenvectors-eigenvalues/14.jpg)
![](images/eigenvectors-eigenvalues/15.jpg)
![](images/eigenvectors-eigenvalues/16.jpg)
![](images/eigenvectors-eigenvalues/17.jpg)
![](images/eigenvectors-eigenvalues/18.jpg)
![](images/eigenvectors-eigenvalues/19.jpg)
![](images/eigenvectors-eigenvalues/20.jpg)
![](images/eigenvectors-eigenvalues/21.jpg)
![](images/eigenvectors-eigenvalues/22.jpg)
![](images/eigenvectors-eigenvalues/23.jpg)
![](images/eigenvectors-eigenvalues/24.jpg)
## 特征向量和特征值的作用
如下例所示
![](images/eigenvectors-eigenvalues/25.jpg)
如果能够找到这个旋转的特征向量，也就是留在其张成的空间中的向量，也就找到了旋转轴
![](images/eigenvectors-eigenvalues/26.jpg)
![](images/eigenvectors-eigenvalues/27.jpg)
![](images/eigenvectors-eigenvalues/28.jpg)
![](images/eigenvectors-eigenvalues/29.jpg)
![](images/eigenvectors-eigenvalues/30.jpg)

### 特征值计算
理解线性变换的作用更好的方式是求出其特征向量和特征值
公式为：

![](images/eigenvectors-eigenvalues/31.gif)

即矩阵向量乘积也就是A乘以v,等于特征向量v乘以某个数,因此求解矩阵的特征向量和特征值，也就是求解上面公式的v和常数


首先将等号右边转换为矩阵向量乘积
![](images/eigenvectors-eigenvalues/32.jpg)
![](images/eigenvectors-eigenvalues/33.jpg)
![](images/eigenvectors-eigenvalues/34.jpg)
![](images/eigenvectors-eigenvalues/35.jpg)
![](images/eigenvectors-eigenvalues/36.jpg)
![](images/eigenvectors-eigenvalues/37.jpg)
![](images/eigenvectors-eigenvalues/38.jpg)
![](images/eigenvectors-eigenvalues/39.jpg)
![](images/eigenvectors-eigenvalues/40.jpg)
![](images/eigenvectors-eigenvalues/41.jpg)
![](images/eigenvectors-eigenvalues/42.jpg)
![](images/eigenvectors-eigenvalues/43.jpg)
![](images/eigenvectors-eigenvalues/44.jpg)
![](images/eigenvectors-eigenvalues/45.jpg)
![](images/eigenvectors-eigenvalues/46.jpg)
![](images/eigenvectors-eigenvalues/47.jpg)
![](images/eigenvectors-eigenvalues/48.jpg)
如果矩阵A为

![](images/eigenvectors-eigenvalues/55.gif)
![](images/eigenvectors-eigenvalues/49.jpg)
![](images/eigenvectors-eigenvalues/50.jpg)
![](images/eigenvectors-eigenvalues/51.jpg)
![](images/eigenvectors-eigenvalues/52.jpg)
![](images/eigenvectors-eigenvalues/53.jpg)
![](images/eigenvectors-eigenvalues/54.jpg)
二维矩阵不一定存在特征向量和特征值

例如90度逆时针旋转
![](images/eigenvectors-eigenvalues/56.jpg)
![](images/eigenvectors-eigenvalues/57.jpg)
![](images/eigenvectors-eigenvalues/58.jpg)
![](images/eigenvectors-eigenvalues/59.jpg)
![](images/eigenvectors-eigenvalues/60.jpg)
另一个例子是剪切变换
![](images/eigenvectors-eigenvalues/61.jpg)
![](images/eigenvectors-eigenvalues/62.jpg)
![](images/eigenvectors-eigenvalues/63.jpg)
![](images/eigenvectors-eigenvalues/64.jpg)
属于单个特征值的特征向量可以不止在一条直线上

例如将所有向量变换为两倍的矩阵，唯一的特征值是2

![](images/eigenvectors-eigenvalues/65.jpg)

## 特征基与对角矩阵
![](images/eigenvectors-eigenvalues/66.jpg)
![](images/eigenvectors-eigenvalues/67.jpg)
![](images/eigenvectors-eigenvalues/68.jpg)
![](images/eigenvectors-eigenvalues/69.jpg)
![](images/eigenvectors-eigenvalues/70.jpg)
![](images/eigenvectors-eigenvalues/71.jpg)
![](images/eigenvectors-eigenvalues/72.jpg)
对角矩阵在很多方面都很容易处理

比如矩阵与自己相乘的结果更容易计算
![](images/eigenvectors-eigenvalues/73.jpg)
![](images/eigenvectors-eigenvalues/74.jpg)

### 如何在另一个坐标系中，表述当前坐标系所描述变换
![](images/eigenvectors-eigenvalues/75.jpg)
![](images/eigenvectors-eigenvalues/76.jpg)
例如矩阵
![](images/eigenvectors-eigenvalues/89.gif)
它的特征向量为
![](images/eigenvectors-eigenvalues/90.gif),
![](images/eigenvectors-eigenvalues/91.gif)
![](images/eigenvectors-eigenvalues/77.jpg)
![](images/eigenvectors-eigenvalues/78.jpg)
![](images/eigenvectors-eigenvalues/79.jpg)
![](images/eigenvectors-eigenvalues/80.jpg)
![](images/eigenvectors-eigenvalues/81.jpg)
![](images/eigenvectors-eigenvalues/82.jpg)
![](images/eigenvectors-eigenvalues/83.jpg)
![](images/eigenvectors-eigenvalues/84.jpg)
![](images/eigenvectors-eigenvalues/85.jpg)
![](images/eigenvectors-eigenvalues/86.jpg)
![](images/eigenvectors-eigenvalues/87.jpg)
![](images/eigenvectors-eigenvalues/88.jpg)
也就是说，在原坐标系中进行矩阵变换
![](images/eigenvectors-eigenvalues/89.gif)
也就相当于在以(![](images/eigenvectors-eigenvalues/90.gif),
![](images/eigenvectors-eigenvalues/91.gif))为基的新坐标系进行矩阵变换
![](images/eigenvectors-eigenvalues/92.gif)
![](images/eigenvectors-eigenvalues/93.jpg)
![](images/eigenvectors-eigenvalues/94.jpg)
![](images/eigenvectors-eigenvalues/95.jpg)
![](images/eigenvectors-eigenvalues/96.jpg)
![](images/eigenvectors-eigenvalues/97.jpg)