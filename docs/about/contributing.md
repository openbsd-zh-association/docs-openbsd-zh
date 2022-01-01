# 贡献指南

## Git 与 Github 指南

* 用[单独的分支](#切换到单独分支)来保存您的 Git 提交（不要直接 commit 到 main 分支） 
* 添加文章时，一个拉取请求（PR）或提交（commit）对应一篇文章。如果您使用了多个提交，请考虑将您的提交挤压（squash）到一个。
* 提交之前，在本地预览看看。

### 切换到单独分支

```
git fetch upstream
git checkout -b translate-faq4 upstream/master
```

## 构建本站

本站采用 MkDocs 构建，需要 Python 环境。要构建本站，需要安装 Python3。以下流程假设您已经安装，且在 Unix-like 环境下。（Linux 或者 *BSD）

设置 Python 环境：

```
python3 -m venv ./venv 
source ./venv/bin/activate
```

安装 MkDocs 与主题：

```
pip install mkdocs[i18n] mkdocs-material
```

本地预览：

```
mkdocs serve
```

