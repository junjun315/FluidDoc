

.. _cn_api_fluid_layers_create_array:

create_array
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.create_array(dtype)


创建LoDTensorArray数组。它主要用于实现RNN与array_write, array_read和While。

参数: 
    - **dtype** (int |float) — lod_tensor_array中存储元素的数据类型。

返回: lod_tensor_array， 元素数据类型为dtype。

返回类型: Variable。


**代码示例**

..  code-block:: python
  
  data = fluid.layers.create_array(dtype='float32')
  
  

.. _cn_api_fluid_layers_DynamicRNN:

DynamicRNN
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.DynamicRNN(name=None)


动态RNN可以处理一批序列数据,每个样本序列的长度可以不同。这个API自动批量处理它们。

必须设置输入lod，请参考 ``lod_tensor``

**代码示例**

..  code-block:: python

	import paddle.fluid as fluid
	data = fluid.layers.data(name='sentence', dtype='int64', lod_level=1)
	embedding = fluid.layers.embedding(input=data, size=[65535, 32],
					    is_sparse=True)

	drnn = fluid.layers.DynamicRNN()
	with drnn.block():
		word = drnn.step_input(embedding)
	     	prev = drnn.memory(shape=[200])
	     	hidden = fluid.layers.fc(input=[word, prev], size=200, act='relu')
	     	drnn.update_memory(prev, hidden)  # set prev to hidden
	     	drnn.output(hidden)

	 # last是的最后一时间步，也是编码（encoding）得出的最终结果
	last = fluid.layers.sequence_last_step(drnn())


动态RNN将按照timesteps展开开序列。用户需要在with block中定义如何处理处理每个timestep。

memory用于缓存分段数据。memory的初始值可以是零，也可以是其他变量。

动态RNN可以将多个变量标记为其输出。使用drnn()获得输出序列。

.. py:method:: step_input(x)
  
    将序列标记为动态RNN输入。

参数:
    	- **x** (Variable) - 输入序列	
	
    	
返回:当前的输入序列中的timestep。

.. py:method:: static_input(x)

将变量标记为RNN输入。输入不会分散到timestep中。

参数:
    	- **x** (Variable) - 输入序列

返回:可以访问的RNN的输入变量,。

.. py:method:: block(*args, **kwds)

用户在RNN中定义operators的block。有关详细信息，请参阅class ``docstring`` 。

.. py:method:: memory(init=None, shape=None, value=0.0, need_reorder=False, dtype='float32')

为动态rnn创建一个memory 变量。
    
如果 ``init`` 不是None， ``memory`` 将由这个变量初始化。参数 ``need_reorder`` 用于将memory重新排序作为输入变量。当memory初始化依赖于输入样本时，应该将其设置为true。

**例如**

..  code-block:: python
  
  	import paddle.fluid as fluid
  	sentence = fluid.layers.data(
                 name='sentence', dtype='float32', shape=[32])
	boot_memory = fluid.layers.data(
                 name='boot', dtype='float32', shape=[10])

	drnn = fluid.layers.DynamicRNN()
	with drnn.block():
	     word = drnn.step_input(sentence)
	     memory = drnn.memory(init=boot_memory, need_reorder=True)
	     hidden = fluid.layers.fc(
			 input=[word, memory], size=10, act='tanh')
	     drnn.update_memory(ex_mem=memory, new_mem=hidden)
	     drnn.output(hidden)
	   
	rnn_output = drnn()



否则，如果已经设置 ``shape`` 、 ``value`` 、 ``dtype`` ，memory将被 ``value`` 初始化
  
..  code-block:: python
  
	import paddle.fluid as fluid

	sentence = fluid.layers.data(
			name='sentence', dtype='float32', shape=[32])

	drnn = fluid.layers.DynamicRNN()
	with drnn.block():
	    word = drnn.step_input(sentence)
	    memory = drnn.memory(shape=[10], dtype='float32', value=0)
	    hidden = fluid.layers.fc(
		    input=[word, memory], size=10, act='tanh')
	    drnn.update_memory(ex_mem=memory, new_mem=hidden)
	    drnn.output(hidden)
	rnn_output = drnn()


参数：
    - **init** (Variable|None) – 初始化的Variable
    - **shape** (list|tuple) – memory shape. 注意形状不包含batch的大小
    - **value** (float) – 初始化的值
    - **need_reorder** (bool) – memory初始化依赖于输入样本时设置为True
    - **dtype** (str|numpy.dtype) – 初始化memory的数据类型

返回：memory Variable


.. py:method:: update_memory(ex_mem, new_mem)

将内存从 ``ex_mem`` 更新到 ``new_mem`` 。注意， ``ex_mem`` 和 ``new_mem`` 的 ``shape`` 和数据类型必须相同。

参数：
	- **ex_mem** （memory Variable）-  memory 变量（Variable） 
	- **new_mem** （memory Variable）- RNN块中生成的平坦变量（plain  variable）

返回：None


.. py:method:: output(*outputs)

标记RNN输出变量。

参数:
    - **\*outputs** - 输出变量。

返回:None
 
 
.. _cn_api_fluid_layers_StaticRNN:

StaticRNN
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.StaticRNN(name=None)


用于创建static RNN。RNN将有自己的参数，比如输入、输出、memory、状态和长度。

.. py:method:: memory(init=None, shape=None, batch_ref=None, init_value=0.0, init_batch_dim_idx=0, ref_batch_dim_idx=1)

参数：
    - **init** - boot memory，如果没有设置，则必须提供一个shape
    - **shape** - boot memory的形状
    - **batch_ref** - batch引用
    - **init_value** - boot memory的初始化值
    - **init_batch_dim_idx** - init维度中的batch大小的索引
    - **ref_batch_dim_idx** - batch_ref维度中的batch大小的索引



 
.. _cn_api_fluid_layers_shuffle:

shuffle
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.shuffle(reader, buffer_size)

