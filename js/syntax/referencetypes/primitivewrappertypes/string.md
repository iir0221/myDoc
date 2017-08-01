* String\(**primitive,wrapper**\)

  * multiline
    ```js
    console.log(`string text on first line
    string text on second line `);
    ```
  * 转义字符：
    * \n  \t   \b   \r   \   \'   \" 
  * interpolation

  ```js
  var a=1, b=2;
  console.log("Sum of values is :" + (a+b) + " and multiplication is :" + (a*b));
  // 简写
  console.log(`Sum of values is :${a+b} and multiplication is : ${a*b}`);
  ```

