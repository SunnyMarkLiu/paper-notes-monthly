### [Tensorflow: The Confusing Parts (2)](https://jacobbuckman.com/post/tensorflow-the-confusing-parts-2/)
- 节点命名并非必要，但在调试时非常有用。当 Tensorflow 代码崩溃时，error trace 将指向一个特定的操作。如果有很多同类型的操作，那么很难确定是哪一个出了问题。而通过明确命名每个节点，可以获得信息详细的 error trace，并更快地识别问题。
- 随着图形越来越复杂，手动命名所有内容变得愈加困难。Tensorflow 通过将一段图形创建代码封装在 `with tf.variable_scope(scope_name):` 语句中，创建的所有节点名称都将自动以 scope_name 字符串作为前缀，用斜杠分隔。
- 训练好的神经网络包括两个基本组成部分：
  - 已经学习过某些任务优化的网络权重
  - 说明如何利用权重获得结果的网络图

- Tensorflow 用于保存和加载的内置工具使用很方便：只需创建一个 tf.train.Saver()。

```python
import tensorflow as tf
a = tf.get_variable('a', [])
b = tf.get_variable('b', [])
init = tf.global_variables_initializer()

saver = tf.train.Saver()
sess = tf.Session()
sess.run(init)
saver.save(sess, './tftcp.model')
```
- 输出四个文件，重建模型所需的信息被分散到它们当中。如果想复制或者备份模型，需要有四个文件（前缀为文件名）。下面简述答案：
  - tftcp.model.data-00000-of-00001 包含模型权重，它可能这里最大的文件。
  - tftcp.model.meta 是模型的网络结构，它包含重建图形所需的所有信息。
  - tftcp.model.index 是连接权重和网络结构的索引结构。用于在数据文件中找到对应节点的参数。
  - checkpoint 实际上不需要重建模型，但如果在整个训练过程中保存了多个版本的模型，那它会跟踪所有内容。

- restore 运算将值从文件移动到会话的变量中。由于会话不再包含任何空值变量，因此不再需要初始化。（如果不小心，会适得其反：还原后运行 init 会使随机初始化的值覆盖加载的值。）
- 在创建 saver 之前确保已经创建了所有的变量。
- 当然，在某些特定的情况下，可能只需保存变量的一个子集。当创建 var_list 以期望它跟踪可用变量子集时，tf.train.Saver 允许传递 var_list。
```python
import tensorflow as tf
a = tf.get_variable('a', [])
b = tf.get_variable('b', [])
c = tf.get_variable('c', [])
saver = tf.train.Saver(var_list=[a,b])
print saver._var_list
```
- 加载模型时，如果只加载保存模型的部分组件，并且name不一致，可通过 `saver = tf.train.Saver(var_list={'a': d})` 进行映射。

```Python
import tensorflow as tf
a = tf.get_variable('a', [])  # 只加载 a
b = tf.get_variable('b', [])
init = tf.global_variables_initializer()
saver = tf.train.Saver()
sess = tf.Session()
sess.run(init)
saver.save(sess, './tftcp.model')

import tensorflow as tf
d = tf.get_variable('d', [])
init = tf.global_variables_initializer()
saver = tf.train.Saver(var_list={'a': d}) # 只加载 a，映射到 d
sess = tf.Session()
sess.run(init)
saver.restore(sess, './tftcp.model')
sess.run(d)
```
- 加载模型时，可通过 `tf.contrib.framework.list_variables('./tftcp.model')` 查看模型的变量。
