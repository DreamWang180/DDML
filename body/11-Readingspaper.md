### ddml

Ahrens, A., Hansen, C.B., Schaffer, M.E., and Wiemann, T. (2023). ddml: Double/Debiased Machine Learning in Stata. arXiv preprint arXiv:2301.09397.

我们介绍了 Stata 中的 **ddml** 包，用于双重/去偏机器学习（Double/Debiased Machine Learning，简称 DDML）。该包支持五种不同计量经济学模型的因果参数估计，能够灵活估计内生变量在功能形式未知和/或外生变量众多的情况下的因果效应。**ddml** 与 Stata 中许多现有的监督学习程序兼容。我们推荐将 DDML 与堆叠估计（stacking estimation）结合使用，堆叠估计将多个机器学习模型结合成最终的预测器。我们提供了蒙特卡洛证据来支持我们的推荐。

[链接到工作论文](#)

[Bibtex 文件](#)

---

### pystacked

Ahrens, A., Hansen, C.B., and Schaffer, M.E. (2022). pystacked: Stacking Generalization and Machine Learning in Stata. arXiv preprint arXiv:2208.10896.

**pystacked** 实现了堆叠泛化（stacked generalization，Wolpert，1992），用于通过 Python 的 scikit-learn 进行回归和二分类。堆叠将多个监督学习模型（“基础”或“第 0 层”模型）组合成一个单一的学习器。当前支持的基础学习器包括正则化回归、随机森林、梯度提升树、支持向量机和前馈神经网络（多层感知机）。**pystacked** 也可以作为一个常规机器学习程序使用，拟合单个基础学习器，因此为 scikit-learn 的机器学习算法提供了一个易于使用的 API。

[链接到工作论文](#)

[Bibtex 文件](#)

---

### lassopack

Ahrens, A., Hansen, C.B., Schaffer, M.E. (2020). lassopack: Model Selection and Prediction with Regularized Regression in Stata. The Stata Journal, 20(1):176-235. doi:10.1177/1536867X20909697

在这篇文章中，我们介绍了 **lassopack**，一个用于 Stata 的正则化回归程序集。**lassopack** 实现了 Lasso、平方根 Lasso、弹性网、岭回归、自适应 Lasso 和后验普通最小二乘法（OLS）。这些方法适用于高维设置，其中预测变量 $p$ 可能很大，甚至大于观察值的数量 $n$。我们提供了三种选择惩罚（“调优”）参数的方法：信息准则（在 lasso2 中实现）、K 折交叉验证以及用于横截面、面板和时间序列数据的 h 步前滚动交叉验证（cvlasso），以及理论驱动（“严格”或插件）惩罚方法用于横截面和面板数据的 Lasso 和平方根 Lasso（rlasso）。我们讨论了每种方法的理论框架和实践考虑。我们还提供了蒙特卡洛结果，以比较不同惩罚方法的表现。

[下载早期的 arXiv 版本 - Stata Journal 论文](#)

如果您在访问 SJ 版本时遇到问题，请随时联系我（AA）。

[Bibtex 文件](#)

---

### 幻灯片

来自我们在 2018 年伦敦 Stata 大会上的演讲幻灯片可在[此处](#)查看。
**pystacked** 的演讲幻灯片可以在[这里](#)找到。
**ddml** 的演讲幻灯片可以在[这里](#)找到。