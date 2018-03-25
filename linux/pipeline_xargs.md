管道是实现“将前面的标准输出作为后面的标准输入”
xargs是实现“将标准输入作为命令的参数”

你可以试试运行：

代码:
echo "--help"|cat
echo "--help"|xargs cat