使用python装饰器用shuffle 装饰 reader

参数:
    - **reader** (Variable) – 用shuffle装饰的reader
    - **buffer_size** (int) – reader中buffer的大小

返回:用shuffle装饰后的reader

返回类型:Variable


.. _cn_api_fluid_layers_double_buffer:

double_buffer
>>>>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.double_buffer(reader, place=None, name=None)


生成一个双缓冲队列reader. 数据将复制到具有双缓冲队列的位置（由place指定），如果 ``place=none`` 则将使用executor执行的位置。

参数:
  - **reader** (Variable) – 需要wrap的reader
  - **place** (Place) – 目标数据的位置. 默认是executor执行样本的位置.
  - **name** (str) – Variable 的名字. 默认为None，不关心名称时也可以设置为None


返回： 双缓冲队列的reader


**代码示例**

..  code-block:: python

	reader = fluid.layers.open_files(filenames=['somefile'],
					 shapes=[[-1, 784], [-1, 1]],
					 dtypes=['float32', 'int64'])
	reader = fluid.layers.double_buffer(reader)
	img, label = fluid.layers.read_file(reader)




.. _cn_api_fluid_layers_py_reader:

py_reader
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.py_reader(capacity, shapes, dtypes, lod_levels=None, name=None, use_double_buffer=True)


创建一个由在Python端提供数据的reader

该layer返回一个Reader Variable。reader提供了 ``decorate_paddle_reader()`` 和 ``decorate_tensor_provider()`` 来设置Python generator，作为Python端的数据源。在c++端调用 ``Executor::Run()`` 时，来自generator的数据将被自动读取。与 ``DataFeeder.feed()`` 不同，数据读取进程和  ``Executor::Run()`` 进程可以使用 ``py_reader`` 并行运行。reader的 ``start()`` 方法应该在每次数据传递开始时调用，在传递结束和抛出  ``fluid.core.EOFException`` 后执行 ``reset()`` 方法。注意， ``Program.clone()`` 方法不能克隆 ``py_reader`` 。

参数:	
  - **capacity** (int) –  ``py_reader`` 维护的缓冲区容量
  - **shapes** (list|tuple) –数据形状的元组或列表.
  - **dtypes** (list|tuple) –  ``shapes`` 对应元素的数据类型
  - **lod_levels** (list|tuple) – lod_level的整型列表或元组
  - **name** (basestring) – python 队列的前缀名称和Reader 名称。不会自动生成。
  - **use_double_buffer** (bool) – 是否使用双缓冲

返回:    reader，从reader中可以获取feed的数据

返回类型:	Variable
	


**代码示例**

1.py_reader 基本使用如下代码

