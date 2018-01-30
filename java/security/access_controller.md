# AccessControl
大多数安全管理器都是在存取控制器的基础上实现的

## CodeSource
* CodeSource
```
public CodeSource(URL url,
                  Certificate[] certs)
构造一个 CodeSource 并将其与指定位置和证书集合相关联。
参数：
url - 位置 (URL)。
certs - 证书。它可以为 null。复制数组的内容，以防随后进行修改。
CodeSource
public CodeSource(URL url,
                  CodeSigner[] signers)
构造一个 CodeSource 并将其与指定位置和代码签名者集合相关联。
参数：
url - 位置 (URL)。
signers - 代码签名者。它可以为 null。复制数组的内容，以防随后进行修改。
从以下版本开始：
1.5
```
* hashCode
```
public int hashCode()
返回此对象的哈希码值。
覆盖：
类 Object 中的 hashCode
返回：
此对象的哈希码值。
另请参见：
Object.equals(java.lang.Object), Hashtable
```
* equals
```
public boolean equals(Object obj)
测试指定对象与此对象之间的相等性。
如果两个 CodeSource 对象的位置具有相同值并且其签名者证书链也具有相同值，
则认为这两个对象相等。不要求证书链具有相同的顺序。
覆盖：
类 Object 中的 equals
参数：
obj - 要与此对象进行相等性测试的对象。
返回：
如果对象被视为相等，则返回 true；否则返回 false。
另请参见：
Object.hashCode(), Hashtable
```
* getLocation
```
public final URL getLocation()
返回与此 CodeSource 关联的位置。
返回：
位置 (URL)。
```
* getCertificates
```
public final Certificate[] getCertificates()
返回构造codeSource对象式所用证书数组的副本；如果不存在副本，则返回 null。
```
* getCodeSigners
```
public final CodeSigner[] getCodeSigners()
返回与此 CodeSource 关联的代码签名者。
如果此 CodeSource 对象是使用 CodeSource(URL url, Certificate[] certs) 构造方法创建的，
则提取其证书链并使用它们来创建一个 CodeSigner 对象数组。
注意，仅检查 X.509 证书，所有其他证书类型都将被忽略。

返回：
代码签名者数组的副本；如果不存在副本，则返回 null。
从以下版本开始：
1.5
```
* implies
```
public boolean implies(CodeSource codesource)
判断当前codeSource能否表示参数所指定的codeSource,
一个代码源能够表示另一个代码源的条件是，前者必须包含后者的所有证书，
而且由前者的URL地址可以获取后者的URL地址。
```
## Permission
