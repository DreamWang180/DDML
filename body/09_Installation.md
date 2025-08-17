### 安装 ddml

#### 从 SSC 安装

您可以通过 SSC 安装 `ddml`：

```stata
ssc install ddml, replace
```

#### 从 Github 安装

我们建议使用 Github 上更新更频繁的版本，您可以通过以下命令安装最新版：

```stata
net install ddml, /// 
    from(https://raw.githubusercontent.com/aahrens1/ddml/master) /// 
    replace
```

如果您想安装旧版本，可以使用以下命令（例如安装 v1.2 版本）：

```stata
net install ddml, /// 
    from(https://raw.githubusercontent.com/aahrens1/ddml/v1.2) /// 
    replace
```

请定期检查更新。

### 离线安装

如果您想在没有网络连接的环境下使用 `ddml`，建议您从 Github 下载包文件。点击绿色的 “Code” 按钮，然后选择 “Download ZIP”。下载并解压后，使用以下命令进行离线安装，确保 `from()` 指向您下载并解压后的仓库文件夹：

```stata
net install ddml, /// 
    from(path_to_unzipped_repository) /// 
    replace
```