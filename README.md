# Multi-Thread do you know how much?
**为什么要使用多线程? 学习多线程的目的: 将耗时操作放到后台去执行,   这也是学习多线程最主要的目的!   
那么怎样能看出哪些操作是耗时较多的呢,这里我们就来模拟一下内存几个区不同的耗时情况:**

假设有一个新闻类的app，如果我们按照在UI阶段的方法，使用plist加载本地数据，那么这个app上的数据都是死的，用户看来看去都是固定死的几条“新闻”，最终的结果就是，用户会删掉这个app。
    没有数据的app犹如一潭死水，没有生机！那么，怎么来实时地获取数据呢？只有通过网络从远程服务器的数据库中获取实时数据。这样我们的app才能够保持活力！
    但是，从网络上获取数据的时候会存在一个问题，比方说：下载一个小电影，通常是比较消耗时间的。也就是说，从网络上获取数据的操作属于耗时操作。
    那么，耗时操作会对我们的app产生什么影响呢？给大家提示一下，既然我们现在要学习多线程，那就说明我们之前写的所有代码都是在单线程上执行的。这里给大家举个例子，过河，如果把河上的桥比作线程，那么我们之前都是在走独木桥。走独木桥有什么特点呢？假设有10个人要过河，但是第1个人张三跟人打架，腿瘸了，那么张三过桥就会非常墨迹。后面的人想过桥，没门，必须等张三过去了，后面的人才可以过河。
这个例子放到程序里面，就是网络操作比较耗时，如果网络操作没有执行完毕，用户的其它操作就会被阻塞，结果就是用户会感到非常卡顿，然后就是各种删删删了。
而多线程就是专门用来解决这种问题的！

所以在引入多线程之前，我们先来做一个模拟耗时操作的演练。

### 1、代码一：循环测试

```
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    // 单线程
    [self demo];
}

#pragma mark - 模拟耗时操作

- (void)demo {
    
    NSLog(@"bengin");
    for (int i = 0; i < 10000000; i++) {
        
    }
    NSLog(@"end");
}

```
打印台输出结果为: 
 

![代码1输出结果](http://upload-images.jianshu.io/upload_images/1483059-14787688d8035193.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


####  通过输出结果可知：循环的速度非常非常快; 仅为**0.025s**




### 2、代码二：操作内存的栈空间


```
-	(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
// 单线程
    [self demo];
}
#pragma mark - 模拟耗时操作
- (void)demo {
    NSLog(@"bengin");
    for (int i = 0; i < 10000000; i++) {
        int n = i;
    }
    NSLog(@"end");
}

```
打印台输出结果为: 

![代码2输出结果](https://static.oschina.net/uploads/img/201602/15181001_xaaO.png "代码2输出结果")

#### 通过输出结果可知：操作内存的栈空间，速度同样非常快。仅为 0.026s



### 3、代码三：操作内存的常量区

```
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    // 单线程
    [self demo];
}
#pragma mark - 模拟耗时操作

- (void)demo {
    
    NSLog(@"bengin");
    for (int i = 0; i < 10000000; i++) {
        // 使用@""定义的字符串保存在常量区
        NSString *str = @"hello";
    }
    NSLog(@"end");
}

```
打印台输出结果为: 

![代码3输出结果](https://static.oschina.net/uploads/img/201602/15181236_exUF.png "在这里输入图片标题")
#### 通过输出结果可知： 操作内存的常量区, 速度比较快(比操作栈区稍微慢点) 0.099s



### 4、代码四：操作内存的堆空间

```
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    // 单线程
    [self demo];
}
#pragma mark - 模拟耗时操作

- (void)demo {
    NSLog(@"bengin");
    for (int i = 0; i < 10000000; i++) {
        // 使用 stringWithFormat 拼接的字符串保存在堆区
        NSString *str = [NSString stringWithFormat:@"hello - %d", i];
    }
    NSLog(@"end");
}

```
打印台输出结果为: 

![代码4输出结果](https://static.oschina.net/uploads/img/201602/15181428_mu5G.png "在这里输入图片标题")
#### 通过输出结果可知：操作内存的堆空间，速度比操作常量区慢；循环非常消耗CPU资源: 时间为10.597s
    

### 5、代码五：I/O操作

```
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    // 单线程
    [self demo];
}
#pragma mark - 模拟耗时操作

- (void)demo {
    NSLog(@"bengin");
    for (int i = 0; i < 10000000; i++) {
        // I/O操作
        NSLog(@"%d", i);
    }
    NSLog(@"end");
}

```
打印台输出结果为: 

![代码5输出结果](https://static.oschina.net/uploads/img/201602/15181629_kwaC.png "在这里输入图片标题")

#### 从输出结果可知：I/O操作，速度非常慢。



### 6、代码六：引入多线程技术

```
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    // 多线程
    [self performSelectorInBackground:@selector(demo) withObject:nil];
}
#pragma mark - 模拟耗时操作
- (void)demo {
    
    NSLog(@"bengin");
    for (int i = 0; i < 10000000; i++) {
        // I/O操作
        NSLog(@"%d", i);
    }
    NSLog(@"end");
}

```
打印台输出结果为:

![代码6输出结果](https://static.oschina.net/uploads/img/201602/15182143_VR2q.png "在这里输入图片标题")

#### 由输出结果可知：引入多线程技术之后，即便是I/O操作这种耗时操作，也不会造成程序卡顿。

### 7、小结与思考
#### 小结：
##### (1) 耗时操作的后果：如果只有主线程，会造成程序卡顿，用户体验极差。
##### (2) 学习多线程的目的：将耗时操作放到后台线程去执行。
##### (3) 通过耗时操作演练可知，操作效率的顺序：
 I/O操作 < 堆区 < 常量区 < 栈区。
##### (4) 使用@””定义的字符串保存在常量区，使用stringWithFormat拼接的字符串保存在堆区。
##### (5) 网络操作也属于耗时操作，通过多线程技术可以将耗时的网络操作放到后台线程去执行，从而提高程序执行效率，改善用户体验。
##### (6) - (void)performSelectorInBackground:(SEL)aSelector withObject:(nullable id)arg  在后台执行某方法。

#### 思考：
#####（1）耗时操作会对我们的应用程序产生什么影响？
 耗时操作的后果：在主线程，耗时操作会造成程序卡顿，用户会以为程序死了，用户体验极差。
#####（2）耗时操作造成的程序卡顿问题该怎么解决？
要想解决程序卡顿问题，就需要使用多线程技术，将耗时操作放到子线程去执行。


综上所述,就可以看出多线程在我们实际开发中,是多么的重要!!!