..  code-block:: python

	import paddle.v2
	import paddle.fluid as fluid
	import paddle.dataset.mnist as mnist

	reader = fluid.layers.py_reader(capacity=64,
					shapes=[(-1,3,224,224), (-1,1)],
					dtypes=['float32', 'int64'])
	reader.decorate_paddle_reader(
	    paddle.v2.reader.shuffle(paddle.batch(mnist.train())

	img, label = fluid.layers.read_file(reader)
	loss = network(img, label) # 一些网络定义

	fluid.Executor(fluid.CUDAPlace(0)).run(fluid.default_startup_program())

	exe = fluid.ParallelExecutor(use_cuda=True, loss_name=loss.name)
	for epoch_id in range(10):
	    reader.start()
	    try:
		while True:
		    exe.run(fetch_list=[loss.name])
	    except fluid.core.EOFException:
		reader.reset()





2.训练和测试应使用不同的名称创建两个不同的py_reader，例如：

..  code-block:: python

	import paddle.v2
	import paddle.fluid as fluid
	import paddle.dataset.mnist as mnist

	def network(reader):
	    img, label = fluid.layers.read_file(reader)
	    # 此处我们省略了一些网络定义
	    return loss

	train_reader = fluid.layers.py_reader(capacity=64,
					      shapes=[(-1,3,224,224), (-1,1)],
					      dtypes=['float32', 'int64'],
					      name='train_reader')
	train_reader.decorate_paddle_reader(
	    paddle.v2.reader.shuffle(paddle.batch(mnist.train())

	test_reader = fluid.layers.py_reader(capacity=32,
					     shapes=[(-1,3,224,224), (-1,1)],
					     dtypes=['float32', 'int64'],
					     name='test_reader')
	test_reader.decorate_paddle_reader(paddle.batch(mnist.test(), 512))

	# 新建 train_main_prog 和 train_startup_prog
	train_main_prog = fluid.Program()
	train_startup_prog = fluid.Program()
	with fluid.program_guard(train_main_prog, train_startup_prog):
	    # 使用 fluid.unique_name.guard() 实现与test program的参数共享
	    with fluid.unique_name.guard():
		train_loss = network(train_reader) # 一些网络定义
		adam = fluid.optimizer.Adam(learning_rate=0.01)
		adam.minimize(loss)

	# Create test_main_prog and test_startup_prog
	test_main_prog = fluid.Program()
	test_startup_prog = fluid.Program()
	with fluid.program_guard(test_main_prog, test_startup_prog):
	    # 使用 fluid.unique_name.guard() 实现与train program的参数共享
	    with fluid.unique_name.guard():
		test_loss = network(test_reader)

	fluid.Executor(fluid.CUDAPlace(0)).run(train_startup_prog)
	fluid.Executor(fluid.CUDAPlace(0)).run(test_startup_prog)

	train_exe = fluid.ParallelExecutor(use_cuda=True,
			loss_name=train_loss.name, main_program=train_main_prog)
	test_exe = fluid.ParallelExecutor(use_cuda=True,
			loss_name=test_loss.name, main_program=test_main_prog)
	for epoch_id in range(10):
	    train_reader.start()
	    try:
		while True:
		    train_exe.run(fetch_list=[train_loss.name])
	    except fluid.core.EOFException:
		train_reader.reset()

	    test_reader.start()
	    try:
		while True:
		    test_exe.run(fetch_list=[test_loss.name])
	    except fluid.core.EOFException:
		test_reader.reset()




.. _cn_api_fluid_layers_log:

log
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.log(x, name=None)


给定输入张量，计算其每个元素的自然对数

.. math::
                  \\Out=ln(x)\\
 

参数:
  - **x** (Variable) – 输入张量
  - **name** (str|None, default None) – 该layer的名称，如果为None，自动命名

返回：给定输入张量计算自然对数

返回类型:	变量（variable）


**代码示例**

..  code-block:: python

  output = fluid.layers.log(x)



.. _cn_api_fluid_layers_conv2d_transpose:

conv2d_transpose
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.conv2d_transpose(input, num_filters, output_size=None, filter_size=None, padding=0, stride=1, dilation=1, groups=None, param_attr=None, bias_attr=None, use_cudnn=True, act=None, name=None)

2-D卷积转置层（Convlution2D transpose layer）

该层根据 输入（input）、滤波器（filter）和卷积核膨胀（dilations）、步长（stride）、填充（padding）来计算输出。输入(Input)和输出(Output)为NCHW格式，其中 ``N`` 为batch大小， ``C`` 为通道数（channel），``H`` 为特征高度， ``W`` 为特征宽度。参数(膨胀、步长、填充)分别都包含两个元素。这两个元素分别表示高度和宽度。欲了解卷积转置层细节，请参考下面的说明和 参考文献_ 。如果参数 ``bias_attr`` 和 ``act`` 不为 ``None``，则在卷积的输出中加入偏置，并对最终结果应用相应的激活函数。

.. _参考文献: http://www.matthewzeiler.com/wp-content/uploads/2017/07/cvpr2010.pdf

输入X和输出Out函数关系X，有等式如下：

.. math::
                        \\Out=\sigma (W*X+b)\\
                      


输入 :math:`X` 和输出 :math:`Out` O函数关系如下：

.. math::
			   
                        \\Out=\sigma (W*X+b)\\

其中：
    -  :math:`X` : 输入张量，具有 ``NCHW`` 格式

    -  :math:`W` : 滤波器张量，，具有 ``NCHW`` 格式

    -  :math:`*` : 卷积操作

    -  :math:`b` : 偏置（bias），二维张量，shape为 ``[m,1]``

    -  :math:`σ` : 激活函数
 
    -  :math:`Out` : 输出值，Out和 ``X`` 的 ``shape`` 可能不一样

**样例**：

输入：

.. math:: 

    输入张量的 ``shape`` :  ``（N，C_{in}， H_{in}， W_{in})``

    滤波器（filter） ``shape`` ： ``（C_{in}, C_{out}, H_f, W_f)``  

输出：
        
.. math:: 
    输出张量的 ``shape`` ： ``（N，C_{out}, H_{out}, W_{out})``

其中

.. math:: 

                \\H'_{out} = (H_{in}-1)*strides[0]-2*paddings[0]+dilations[0]*(H_f-1)+1\\
            
		\\W'_{out} = (W_{in}-1)*strides[1]-2*paddings[1]+dilations[1]*(W_f-1)+1 \\
            
		\\H_{out}\in[H'_{out},H'_{out} + strides[0])\\
            
		\\W_{out}\in[W'_{out},W'_{out} + strides[1])\\

            
                


参数:
	- **input**（Variable）- 输入张量，格式为[N, C, H, W]
	- **num_filters** (int) - 滤波器（卷积核）的个数，与输出的图片的通道数（ channel ）相同
	- **output_size** (int|tuple|None) - 输出图片的大小。如果output_size是一个元组（tuple），则该元形式为（image_H,image_W),这两个值必须为整型。如果output_size=None,则内部会使用filter_size、padding和stride来计算output_size。如果output_size和filter_size是同时指定的，那么它们应满足上面的公式。
	- **filter_size** (int|tuple|None) - 滤波器大小。如果filter_size是一个tuple，则形式为(filter_size_H, filter_size_W)。否则，滤波器将是一个方阵。如果filter_size=None，则内部会计算输出大小。
	- **padding (int|tuple)** - 填充大小。如果padding是一个元组，它必须包含两个整数(padding_H、padding_W)。否则，padding_H = padding_W = padding。默认:padding = 0。
	- **stride** (int|tuple) - 步长大小。如果stride是一个元组，那么元组的形式为(stride_H、stride_W)。否则，stride_H = stride_W = stride。默认:stride = 1。
	- **dilation** (int|元组) - 膨胀大小。如果dilation是一个元组，那么元组的形式为(dilation_H, dilation_W)。否则，dilation_H = dilation_W = dilation_W。默认:dilation= 1。
	- **groups** (int) - Conv2d转置层的groups个数。从Alex Krizhevsky的CNN Deep论文中的群卷积中受到启发，当group=2时，前半部分滤波器只连接到输入通道的前半部分，而后半部分滤波器只连接到输入通道的后半部分。默认值:group = 1。
	- **param_attr** (ParamAttr|None) - conv2d_transfer中可学习参数/权重的属性。如果param_attr值为None或ParamAttr的一个属性，conv2d_transfer使用ParamAttrs作为param_attr的值。如果没有设置的param_attr初始化器，那么使用Xavier初始化。默认值:None。
	- **bias_attr** (ParamAttr|bool|None) - conv2d_tran_bias中的bias属性。如果设置为False，则不会向输出单元添加偏置。如果param_attr值为None或ParamAttr的一个属性，将conv2d_transfer使用ParamAttrs作为，bias_attr。如果没有设置bias_attr的初始化器，bias将初始化为零。默认值:None。
	- **use_cudnn** (bool) - 是否使用cudnn内核，只有已安装cudnn库时才有效。默认值:True。
	- **act** (str) -  激活函数类型，如果设置为None，则不使用激活函数。默认值:None。
	-  **name** (str|None) -  该layer的名称(可选)。如果设置为None， 将自动命名该layer。默认值:True。


