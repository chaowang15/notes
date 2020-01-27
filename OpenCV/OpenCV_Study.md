<!-- 本文同时包含了多种markdown的使用技巧，参见相关注释 -->

# OpenCV 学习笔记

## cv::Mat Usage 使用说明

参考：[Mat - The Basic Image Container](https://docs.opencv.org/2.4/doc/tutorials/core/mat_the_basic_image_container/mat_the_basic_image_container.html)


### Mat构成

cv::Mat的构成很特殊。它包含两个部分：
- **矩阵头部**（matrix header），它包含了诸如矩阵的尺寸和存储方式等。因此，Mat的header的大小通常是固定的，而整个Mat大小显然不固定，因为矩阵主体数据不固定。
- **一个指向矩阵主体数据的指针**。也就是说，两个不同的cv::Mat的指针可以指向同一块数据区域。这样的好处是，图片通常都比较大，这样无需分配额外内存来存储多余的图片。当然缺点很明显，如果Mat指向的数据区域被release的话，所有的指向它的Mat都会失效。这样的话，将来再次尝试获取某个像素的话就会出错。

因此，Mat的Copy constructor和Assignment operator都是**仅仅拷贝了矩阵的header，并未拷贝矩阵主体**。换句话说，如果原始的Mat被release的话，新的拷贝Mat所指向的矩阵主体就会为空。同样的，如果修改拷贝的Mat，那么原始的Mat指向的矩阵主体也会改变。

```C++
// 此时只是定义了矩阵的header部分，还未给矩阵主体分配空间
cv::Mat A, C; 
// 读入图片同时为矩阵主体分配内存空间
A = cv::imread(argv[1], CV_LOAD_IMAGE_COLOR); 
// 注意：这里的B只拷贝了A的header而已，它的指针指向A指向的矩阵主体部分。
// 此后如果修改B的话，A指向的内容也会被改变，这很危险。
cv::Mat B(A);
// Assignment operator和上面的Copy Constructor作用相同，还是只拷贝了header部分。
C = A; 
```

那么，如果真的想要**整体拷贝**一个Mat全部内容怎么办？有两个函数：`clone()`以及`copyTo()`:
```C++
Mat F = A.clone(); // 直接硬拷贝
Mat G;
A.copyTo(G); // copyTo和clone()结果完全一样，就是用法稍有不同
```

### 初始化

- Mat constructors官方文档：https://docs.opencv.org/2.4/modules/core/doc/basic_structures.html#mat-mat
- Mat初始化的一些例子：https://docs.opencv.org/2.4/doc/tutorials/core/mat_the_basic_image_container/mat_the_basic_image_container.html


### Mat成员函数

```C++
if (img.depth() == CV_8U){
    // depth()函数获取的是深度，即每个像素点中每个元素的类型。这里就是8-bit unsigned char类型
}
if (img.type() == CV_8UC3){
    // type()函数获取的是详细类型，这里就是8-bit unsigned char且有3个channels
}
if (img.channels() == 3){
    // channels()获取的是每个像素点的元素个数，例如如果是BGR图片，这个值就是3
}
```

### 其它使用方法

```c++
// 获取一个Mat中的一小块矩形部分
cv::Mat small_img = img(cv::Rect(0, 0, 100, 200)); // 注意这里依然是只拷贝了header
// 修改这个小块。注意此时img指向的矩阵主体也会改变，因为它们指向同一个主体部分
small_img.setTo(0);

```


## Access an image 操作图片像素

参考：[How to scan images, lookup tables and time measurement with OpenCV](https://docs.opencv.org/2.4/doc/tutorials/core/how_to_scan_images/how_to_scan_images.html#the-efficient-way)

OpenCV中，一张图片中每个像素点可能只有一个unsigned char变量，即8-bit元素，例如灰度图片；也可能是多个unsigned char变量，例如RGB或者RGBA彩色图片。这里每个像素中有几个量就叫做**通道**（channels）。灰度图片（grayscale）就是单通道，RGB图片就是三通道。下图演示了RGB图片中的数据存储：

<!--- 这是如何在Markdown中添加图片。如果想要增加本地图片，就把下面的链接替换成本地图片的路径即可。如果图片和该md文档在同一路径下，那么只用图片名就行。下面括号内最后的一个字符串是title text，当鼠标在图片处悬停时会显示 --->
![OpenCV_image_channels](https://docs.opencv.org/2.4/_images/math/b6df115410caafea291ceb011f19cc4a19ae6c2c.png "OpenCV Image Channels")


**注意**：
- OpenCV中读入彩色图片到`cv::Mat`中后，默认是按照BGR顺序，即蓝绿红，如上图所示。而通常很多图片存储后是按照RGB顺序的。这点一定要注意。
- 可以通过`cv::Mat`类的成员函数`channels()`获取该图片的通道个数。


### Get one pixel 获取一个像素

如果只是想要获取某一个像素的话，可以使用OpenCV中为Mat定义的at()函数：
```c++
// 灰度图像每个像素点就是一个unsigne char
unsigned char gray = gray_img.at<uchar>(i,j); 
// 彩色图像每个像素点包含RGB这三个channels（有时候是RGBA四个）
unsigned char b = color_img.at<cv::Vec3b>(i,j)[0]; 
unsigned char g = color_img.at<cv::Vec3b>(i,j)[1];
unsigned char r = color_img.at<cv::Vec3b>(i,j)[2];
```

### Scan entire image 遍历整张图片

#### 1. The efficient way 最快的方法：基于C指针

需要注意的是，虽然一张图片的物理尺寸是`width x height`，但是实际上，由于内存通常都足够大，因此OpenCV中的cv::Mat读入图片后，其实通常是**将图片中全部的行按照顺序存储到一行**中的。即，类似于将所有的行“平铺”到一行，这样的话，一张图片其实就是一个一维数组。这样做的好处是，可以加快扫描整张图片的速度。当然，这只是底层的存储方式有变化而已，该图片的顶层的各种参数全都不变。当然这也不一定，因此在实现中，可以用`isContinuous()`函数来判断一个`cv::Mat`变量是否是连续存储的。

如果是只存储一行，那么最快的获取每个像素的方法就是使用C中的操作符`[]`。下面代码的作用是，遍历整张图片，并将每个像素的灰度值通过table来in-place得映射到另一个值。

<!-- 添加代码 -->
```C++
// 遍历整张图片，并将每个像素的灰度值通过table来in-place得映射到另一个值
Mat& ScanImageAndReduceC(Mat& I, const uchar* const table)
{
    // accept only char type matrices
    CV_Assert(I.depth() == CV_8U);
    int channels = I.channels();
    int nRows = I.rows;
    int nCols = I.cols * channels;
    // 检查一下是否是连续存储的。如果是，则更新行和列的值。
    if (I.isContinuous())
    {
        nCols *= nRows;
        nRows = 1;
    }
    int i,j;
    uchar* p;
    for( i = 0; i < nRows; ++i)
    {
        p = I.ptr<uchar>(i); // 先获取每一行开头的指针
        for ( j = 0; j < nCols; ++j)
        {
            // p[j]就是每个像素了。
            // Here it maps each pixel's grayscale value to another value.
            p[j] = table[p[j]];
        }
    }
    return I;
}
```
上面代码调用了`isContinuous()`来判断是否为连续存储的（即一行）。

### 2. The iterator (safe) way 迭代器做法（更安全）

上面的基于C的快速法中，用户需要自行确保判断了全部的channels、跳过有可能出现的行和行之间的gaps，并且确保不能越界。相比之下，使用C++的iterator更加安全一些。只需获取Mat的起始和终止指针，在中间遍历即可。不过，该方法还是比上面的C指针的方法慢一些。下面代码作用和上面的代码相同，只是增加了对BGR彩色图片的处理。

```C++
// 依然是遍历整张图片，并将每个像素的每个通道的颜色值通过table来in-place得映射到另一个值
Mat& ScanImageAndReduceIterator(Mat& I, const uchar* const table)
{
    // accept only char type matrices
    CV_Assert(I.depth() == CV_8U);
    const int channels = I.channels();
    switch(channels)
    {
    case 1:
        {
            MatIterator_<uchar> it, end;
            for( it = I.begin<uchar>(), end = I.end<uchar>(); it != end; ++it)
                *it = table[*it];
            break;
        }
    case 3:
        {
            MatIterator_<Vec3b> it, end;
            for( it = I.begin<Vec3b>(), end = I.end<Vec3b>(); it != end; ++it)
            {
                (*it)[0] = table[(*it)[0]];
                (*it)[1] = table[(*it)[1]];
                (*it)[2] = table[(*it)[2]];
            }
        }
    }
    return I;
}
```

#### 3. On-the-fly address calculation 即时计算每个像素的地址

另一种方法是，我们直接使用前面讲述的获取每个像素的函数`at()`，获取每个像素点的位置指针就行了。但是，这种方法通常**不推荐**，因为它通常用于随机获取任意位置的像素。

```C++
Mat& ScanImageAndReduceRandomAccess(Mat& I, const uchar* const table)
{
    // accept only char type matrices
    CV_Assert(I.depth() == CV_8U);
    const int channels = I.channels();
    switch(channels)
    {
    case 1:
        {
            for( int i = 0; i < I.rows; ++i)
                for( int j = 0; j < I.cols; ++j )
                    I.at<uchar>(i,j) = table[I.at<uchar>(i,j)]; // 和前一种方法唯一的不同之处
            break;
        }
    case 3:
        {
        // 下面使用了Mat_类型，它是一种Mat的指针类型。使用它的好处是，在循环中就不用每次都要显式的像I.at<Vec3b>这样指定类型了。当然它的速度其实是和直接使用.at<Vec3b>一模一样，因此它的好处只是让代码更简洁一点而已。
         Mat_<Vec3b> _I = I;
         for( int i = 0; i < I.rows; ++i)
            for( int j = 0; j < I.cols; ++j )
               {
                   _I(i,j)[0] = table[_I(i,j)[0]];
                   _I(i,j)[1] = table[_I(i,j)[1]];
                   _I(i,j)[2] = table[_I(i,j)[2]];
            }
         I = _I;
         break;
        }
    }

    return I;
}
```

#### 4. The Core Function 直接使用OpenCV相关函数

OpenCV定义了大量的和图片处理相关的函数供使用。因此，如果满足需求的话，可以直接使用这些函数，而不用再遍历整张图片每个像素了。例如，上面的例子所做的事情都是遍历整张图片，并将每个像素的每个通道的颜色值通过table来in-place得映射到另一个值。即，输入的table充当的是look-up table的作用。而OpenCV中已经定义了一个LUT()的函数来做这件事情（参见：[https://docs.opencv.org/2.4/modules/core/doc/operations_on_arrays.html#lut](https://docs.opencv.org/2.4/modules/core/doc/operations_on_arrays.html#lut)。实验结果表明，直接使用LUT()是本章这些方法中最快的。这是因为，OpenCV中定义的函数通常使用了基于Intel TBB（Threaded Building Blocks）的多线程以加速。

```C++
Mat& ScanImageAndReduceCoreFunction(Mat& I, const uchar* const table)
{
    // 定义look-up table变量是一个1x256的矩阵
    Mat lookUpTable(1, 256, CV_8U);
    uchar* p = lookUpTable.data;
    for( int i = 0; i < 256; ++i)
        p[i] = table[i]; // 因为lookUpTable只有一行，因此直接赋值即可
    cv::Mat J = I.clone();
    cv::LUT(I, lookUpTable, J); // 调用已定义的函数，J是输出
    return J;
}
```

#### Summary 用法总结

在实现中，如果你的需求可以直接使用OpenCV的函数实现，那么直接使用它即可。显然它是最快并且基本上是最安全的方法。否则，尽可能使用上面第1中方法，即基于C指针的快速法。

## Camera Calibration

https://docs.opencv.org/2.4/doc/tutorials/calib3d/camera_calibration/camera_calibration.html

<!-- 添加Latex格式的公式 -->
$$
\begin{aligned}
\mathbf{b} = \begin{bmatrix}
     a & 1 \\
     b & 2
\end{bmatrix}
\tag{1.1}
\end{aligned}
$$ 