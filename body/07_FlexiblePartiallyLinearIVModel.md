### 灵活的部分线性工具变量（IV）模型

#### 准备工作

我们加载数据、定义全局宏并设置种子。

```stata
. use https://statalasso.github.io/dta/BLP_CHS.dta, clear
. global Y y
. global D price
. global X hpwt air mpd space
. global Z Zbase*
. set seed 42
```

#### 步骤 1: 初始化

我们初始化模型。

```stata
. ddml init ivhd
```

#### 步骤 2: 添加学习器

我们按照通常的方式添加学习器以估计 $E[Y \mid X]$。

```stata
. ddml E[Y|X]: reg $Y $X
Learner Y1_reg added successfully.

. ddml E[Y|X]: pystacked $Y $X, type(reg)
Learner Y2_pystacked added successfully.
```

在添加学习器以估计 $E[D \mid Z, X]$ 和 $E[D \mid X]$ 时，我们需要注意一些特殊事项。具体来说，估计 $E[D \mid X]$ 依赖于估计 $E[D \mid X, Z]$。更具体地说，我们首先得到拟合值 $\hat{D} = E[D \mid X, Z]$，然后用这些值与 $X$ 进行拟合，来估计 $E[\hat{D} \mid X]$。

在添加学习器以估计 $E[D \mid Z, X]$ 时，我们需要为每个学习器提供名称（例如 `learner(name)`）。

```stata
. ddml E[D|Z,X], learner(Dhat_reg): reg $D $X $Z
Learner Dhat_reg added successfully.

. ddml E[D|Z,X], learner(Dhat_pystacked): pystacked $D $X $Z, type(reg)
Learner Dhat_pystacked added successfully.
```

在添加学习器以估计 $E[D \mid X]$ 时，我们明确引用上一阶段的学习器（例如，`learner(Dhat_reg)`），并提供处理变量的名称（`vname($D)`）。最后，我们用占位符 `{D}` 来代替因变量。

```stata
. ddml E[D|X], learner(Dhat_reg) vname($D): reg {D} $X
Learner Dhat_reg_h added successfully.

. ddml E[D|X], learner(Dhat_pystacked) vname($D): pystacked {D} $X, type(reg)
Replacing existing learner Dhat_pystacked_h...
Learner Dhat_pystacked_h added successfully.
```

#### 步骤 3-4: 交叉拟合与估计

现在，我们可以进行交叉拟合和估计。

```stata
. ddml crossfit
Cross-fitting E[Y|X,Z] equation: y
Cross-fitting fold 1 2 3 4 5 ...completed cross-fitting
Cross-fitting E[D|X,Z] and E[D|X] equation: price
Cross-fitting fold 1 2 3 4 5 ...completed cross-fitting
. ddml estimate, robust
```

**DDML 估计结果：**

```markdown
spec  r     Y learner     D learner         b        SE    DH learner
 opt  1  Y2_pystacked Dhat_pystac~d    -0.098  ( 0.008) Dhat_pystac~h
opt = minimum MSE specification for that resample.

Min MSE DDML model
y-E[y|X]  = Y2_pystacked_1                         Number of obs   =      2217
E[D|X,Z]  = Dhat_pystacked_1
E[D|X]    = Dhat_pystacked_h_1
Orthogonalised D = D - E[D|X]; optimal IV = E[D|X,Z] - E[D|X].
------------------------------------------------------------------------------
             |               Robust
       share | Coefficient  std. err.      z    P>|z|     [95% conf. interval]
-------------+----------------------------------------------------------------
       price |  -.0979042   .0075006   -13.05   0.000     -.112605   -.0832033
       _cons |   .0033532   .0215627     0.16   0.876    -.0389089    .0456154
------------------------------------------------------------------------------
```

#### 手动估计

如果您想知道 `ddml` 在后台执行了什么，可以使用以下命令：

```stata
. ddml estimate, allcombos spec(8) rep(1) robust
```

**DDML 估计结果：**

```markdown
spec  r     Y learner     D learner         b        SE    DH learner
   1  1        Y1_reg      Dhat_reg    -0.137  ( 0.012)    Dhat_reg_h
   2  1        Y1_reg      Dhat_reg     0.369  ( 0.207) Dhat_pystac~h
   3  1        Y1_reg Dhat_pystac~d    -0.089  ( 0.005)    Dhat_reg_h
   4  1        Y1_reg Dhat_pystac~d    -0.114  ( 0.009) Dhat_pystac~h
   5  1  Y2_pystacked      Dhat_reg    -0.096  ( 0.011)    Dhat_reg_h
   6  1  Y2_pystacked      Dhat_reg    -0.212  ( 0.087) Dhat_pystac~h
   7  1  Y2_pystacked Dhat_pystac~d    -0.042  ( 0.004)    Dhat_reg_h
*  8  1  Y2_pystacked Dhat_pystac~d    -0.098  ( 0.008) Dhat_pystac~h
* = minimum MSE specification for that resample.

Min MSE DDML model, specification 8
y-E[y|X]  = Y2_pystacked_1                         Number of obs   =      2217
E[D|X,Z]  = Dhat_pystacked_1
E[D|X]    = Dhat_pystacked_h_1
Orthogonalised D = D - E[D|X]; optimal IV = E[D|X,Z] - E[D|X].
------------------------------------------------------------------------------
             |               Robust
       share | Coefficient  std. err.      z    P>|z|     [95% conf. interval]
-------------+----------------------------------------------------------------
       price |  -.0979042   .0075006   -13.05   0.000     -.112605   -.0832033
       _cons |   .0033532   .0215627     0.16   0.876    -.0389089    .0456154
------------------------------------------------------------------------------
```

#### 生成中介变量与工具变量

```stata
. gen Dtilde = $D - Dhat_pystacked_h_1
. gen Zopt = Dhat_pystacked_1 - Dhat_pystacked_h_1
```

#### 工具变量回归

```stata
. ivreg Y2_pystacked_1 (Dtilde=Zopt), robust
```

**工具变量 2SLS 回归**

```markdown
Instrumental variables 2SLS regression          Number of obs     =      2,217
                                                F(1, 2215)        =     170.38
                                                Prob > F          =     0.0000
                                                R-squared         =     0.1175
                                                Root MSE          =     1.0152

------------------------------------------------------------------------------
             |               Robust
Y2_pystack~1 | Coefficient  std. err.      t    P>|t|     [95% conf. interval]
-------------+----------------------------------------------------------------
      Dtilde |  -.0979042   .0075006   -13.05   0.000    -.1126131   -.0831953
       _cons |   .0033532   .0215627     0.16   0.876     -.038932    .0456385
------------------------------------------------------------------------------
Instrumented: Dtilde
 Instruments: Zopt
```