返回：	存储卷积转置结果的张量。

返回类型:	变量（variable）

抛出异常:

    -  ``ValueError`` : 如果输入的shape、filter_size、stride、padding和groups不匹配，抛出ValueError

**代码示例**

..  code-block:: python
  
    data = fluid.layers.data(name='data', shape=[3, 32, 32], dtype='float32')
    conv2d_transpose = fluid.layers.conv2d_transpose(input=data, num_filters=2, filter_size=3)
    
  

.. _cn_api_fluid_layers_conv3d_transpose:

conv3d_transpose
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.conv3d_transpose(input, num_filters, output_size=None, filter_size=None, padding=0, stride=1, dilation=1, groups=None, param_attr=None, bias_attr=None, use_cudnn=True, act=None, name=None)

3-D卷积转置层（Convlution3D transpose layer)

该层根据 输入（input）、滤波器（filter）和卷积核膨胀（dilations）、步长（stride）、填充来计算输出。输入(Input)和输出(Output)为NCDHW格式。其中 ``N`` 为batch大小， ``C`` 为通道数（channel）, ``D``  为特征深度, ``H`` 为特征高度， ``W`` 为特征宽度。参数(膨胀、步长、填充)分别包含两个元素。这两个元素分别表示高度和宽度。欲了解卷积转置层细节，请参考下面的说明和 参考文献_ 。如果参数 ``bias_attr`` 和 ``act`` 不为None，则在卷积的输出中加入偏置，并对最终结果应用相应的激活函数

.. _参考文献: http://www.matthewzeiler.com/wp-content/uploads/2017/07/cvpr2010.pdf

输入X和输出Out函数关系X，有等式如下：

.. math::
                        \\Out=\sigma (W*X+b)\\

其中：
    -  :math:`X` : 输入张量，具有 ``NCDHW`` 格式

    -  :math:`W` : 滤波器张量，，具有 ``NCDHW`` 格式

    -  :math:`*` : 卷积操作

    -  :math:`b` : 偏置（bias），二维张量，shape为[m,1]

    -  :math:`σ` : 激活函数
 
    -  :math:`Out` : 输出值， ``Out`` 和 ``X`` 的 shape可能不一样


**样例**

Input:

.. math::   
	
		Input shape: (N,C_{in},D_{in},H_{in},W_{in})

		Filter shape: (C_{in},C_{out},D_f,H_f,W_f)

	

Output:

.. math::   
	
		Output shape: (N,C_{out},D_{out},H_{out},W_{out})

	
其中：

.. math::   
		


		D_{out}=(D_{in}-1)*strides[0]-2*paddings[0]+dilations[0]*(D_f-1)+1
	
		H_{out}=(H_{in}-1)*strides[1]-2*paddings[1]+dilations[1]*(H_f-1)+1
	
		W_{out}=(W_{in}-1)*strides[2]-2*paddings[2]+dilations[2]*(W_f-1)+1
		


参数:
	- **input**（Variable）- 输入张量，格式为[N, C, D, H, W]
	- **num_filters** (int) - 滤波器（卷积核）的个数，与输出的图片的通道数（channel）相同
	- **output_size** (int|tuple|None) - 输出图片的大小。如果output_size是一个元组（tuple），则该元形式为（image_H,image_W),这两个值必须为整型。如果output_size=None,则内部会使用filter_size、padding和stride来计算output_size。如果output_size和filter_size是同时指定的，那么它们应满足上面的公式。
	- **filter_size** (int|tuple|None) - 滤波器大小。如果filter_size是一个tuple，则形式为(filter_size_H, filter_size_W)。否则，滤波器将是一个方阵。如果filter_size=None，则内部会计算输出大小。
	- **padding** (int|tuple) - 填充大小。如果padding是一个元组，它必须包含两个整数(padding_H、padding_W)。否则，padding_H = padding_W = padding。默认:padding = 0。
	- **stride** (int|tuple) - 步长大小。如果stride是一个元组，那么元组的形式为(stride_H、stride_W)。否则，stride_H = stride_W = stride。默认:stride = 1。
	- **dilation** (int|元组) - 膨胀大小。如果dilation是一个元组，那么元组的形式为(dilation_H, dilation_W)。否则，dilation_H = dilation_W = dilation_W。默认:dilation= 1。
	- **groups** (int) - Conv2d转置层的groups个数。从Alex Krizhevsky的CNN Deep论文中的群卷积中受到启发，当group=2时，前半部分滤波器只连接到输入通道的前半部分，而后半部分滤波器只连接到输入通道的后半部分。默认值:group = 1。
	- **param_attr** (ParamAttr|None) - conv2d_transfer中可学习参数/权重的属性。如果param_attr值为None或ParamAttr的一个属性，conv2d_transfer使用ParamAttrs作为param_attr的值。如果没有设置的param_attr初始化器，那么使用Xavier初始化。默认值:None。
	- **bias_attr** (ParamAttr|bool|None) - conv2d_tran_bias中的bias属性。如果设置为False，则不会向输出单元添加偏置。如果param_attr值为None或ParamAttr的一个属性，将conv2d_transfer使用ParamAttrs作为，bias_attr。如果没有设置bias_attr的初始化器，bias将初始化为零。默认值:None。
	- **use_cudnn** (bool) - 是否使用cudnn内核，只有已安装cudnn库时才有效。默认值:True。
	- **act** (str) -  激活函数类型，如果设置为None，则不使用激活函数。默认值:None。
	- **name** (str|None) - 该layer的名称(可选)。如果设置为None， 将自动命名该layer。默认值:True。


返回：	存储卷积转置结果的张量。

返回类型:	变量（variable）

抛出异常:

    -  ``ValueError``  : 如果输入的shape、filter_size、stride、padding和groups不匹配，抛出ValueError


**代码示例**

