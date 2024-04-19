## VST
### VST (variance stabilizing transformation)
&emsp;&emsp; 在实际的成像系统中，噪声的分布特性是非常复杂的。对于相机传感器直接输出的原始数据，也称为raw图像，一般情况下噪声用混合Poisson-Gaussian分布进行建模。（在极暗的环境下还会有较为显著的行噪声，具体可以参考CVPR2020的ELD）。<br>
&emsp;&emsp; 假设 $\mathbf{Y}$ , $\mathbf{X}$ 和 $\mathbf{N}$ 分别为含噪的输入图像，对应的干净图像，以及噪声分量，则有： $$\mathbf{Y} \sim \mathcal{N}\left(\mathbf{X}, \boldsymbol{\Sigma}^{2}\right)$$ <br>
其中噪声方差： $$\Sigma^{2} = \alpha x + \sigma ^{2} $$ <br>
参数 $\alpha$ 和 $\sigma$ 分别表示混合噪声中Poisson分量和Gaussian分量的强度。<br>
&emsp;&emsp; 针对这种混合分布噪声，一类处理方法是对噪声进行重新建模，并且根据新建模的噪声设计特定的去噪过程，另一类处理方法是将新建模的噪声过程转换成高斯白噪声，然后采用已经研究过的针对高斯白噪声有效的去噪算法，两种方法各有优劣，但是第二种方法有两个好处，其一直方便模块化，第二是直接利用现有的去噪算法，节约时间和研发成本。这个将特定噪声转换成高斯白噪声的过程我们称之为VST，其反过程是将去噪后的信号转换到原始分布，称之为inverse VST。<br>

<div align=center>
<img src="https://github.com/623-wzy/wzy/blob/main/image/20180809142357833.png"/>
</div>

### AT (Anscombe transform)
##### Anscombe变换
&emsp;&emsp; **Anscombe变换**以Francis Anscombe的名字命名，是一种方差稳定变换，它将具有泊松分布的随机变量变换为具有近似标准高斯分布的随机变量。Anscombe变换广泛应用于光子限制成像（天文学、X 射线），其中图像自然遵循泊松定律。Anscombe变换通常用于对数据进行预处理，以使标准差近似恒定。然后采用针对加性高斯白噪声框架设计的去噪算法；然后通过对去噪数据应用逆安斯科姆变换来获得最终估计。<br>
&emsp;&emsp; 对于泊松分布，平均值 $m$ 与方差 $\nu$ 不独立： $m=\nu$ 。Anscombe变换： $$A：x\mapsto 2\sqrt{x+3/8} $$ <br>
&emsp;&emsp; 旨在变换数据，使方差大约为1，以获得足够大的均值；对于均值为零方差仍为0。它转换泊松数据 $x$ （均值 $m$ ）到均值为 
$2 \sqrt{m+\frac{3}{8}}-\frac{1}{4 m^{1 / 2}}+O\left(\frac{1}{m^{3 / 2}}\right)$ 和 标准差为 $1+O\left(\frac{1}{m^{2}}\right)$ 的近似高斯数据。对于m比较大的情况，这种近似会更加准确。至于为何选择常数项为3/8，是因为 $2\sqrt{x+c} $ 的方差变换形式会有一个附加项为 $\frac{\frac{3}{8} -c}{m}$ 。<br>

<div align=center>
<img src="https://github.com/623-wzy/wzy/blob/main/image/20181025105803813.png"/>
</div>

&emsp;&emsp; Anscombe变换动画。 $\mu$ 是Anscombe变换的泊松分布的平均值，并且通过减去 $2\sqrt{m+\frac{3}{8}}-\frac{1}{4 m^{1 / 2}}$ 进行标准化， $\sigma$ 是其标准差。可以发现随着m变化， $m^{\frac{3}{2} } \mu 和m^{2}(\sigma -1)$ 大致保持[0,10]之间的波动，进一步支持了Anscombe变换后均值和方差的估计。<br>

<div align=center>
<img src="https://github.com/623-wzy/wzy/blob/main/image/Anscombe_transform_animated.gif"/>
</div>

#### Anscombe逆变换
&emsp;&emsp; 当Anscombe变换应用于降噪时，还需要对处理后数据进行**Anscombe逆变换**以返回方差稳定的降噪后数据。Anscombe变换的代数逆形式： $$A^{-1} ：y\mapsto (\frac{y}{2})^{2}-\frac{3}{8} $$ <br>
&emsp;&emsp; 由于平方根是非线性变换，代数逆形式往往会给Anscombe变换的平均值m的估计带来预期外的偏差。为了消除偏差影响：<br>
&emsp;&emsp; **渐进无偏逆：** $$y\mapsto (\frac{y}{2})^{2}-\frac{1}{8} $$ <br>
&emsp;&emsp; **渐进无偏逆虽然减轻了偏差问题，但不适用光子限制成像，隐式映射提供了精确无偏逆：** $$\mathrm{E}\left[\left.2 \sqrt{x+\frac{3}{8}} \right\rvert\, m\right]=2 \sum_{x=0}^{+\infty}\left(\sqrt{x+\frac{3}{8}} \cdot \frac{m^{x} e^{-m}}{x !}\right) \mapsto m$$ <br>
&emsp;&emsp; **精确无偏逆的封闭形式近似：** $$y\mapsto \frac{1}{4} y^{2} - \frac{1}{8} + \frac{1}{4}\sqrt{\frac{3}{2}} y^{-1} - \frac{11}{8} y^{-2} + \frac{5}{8} \sqrt{\frac{3}{2}} y^{-3} $$ <br>
### GAT (Generalized Anscombe transform)
&emsp;&emsp; 广义Anscombe变换表示为： $$f(\hat{x} )\begin{cases}
\frac{2}{a} \sqrt{a\hat{x} + \frac{3}{8} a^{2} + \hat{\sigma} - am}   & \text x> -\frac{3}{8} a - \frac{\hat{\sigma}}{a} + m \\
0  & \text x\le -\frac{3}{8} a - \frac{\hat{\sigma}}{a} + m
\end{cases}$$ <br>
当 $a=1，\sigma=0，m=0$ 时，泊松高斯分布退化为泊松分布，GAT也退化为AT。对上述公式归一化： $$x=\frac{\hat{x}-m}{a}，\sigma=\frac{\hat{\sigma}}{a}$$ <br>
&emsp;&emsp; 广义Anscombe逆变换表示为： $$ y\mapsto \frac{1}{4} y^{2} - \frac{1}{8} + \frac{1}{4}\sqrt{\frac{3}{2}} y^{-1} - \frac{11}{8} y^{-2} + \frac{5}{8} \sqrt{\frac{3}{2}} y^{-3} - \sigma^{2} , \hat{y} =a * \hat{y}+m ，\sigma = \frac{\hat{\sigma}}{a}$$ <br>

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
3.[VST-wiki](https://en.wikipedia.org/wiki/Variance-stabilizing_transformation) <br>
4.[Anscombe-wiki](https://en.wikipedia.org/wiki/Anscombe_transform#cite_note-Anscombe1948-1) <br>

