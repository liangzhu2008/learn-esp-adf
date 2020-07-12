# Learn-to-esp-adf

learn to esp-adf-v2.0， 学习 esp-adf-v2.0 框架

## Install Graphviz (Windows)

* 从 <https://graphviz.org/_pages/Download/Download_windows.html> 下载 Graphviz 2.38 Stable Release 安装包。

* 执行安装包。

* 在 PATH 环境变量添加安装目录，例如: `C:\Program Files (x86)\Graphviz2.38\bin` 。

* 在 cmd 命令行模式下， 执行 `dot -version` ，检查安装结果。

## Install sphinx

* 安装 spinx 及相关包

  ```bash
  pip install sphinx recommonmark sphinx_rtd_theme sphinxcontrib-plantuml
  ```

  * sphinx: 一个基于 Python 的文档生成工具
  * recommonmark: sphinx 对 Markdown 支持包
  * sphinx_rtd_theme： 一个 sphinx 的主题包
  * sphinxcontrib-plantuml： uml 工具包

  **note**: 安装 sphinxcontrib-plantuml 时，会有一个错误提示 *attributeerror '_namespacepath' object has no attribute 'sort'*， 需要执行 `python -m pip install --upgrade pip setuptools wheel`。

* 创建目录及模板

  ```bash
  mkdir docs
  cd docs
  sphinx-quickstart
  ```

* 从 https://plantuml.com/zh/download 下载 plantuml.xxx.jar。放到 `docs` 目录下，并执行`java -jar plantuml.xxx.jar -version`检查安装结果。

* 修改 conf.py，加入 Markdown 支持，UML 支持

  ```python
  extensions = ['recommonmark']
  extensions = ['sphinx.ext.autodoc']
  extensions = ['sphinxcontrib.plantuml']

  plantuml = 'java -jar plantuml.1.2020.15.jar'
  ```

* 修改 conf.py，加入 sphinx_rtd_theme 主题支持。修改

  ``` python
  html_theme = 'alabaster'
  ```

  为

  ``` python
  import sphinx_rtd_theme
  html_theme = "sphinx_rtd_theme"
  html_theme_path = [sphinx_rtd_theme.get_html_theme_path()]
  ```

* 生成 html 文档

  ```bash
  make html
  ```

* 打开 _build\html\index.html 即可。