..  code-block:: python
  
    data = fluid.layers.data(name='data', shape=[3, 12, 32, 32], dtype='float32')
    conv3d_transpose = fluid.layers.conv3d_transpose(input=data, num_filters=2, filter_size=3)


.. _cn_api_fluid_layers_im2sequence:

im2sequence
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.im2sequence(input, filter_size=1, stride=1, padding=0, input_image_size=None, out_stride=1, name=None)

从输入张量中提取图像张量，与im2col相似，shape={input.batch_size * output_height * output_width, filter_size_H * filter_size_W * input.通道}。这个op使用filter / kernel扫描图像并将这些图像转换成序列。一个图片展开后的timestep的个数为output_height * output_width，其中output_height和output_width由下式计算:


.. math:: 
                        output_size=\frac{1+(2∗padding+img_size−block_size+stride−1)}{stride}

每个timestep的维度为block_y * block_x * input.channels。

参数:
	- **input**（Variable）- 输入张量，格式为[N, C, H, W]
	- **filter_size** (int|tuple|None) - 滤波器大小。如果filter_size是一个tuple，它必须包含两个整数(filter_size_H, filter_size_W)。否则，过滤器将是一个方阵。
    	- **stride** (int|tuple) - 步长大小。如果stride是一个元组，它必须包含两个整数(stride_H、stride_W)。否则，stride_H = stride_W = stride。默认:stride = 1。
    	- **padding** (int|tuple) - 填充大小。如果padding是一个元组，它可以包含两个整数(padding_H, padding_W)，这意味着padding_up = padding_down = padding_H和padding_left = padding_right = padding_W。或者它可以使用(padding_up, padding_left, padding_down, padding_right)来指示四个方向的填充。否则，标量填充意味着padding_up = padding_down = padding_left = padding_right = padding Default: padding = 0。
    	- **input_image_size** (Variable) - 输入包含图像的实际大小。它的维度为[batchsize，2]。该参数可有可无，是用于batch推理。
    	- **out_stride** (int|tuple) - 通过CNN缩放图像。它可有可无，只有当input_image_size不为空时才有效。如果out_stride是tuple，它必须包含(out_stride_H, out_stride_W)，否则，out_stride_H = out_stride_W = out_stride。
    	- **name** (int) - 该layer的名称，可以忽略。

返回：	LoDTensor shaoe为{batch_size * output_height * output_width, filter_size_H * filter_size_W * input.channels}。如果将输出看作一个矩阵，这个矩阵的每一行都是一个序列的step。

返回类型:	output

::

	Given:

    x = [[[[ 6.  2.  1.]
    	[ 8.  3.  5.]
    	[ 0.  2.  6.]]

    [[ 2.  4.  4.]
    [ 6.  3.  0.]
    [ 6.  4.  7.]]]

    [[[ 6.  7.  1.]
    [ 5.  7.  9.]
    [ 2.  4.  8.]]

    [[ 1.  2.  1.]
    [ 1.  3.  5.]
    [ 9.  0.  8.]]]]

    x.dims = {2, 2, 3, 3}

    And:

    filter = [2, 2]
    stride = [1, 1]
    padding = [0, 0]

    Then:

    output.data = [[ 6.  2.  8.  3.  2.  4.  6.  3.]
    [ 2.  1.  3.  5.  4.  4.  3.  0.]
    [ 8.  3.  0.  2.  6.  3.  6.  4.]
    [ 3.  5.  2.  6.  3.  0.  4.  7.]
    [ 6.  7.  5.  7.  1.  2.  1.  3.]
    [ 7.  1.  7.  9.  2.  1.  3.  5.]
    [ 5.  7.  2.  4.  1.  3.  9.  0.]
    [ 7.  9.  4.  8.  3.  5.  0.  8.]]

    output.dims = {8, 8}

    output.lod = [[4, 4]]


**代码示例**

..  code-block:: python
  
    output = fluid.layers.im2sequence(
    input=layer, stride=[1, 1], filter_size=[2, 2])


.. _cn_api_fluid_layers_nce:

nce
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.nce(input, label, num_total_classes, sample_weight=None, param_attr=None, bias_attr=None, num_neg_samples=None, name=None)

计算并返回噪音对比估计（ noise-contrastive estimation training loss）。
`请参考 See Noise-contrastive estimation: A new estimation principle for unnormalized statistical models 
<http://www.jmlr.org/proceedings/papers/v9/gutmann10a/gutmann10a.pdf>`_ 
该operator默认使用均匀分布进行抽样。

参数:
	- **input** (Variable) -  特征
	- **label** (Variable) -  标签
    	- **num_total_classes** (int) - 所有样本中的类别的总数
    	- **sample_weight** (Variable|None) - 存储每个样本权重，shape为[batch_size, 1]存储每个样本的权重。每个样本的默认权重为1.0
    	- **param_attr** (ParamAttr|None) - 可学习参数/ nce权重的参数属性。如果它没有被设置为ParamAttr的一个属性，nce将创建ParamAttr为param_attr。如没有设置param_attr的初始化器，那么参数将用Xavier初始化。默认值:None
    	- **bias_attr** (ParamAttr|bool|None) -  nce偏置的参数属性。如果设置为False，则不会向输出添加偏置（bias）。如果值为None或ParamAttr的一个属性，则bias_attr=ParamAtt。如果没有设置bias_attr的初始化器，偏置将被初始化为零。默认值:None
    	- **num_neg_samples** (int) - 负样例的数量。默认值是10
    	- **name** (str|None) - 该layer的名称(可选)。如果设置为None，该层将被自动命名

返回：	nce loss

返回类型:	变量（Variable）


**代码示例**

