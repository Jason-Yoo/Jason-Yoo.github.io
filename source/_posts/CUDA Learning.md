CUDA Learning

##### GpuMat

一、简介
GpuMat可以从其命名看出，它是“GPU”版本的Mat，绝大部分接口和Mat相同，功能也类似。

和Mat相比，GpuMat多了两个成员函数upload和download，分别用于把数据从内存上传(通过总线传输)到显存和从显存下载(通过总线传输)到内存。

GpuMat和Mat都有数据指针(指向一块存储区域，其中存放着图像数据)，不过GpuMat的数据指针是指向显存上的某一块区域，而Mat的数据指针是指向内存上的某一块区域。

GpuMat仅支持二维数据；GpuMat::isContinuous() == false，这是为了对齐内存，当一行的数据不是8字节(具体是几字节不知道)的倍数时，为了提高访问速度，这一行数据的末尾会填充空白字节，所以一行的实际内存大小保存在GpuMat的成员step里面。

二、访问GpuMat的每个元素
要访问GpuMat的每一个元素，实现自定义的算法，就得自己重新实现一个Kernel，然后把GpuMat作为参数传进去。但实际上，为了提高程序性能，一般不直接使用GpuMat作为参数，而是使用它的精简版PtrStepSz或者PtrStep代替。

Block数量只能多不能少，否则有的像素访问不到。
观察全局Thread索引是怎么算的。
在Kernel里面一定要判断是否越界。当然，rows和cols分别是threadsPerBlock.x和threadsPerBlock.y的整倍数时，不需要判断。
访问src的一个元素的方法是src(行坐标, 列坐标)。



##### **Kernel**

Kernel是在GPU上执行的函数，访问的数据都应该在显存中；函数没有返回值，需用void作为返回类型；语法和C++相同，也能使用C++的一些标准库函数(因为这些库函数有GPU实现，不过函数名字和参数相同而已)。kernel是函数的名字，可以随便改。



##### 线程组织模型

GPU有很多个流处理器，每个流处理器相互独立，可以执行不同的代码；每个流处理器里面还有很多小核心，这些核心在同一时刻执行相同的代码，不过可以通过索引去访问不同的数据。在CUDA的线程模型里面，这些小核心对应的概念叫做Thread，每个Thread都可以计算出一个全局唯一从0开始的索引(索引可以是一维的，可以是二维的，甚至可以时是三维的)。
Grid由许多个Block组成，一个Block由许多Thread组成。

一维：Block的索引为(b_x)，Thread的索引是(t_x)
二维：Block的索引为(b_x, b_y)，Thread的索引是(t_x, t_y)
三维：Block的索引为(b_x, b_y, b_z)，Thread的索引是(t_x, t_y, t_z)