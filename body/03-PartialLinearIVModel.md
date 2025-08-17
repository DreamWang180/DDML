### 部分线性工具变量（IV）模型

#### 准备工作

我们加载数据、定义全局宏并设置种子。

```stata
. use https://statalasso.github.io/dta/AJR.dta, clear
. global Y logpgp95
. global D avexpr
. global Z logem4
. global X lat_abst edes1975 avelf temp* humid* steplow-oilres
. set seed 42
```

#### 步骤 1: 初始化

由于数据集非常小，我们考虑使用 30 个交叉拟合折叠。

```stata
. ddml init iv, kfolds(30)
```

#### 步骤 2: 添加学习器

部分线性 IV 模型有三个条件期望：

* $E[Y \mid X]$
* $E[D \mid X]$
* $E[Z \mid X]$

对于每个简化方程，我们添加两个学习器：`regress` 和 `rforest`。

由于 `rforest` 的 `predict` 命令不支持变量类型，因此需要为 `rforest` 添加选项 `vtype(none)`。

```stata
. ddml E[Y|X]: reg $Y $X
Learner Y1_reg added successfully.
. ddml E[Y|X], vtype(none): rforest $Y $X, type(reg)
Learner Y2_rforest added successfully.
. ddml E[D|X]: reg $D $X
Learner D1_reg added successfully.
. ddml E[D|X], vtype(none): rforest $D $X, type(reg)
Learner D2_rforest added successfully.
. ddml E[Z|X]: reg $Z $X
Learner Z1_reg added successfully.
. ddml E[Z|X], vtype(none): rforest $Z $X, type(reg)
Learner Z2_rforest added successfully.
```

#### 步骤 3/4: 交叉拟合与估计

我们使用 `shortstack` 选项来组合基础学习器。简短堆叠是比堆叠更节省计算资源的替代方法。堆叠依赖于交叉验证的预测值来获得基础学习器的相对权重，而简短堆叠则使用交叉拟合的预测值。

```stata
. qui ddml crossfit
. ddml estimate, robust
```

**DDML 估计结果：**

```markdown
spec  r     Y learner     D learner         b        SE     Z learner
 opt  1    Y2_rforest    D2_rforest     0.772  ( 0.207)              
  ss  1  [shortstack]          [ss]     0.716  ( 0.196)          [ss]
opt = minimum MSE specification for that resample.

Shortstack DDML model
y-E[y|X]  = logpgp95_ss_1                          Number of obs   =        64
D-E[D|X,Z]= avexpr_ss_1
Z-E[Z|X]  = logem4_ss_1
------------------------------------------------------------------------------
             |               Robust
    logpgp95 | Coefficient  std. err.      z    P>|z|     [95% conf. interval]
-------------+----------------------------------------------------------------
      avexpr |   .7158468   .1958356     3.66   0.000     .3320162    1.099677
       _cons |  -.0308525   .0914993    -0.34   0.736    -.2101878    .1484828
------------------------------------------------------------------------------
```

#### 手动估计

如果您想知道 `ddml` 在后台执行了什么，可以使用以下命令：

```stata
. ddml estimate, allcombos spec(8) rep(1) robust
```

**DDML 估计结果：**

```markdown
spec  r     Y learner     D learner         b        SE     Z learner
   1  1        Y1_reg        D1_reg     0.378  ( 0.125)        Z1_reg
   2  1        Y1_reg        D1_reg    -0.187  ( 1.573)    Z2_rforest
   3  1        Y1_reg    D2_rforest     2.413  ( 3.594)        Z1_reg
   4  1        Y1_reg    D2_rforest     0.083  ( 0.475)    Z2_rforest
   5  1    Y2_rforest        D1_reg     0.123  ( 0.207)        Z1_reg
   6  1    Y2_rforest        D1_reg    -1.749  ( 4.690)    Z2_rforest
   7  1    Y2_rforest    D2_rforest     0.783  ( 0.504)        Z1_reg
*  8  1    Y2_rforest    D2_rforest     0.772  ( 0.207)    Z2_rforest
  ss  1  [shortstack]          [ss]     0.716  ( 0.196)          [ss]
* = minimum MSE specification for that resample.

Min MSE DDML model, specification 8
y-E[y|X]  = Y2_rforest_1                           Number of obs   =        64
D-E[D|X,Z]= D2_rforest_1
Z-E[Z|X]  = Z2_rforest_1
------------------------------------------------------------------------------
             |               Robust
    logpgp95 | Coefficient  std. err.      z    P>|z|     [95% conf. interval]
-------------+----------------------------------------------------------------
      avexpr |    .772314   .2068282     3.73   0.000     .3669382     1.17769
       _cons |  -.0119092   .1009289    -0.12   0.906    -.2097263    .1859079
------------------------------------------------------------------------------
```

#### 工具变量回归

```stata
. ivreg Y2_rf (D2_rf = Z2_rf), robust
```

**工具变量 2SLS 回归**

```markdown
Instrumental variables 2SLS regression          Number of obs     =         64
                                                F(1, 62)          =      13.94
                                                Prob > F          =     0.0004
                                                R-squared         =          .
                                                Root MSE          =     .80209

------------------------------------------------------------------------------
             |               Robust
Y2_rforest_1 | Coefficient  std. err.      t    P>|t|     [95% conf. interval]
-------------+----------------------------------------------------------------
D2_rforest_1 |    .772314   .2068282     3.73   0.000     .3588703    1.185758
       _cons |  -.0119092   .1009289    -0.12   0.906    -.2136633    .1898448
------------------------------------------------------------------------------
Instrumented: D2_rforest_1
 Instruments: Z2_rforest_1
```