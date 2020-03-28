
参考链接 [https://www.zhihu.com/question/306213462](https://www.zhihu.com/question/306213462)

作者：Hengkai Guo
链接：https://www.zhihu.com/question/306213462/answer/562776112
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

对于两路输入来说，如果是通道数相同且后面带卷积的话，add等价于concat之后对应通道共享同一个卷积核。下面具体用式子解释一下。由于每个输出通道的卷积核是独立的，我们可以只看单个通道的输出。假设两路输入的通道分别为X1, X2, ..., Xc和Y1, Y2, ..., Yc。那么concat的单个输出通道为（*表示卷积）：

<img src='https://www.zhihu.com/equation?tex=Z_%7Bconcat%7D+%3D+%5Csum_%7Bi%3D1%7D%5E%7Bc%7D%7BX_i%7D%5Cast%7BK_i%7D%2B+%5Csum_%7Bi%3D1%7D%5E%7Bc%7D%7BY_i%7D%5Cast%7BK_%7Bi%2Bc%7D%7D'></img>

而add的单个输出通道为：

<img src='https://www.zhihu.com/equation?tex=Z_%7Badd%7D+%3D+%5Csum_%7Bi%3D1%7D%5E%7Bc%7D%28%7BX_i+%2B+Y_i%7D%29%5Cast%7BK_i%7D+%3D+%5Csum_%7Bi%3D1%7D%5E%7Bc%7D%7BX_i%7D%5Cast%7BK_i%7D+%2B+%5Csum_%7Bi%3D1%7D%5E%7Bc%7D%7BY_i%7D%5Cast%7BK_i%7D'>



因此add相当于加了一种prior，当两路输入可以具有“对应通道的特征图语义类似”（可能不太严谨）的性质的时候，可以用add来替代concat，这样更节省参数和计算量（concat是add的2倍）。FPN[1]里的金字塔，是希望把分辨率最小但语义最强的特征图增加分辨率，从性质上是可以用add的。如果用concat，因为分辨率小的特征通道数更多，计算量是一笔不少的开销。<img src="https://pic2.zhimg.com/50/v2-96612574601b602f8a247bd926d22877_hd.jpg" data-caption="" data-size="normal" data-rawwidth="918" data-rawheight="794" data-default-watermark-src="https://pic3.zhimg.com/50/v2-ad27e34208aff216c362dfc0541b0f4d_hd.jpg" class="origin_image zh-lightbox-thumb" width="918" data-original="https://pic2.zhimg.com/v2-96612574601b602f8a247bd926d22877_r.jpg"/>CPN[2]为了进一步减少计算量，对于分辨率小的特征图在add前用1x1的卷积减少了通道数。<img src="https://pic2.zhimg.com/50/v2-05ff5c3e24b56333d131b7156dbfc96f_hd.jpg" data-caption="" data-size="normal" data-rawwidth="1588" data-rawheight="600" data-default-watermark-src="https://pic1.zhimg.com/50/v2-241f40aa6a626e6b1141a472813eb5e8_hd.jpg" class="origin_image zh-lightbox-thumb" width="1588" data-original="https://pic2.zhimg.com/v2-05ff5c3e24b56333d131b7156dbfc96f_r.jpg"/>又比如edge detection[3]的工作，里面不同层输出的edge map也通过weighted add融合在一起，这是因为这些输出的语义本来就是相同的，都用了label的loss来约束。<img src="https://pic1.zhimg.com/50/v2-894c24f97cfb43db8dadabd3e51f85df_hd.jpg" data-caption="" data-size="normal" data-rawwidth="1804" data-rawheight="1314" data-default-watermark-src="https://pic4.zhimg.com/50/v2-294682f5bf7ada47193f2d0e4083dd04_hd.jpg" class="origin_image zh-lightbox-thumb" width="1804" data-original="https://pic1.zhimg.com/v2-894c24f97cfb43db8dadabd3e51f85df_r.jpg"/>还有一个例子是ResNet[4]的skip connection。这里的add主要是为了保持mapping的identity性质，使梯度回传得更加容易。同样的操作在LSTM[5]里的cell state也能看到。<img src="https://pic1.zhimg.com/50/v2-3acafc808b7ef5ecfed2b43ef41d60d2_hd.jpg" data-caption="" data-size="normal" data-rawwidth="226" data-rawheight="472" data-default-watermark-src="https://pic3.zhimg.com/50/v2-8e11912b342095bfb02899f312e0bad2_hd.jpg" class="content_image" width="226"/>当然，如果不在乎计算量且数据足够的时候，用concat也是可以的，因为这两个本身就是包含关系。实际上concat在skip connection里用的也比add更普遍，比如题主提到的U-Net[6]、DenseNet[7]。<img src="https://pic3.zhimg.com/50/v2-cf0357503edb8414d1081c02ec2faa8a_hd.jpg" data-caption="" data-size="normal" data-rawwidth="1282" data-rawheight="920" data-default-watermark-src="https://pic2.zhimg.com/50/v2-90578c704181487bbd85665d5c271113_hd.jpg" class="origin_image zh-lightbox-thumb" width="1282" data-original="https://pic3.zhimg.com/v2-cf0357503edb8414d1081c02ec2faa8a_r.jpg"/>
