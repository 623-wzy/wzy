## VST
### 一、噪声模型
&emsp;&emsp; 在实际的成像系统中，噪声的分布特性是非常复杂的。对于相机传感器直接输出的原始数据，也称为raw图像，一般情况下噪声用混合Poisson-Gaussian分布进行建模。（在极暗的环境下还会有较为显著的行噪声，具体可以参考CVPR2020的ELD）。
假设 $\mathbf{Y}$ , $\mathbf{X}$ 和 $\mathbf{N}$ 分别为含噪的输入图像，对应的干净图像，以及噪声分量，则有： $$\mathbf{Y} \sim \mathcal{N}\left(\mathbf{X}, \boldsymbol{\Sigma}^{2}\right)$$ <br>

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
