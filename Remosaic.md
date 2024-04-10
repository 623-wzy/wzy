## Remosaic

&emsp;&emsp; **SSIM(structural similarity index)，结构相似性，是一种衡量两幅图像相似度的指标。** 该指标首先由德州大学奥斯丁分校的图像和视频工程实验室(Laboratory for Image and Video Engineering)提出。源自论文《Image Quality Assessment: From Error Visibility to Structural Similarity》。<br>
&emsp;&emsp; 在SSIM被提出之前被广泛应用的是MSE(均方误差)，因为它计算简单，物理意义明确。<br>
&emsp;&emsp; MSE公式： $$MSE = \frac{1}{mn} \sum_{i=0}^{m-1} \sum_{j=0}^{n-1} [I(i,j) - K(i,j)]^{2}$$ 
&emsp;&emsp; 下图可以看出，MSE 反应的距离和人力的直观感受有很大差别。
### Quadra Bayer
&emsp;&emsp; 手机越做越紧凑需要模组和芯片尺寸越做越小，在尺寸一定的基础上，高像素和大像素，对于手机摄像头来说，一直是一对矛盾的存在。然而,**高像素所带来的高分辨率画质，和大像素带给暗态高感度低噪声的画质，两者都非常重要**。为了两者兼得，4cell1芯片应运而生，也有称之为 “Quad bayer”、“Tetra cell”、“Four cell”等，该芯片基于经典的Bayer阵列，将每一种颜色以4个pixel组合排列，成功让一款摄像头能在高像素和大像素间自由切换。
