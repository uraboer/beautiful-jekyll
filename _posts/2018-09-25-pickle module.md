##  pickle module

#### pickle简介

pickle模块是python中用来持久化对象的一个模块。所谓对对象进行持久化，即将对象的数据类型、存储结构、存储内容等所有信息作为文件保存下来以便下次使用

#### 用pickle保存对象到文件

```python
import pickle

mydata={'a':'123','b':[1,2,3,4]}

# 打开（或创建）一个文件，打开方式为二进制写入
file_to_save=open('mydata.pkl','wb')

# 通过dump函数将mydata保存到文件中
# 第一个参数是要保存的文件名
# 第二个参数是写入到的类文件对象file。file必须有write()接口， file可以是一个以'w'方式打开的文件或者一个StringIO对象或者其他任何实现write()接口的对象。如果protocol>=1，文件对象需要是二进制模式打开的。
# 第三个参数为序列化使用的协议版本，0：ASCII协议，所序列化的对象使用可打印的ASCII码表示；1：老式的二进制协议；2：2.3版本引入的新二进制协议，较以前的更高效；-1：使用当前版本支持的最高协议。其中协议0和1兼容老版本的python。protocol默认值为0。
pickle.dump(data1, file_to_save, -1)

# 关闭文件对象
file_to_save.close()
```



#### 用pickle从文件中读取对象

```python
import pickle

file_to_read=open('mydata.pkl','rb')

mydata=pickle.load(file_to_read)

print mydata

file_to_read.close()
```



