## SSIM

&emsp;&emsp; **SSIM(structural similarity index)，结构相似性，是一种衡量两幅图像相似度的指标。** 该指标首先由德州大学奥斯丁分校的图像和视频工程实验室(Laboratory for Image and Video Engineering)提出。源自论文《[Image Quality Assessment: From Error Visibility to Structural Similarity](https://github.com/623-wzy/wzy/blob/main/paper/Image%20Quality%20Assessment%20%20From%20Error%20Visibility%20to%20Structural%20Similarity.pdf)》。<br>
&emsp;&emsp; 在SSIM被提出之前被广泛应用的是MSE(均方误差)，因为它计算简单，物理意义明确。MSE公式： $$MSE = \frac{1}{mn} \sum_{i=0}^{m-1} \sum_{j=0}^{n-1} [I(i,j) - K(i,j)]^{2}$$ <br>
&emsp;&emsp; 下图可以看出，MSE 反应的距离和人力的直观感受有很大差别。
<div align=center>
<img src="https://github.com/623-wzy/wzy/blob/main/image/5.jpeg"/>
</div>
&emsp;&emsp; 由于MSE不能表达人的视觉系统对图片的直观感受（涉及生物学），因此作者提出了更为科学的SSIM 评价指标。SSIM 更侧重于两图的结构相似性，而不是逐像素计算亮度的差异。<br>
&emsp;&emsp; 作者将两幅图的相似性比较拆成了三个维度：亮度（luminance） $l(x,y)$ ，对比度（contrast） $c(x,y)$ ，结构（structure） $s(x,y)$ 。<br>
&emsp;&emsp; 最终x和y的相似度为这三者的函数：$$ S\left ( x,y \right ) = f\left ( l\left ( x,y \right ) ,c\left ( x,y \right ) ,s\left ( x,y \right )  \right )$$ <br>
&emsp;&emsp; 作者设计的三个公式定量计算三者的相似性，公示的设计遵顼三个原则：<br>
&emsp;&emsp; -对称性： $S(x,y)=S(y,x)$ <br>
&emsp;&emsp; -有界性： $S(x,y)<=1$ <br>
&emsp;&emsp; -极限值唯一： $S(x,y)=1 当且仅当x=y$ <br>
&emsp;&emsp; 给定两个图像 x和y ，一张为未经压缩的无失真图像，另一张为失真后的图像，两张图像的结构相似性可按照以下方式求出
<div align=center>
<img src="https://github.com/623-wzy/wzy/blob/main/image/1.png"/>
</div>
&emsp;&emsp; 其中， $\mu_{X}$ 、 $\mu_{Y}$ 分别表示图像 $X$ 和 $Y$ 的均值， $\sigma_{X}$ 、 $\sigma_{Y}$ 分别表示图像 $X$ 和 $Y$ 的方差， $\sigma_{X}Y$ 表示图像 $X$ 和 $Y$ 的协方差。<br>
&emsp;&emsp; SSIM分别从亮度、对比度、结构三方面度量图像相似性。
<div align=center>
<img src="https://github.com/623-wzy/wzy/blob/main/image/2.png"/>
</div>
&emsp;&emsp; C1、C2、C3为常数，为了避免分母为0的情况，通常取 $C1 = (K1L)^{2}, C2=(K2 ∗ L)^{2}, C3 = C2/2$ , 一般地K1=0.01, K2=0.03, L是灰度的动态范围，和图像数据的类型有关，如果是uint8 类型则L=255，如果是float则L=1。<br>
&emsp;&emsp; 上面的公式不能应用于整幅图，因为在整幅图的跨度上均值和方差往往变化剧烈。作者采用 sliding window 以步长为 1 计算两幅图各个对应 sliding window 下的 patch 的 SSIM，然后取平均值作为两幅图整体的 SSIM，称为 Mean SSIM。简写为 MSSIM。
假如整幅图有 M 个 patch，那么 MSSIM 公式为： $$MSSIM(X,Y)=\frac{1}{M} \sum_{i=1}^{M}SSIM(x_{i},y_{i}) $$ <br>
使用MSSIM 计算相似度：<br>
<div align=center>
<img src="https://github.com/623-wzy/wzy/blob/main/image/6.jpeg"/>
</div>

##### 参考文献
[SSIM 的原理和代码实现](https://cloud.tencent.com/developer/article/1438942)


