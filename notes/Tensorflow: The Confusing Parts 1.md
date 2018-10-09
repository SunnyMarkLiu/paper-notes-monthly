### [Tensorflow: The Confusing Parts (1)](https://jacobbuckman.com/post/tensorflow-the-confusing-parts-1/)
- 会话（session）的作用是处理内存分配和优化，使我们能够实际执行由计算图指定的计算。你可以将计算图想象为我们想要执行的计算的「模版」：它列出了所有步骤。为了使用计算图，我们需要启动一个会话，它使我们能够实际地完成任务
- 创建变量采用 `tf.get_variable()` 方法，当首次创建变量节点时，它的值基本上为「null」，并且任何试图对它求值的操作都会引发这个异常。计算图中创建了变量，必须要进行初始化操作：
  - tf.assign()
```python
import tensorflow as tf
count_variable = tf.get_variable("count", [])
zero_node = tf.constant(0.)
assign_node = tf.assign(count_variable, zero_node)
sess = tf.Session()
sess.run(assign_node)
print sess.run(count_variable)
```
 - initializer
```python
import tensorflow as tf
const_init_node = tf.constant_initializer(0.)
count_variable = tf.get_variable("count", [], initializer=const_init_node)
init = tf.global_variables_initializer()
sess = tf.Session()
sess.run(init)
print sess.run(count_variable)
```
- 变量共享涉及创建作用域并设置「reuse = True」，如果你想在多个地方使用单个变量，只需以编程方式记录指向该变量节点的指针，并在需要时重新使用它。In other words, you should have **only a single call** of `tf.get_variable()` for each parameter you intend to store in memory.
- `optimizer = tf.train.GradientDescentOptimizer(1e-3)` 不会向计算图中添加节点。它只是创建一个包含有用的帮助函数的 Python 对象。`train_op = optimizer.minimize(loss)` 将一个节点添加到图中，并将一个指针存储在变量 train_op 中。train_op 节点没有输出，但是有一个十分复杂的计算。
```Python
optimizer = tf.train.GradientDescentOptimizer(1e-3)
train_op = optimizer.minimize(loss)
```
- train_op 回溯输入和损失的计算路径，寻找变量节点。对于它找到的每个变量节点，计算该变量对于损失的梯度。然后计算该变量的新值：当前值减去梯度乘以学习率的积。最后，它执行赋值操作更新变量的值。
