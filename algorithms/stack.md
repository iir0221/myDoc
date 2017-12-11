# 栈
## 平衡符号(有效的括号序列)
* 给定一个字符串所表示的括号序列，包含以下字符： '(', ')', '{', '}', '[' and ']'， 判定是否是有效的括号序列。
* 样例
    * 括号必须依照 "()" 顺序表示， "()[]{}" 是有效的括号，但 "([)]"则是无效的括号。
* 解析
    * 根据报错处理未考虑到的情况
```java
public class Solution {
    /*
     * @param s: A string
     * @return: whether the string is a valid parentheses
     */
    public boolean isValidParentheses(String s) {
        // write your code here
        
        char[] charArray = s.toCharArray();

        Stack<Character> stack = new Stack();

        for(char c:charArray) {
            if(c=='('||c=='{'||c=='[') {
                stack.push(c);
                continue;
            }

            if(c==')') {
                if(!stack.isEmpty()) {
                    if(stack.pop()=='(')
                        continue;
                    else
                        return false;
                } else
                    return false;
            }

            if(c=='}') {
                if(!stack.isEmpty()) {
                    if(stack.pop()=='{')
                        continue;
                    else
                        return false;
                } else
                    return false;
            }

            if(c==']') {
                if(!stack.isEmpty()) {
                    if(stack.pop()=='[')
                        continue;
                    else
                        return false;
                } else
                    return false;
            }
        }

        if(stack.isEmpty())
            return true;
        else
            return false;
    }
}
```

## 后缀表达式（逆波兰表示式求值）

* 求逆波兰表达式的值。

    * 在逆波兰表达法中，其有效的运算符号包括 +, -, *, / 。每个运算对象可以是整数，也可以是另一个逆波兰计数表达。
* 样例
    * ["2", "1", "+", "3", "*"] -> ((2 + 1) * 3) -> 9
    * ["4", "13", "5", "/", "+"] -> (4 + (13 / 5)) -> 6
* 解析
    * 数字压入栈中，符号则取出栈中的两个数字，进行相应计算，再将计算结果压入栈中
```java
public class Solution {
    /*
     * @param tokens: The Reverse Polish Notation
     * @return: the value
     */
    public int evalRPN(String[] tokens) {
        // write your code here
        
        if(tokens.length==0)
            return 0;

        int result = 0;
        Stack<Integer> stack = new Stack<>();
        for(String token:tokens) {
            if(token.equals("+")) {
                int first = stack.pop();
                int second = stack.pop();

                result = second + first;
                stack.push(result);
            }  else if(token.equals("-")) {
                int first = stack.pop();
                int second = stack.pop();

                result = second - first;
                stack.push(result);
            } else if(token.equals("*")) {
                int first = stack.pop();
                int second = stack.pop();
                result = second * first;
                stack.push(result);
            } else if(token.equals("/")) {
                int first = stack.pop();
                int second = stack.pop();

                result = second / first;
                stack.push(result);
            }  else {
                stack.push(Integer.valueOf(token));
            }

        }

        return stack.pop();
    }
}
```

## 中序到后续的转换（将表达式转换为逆波兰表达式）
* 给定一个表达式字符串数组，返回该表达式的逆波兰表达式（即去掉括号）。
* 例子
    * 对于 [3 - 4 + 5]的表达式（该表达式可表示为["3", "-", "4", "+", "5"]），返回 [3 4 - 5 +]（该表达式可表示为 ["3", "4", "-", "5", "+"]）。
* 解析
    * 1.优先输出优先级高的运算符 
    * 2.对于相同优先级的运算符，优先输出前面的运算符
```java
public class Solution {
    /*
     * @param expression: A string array
     * @return: The Reverse Polish notation of this expression
     */
    public List<String> convertToRPN(String[] expression) {
        // write your code here
        List<String> result = new ArrayList<>();
        Stack<String> stack = new Stack<>();
        String now =null;
        for(String s:expression) {
            if(s.equals("+") || s.equals("-")) {
                while (!stack.isEmpty()) {
                    now = stack.peek();
                    if (now.equals("(")) {
                        break;
                    } else {
                        result.add(stack.pop());
                    }
                }
                stack.push(s);
            } else if(s.equals("*") || s.equals("/")) {
                while (!stack.isEmpty()) {
                    now = stack.peek();
                    if (now.equals("*") || now.equals("/"))
                        result.add(stack.pop());
                    else {
                        break;
                    }
                }
                stack.push(s);
            } else if(s.equals("(")) {
                stack.push(s);
            } else if(s.equals(")")) {
                while(!stack.isEmpty() ) {
                    now = stack.pop();
                    if(now.equals("(")) {
                        break;
                    }
                    result.add(now);
                }
            } else
                result.add(s);
        }

        while (!stack.isEmpty()) {
            result.add(stack.pop());
        }
        return result;
    }
}
```

## 表达式求值
* 先进行中序到后续的转换，得到逆波兰表达式
* 再计算逆波兰表达式的值

## 表达式展开
* 给出一个表达式 s，此表达式包括数字，字母以及方括号。在方括号前的数字表示方括号内容的重复次数(括号内的内容可以是字符串或另一个表达式)，请将这个表达式展开成一个字符串。
* 例子
    * S = abc3[a] 返回 abcaaa
    * S = 3[abc] 返回 abcabcabc
    * S = 4[ac]dy 返回 acacacacdy
    * S = 3[2[ad]3[pf]]xyz 返回 adadpfpfpfadadpfpfpfadadpfpfpfxyz
* 解析
    * 将中间结果存在Stack中
```java
public class Solution {
    /*
     * @param s: an expression includes numbers, letters and brackets
     * @return: a string
     */
    public String expressionExpand(String s) {
        // write your code here
        Stack<String> stack = new Stack<>();
        String result = "";
        Boolean isDigit = false;
        for(Character c:s.toCharArray()) {
            if(c=='[') {
                isDigit = false;
            } else if(c==']') {
                isDigit = false;
                String tmp="";
                String tmp1 = "";
                while (!stack.isEmpty()) {
                    tmp1 = stack.pop();
                    if(isInteger(tmp1)) {
                        String tmp2="";
                        for(int i=0;i<Integer.valueOf(tmp1);i++) {
                            tmp2 = tmp2+tmp;
                        }
                        if(!tmp2.equals(""))
                            stack.push(tmp2);
                        break;
                    }
                    tmp = tmp1 + tmp;
                }
            } else if(96<c || 123<c || 64<c || 133<c) {
                isDigit = false;
                stack.push(c.toString());
            } else {
                if(isDigit) {
                    String tmp = stack.pop();
                    stack.push(tmp+c);
                } else {
                    stack.push(c.toString());
                }
                isDigit = true;

            }

        }

        while (!stack.isEmpty()) {
            result = stack.pop()+result;
        }

        return result;
    }
    public boolean isInteger(String str) {
        for (int i = str.length();--i>=0;){    
            if (!Character.isDigit(str.charAt(i))){  
                return false;  
            }  
        }  
        return true;  
    }

}
```