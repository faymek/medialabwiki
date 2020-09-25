<center> <h1>numpy 用法笔记</h1></center>

# 矩阵分块

参考：https://stackoverflow.com/questions/16856788/slice-2d-array-into-smaller-2d-arrays

参考：https://stackoverflow.com/a/16873755/190597

将2D array矩阵划分成若干nrows*ncols的子矩阵

```python
def blockshaped(arr, nrows, ncols):
    """
    Return an array of shape (n, nrows, ncols) where
    n * nrows * ncols = arr.size

    If arr is a 2D array, the returned array should look like n subblocks with
    each subblock preserving the "physical" layout of arr.
    """
    h, w = arr.shape
    assert h % nrows == 0, "{} rows is not evenly divisble by {}".format(h, nrows)
    assert w % ncols == 0, "{} cols is not evenly divisble by {}".format(w, ncols)
    return (arr.reshape(h//nrows, nrows, -1, ncols)
               .swapaxes(1,2)
               .reshape(-1, nrows, ncols))

def unblockshaped(arr, h, w):
    """
    Return an array of shape (h, w) where
    h * w = arr.size

    If arr is of shape (n, nrows, ncols), n sublocks of shape (nrows, ncols),
    then the returned array preserves the "physical" layout of the sublocks.
    """
    n, nrows, ncols = arr.shape
    return (arr.reshape(h//nrows, -1, nrows, ncols)
               .swapaxes(1,2)
               .reshape(h, w))

```

**使用：**

```python
c = np.arange(24).reshape((4,6))
print(c)
# [[ 0  1  2  3  4  5]
#  [ 6  7  8  9 10 11]
#  [12 13 14 15 16 17]
#  [18 19 20 21 22 23]]

print(blockshaped(c, 2, 3))
# [[[ 0  1  2]
#   [ 6  7  8]]

#  [[ 3  4  5]
#   [ 9 10 11]]

#  [[12 13 14]
#   [18 19 20]]

#  [[15 16 17]
#   [21 22 23]]]

print(unblockshaped(blockshaped(c, 2, 3), 4, 6))
# [[ 0  1  2  3  4  5]
#  [ 6  7  8  9 10 11]
#  [12 13 14 15 16 17]
#  [18 19 20 21 22 23]]
```

**原理：**

The `h//nrows` makes sense since this keeps the first block's rows together. It also makes sense that you'll need `nrows` and `ncols` to be part of the shape. `-1` tells reshape to fill in whatever number is necessary to make the reshape valid. Armed with the form of the solution, I just tried things until I found the formula that works. I'm sorry I don't have a more insightful explanation for you. 



# 数据保存与读取

参考：https://www.cnblogs.com/wushaogui/p/9142019.html

参考：https://docs.scipy.org/doc/numpy/reference/routines.io.html

在经常性读取大量的数值文件时（比如深度学习训练数据），可以考虑现将数据存储为Numpy格式，然后直接使用numpy去读取，速度相比为转化前快很多.

下面就常用的保存数据到二进制文件和保存数据到文本文件进行介绍：

## 1.保存为二进制文件(.npy/.npz)

### [numpy.save](https://docs.scipy.org/doc/numpy/reference/generated/numpy.save.html#numpy.save)

保存一个数组到一个二进制的文件中，保存格式是`.npy`

**参数介绍**

```python
numpy.save(file, arr, allow_pickle=True, fix_imports=True)
```

> file: 文件名/文件路径
> arr: 要存储的数组
> allow_pickle: 布尔值，允许使用Python pickles保存对象数组
> fix_imports: 为了方便Pyhton2中读取Python3保存的数据

**使用**

```python
>>> import numpy as np 
#生成数据 
>>> x=np.arange(10) 
>>> x 
array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]) 
 
#数据保存 
>>> np.save('save_x',x) 
 
#读取保存的数据 
>>> np.load('save_x.npy') 
array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]) 
```

### [numpy.savez](https://docs.scipy.org/doc/numpy/reference/generated/numpy.savez.html#numpy.savez)

这个同样是保存数组到一个二进制的文件中，但是厉害的是，它可以保存多个数组到同一个文件中,保存格式是`.npz`，它其实就是多个前面np.save的保存的`npy`，再通过打包(未压缩)的方式把这些文件归到一个文件上，里面是就是自己保存的多个`npy`。

**参数介绍**

```python
numpy.savez(file, *args, **kwds)
```

> file: 文件名/文件路径
> *args: 要存储的数组,可以写多个,如果没有给数组指定Key,Numpy将默认从'arr_0','arr_1'的方式命名
> kwds: 可选

**使用**

