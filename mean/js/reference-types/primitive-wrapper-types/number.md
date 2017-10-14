## Number
* Number（primitive,wrapper）

  * Number.MAX\_VALUE

  * Number.MIN\_VALUE

  * parseInt\(val,base\) //val:值，base:进制

  * parseFloat\(val,base\)//val:值，base:进制

  * +可 parse 字符串

    * typeof\(+"42"\)--&gt;'number'
    
## 浮点数

0.1+0.2=0.30000000000000004

注意精度问题

可使用库：

**big.js**

```js
It can be loaded via a script tag in an HTML document for the browser
<script src='./relative/path/to/big.js'></script>


or as a CommonJS, Node.js or AMD module using require.
var Big = require('big.js');


For Node.js, the library is available from the npm registry:
$ npm install big.js
```

