### 设置 Stata 的 Python 集成

`pystacked` 至少需要 Stata 16（或更高版本）、Python 安装（3.8 或更高版本）以及 scikit-learn（0.24 或更高版本）。如果你想使用 `ddml`，也需要安装 Python。

StataCorp 提供了详细的说明，介绍如何设置 Stata 的 Python 集成，分为三个博客条目：链接1、链接2、链接3。

以下是我们简要概述的步骤：

#### 1. Python 安装

你有（至少）三种选择：

* 对许多人来说，最简单的方式是安装 Anaconda，可以从[这里](https://www.anaconda.com/products/individual)下载。Anaconda 是一个 Python 发行版，带有最重要的包、包管理器和编辑器。
* 另外，你也可以从[这里](https://www.python.org/downloads/)下载并安装普通的 Python。
* 如果你已经在系统中安装了一个较新的 Python 版本，可以为 Stata 设置一个单独的 Python 环境。这是可选的，但如果你希望在不同的项目中使用不同的库，这可能会很有用（参见[此处的说明](https://docs.python.org/3/library/venv.html)）。

#### 2. 设置 Python 集成

安装 Python 后，你需要告诉 Stata 如何找到 Python 安装位置。

你可以使用以下命令在系统中搜索 Python 安装：

```
. python search
```

注意，可能会显示多个 Python 安装（例如，MacOS 自带了一个旧版本的 Python），Stata 可能无法找到系统中的所有 Python 安装。

要将 Stata 与特定的 Python 安装链接，使用：

```
python set exec <pyexecutable>, permanently
```

其中 `<pyexecutable>` 可能是例如：`/usr/local/bin/python3`、`C:\Program Files\Python38\python.exe` 或 `C:\Users\<user>\AppData\Local\Programs\Python\Python38\python.exe`，具体取决于你的操作系统以及 Python 的安装位置。

输入 `python query` 来检查安装是否正确链接：

```
.  python query
------------------------------------------------------------------ 
    Python Settings
      set python_exec      /usr/bin/python3
      set python_userpath  

    Python system information
      initialized          no
      version              3.8.9
      architecture         64-bit
```

你可以通过输入 `python` 启动 Python 环境，输入 `end` 返回 Stata 环境：

```
. python
----------------------------------------------- python (type end to exit) ----------
>>> print('hello')
hello
>>> end
------------------------------------------------------------------------------------
```

#### 3. 管理包

`pystacked` 需要 `scikit-learn`（简称 `sklearn`）。你可以在 Stata 中检查 `sklearn` 是否已安装：

```
. python which sklearn
<module 'sklearn' from '/Users/<username>/Library/Python/3.8/lib/python/site-packages/sklearn/__init__.py'>
```

如果 Stata 找不到 `sklearn`，可能是因为你将 Stata 链接到了错误的 Python 安装，或者你还需要安装 `sklearn`。

如果你使用 Anaconda，`sklearn` 会自动包含，你可以通过 Anaconda 的 Python 发行版更新 `scikit-learn`（参见[此处](https://docs.anaconda.com/anaconda/user-guide/tasks/manage-pkgs/)）或在终端中使用 `conda install`。

如果不使用 Anaconda，可以通过 `pip` 安装和更新包。例如，你可以在终端或 Stata 中直接输入以下命令安装 `sklearn`：

```
. shell <Python path> -m pip install -U scikit-learn
```

其中 `<Python path>` 是你希望与 Stata 一起使用的 Python 安装路径。如果你只想使用默认的 Python 安装，也可以将 `<Python path>` 替换为 `python3`（Mac 上）或 `py`（Windows 上）。

#### 4. 检查是否正常工作

为了测试 Stata 的 Python 集成是否在系统上正常工作，请在 Stata 中运行以下测试代码：

```
clear all
use http://www.stata-press.com/data/r16/iris

python:
from sfi import Data
import numpy as np
from sklearn.svm import SVC

# 使用 sfi Data 类将 Stata 变量中的数据拉取到 Python 中
X = np.array(Data.get("seplen sepwid petlen petwid"))
y = np.array(Data.get("iris"))

# 使用数据训练 C-Support Vector Classifier
svc_clf = SVC(gamma='auto')
svc_clf.fit(X, y)
end



clear all 
use http://www.stata-press.com/data/r16/iris

python:
from sfi import Data
import numpy as np
from sklearn.svm import SVC

# Use the sfi Data class to pull data from Stata variables into Python
X = np.array(Data.get("seplen sepwid petlen petwid"))
y = np.array(Data.get("iris"))

# Use the data to train C-Support Vector Classifier
svc_clf = SVC(gamma='auto')
svc_clf.fit(X, y)
end
```

为了测试 `pystacked` 是否正常工作，可以在 Stata 中运行以下测试代码：

```
clear all
use https://statalasso.github.io/dta/cal_housing.dta, clear
set seed 42
gen train=runiform()
replace train=train<.75
set seed 42
pystacked medh longi-medi if train
```



#### 可选：使用 Stata 和 Python 环境

你也可以将 Stata 与 Python 环境结合使用。如果你希望在系统中使用多个版本的 Python，这可能会很有用。如何使用 Python 环境的完整指南可见[此处](https://docs.python.org/3/library/venv.html)。以下是详细的步骤：

##### MacOS:

1. 关闭 Stata 并创建一个文件夹用于保存 Python 环境。我使用的文件夹是 `/Users/myname/python_envs`。
2. 打开终端并导航到该文件夹。在我的示例中，这将是 `cd /Users/myname/python_envs`。
3. 设置 Python 环境：`python3 -m venv myenv`，其中 `myenv` 可以替换为你想要使用的任何名称。
4. 激活环境：`source venv/bin/activate`
5. 安装 `sklearn`：`python3 -m pip install scikit-learn`
6. 退出环境：`deactivate`
7. 打开 Stata 并输入：`python set exec "/Users/myname/python_envs/myenv/bin/python3", perm`

##### Windows:

1. 关闭 Stata 并创建一个文件夹用于保存 Python 环境。我使用的文件夹是 `C:\Users\myname\python_envs`。
2. 打开命令提示符并导航到该文件夹。在我的示例中，这将是 `cd C:\Users\myname\python_envs`。
3. 设置 Python 环境：`py -m venv myenv`，其中 `myenv` 可以替换为你想要使用的任何名称。
4. 激活环境：`.\venv\bin\activate`
5. 安装 `sklearn`：`py -m pip install scikit-learn`
6. 退出环境：`deactivate`
7. 打开 Stata 并输入：`python set exec "C:\Users\myname\python_envs\Scripts\python.exe", perm`