..  code-block:: python

    window_size = 5
    words = []
    for i in xrange(window_size):
    	words.append(layers.data(
    	name='word_{0}'.format(i), shape=[1], dtype='int64'))

    dict_size = 10000
    label_word = int(window_size / 2) + 1

    embs = []
    for i in xrange(window_size):
   	 if i == label_word:
    	continue

    emb = layers.embedding(input=words[i], size=[dict_size, 32],
    param_attr='emb.w', is_sparse=True)
    embs.append(emb)

    embs = layers.concat(input=embs, axis=1)
    loss = layers.nce(input=embs, label=words[label_word],
    num_total_classes=dict_size, param_attr='nce.w',
    bias_attr='nce.b')


.. _cn_api_fluid_layers_hsigmoid:

hsigmoid
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.hsigmoid(input, label, num_classes, param_attr=None, bias_attr=None, name=None)

层次sigmod（ hierarchical sigmoid ）加速语言模型的训练过程。这个operator将类别组织成一个完整的二叉树，每个叶节点表示一个类(一个单词)，每个内部节点进行一个二分类。对于每个单词，都有一个从根到它的叶子节点的唯一路径，hsigmoid计算路径上每个内部节点的损失（cost），并将它们相加得到总损失（cost）。hsigmoid可以把时间复杂度 ``O(N)`` 优化到 ``O(logN)`` ,其中 ``N`` 表示单词字典的大小。

`请参考 Hierarchical Probabilistic Neural Network Language Model <http://www.iro.umontreal.ca/~lisa/pointeurs/hierarchical-nnlm-aistats05.pdf>`_
    
参数:
        - **input** (Variable) - 输入张量，shape为 ``[N×D]`` ,其中 ``N`` 是minibatch的大小，D是特征大小。
        - **label**(Variable) - 训练数据的标签。该tensor的shape为 ``[N×1]``   
        - **num_classes** - (int)，类别的数量不能少于2
        - **param_attr** (ParamAttr|None) - 可学习参数/ hsigmoid权重的参数属性。如果将其设置为ParamAttr的一个属性或None，则将ParamAttr设置为param_attr。如果没有设置param_attr的初始化器，那么使用用Xavier初始化。默认值:没None。
        - **bias_attr** (ParamAttr|bool|None) - hsigmoid偏置的参数属性。如果设置为False，则不会向输出添加偏置。如果将其设置ParamAttr的一个属性或None，则将ParamAttr设置为bias_attr。如果没有设置bias_attr的初始化器，偏置将初始化为零。默认值:None。
        - **name** (str|None) - 该layer的名称(可选)。如果设置为None，该层将被自动命名。默认值:None。
    
    返回:  (Tensor) 层次sigmod（ hierarchical sigmoid） 。shape[N, 1]
    
    返回类型:  Out


**代码示例**

..  code-block:: python
        
	x = fluid.layers.data(name='x', shape=[2], dtype='float32')
    	y = fluid.layers.data(name='y', shape=[1], dtype='int64')
    	out = fluid.layers.hsigmoid(input=x, label=y, num_classes=6)

  
.. _cn_api_fluid_layers_beam_search_decode:

beam_search_decode
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.beam_search_decode(ids, scores, beam_size, end_id, name=None)

束搜索层（Beam Search Decode Layer）通过回溯LoDTensorArray ids，为每个源语句构建完整假设，LoDTensorArray ``ids`` 的lod可用于恢复束搜索树中的路径。请参阅下面的demo中的束搜索使用示例：

    ::

        fluid/tests/book/test_machine_translation.py

参数:
        - **id** (Variable) - LodTensorArray，包含所有回溯步骤重中所需的ids。
        - **score** (Variable) - LodTensorArra，包含所有回溯步骤对应的score。
        - **beam_size** (int) - 束搜索中波束的宽度。
        - **end_id** (int) - 结束token的id。
        - **name** (str|None) - 该层的名称(可选)。如果设置为None，该层将被自动命名。
    
返回：	LodTensor 对（pair）， 由生成的id序列和相应的score序列组成。两个LodTensor的shape和lod是相同的。lod的level=2，这两个level分别表示每个源句有多少个假设，每个假设有多少个id。

返回类型:	变量（variable）


**代码示例**


.. _cn_api_fluid_layers_row_conv:

row_conv
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.row_conv(input, future_context_size, param_attr=None, act=None)

行卷积（Row-convolution operator）称为超前卷积（lookahead convolution）。下面关于DeepSpeech2的paper中介绍了这个operator 
    
    `<http://www.cs.cmu.edu/~dyogatam/papers/wang+etal.iclrworkshop2016.pdf>`_ 

双向的RNN在深度语音模型中很有用，它通过对整个序列执行正向和反向传递来学习序列的表示。然而，与单向RNNs不同的是，在线部署和低延迟设置中，双向RNNs具有难度。超前卷积将来自未来子序列的信息以一种高效的方式进行计算，以改进单向递归神经网络。 row convolution operator 与一维序列卷积不同，计算方法如下:
   
给定输入序列长度 ``t`` 输入维度 ``d`` 和一个大小为上下文大小* ``d`` 的滤波器，输出序列卷积为:

.. math::   
		out_i = \sum_{j=1}^{i+context} in_{j,_:} * W_{i-j}^2 
    
公式中：
        - Out_i : 第i行输出变量 shaoe为[1, D].
        - tau： 未来上下文（featur context）大小
        - Xj: 第i行输出变量 shaoe为[1，0]
        - W_{i-j} : 第(i-j)行参数的形状[1,D]。

 `详细请参考设计文档 https://github.com/PaddlePaddle/Paddle/issues/2228#issuecomment-303903645 <https://github.com/PaddlePaddle/Paddle/issues/2228#issuecomment-303903645>`_  .

参数:
    - **input** (Variable) -- 输入是一个LodTensor，它支持可变时间长度的输入序列。这个LodTensor的内部张量是一个具有形状(T x N)的矩阵，其中T是这个mini batch中的总的timestep，N是输入数据维数。
    - **future_context_size** (int) -- 未来上下文大小。请注意，卷积核的shape是[future_context_size + 1, D]。
    - **param_attr** (ParamAttr) --  参数的属性，包括名称、初始化器等。
    - **act** (str) -- 非线性激活函数。
    
