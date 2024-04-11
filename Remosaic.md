## Remosaic
### Quadra Bayer
&emsp;&emsp; 手机越做越紧凑需要模组和芯片尺寸越做越小，在尺寸一定的基础上，高像素和大像素，对于手机摄像头来说，一直是一对矛盾的存在。然而,**高像素所带来的高分辨率画质，和大像素带给暗态高感度低噪声的画质，两者都非常重要**。为了两者兼得，4cell1芯片应运而生，也有称之为 “Quad bayer”、“Tetra cell”、“Four cell”等，该芯片基于经典的Bayer阵列，将每一种颜色以4个pixel组合排列，成功让一款摄像头能在高像素和大像素间自由切换。<br>

<div align=center>
<img src="https://github.com/623-wzy/wzy/blob/main/image/v2-9539b4f783f2191b5d5fc7e3cc5eb153_1440w.webp"/>
</div>

<div align=center>
<img src="https://github.com/623-wzy/wzy/blob/main/image/v2-d2e863ea5c44e8b5d5677892df05f1fb_1440w.webp"/>
</div>

&emsp;&emsp; **那么实际应用时，什么情况下采用Remosaic方式，什么情况下采用Quard Bayer模式呢？** <br>
&emsp;&emsp; 一般情况下终端会根据ISO来判断采用Quard Bayer模式下拍摄的图片的输出。比如ISO200以下（定义为亮环境），sensor以高像素REMOSAIC以后输出，ISO200以上（定义为暗环境），则以4cell后低像素输出，再通过平台的插值的方法得到高分辨率图像。<br>

### Remosaic

&emsp;&emsp; **Remosaic：** 还原高分辨率图像。从原始的4cell1像素排布，还原成普通拜耳（Bayer）结构的过程，称之为Remosaic。<br>

<div align=center>
<img src="https://github.com/623-wzy/wzy/blob/main/image/v2-b887f04ae680b7bbcb366dab0201fa15_1440w.webp"/>
</div>

&emsp;&emsp; **Remosaic通常分为软件和硬件两种方式。** 软件Remosaic，通过像素互换，或该像素与周围相关像素的联系，根据距离远近计算出一定的权重比例，作为该像素的信号值，通常软件Remosaic算法放在平台端集成。目前，也有部分芯片，通过独立的ISP信号处理变换像素结构，每个感光单元又都能独立显示并且输出数据，可以拍摄出正常硬件直出的Bayer排布，无需额外软件插值。当然二者从速度上来看相差甚远，硬件比软件的Remosaic在处理速度上会快很多，硬件Remosaic可以支持Full size Bayer预览，然而手机端是否要用full size去预览还需要综合考虑功耗等其他因素；软件Remosaic处理需要花费更长的时间，目前仅作为Full Size拍照时候使用。各厂商的remosaic算法各不相同。Sony率先实现了on-sensor remosaic，它之前其它sensor都是在host端软件来进行remosaic操作，必然是只支持静态照相模式，后来各sensor厂也都有了on sensorremosaic，到现在ISP soc上也有了硬件的remosaic。<br>
&emsp;&emsp; remosaic算法同样很复杂，最简单的一个方法，把左图中相邻的R，G，B放到右图中，实现remosaic，可以想见，如果有一条细线正好在最左边的两个R的位置，经过Remosaic之后，这条细线就会在第三个像素的位置出现。除此之外，如果R2被这种remosaic copy到R3之后，所带来的除了空间采样位置的误差之外，它也不能准确代表R3位置的R的色彩。<br>

<div align=center>
<img src="https://github.com/623-wzy/wzy/blob/main/image/640.webp"/>
</div>

&emsp;&emsp; 假设我们把这颗Quad sensor的color filter替换成bayer pattern，做一个‘物理上’的remosaic，与用上边remosaic算法得到的bayer sensor做直接的比较，会发现由于光学与物理crosstalk等的影响，两个RAW图存在着不小的差异，这就导致图像会出现各种瑕疵，完全没有上面样片那样美好。从图像质量的角度来说，主要有以下问题：<br>

- 伪影:本身树枝之间应该是干净的天空，但是由于remosaic造成了伪影。<br>

<div align=center>
<img src="https://github.com/623-wzy/wzy/blob/main/image/640%20(1).webp"/>
</div>

- 高饱和色降饱和:本身应该是连续的黄色的草丛，却出现色彩降饱和，呈现灰色。由于黄色饱和度高，所以比较容易被察觉饱和度的下降。<br>

<div align=center>
<img src="https://github.com/623-wzy/wzy/blob/main/image/640%20(2).webp"/>
</div>

- 锯齿线断裂:稍微有些倾斜的直线，很容易出现这种因为remosaic造成的锯齿或者断续。<br>

<div align=center>
<img src="https://github.com/623-wzy/wzy/blob/main/image/640%20(3).webp"/>
</div>

- 拐角伪点:在高频的拐点，很容易出现伪点，这在demosaic和DPC算法里也容易出现，由于对线条纹理的方向判断错误，错误的插值导致的伪点。<br>

<div align=center>
<img src="https://github.com/623-wzy/wzy/blob/main/image/640%20(4).webp"/>
</div>

<div align=center>
<img src="https://github.com/623-wzy/wzy/blob/main/image/640%20(5).webp"/>
</div>

- 伪彩:可以看到树干和枯草的高频区域出现一些彩色的artefact。<br>

<div align=center>
<img src="https://github.com/623-wzy/wzy/blob/main/image/640%20(6).webp"/>
</div>

&emsp;&emsp; 现在的remosaic算法也逐渐从传统的信号处理方法过渡到基于机器学习/深度学习的图像处理方法。传统的remosaic算法流程大致分为三步：1.首先利用Quadra CFA的插值算法，转换Quad Bayer Raw数据变为RGB数据，类似我们平时看到的Jpeg 2.得到的RGB Image再使用RGB Image转Bayer的算法，将其分解成Bayer Raw Image 3.送回正常ISP进行下一步处理。<br>
&emsp;&emsp; 利用传统的图像质量测量方法，可以比较各remosaic算法的优劣。测试Chart可以选择Image Engineering 的all in one chart。高频的细节，直线，斜边，各种颜色在这个chart里都有，所以可以满足客观评价remosaic的需求。<br>

<div align=center>
<img src="https://github.com/623-wzy/wzy/blob/main/image/640%20(7).webp"/>
</div>

&emsp;&emsp; 各IQ Metrics计算： $(I是待测图像，\hat{I} 是真实图像)$ <br>

### Crosstalk补偿（QPDC）
&emsp;&emsp; 在整个转换过程中，还会涉及到PD补偿、坏点补偿和Crosstalk补偿等一些优化，主要是优化可能在remosaic转换中产生的False Color/Color Noise/解析度丢失等常见问题。对于Quadra CFA的芯片像素，由于所处位置的不同，实际上每个像素点的crosstalk不同导致感光能力有一定差别，通常就需要引入Crosstalk校准来消除这种差异。Crosstalk校准工具，通常将全图分成多个ROI方块，计算各像素通道的能量并确定其补偿数据，芯片再使用这些校准数据让原本不均匀状态的能量分布变得更为平衡。Crosstalk校准补偿在实际图像上的反映主要是去除由于信号差别造成的格子，锯齿状等的色块干扰，这种干扰现象通常在拍摄单一色块的画面会表现得更为明显。
