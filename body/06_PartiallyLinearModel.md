### 部分线性模型

#### 准备工作

我们加载数据，定义全局宏并设置随机种子。

```stata
. use https://github.com/aahrens1/ddml/raw/master/data/sipp1991.dta, clear
. global Y net_tfa
. global D e401
. global X tw age inc fsize educ db marr twoearn pira hown
. set seed 42
```

#### 步骤 1: 初始化 DDML 模型

接下来，我们初始化 ddml 估计并选择模型。`partial` 表示部分线性模型。除非使用 `mname(name)` 选项指定，否则模型将存储在 Mata 对象中，默认名称为“m0”。

**折数**
注意，我们将随机折数设置为 2，以便模型运行得更快。默认值为 `kfolds(5)`。我们建议考虑至少 5-10 折，如果样本量较小，甚至更多。

Number of folds
Note that we set the number of random folds to 2, so that the model runs quickly. The default is kfolds(5). We recommend to consider at least 5-10 folds and even more if your sample size is small.



```stata
. ddml init partial, kfolds(2)
```

#### 步骤 2: 添加机器学习器

我们为估计条件期望 $E[Y|X]$ 添加一个监督学习器。首先，我们添加简单的线性回归。

```stata
. ddml E[Y|X]: reg $Y $X
Learner Y1_reg added successfully.
```

我们可以为每个简化方程添加多个学习器。在这里，我们还使用 `pystacked` 中实现的随机森林学习器。（在下一个示例中，我们展示如何使用 `pystacked` 堆叠多个学习器，但这里我们只使用它实现一个学习器。）

```stata
. ddml E[Y|X]: pystacked $Y $X, type(reg) method(rf)
Learner Y2_pystacked added successfully.
```

我们对条件期望 $E[D|X]$ 进行同样的操作。

```stata
. ddml E[D|X]: reg $D $X
Learner D1_reg added successfully.
. ddml E[D|X]: pystacked $D $X, type(reg) method(rf)
Learner D2_pystacked added successfully.
```

你可以选择检查学习器是否已正确添加。

```stata
. ddml desc

. ddml desc

Model:                  partial, crossfit folds k=2, resamples r=1
Dependent variable (Y): net_tfa
 net_tfa learners:      Y1_reg Y2_pystacked
D equations (1):        e401
 e401 learners:         D1_reg D2_pystacked
```



#### 步骤 3: 交叉拟合

学习器会在训练数据上进行迭代拟合。此步骤可能需要一些时间。

```stata
. ddml crossfit
Cross-fitting E[Y|X] equation: net_tfa
Cross-fitting fold 1 2 ...completed cross-fitting
Cross-fitting E[D|X] equation: e401
Cross-fitting fold 1 2 ...completed cross-fitting
```

#### 步骤 4: 估计

最后，我们获得感兴趣系数的估计值。由于我们为两个简化方程添加了两个学习器，因此有四种可能的规格。默认情况下，显示的结果对应于具有最低样本外均方误差（MSPE）的规格：

```stata
. ddml estimate, robust

. ddml estimate, robust

DDML estimation results:
spec  r     Y learner     D learner         b        SE
   1  1        Y1_reg        D1_reg  5397.308(1130.901)
   2  1        Y1_reg  D2_pystacked  6707.514 (880.374)
*  3  1  Y2_pystacked        D1_reg  7044.822(1127.173)
   4  1  Y2_pystacked  D2_pystacked  6991.835 (755.805)
* = minimum MSE specification for that resample.

Min MSE DDML model, specification 3
y-E[y|X]  = Y2_pystacked_1                         Number of obs   =      9915
D-E[D|X,Z]= D1_reg_1
------------------------------------------------------------------------------
             |               Robust
     net_tfa | Coefficient  std. err.      z    P>|z|     [95% conf. interval]
-------------+----------------------------------------------------------------
        e401 |   7044.822   1127.173     6.25   0.000     4835.603    9254.042
------------------------------------------------------------------------------
```



要估计所有四种规格，我们使用 `allcombos` 选项：

```stata
. ddml estimate, robust allcombos

. ddml estimate, robust allcombos

DDML estimation results:
spec  r     Y learner     D learner         b        SE
   1  1        Y1_reg        D1_reg  5397.208(1130.776)
   2  1        Y1_reg  D2_pystacked  6705.740 (878.656)
*  3  1  Y2_pystacked        D1_reg  7044.518(1126.896)
   4  1  Y2_pystacked  D2_pystacked  6979.699 (753.471)
* = minimum MSE specification for that resample.

Min MSE DDML model
y-E[y|X]  = Y2_pystacked_1                         Number of obs   =      9915
D-E[D|X,Z]= D1_reg_1
------------------------------------------------------------------------------
             |               Robust
     net_tfa | Coefficient  std. err.      z    P>|z|     [95% conf. interval]
-------------+----------------------------------------------------------------
        e401 |   7044.518   1126.896     6.25   0.000     4835.843    9253.193
       _cons |  -317.8379   352.8666    -0.90   0.368    -1009.444     373.768
------------------------------------------------------------------------------
```



估计所有规格后，我们可以检索特定结果。在这里，我们使用依赖 OLS 估计 $E[Y|X]$ 和 $E[D|X]$ 的规格：

```stata
. ddml estimate, robust spec(1) replay

. ddml estimate, robust spec(1) replay

DDML estimation results:
spec  r     Y learner     D learner         b        SE
 opt  1  Y2_pystacked        D1_reg  7044.518(1126.896)
opt = minimum MSE specification for that resample.

DDML model, specification 1
y-E[y|X]  = Y1_reg_1                               Number of obs   =      9915
D-E[D|X,Z]= D1_reg_1
------------------------------------------------------------------------------
             |               Robust
     net_tfa | Coefficient  std. err.      z    P>|z|     [95% conf. interval]
-------------+----------------------------------------------------------------
        e401 |   5397.208   1130.776     4.77   0.000     3180.928    7613.488
       _cons |   -104.854   397.9023    -0.26   0.792     -884.728    675.0201
------------------------------------------------------------------------------
```





#### 常数项的包含

由于残差化后的结果和处理可能在有限样本中不完全为零，**ddml** 默认在部分线性模型的估计阶段包含常数项。从渐进角度来看，截距是非必须的。早期版本的 **ddml**（1.2 之前）未包含常数项。

你可以通过输入以下命令手动获取相同的点估计：

```stata
. reg Y1_reg D1_reg, robust

. reg Y1_reg D1_reg, robust

Linear regression                               Number of obs     =      9,915
                                                F(1, 9914)        =      22.78
                                                Prob > F          =     0.0000
                                                R-squared         =     0.0037
                                                Root MSE          =      39626

------------------------------------------------------------------------------
             |               Robust
    Y1_reg_1 | Coefficient  std. err.      t    P>|t|     [95% conf. interval]
-------------+----------------------------------------------------------------
    D1_reg_1 |   5397.308   1130.901     4.77   0.000     3180.512    7614.105
------------------------------------------------------------------------------
```



或者通过图形展示：

```stata
. twoway (scatter Y1_reg D1_reg) (lfit Y1_reg D1_reg)
```

其中，`Y1_reg` 和 `D1_reg` 是 `net_tfa` 和 \`e401\` 的正交化版本。

要详细描述 **ddml** 模型设置或结果，可以使用 `ddml describe` 并附上相关选项（例如 `sample`、`learners`、`crossfit`、`estimates`），或者使用 `all` 选项一次性描述所有内容：

```stata
. ddml describe, all
```