返回: 输出(Out)是一个LodTensor，它支持可变时间长度的输入序列。这个LodTensor的内部量是一个形状为 T x N 的矩阵，和X的 shape 一样。


**代码示例**

..  code-block:: python

	import paddle.fluid as fluid
     
     	x = fluid.layers.data(name='x', shape=[16],
                        dtype='float32', lod_level=1)
	out = fluid.layers.row_conv(input=x, future_context_size=2)


.. _cn_api_fluid_layers_smooth_l1:

smooth_l1
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.smooth_l1(x, y, inside_weight=None, outside_weight=None, sigma=None)

该layer计算变量x1和y 的smooth L1 loss，它以x和y的第一维大小作为批处理大小。对于每个实例，按元素计算smooth L1 loss，然后计算所有loss。输出变量的形状是[batch_size, 1]


参数:
        - **x** (Variable) - rank至少为2的张量。输入x的smmoth L1 loss 的op，shape为[batch_size, dim1，…],dimN]。
        - **y** (Variable) - rank至少为2的张量。与 ``x`` 形状一致的的smooth L1 loss  op目标值。
        - **inside_weight** (Variable|None) - rank至少为2的张量。这个输入是可选的，与x的形状应该相同。如果给定， ``(x - y)`` 的结果将乘以这个张量元素。
        - **outside_weight** (变量|None) - 一个rank至少为2的张量。这个输入是可选的，它的形状应该与 ``x`` 相同。如果给定，那么 smooth L1 loss 就会乘以这个张量元素。
        - **sigma** (float|None) - smooth L1 loss layer的超参数。标量，默认值为1.0。
   
返回：	smooth L1 loss, shape为 [batch_size, 1]

    

**代码示例**

..  code-block:: python
        
    data = fluid.layers.data(name='data', shape=[128], dtype='float32')
    label = fluid.layers.data(
    name='label', shape=[100], dtype='float32')
    fc = fluid.layers.fc(input=data, size=100)
    out = fluid.layers.smooth_l1(x=fc, y=label)


.. _cn_api_fluid_layers_ctc_greedy_decoder:

greedy_decoder
>>>>>>>>>>>>

.. py:class::paddle.fluid.layers.ctc_greedy_decoder(input, blank, name=None)

此op用于贪婪策略解码序列，步骤如下:
    
    1. 获取输入中的每一行的最大值索引。又名numpy.argmax(input, axis=0)。
    2. 对于step1结果中的每个序列，在两个空格之间合并重复token并删除所有空格。


A simple example as below:

  ::

        Given:

        input.data = [[0.6, 0.1, 0.3, 0.1],
              [0.3, 0.2, 0.4, 0.1],
              [0.1, 0.5, 0.1, 0.3],
              [0.5, 0.1, 0.3, 0.1],

              [0.5, 0.1, 0.3, 0.1],
              [0.2, 0.2, 0.2, 0.4],
              [0.2, 0.2, 0.1, 0.5],
              [0.5, 0.1, 0.3, 0.1]]

        input.lod = [[4, 4]]

        Then:

        output.data = [[2],
                       [1],
                       [3]]

        output.lod = [[2, 1]]