```python
>>> import numpy as np 
#生成数据 
>>> x=np.arange(10) 
>>> x 
array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]) 
>>> y=np.sin(x) 
>>> y 
array([ 0.        ,  0.84147098,  0.90929743,  0.14112001, -0.7568025 , 
       -0.95892427, -0.2794155 ,  0.6569866 ,  0.98935825,  0.41211849]) 
        
#数据保存 
>>> np.save('save_xy',x,y) 
 
#读取保存的数据 
>>> npzfile=np.load('save_xy.npz') 
>>> npzfile  #是一个对象,无法读取 
<numpy.lib.npyio.NpzFile object at 0x7f63ce4c8860> 
 
#按照组数默认的key进行访问 
>>> npzfile['arr_0'] 
array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]) 
>>> npzfile['arr_1'] 
array([ 0.        ,  0.84147098,  0.90929743,  0.14112001, -0.7568025 , 
       -0.95892427, -0.2794155 ,  0.6569866 ,  0.98935825,  0.41211849]) 
```

更加神奇的是,你可以不适用Numpy默认给数组的Key，而是自己给数组有意义的Key，这样就可以不用去猜测自己加载数据是否是自己需要的。

```python
#数据保存 
>>> np.savez('newsave_xy',x=x,y=y) 
 
#读取保存的数据 
>>> npzfile=np.load('newsave_xy.npz') 
 
#按照保存时设定组数key进行访问 
>>> npzfile['x'] 
array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]) 
>>> npzfile['y'] 
array([ 0.        ,  0.84147098,  0.90929743,  0.14112001, -0.7568025 , 
       -0.95892427, -0.2794155 ,  0.6569866 ,  0.98935825,  0.41211849]) 
```

简直不要太爽，深度学习中，有时候你保存了训练集、验证集、测试集，还包括他们的标签，用这个方式存储起来，要啥加载啥，文件数量大大减少，也不会到处改文件名去。

### [numpy.savez_compressed](https://docs.scipy.org/doc/numpy/reference/generated/numpy.savez_compressed.html#numpy.savez_compressed)

这个就是在前面numpy.savez的基础上加了压缩，前面我介绍时尤其注明numpy.savez是得到的文件打包，不压缩的。这个文件就是对文件进行打包时使用了压缩，可以理解为压缩前各`npy`的文件大小不变，使用该函数比前面的numpy.savez得到的`npz`文件更小。

注：函数所需参数和numpy.savez一致，用法完成一样。

## 2.保存到文本文件

### [numpy.savetxt](https://docs.scipy.org/doc/numpy/reference/generated/numpy.savetxt.html#numpy.savetxt)

保存数组到文本文件上，可以直接打开查看文件里面的内容。

**参数介绍**
```python
numpy.savetxt(fname, X, fmt='%.18e', delimiter=' ', newline='\n', header='', footer='', comments='# ', encoding=None)
```

> fname:文件名/文件路径,如果文件后缀是`.gz`,文件将被自动保存为`.gzip`格式,np.loadtxt可以识别该格式
> X:要存储的1D或2D数组
> fmt:控制数据存储的格式
> delimiter:数据列之间的分隔符
> newline:数据行之间的分隔符
> header:文件头步写入的字符串
> footer:文件底部写入的字符串
> comments:文件头部或者尾部字符串的开头字符,默认是'#'
> encoding:使用默认参数

**使用**

```python
>>> import numpy as np 
#生成数据 
>>> x = y = z = np.ones((2,3)) 
>>> x 
array([[1., 1., 1.], 
       [1., 1., 1.]]) 
        
#保存数据 
np.savetxt('test.out', x) 
np.savetxt('test1.out', x,fmt='%1.4e') 
np.savetxt('test2.out', x, delimiter=',') 
np.savetxt('test3.out', x,newline='a') 
np.savetxt('test4.out', x,delimiter=',',newline='a') 
np.savetxt('test5.out', x,delimiter=',',header='abc') 
np.savetxt('test6.out', x,delimiter=',',footer='abc')
```

保存下来的文件都是友好的，可以直接打开看看有什么变化。

### [numpy.loadtxt](https://docs.scipy.org/doc/numpy/reference/generated/numpy.loadtxt.html#numpy.loadtxt)

根据前面定制的保存格式，相应的加载数据的函数也得变化。

**参数介绍**

```python
numpy.loadtxt(fname, dtype=<class 'float'>, comments='#', delimiter=None, converters=None, skiprows=0, usecols=None, unpack=False, ndmin=0, encoding='bytes')
```

> fname: 文件名/文件路径,如果文件后缀是`.gz`或`.bz2`,文件将被解压,然后再载入
> dtype: 要读取的数据类型
> comments: 文件头部或者尾部字符串的开头字符,用于识别头部,尾部字符串
> delimiter: 划分读取上来值的字符串
> converters: 数据行之间的分隔符

**使用**

```python
np.loadtxt('test.out') 
np.loadtxt('test2.out', delimiter=',') 
```
