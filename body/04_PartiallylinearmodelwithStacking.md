### 部分线性模型与堆叠回归

堆叠回归是一种简单且强大的方法，用于结合多个学习器的预测结果。在 Stata 中可以通过 **pystacked** 包实现（见此处）。以下是使用部分线性模型的示例，但它也可以与任何由 **ddml** 支持的模型一起使用。

#### 步骤 1: 初始化

准备：使用上述数据和全局变量。为这个新的估计使用名称 `m1`，以区别于使用默认名称 `m0` 的前一个示例。这样可以为比较提供多个估计。此外，还指定了 5 次交叉拟合重复。

```stata
. set seed 42
. ddml init partial, kfolds(2) reps(5) mname(m1)
```

**交叉拟合重复次数**
DDML 的结果取决于精确的交叉拟合折数分割。我们建议在不同的随机折数上多次重新运行（最终）模型；请参见选项 `reps(integer)`。

#### 步骤 2: 添加学习器

为估计条件期望添加监督学习器。堆叠集成中的第一个学习器是 OLS。我们还使用交叉验证的 Lasso、Ridge 和两种不同设置的随机森林，并将它们保存在以下宏中：

```stata
. global rflow max_features(5) min_samples_leaf(1) max_samples(.7)
. global rfhigh max_features(5) min_samples_leaf(10) max_samples(.7)
```

在每一步中，我们添加了 `mname(m1)` 选项，以确保学习器不会被添加到仍在内存中的 `m0` 模型中。我们还使用 `learner(varname)` 选项指定包含估计条件期望的变量名，以避免覆盖使用默认命名为 `m0` 模型创建的变量。

```stata
. ddml E[Y|X], mname(m1) learner(Y_m1): pystacked $Y $X            || /// 
>                                method(ols)                       || /// 
>                                method(lassocv)                   || /// 
>                                method(ridgecv)                   || /// 
>                                method(rf) opt($rflow)            || /// 
>                                method(rf) opt($rfhigh), type(reg)
Learner Y_m1 added successfully.
```

```stata
. ddml E[D|X], mname(m1) learner(D_m1): pystacked $D $X            || /// 
>                                method(ols)                       || /// 
>                                method(lassocv)                   || /// 
>                                method(ridgecv)                   || /// 
>                                method(rf) opt($rflow)            || /// 
>                                method(rf) opt($rfhigh), type(reg)
Learner D_m1 added successfully.
```

#### 选项说明

注意：“：”前和第一个逗号后的选项指的是 **ddml**，最后逗号后的选项指的是估计命令。确保不要混淆这两类选项。

检查学习器是否已正确添加（省略输出）：

```stata
. ddml desc, mname(m1) learners
```

#### 步骤 3/4: 交叉拟合和估计

```stata
. qui ddml crossfit, mname(m1)
```

```stata
. ddml estimate, mname(m1) robust
```

**DDML 估计结果**：

```markdown
spec  r     Y learner     D learner         b        SE
 opt  1          Y_m1          D_m1  7362.283 (937.426)
 opt  2          Y_m1          D_m1  6958.283 (899.946)
 opt  3          Y_m1          D_m1  6531.201 (872.895)
 opt  4          Y_m1          D_m1  6532.662 (952.414)
 opt  5          Y_m1          D_m1  6672.368 (981.239)
opt = minimum MSE specification for that resample.

Mean/med.   Y learner     D learner         b        SE
 mse mn     [min-mse]         [mse]  6811.360 (973.863)
 mse md     [min-mse]         [mse]  6672.368 (962.606)

Median over min-mse specifications
y-E[y|X]  = Y_m1                                   Number of obs   =      9915
D-E[D|X,Z]= D_m1
------------------------------------------------------------------------------
             |               Robust
     net_tfa | Coefficient  std. err.      z    P>|z|     [95% conf. interval]
-------------+----------------------------------------------------------------
        e401 |   6672.368   962.6062     6.93   0.000     4785.695    8559.042
------------------------------------------------------------------------------
```

**5 次重新抽样的总结**：

```markdown
       D eqn      mean       min       p25       p50       p75       max
        e401   6811.3596 6531.2007 6532.6626 6672.3682 6958.2832 7362.2832
```

检查 **pystacked** 使用的学习器权重（未显示）：

```stata
. ddml extract, mname(m1) show(pystacked)
```