参数:
        - **input** (Variable) - (LoDTensor<float>)，变长序列的概率，它是一个具有LoD信息的二维张量。它的形状是[Lp, num_classes + 1]，其中Lp是所有输入序列长度的和，num_classes是真正的类别。(不包括空白标签)。
        - **blank** (int) - Connectionist Temporal Classification (CTC) loss空白标签索引,  属于半开区间[0,num_classes + 1）。
        - **name** (str) - -此层的名称。可选。
   
返回：	
        - CTC贪婪解码结果。如果结果中的所有序列都为空，则LoDTensor 为[-1]，其中LoD[[]] dims[1,1]。

返回类型： 变量（Variable）
    

**代码示例**

..  code-block:: python
        
    x = fluid.layers.data(name='x', shape=[8], dtype='float32')

    cost = fluid.layers.ctc_greedy_decoder(input=x, blank=0)



.. _cn_api_fluid_layers_pad:

pad
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.pad(x, paddings, pad_value=0.0, name=None)

在张量上加上一个由 ``pad_value`` 给出的常数值，填充宽度由 ``paddings`` 指定。
其中，维度 ``i`` 中 ``x`` 内容前填充的值个数用 ``paddings[i]`` 表示，维度 ``i`` 中 ``x`` 内容后填充的值个数用 ``paddings[i+1]`` 表示。
   
一个例子:

::

        Given:

         x = [[1, 2], [3, 4]]

        paddings = [0, 1, 1, 2]

        pad_value = 0

        Return:

        out = [[0, 1, 2, 0, 0]
            [0, 3, 4, 0, 0]
            [0, 0, 0, 0, 0]]



    参数:
        - **x** (Variable)——输入张量变量。
        - **paddings** (list)-一个整数列表。按顺序填充在每个维度上填充元素。 ``padding`` 长度必须是 ``rank(x)×2``
        - **pad_value** (float) -用来填充的常量值。
        - **name** (str|None) -这个层的名称(可选)。如果设置为None，该层将被自动命名。
   
    返回：	填充后的张量变量

    返回类型： 变量（Variable）
    

**代码示例**

..  code-block:: python
        
    out = fluid.layers.pad(
    x=x, paddings=[0, 1, 1, 2], pad_value=0.)


.. _cn_api_fluid_layers_roi_pool :

roi_pool
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.roi_pool(input, rois, pooled_height=1, pooled_width=1, spatial_scale=1.0)

ROIPool operator
    
roi池化是对非均匀大小的输入执行最大池化，以获得固定大小的特征映射(例如7*7)。
    
该operator有三个步骤:

        1. 用pooled_width和pooled_height将每个区域划分为大小相等的部分
        2. 在每个部分中找到最大的值
        3. 将这些最大值复制到输出缓冲区

Faster-RCNN.使用了roi池化。roi关于roi池化请参考 https://stackoverflow.com/questions/43430056/what-is-roi-layer-in-fast-rcnn

    参数:    
        - **input** (Variable) - 张量，ROIPoolOp的输入。输入张量的格式是NCHW。其中N为batch大小，C为输入通道数，H为特征高度，W为特征宽度
        - **roi** (Variable) -  roi区域。
        - **pooled_height** (integer) - (int，默认1)，池化输出的高度。默认:1
        - **pooled_width** (integer) -  (int，默认1) 池化输出的宽度。默认:1
        - **spatial_scale** (float) - (float，默认1.0)，用于将ROI coords从输入规模转换为池化时使用的规模。默认1.0
    
    返回:
        (张量)，ROIPoolOp的输出是一个shape为(num_rois, channel, pooled_h, pooled_w)的4d张量。
    
    返回类型: 变量（Variable）
    

    **代码示例**

..  code-block:: python
        
	pool_out = fluid.layers。roi_pool(输入=x, rois=rois, 7,7,1.0)


.. _cn_api_fluid_layers_dice_loss:

dice_loss
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.dice_loss(input, label, epsilon=1e-05)

dice_loss是比较两批数据相似度，通常用于二值图像分割，即标签为二值。
    
dice_loss定义为:

.. math::       
		dice_loss = 1- frac{2 * intersection_area}{total_rea} = frac{((total_area−intersection_area)−intersection_area)}{total_area}=frac{union_area−intersection_area}{total_area}           

参数:
    - **input** (Variable) - rank>=2的预测。第一个维度是batch大小，最后一个维度是类编号。
    - **label**（Variable）- 与输入tensor rank相同的正确的标注数据（groud truth）。第一个维度是batch大小，最后一个维度是1。
    - **epsilon** (float) - 将会加到分子和分母上。如果输入和标签都为空，则确保dice为1。默认值:0.00001
    
    返回: dice_loss shape为[1]。

    返回类型:  dice_loss(Variable)

**代码示例**

..  code-block:: python
        
	predictions = fluid.layers.softmax(x)
    	loss = fluid.layers.dice_loss(input=predictions, label=label, 2)



.. _cn_api_fluid_layers_image_resize:

image_resize
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.image_resize(input, out_shape=None, scale=None, name=None, resample='BILINEAR')

调整一批图片的大小
    
输入张量的shape为(num_batch, channels, in_h, in_w)，并且调整大小只适用于最后两个维度(高度和宽度)。
    
支持重新取样方法: 双线性插值
    
参数:
    - **input** (Variable) - 图片调整层的输入张量，这是一个shape=4的张量(num_batch, channels, in_h, in_w)
    - **out_shape** (list|tuple|Variable|None) - 图片调整层的输出，shape为(out_h, out_w)。默认值:None
    - **scale** (float|None)-输入的高度或宽度的乘数因子 。 out_shape和scale至少要设置一个。out_shape的优先级高于scale。默认值:None
    - **name** (str|None) - 该层的名称(可选)。如果设置为None，该层将被自动命名
    - **resample** (str) - 重采样方法。目前只支持“双线性”。默认值:双线性插值

返回： 4维tensor，shape为 (num_batches, channls, out_h, out_w).

返回类型:	变量（variable）


**代码示例**

..  code-block:: python
        
	out = fluid.layers.image_resize(input, out_shape=[12, 12]) 
  



.. _cn_api_fluid_layers_image_resize_short:

image_resize_short
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.image_resize_short(input, out_short_len, resample='BILINEAR')

调整一批图片的大小。输入图像的短边将被调整为给定的out_short_len 。输入图像的长边按比例调整大小，最终图像的长宽比保持不变。

参数:
        - **input** (Variable) -  图像调整图层的输入张量，这是一个4维的形状张量(num_batch, channels, in_h, in_w)。
        - **out_short_len** (int) -  输出图像的短边长度。
        - **resample** (str) - resample方法，默认为双线性插值。
    
返回：	4维张量，shape为(num_batch, channls, out_h, out_w)

返回类型:	变量（variable）



.. _cn_api_fluid_layers_image_resize_bilinear:

resize_bilinear
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.resize_bilinear(input, out_shape=None, scale=None, name=None)

双线性插值是对线性插值的扩展,即二维变量方向上(如h方向和w方向)插值。关键思想是先在一个方向上执行线性插值，然后再在另一个方向上执行线性插值。

 `详情请参阅维基百科 https://en.wikipedia.org/wiki/Bilinear_interpolation <https://en.wikipedia.org/wiki/Bilinear_interpolation>`_ 

参数:
        - **input** (Variable) - 双线性插值的输入张量，是一个shpae为(N x C x h x w)的4d张量。
        - **out_shape** (Variable) - 一维张量，包含两个数。第一个数是高度，第二个数是宽度。
        - **scale** (float|None) - 用于输入高度或宽度的乘数因子。out_shape和scale至少要设置一个。out_shape的优先级高于scale。默认值:None。
        - **name** (str|None) - 输出变量名。
    
返回：	输出的维度是(N x C x out_h x out_w)



.. _cn_api_fluid_layers_gather:

gather
>>>>>>>>>>>>

.. py:class:: paddle.fluid.layers.gather(input, index)

收集层（gather layer）

根据索引index获取X的最外层维度的条目，并将它们串连在一起。

                        Out=X[Index]

    ::

        X = [[1, 2],
             [3, 4],
             [5, 6]]

        Index = [1, 2]

        Then:

        Out = [[3, 4],
               [5, 6]]


参数:
        - **input** (Variable) - input 的rank >= 1。
        - **index** (Variable) - index的rank = 1。
    
返回：	output (Variable)

**代码示例**

..  code-block:: python
        
	output = fluid.layers.gather(x, index)
