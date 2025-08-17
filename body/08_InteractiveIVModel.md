### 交互式工具变量（IV）模型

#### 准备工作

我们加载数据，定义全局宏并设置种子。

```stata
. use http://fmwww.bc.edu/repec/bocode/j/jtpa.dta,clear
. global Y earnings
. global D training
. global Z assignmt
. global X sex age married black hispanic
. set seed 42
```

#### 步骤 1: 初始化

我们初始化模型。

```stata
. ddml init interactiveiv, kfolds(5)
```

#### 步骤 2: 添加学习器

我们使用堆叠（在 `pystacked` 中实现）并为每个简化方程添加两个基础学习器。

```stata
. ddml E[Y|X,Z]: pystacked $Y c.($X)# #c($X), type(reg) m(ols lassocv)
Learner Y1_pystacked added successfully.

. ddml E[D|X,Z]: pystacked $D c.($X)# #c($X), type(class) m(logit lassocv)
Learner D1_pystacked added successfully.

. ddml E[Z|X]: pystacked $Z c.($X)# #c($X), type(class) m(logit lassocv)
Learner Z1_pystacked added successfully.
```

#### 步骤 3-4: 交叉拟合与估计

```stata
. ddml crossfit
Cross-fitting E[y|X,Z] equation: earnings
Cross-fitting fold 1 2 3 4 5 ...completed cross-fitting
Cross-fitting E[D|X,Z] equation: training
Cross-fitting fold 1 2 3 4 5 ...completed cross-fitting
Cross-fitting E[Z|X]: assignmt
Cross-fitting fold 1 2 3 4 5 ...completed cross-fitting

. ddml estimate
```

**DDML 估计结果 (LATE)：**

```markdown
spec  r    Y0 learner    Y1 learner    D0 learner    D1 learner         b        SE     Z learner
 opt  1  Y1_pystacked  Y1_pystacked  D1_pystacked  D1_pystacked  1817.206 (512.772)  Z1_pystacked
opt = minimum MSE specification for that resample.

E[y|X,D=0]   = Y1_pystacked0_1                     Number of obs   =     11204
E[y|X,D=1]   = Y1_pystacked1_1
E[D|X,Z=0]   = D1_pystacked0_1
E[D|X,Z=1]   = D1_pystacked1_1
E[Z|X]       = Z1_pystacked_1
------------------------------------------------------------------------------
             |               Robust
    earnings | Coefficient  std. err.      z    P>|z|     [95% conf. interval]
-------------+----------------------------------------------------------------
    training |   1817.206   512.7723     3.54   0.000     812.1911    2822.221
------------------------------------------------------------------------------
```

#### 短堆叠（Short-stacking）

如果我们想要使用短堆叠而不是堆叠，步骤如下：

初始化：

```stata
. set seed 42
. ddml init interactiveiv, kfolds(5)
warning - model m0 already exists
all existing model results and variables will
be dropped and model m0 will be re-initialized
```

添加基础学习器：

```stata
. ddml E[Y|X,Z]: reg $Y $X
Learner Y1_reg added successfully.
. ddml E[Y|X,Z]: pystacked $Y c.($X)# #c($X), type(reg) m(lassocv)
Learner Y2_pystacked added successfully.
. ddml E[D|X,Z]: logit $D $X
Learner D1_logit added successfully.
. ddml E[D|X,Z]: pystacked $D c.($X)# #c($X), type(class) m(lassocv)
Learner D2_pystacked added successfully.
. ddml E[Z|X]: logit $Z $X
Learner Z1_logit added successfully.
. ddml E[Z|X]: pystacked $Z c.($X)# #c($X), type(class) m(lassocv)
Learner Z2_pystacked added successfully.
```

使用短堆叠选项进行交叉拟合：

```stata
. ddml crossfit, shortstack
Cross-fitting E[y|X,Z] equation: earnings
Cross-fitting fold 1 2 3 4 5 ...completed cross-fitting...completed short-stacking
Cross-fitting E[D|X,Z] equation: training
Cross-fitting fold 1 2 3 4 5 ...completed cross-fitting...completed short-stacking
Cross-fitting E[Z|X]: assignmt
Cross-fitting fold 1 2 3 4 5 ...completed cross-fitting...completed short-stacking
```

估计阶段：

```stata
. ddml estimate, robust
```

**短堆叠 DDML 模型 (LATE)：**

```markdown
spec  r    Y0 learner    Y1 learner    D0 learner    D1 learner         b        SE     Z learner
 opt  1  Y2_pystacked  Y2_pystacked      D1_logit  D2_pystacked  1824.422 (514.303)  Z2_pystacked
  ss  1  [shortstack]          [ss]          [ss]          [ss]  1818.488 (513.133)          [ss]
opt = minimum MSE specification for that resample.

Shortstack DDML model (LATE)
E[y|X,D=0]   = earnings_ss0_1                      Number of obs   =     11204
E[y|X,D=1]   = earnings_ss1_1
E[D|X,Z=0]   = training_ss0_1
E[D|X,Z=1]   = training_ss1_1
E[Z|X]       = assignmt_ss_1
------------------------------------------------------------------------------
             |               Robust
    earnings | Coefficient  std. err.      z    P>|z|     [95% conf. interval]
-------------+----------------------------------------------------------------
    training |   1818.488    513.133     3.54   0.000     812.7663    2824.211
------------------------------------------------------------------------------
```