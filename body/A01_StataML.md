# 欢迎访问 Stata ML 页面

在本网站上，我们介绍了用于机器学习的 Stata 软件包。这些软件包包括旨在进行预测、模型选择和因果推断的功能。

### lassopack
The package lassopack implements lasso (Tibshirani 1996), square-root lasso (Belloni et al. 2011), elastic net (Zou & Hastie 2005), ridge regression (Hoerl & Kennard 1970), adaptive lasso (Zou 2006) and post-estimation OLS. lassopack also supports logistic lasso.

lassopack 实现了以下算法：

- Lasso（Tibshirani 1996）
- 平方根 lasso（Belloni 等人 2011）
- 弹性网（Zou & Hastie 2005）
- 岭回归（Hoerl & Kennard 1970）
- 自适应 lasso（Zou 2006）
- 后估计 OLS

lassopack 还支持逻辑回归 lasso。

### pdslasso
pdslasso 提供了有助于结构模型中因果推断的方法。该软件包允许从大量变量中选择控制变量和/或工具变量，适用于研究者希望估计一个或多个（可能是内生的）因果变量的因果影响的情境。

### pystacked
pystacked 实现了堆叠回归（Wolpert, 1992），通过 scikit-learn 的 `sklearn.ensemble.StackingRegressor` 和 `sklearn.ensemble.StackingClassifier`。堆叠方法将多个监督式机器学习模型（“基础学习器”）的预测结果结合起来，最终输出更优的预测结果。

### ddml
ddml 实现了双重/去偏机器学习（Double/Debiased Machine Learning, DDML）方法，支持五种不同的估计器，可灵活估计内生变量的因果效应，适用于具有未知函数形式和/或大量外生变量的情境。ddml 与 Stata 中许多现有的监督式机器学习程序兼容。
