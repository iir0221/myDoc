# 递归
* 当一个函数用它自己来定义时，就称为是递归
* 递归设计法则
    * 基准情形（必须有某些基准情形）
    * 不断推进（朝基准情形）
    * 设计法则（所有递归调用都能运行）
    * 合成效益法则（尽量不要在不同的递归操作中做重复性操作）

## 斐波那契数列
* f(0)=1
* f(1)=1
* f(n)=f(n-1)+f(n-2)(n>1)
* 用递归求解斐波那契数列违反第四条准则，效率低下
```java
public int fibonacci(int n) {
    if(n==1)
        return 0;
    if(n==2)
        return 1;
    return fibonacci(n-1)+fibonacci(n-2);
    
}
```
* O(N)
```java    
public int fibonacci(int n) {
    if(n==1)
        return 0;
    if(n==2)
        return 1;

    int first = 0;
    int second = 1;
    int result = 0;

    for(int i=3;i<=n;i++) {
        result = first + second;
        first = second;
        second = result;
    }
    
    return result;
}
```

## 欧几里得算法
欧几里德算法又称辗转相除法，是指用于计算两个正整数a，b的最大公约数。应用领域有数学和计算机两个方面。计算公式gcd(a,b) = gcd(b,a mod b)。

### 定理
定理：两个整数的最大公约数等于其中较小的那个数和两数相除余数的最大公约数。

最大公约数（Greatest Common Divisor）缩写为GCD。

gcd(a,b) = gcd(b,a mod b) (不妨设a>b 且r=a mod b ,r不为0)
```java
//gcd(m,n)=gcd(n,m%n)
public class Euclidean {

    public int divisor(int m,int n) {
        if (m % n == 0) {
            return n;
        }
        else {
            return divisor(n,m % n);
        }
    }

    public int gcd(int m,int n) {
        if(m>n)
            return divisor(m,n);
        else
            return divisor(n,m);
    }
}
```

## 总结
* 注意边界的表示
例如[解法3中](subarray.md)
    * 注意使用0，left，right，nums.length
    * 在非递归问题中直接使用0，nums.length作为边界
    * 递归中则应该使用left，right