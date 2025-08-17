### 交互式模型

#### 准备

我们加载数据、定义全局宏并设置种子。

```stata
. webuse cattaneo2, clear
(Excerpt from Cattaneo (2010) Journal of Econometrics 155: 138–154)
. global Y bweight
. global D mbsmoke
. global X mage prenatal1 mmarried fbaby mage medu
. set seed 42
```

#### 步骤 1: 初始化

我们使用 5 个折叠和 5 次重采样；也就是说，我们使用随机选择的折叠估计模型 5 次。

```stata
. ddml init interactive, kfolds(5) reps(5)
```

#### 步骤 2: 添加学习器

我们需要估计以下条件期望：

* $E[Y \mid X, D = 0]$
* $E[Y \mid X, D = 1]$
* $E[D \mid X]$

前两个条件期望一起添加。

我们考虑了两个监督学习器：线性回归和梯度提升树（在 **pystacked** 中实现）。注意，对于 $E[Y \mid X, D]$ 使用梯度提升回归树，而对于 $E[D \mid X]$ 使用梯度提升分类树。

```stata
. ddml E[Y|X,D]: reg $Y $X
Learner Y1_reg added successfully.
. ddml E[Y|X,D]: pystacked $Y $X, type(reg) method(gradboost)
Learner Y2_pystacked added successfully.
. ddml E[D|X]: logit $D $X
Learner D1_logit added successfully.
. ddml E[D|X]: pystacked $D $X, type(class) method(gradboost)
Learner D2_pystacked added successfully.
```

#### 步骤 3: 交叉拟合

```stata
. ddml crossfit
```

#### 步骤 4: 估计

在最终估计步骤中，我们可以估计平均处理效应（默认）或处理组的平均处理效应（atet；输出未显示）。

```stata
. ddml estimate
```

**DDML 估计结果（ATE）**：

```markdown
spec  r    Y0 learner    Y1 learner     D learner         b        SE
 opt  1  Y1_pystacked  Y1_pystacked  D1_pystacked  -219.583  (26.027)
 opt  2  Y1_pystacked  Y1_pystacked  D1_pystacked  -220.967  (25.586)
 opt  3  Y1_pystacked  Y1_pystacked  D1_pystacked  -227.103  (26.256)
 opt  4  Y1_pystacked  Y1_pystacked  D1_pystacked  -221.207  (25.830)
 opt  5  Y1_pystacked  Y1_pystacked  D1_pystacked  -224.497  (25.840)
opt = minimum MSE specification for that resample.

Mean/med.  Y0 learner    Y1 learner     D learner         b        SE
 mse mn     [min-mse]         [mse]         [mse]  -222.672  (26.045)
 mse md     [min-mse]         [mse]         [mse]  -221.207  (26.049)

Median over 5 min-mse specifications (ATE)
E[y|X,D=0]   = Y1_pystacked                        Number of obs   =      4642
E[y|X,D=1]   = Y1_pystacked
E[D|X]       = D1_pystacked
------------------------------------------------------------------------------
             |               Robust
     bweight | Coefficient  std. err.      z    P>|z|     [95% conf. interval]
-------------+----------------------------------------------------------------
     mbsmoke |  -221.2069    26.0486    -8.49   0.000    -272.2612   -170.1526
------------------------------------------------------------------------------
```

**5 次重新抽样的总结**：

```markdown
       D eqn      mean       min       p25       p50       p75       max
     mbsmoke   -222.6716 -227.1032 -224.4971 -221.2069 -220.9675 -219.5831
```

#### 继续估计 ATET：

```stata
. qui ddml estimate, atet
```

请注意，我们已指定 5 次重采样迭代（`reps(5)`）。默认情况下，显示每次重采样迭代的最小 MSE 规格的中位数。底部显示了重采样迭代的摘要统计表。

---

### 简短堆叠

如果我们想使用相同的两个基础学习器，但采用简短堆叠而不是堆叠方法，可以将学习器单独输入，并使用 `shortstack` 选项：

```stata
. set seed 42
. ddml init interactive, kfolds(5) reps(5)
. ddml E[Y|X,D]: reg $Y $X
. ddml E[Y|X,D]: pystacked $Y $X, type(reg) method(gradboost)
. ddml E[D|X]: logit $D $X
. ddml E[D|X]: pystacked $D $X, type(class) method(gradboost)
. ddml crossfit, shortstack
. ddml estimate
```