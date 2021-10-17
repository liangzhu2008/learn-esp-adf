# Learn ESP-ADF

学习 ESP-ADF 框架.

* [阅读文档](https://learn-esp-adf.readthedocs.org)
* [工具安装](#工具安装)
* [生成文档](#本地生成文档)

## 工具安装

安装生成文档的工具。除非你要自已在本地生成文档，你才要执行这一步。

### 准备

* 安装 Python （sphinx 需要）。

<!-- 2. 安装 Java （sphinxcontrib-plantuml 需要）。

### 安装 Graphviz (Windows， for plantuml)

1. 从 <https://graphviz.org/_pages/Download/Download_windows.html> 下载 Graphviz 2.38 Stable Release 安装包。
2. 执行安装包。
3. 在 PATH 环境变量添加安装目录，例如: `C:\Program Files (x86)\Graphviz2.38\bin` 。
4. 在 cmd 命令行模式下， 执行 `dot -version` ，检查安装结果。 -->

### 安装 Sphinx

1. 安装软件包（需要先安装 Python）

    ```sh
    pip install sphinx sphinx_intl sphinx_rtd_theme recommonmark plantweb
    ```

    * sphinx: 文档生成工具
    * sphinx_intl: 多语言工具
    * recommonmark: sphinx支持markdown的插件
    * sphinx_rtd_theme: sphinx的readthedocs主题插件
    * plantweb: uml 生成工具

    <!-- # pip install sphinx sphinx_intl sphinx_rtd_theme recommonmark sphinxcontrib-plantuml -->
    <!-- **note**: 安装 sphinxcontrib-plantuml 时，会有一个错误提示 `attributeerror '_namespacepath' object has no attribute 'sort'`， 需要执行 `python -m pip install --upgrade pip setuptools wheel`。 -->

2. 创建存放文档的目录，执行 sphinx-quickstart 命令

    ```sh
    cd /path/to/project
    mkdir docs
    cd docs
    sphinx-quickstart
    ```

    sphinx-quickstart 会创建基本配置。一般情况下，你只要接受默认值就行了。当上述命令执行完后，在 ```docs```目录下，你会找以 ```index.rst```和 ```conf.py```。 你可以编辑这两个文件，加入一些项目信息。

    * **Makefile** : 批处理指令，使用 make 命令时，用来构建文档输出。
    * **_build** : 用于存放最终生成的文档。
    * **_static** : 所有不属于文档源代码的文件（如图像）均存放于此处，构建时会它们链接在一起。
    * **conf.py** : 一个 Python 文件，存放 Sphinx 的配置值，包括执行 sphinx-quickstart 时选中的那些值。
    * **index.rst** : 文档项目的 root 目录。如果将文档有多个文件，该目录会连接这些文件。

    <!-- 3. 从 <https://plantuml.com/zh/download> 下载 plantuml.xxx.jar。放到 `docs\tool\` 目录下，并执行`java -jar tool\plantuml.xxx.jar -version`检查安装结果。显示如下结果则安装正常：
        ```log
        PlantUML version 1.2020.15 (Sun Jun 28 19:39:45 CST 2020)
        (GPL source distribution)
        Java Runtime: OpenJDK Runtime Environment
        JVM: OpenJDK 64-Bit Server VM
        Default Encoding: MS950_HKSCS
        Language: zh
        Country: HK
        PLANTUML_LIMIT_SIZE: 4096
        Dot version: dot - graphviz version 2.38.0 (20140413.2041)
        Installation seems OK. File generation OK
        ``` -->

3. 修改```conf.py```，加入 Markdown 支持，UML 支持

    ```python
    extensions = ['recommonmark']
    extensions = ['plantweb.directive']
    ```

    <!-- 
    extensions = ['sphinx.ext.autodoc']
    #extensions = ['sphinxcontrib.plantuml']
    import os
    plantuml_relative_path_ = r'tool\plantuml.1.2020.15.jar'
    plantuml = 'java -jar ' + os.path.join(os.path.abspath(os.getcwd()), plantuml_relative_path_) -->

4. 修改 ```conf.py```，使用 sphinx_rtd_theme 风格。修改

    ```python
    html_theme = 'alabaster'
    ```

    为

    ```python
    import sphinx_rtd_theme
    html_theme = "sphinx_rtd_theme"
    html_theme_path = [sphinx_rtd_theme.get_html_theme_path()]
    ```

## 本地生成文档

在 `docs\` 目录下执行命令，则生成 html 格式输出

```shell
make html
```

输出的html文件在 ```docs\_build\html```目录下， 打开 ```index.html```即可。

## LICENSE

[Learn ESP-ADF](https://github.com/liangzhu2008/learn-esp-adf) 采用 [Apache-2.0 License](https://github.com/liangzhu2008/learn-esp-adf/blob/master/LICENSE) 授权。

 <!-- [ESP-IDF](https://github.com/espressif/esp-idf) 采用 [Apache-2.0 License](https://github.com/espressif/esp-idf/blob/master/LICENSE) 授权。 -->

<!-- [ESP-ADF](https://github.com/espressif/esp-adf) 采用 [ESPRESSIF MIT License](https://github.com/espressif/esp-adf/blob/master/LICENSE) 授权。 -->



----END----
