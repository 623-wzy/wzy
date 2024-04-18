## VST
### 一、噪声模型
&emsp;&emsp; 在实际的成像系统中，噪声的分布特性是非常复杂的。对于相机传感器直接输出的原始数据，也称为raw图像，一般情况下噪声用混合Poisson-Gaussian分布进行建模。（在极暗的环境下还会有较为显著的行噪声，具体可以参考CVPR2020的ELD）。
假设 $\mathbf{Y}$ , $\mathbf{X}$ 和 $\mathbf{N}$ 分别为含噪的输入图像，对应的干净图像，以及噪声分量，则有： $$\mathbf{Y} \sim \mathcal{N}\left(\mathbf{X}, \boldsymbol{\Sigma}^{2}\right)$$ <br>
其中噪声方差： $$\Sigma^{2} = \alpha x + \sigma ^{2} $$ <br>
参数 $\alpha$ 和 $\sigma$ 分别表示混合噪声中Poisson分量和Gaussian分量的强度。<br>
&emsp;&emsp; 在现实的物理过程中，有许多需要去噪的过程并不是仅仅被高斯噪声所干扰，针对这样的过程，一类处理方法是对噪声进行重新建模，并且根据新建模的噪声设计特定的去噪过程，另一类处理方法是将新建模的噪声过程转换成高斯白噪声，然后采用已经研究过的针对高斯白噪声有效的去噪算法，两种方法各有优劣，但是第二种方法有两个好处，其一直方便模块化，第二是直接利用现有的去噪算法，避免重新投入资源尽心开发，节约时间和研发成本。这个将特定噪声转换成高斯白噪声的过程我们称之为VST，其反过程是将去噪后的信号转换到原始分布，称之为inverse VST。上述整体流程如下图所示：

<div align=center>
<img src="https://github.com/623-wzy/wzy/blob/main/image/20180809142357833.png"/>
</div>

```python
def gat(z,sigma,alpha,g):
    _alpha=torch.ones_like(z)*alpha
    _sigma=torch.ones_like(z)*sigma
    z=z/_alpha
    _sigma=_sigma/_alpha
    f=(2.0)*torch.sqrt(torch.max(z+(3.0/8.0)+_sigma**2,torch.zeros_like(z)))
    return f

def inverse_gat(z,sigma1,alpha,g,method='asym'):
   # with torch.no_grad():
    sigma=sigma1/alpha
    if method=='closed_form':
        exact_inverse = ( np.power(z/2.0, 2.0) +
              0.25* np.sqrt(1.5)*np.power(z, -1.0) -
              11.0/8.0 * np.power(z, -2.0) +
              5.0/8.0 * np.sqrt(1.5) * np.power(z, -3.0) -
              1.0/8.0 - sigma**2 )
        exact_inverse=np.maximum(0.0,exact_inverse)
    elif method=='asym':
        exact_inverse=(z/2.0)**2-1.0/8.0-sigma
    else:
        raise NotImplementedError('Only supports the closed-form')
    if alpha !=1:
        exact_inverse*=alpha
    if g!=0:
        exact_inverse+=g
    return exact_inverse
```
##### 参考文献
1.[FBI-Denoise(code)](https://github.com/csm9493/FBI-Denoiser) <br>
2.[FBI-Denoise(知乎)](https://zhuanlan.zhihu.com/p/435957028) <br>
