# Nox 教程

**英文地址** ：[nox tutorial](https://nox.thea.codes/en/stable/tutorial.html)



本教程将引导你学会安装、配置和运行 Nox。

## 安装

Nox 可以通过[pip](https://pip.readthedocs.org/)轻松安装：

```python
python3 -m pip install nox
```

你可能希望使用[用户站点](https://packaging.python.org/tutorials/installing-packages/%23installing-to-the-user-site#installing-to-the-user-site)(user site)来避免对全局的 Python install 造成混乱：  

```python
python3 -m pip install --user nox
```

或者，你也可以更精致，使用[pipx](https://packaging.python.org/guides/installing-stand-alone-command-line-tools/)：

```python
pipx install nox
```

无论用哪种方式，Nox 通常是要全局安装的，类似于 tox、pip和其它类似的工具。

如果你有兴趣在[docker](https://www.docker.com/) 内运行 nox，可以使用 DockerHub 上的[thekevjames/nox镜像](https://hub.docker.com/r/thekevjames/nox)，它包含所有 nox 版本的构建与及所有支持的 Python 版本。

如果你想在[GitHub Actions中](https://github.com/features/actions)运行 nox ，则可以使用[Activatedleigh/setup-nox action](https://github.com/marketplace/actions/setup-nox)，它将安装最新的 nox，并令 GitHub Actions 环境提供的所有 Python 版本可用。

## 编写配置文件

Nox 通过项目目录中一个名为 noxfile.py 的文件作配置 。这是一个 Python文件，定义了一组会话（sessions）。一个会话是一个环境和一组在这个环境中运行的命令。如果你熟悉 tox，会话就类似于它的环境。如果你熟悉 GNU Make，会话则类似于它的 target。       

会话使用 @nox.session 装饰器作声明。这方式类似于 Flask 使用 @app.route。   

下面是一个基本的 Nox 文件，对 example.py 运行[flake8](http://flake8.pycqa.org/en/latest/)（你可以自己创建example.py）：

```python
import nox

@nox.session
def lint(session):
    session.install("flake8")
    session.run("flake8", "example.py")
```

## 第一次运行 Nox

现在，你已经安装了 Nox 并拥有一个配置文件， 那就可以运行 Nox 了！在终端中打开项目的目录，然后运行`nox` 。你应该会看到类似这样的内容： 

```python
$ nox
nox > Running session lint
nox > Creating virtualenv using python3.7 in .nox/lint
nox > pip install flake8
nox > flake8 example.py
nox > Session lint was successful.
```

✨现在你已第一次成功地使用 Nox 啦！✨

本教程的其余部分将带你学习其它可以用 Nox 完成的常见操作。如果需要的话，你还可以跳至[命令行用法](https://nox.thea.codes/en/stable/usage.html)和[配置＆API](https://nox.thea.codes/en/stable/config.html)文档。

## 安装依赖项

Nox 基本上是将 session.install 传递给 pip ，因此你可以用通常的方式来安装东西。这里有一些例子：

（1）一次安装一个或多个包：

```python
@nox.session
def tests(session):
    # same as pip install pytest protobuf>3.0.0
    session.install("pytest", "protobuf>3.0.0")
    ...
```

（2）根据 requirements.txt 文件安装：  

```python
@nox.session
def tests(session):
    # same as pip install -r -requirements.txt
    session.install("-r", "requirements.txt")
    ...
```

（3）如果你的项目是一个 Python 包，而你想安装它：

```python
@nox.session
def tests(session):
    # same as pip install .
    session.install(".")
    ...
```

## 运行命令

session.run 函数可让你在会话的虚拟环境的上下文中运行命令。以下是一些示例：  

（1）你可以安装和运行 Python 工具：

```python
@nox.session
def tests(session):
    session.install("pytest")
    session.run("pytest")
```


（2）如果你想给一个程序传递更多的参数，只需给 run 添加更多参数即可： 

```python
@nox.session
def tests(session):
    session.install("pytest")
    session.run("pytest", "-v", "tests")
```

（3）你还可以传递环境变量：

```python
@nox.session
def tests(session):
    session.install("black")
    session.run(
        "pytest",
        env={
            "FLASK_DEBUG": "1"
        }
    )
```

有关运行程序的更多选项和示例，请参见[nox.sessions.Session.run()](https://nox.thea.codes/en/stable/config.html%23nox.sessions.Session.run#nox.sessions.Session.run)。

## 选择要运行的会话

一旦你的 Noxfile 中有多个会话，你会注意到 Nox 将默认运行所有的会话。尽管这很有用，但是通常一次只需要运行一两个。

你可以使用`--sessions`参数（或`-s`）来选择要运行的会话。你可以使用`--list`参数显示哪些会话可用，哪些将会运行。这里有一些例子：

这是一个具有三个会话的 Noxfile：

```python
import nox

@nox.session
def test(session):
    ...

@nox.session
def lint(session):
    ...

@nox.session
def docs(session):
    ...
```

如果你只运行`nox --list` ，则会看到所有会话都被选中：   

```python
Sessions defined in noxfile.py:

* test
* lint
* docs

sessions marked with * are selected,
sessions marked with - are skipped.
```

如果你运行`nox --list --sessions lint`，Nox 将只运行 lint 会话：    

```python
nox > Running session lint
nox > Creating virtualenv using python3 in .nox/lint
nox > ...
nox > Session lint was successful.
```

还有更多选择和运行会话的方法！你可以在[命令行用法](https://nox.thea.codes/en/stable/usage.html)中阅读更多有关调用 Nox 的信息。

## 针对不同的多个 Python 进行测试

许多项目需要支持一个特定的 Python 版本或者多个 Python 版本。你可以通过给 @nox.session 指定 Python，来使 Nox 针对多个解释器运行会话。这里有一些例子：

（1）如果你希望会话仅针对 Python 的单个版本运行：

```python
@nox.session(python="3.7")
def test(session):
    ...
```

（2）如果你希望会话在 Python 的多个版本上运行：

```
@nox.session(python=["2.7", "3.5", "3.7"])
def test(session):
    ...
```

你会注意到，运行`nox --list`将显示此会话已扩展为三个不同的会话：   

```python
Sessions defined in noxfile.py:

* test-2.7
* test-3.5
* test-3.7
```

你可以使用`nox --sessions test`运行所有 test 会话，也可以使用列表中显示的全名来运行单个 test 会话，例如，`nox --sessions test-3.5`。有关选择会话的更多详细信息，请参见[命令行用法](https://nox.thea.codes/en/stable/usage.html)文档。           

你可以在[会话的virtualenv配置](https://nox.thea.codes/en/stable/config.html%23virtualenv-config#virtualenv-config)里，阅读到更多关于配置会话所用的虚拟环境的信息。

## 与 conda 一起测试

一些项目，特别是在数据科学社区，需要在 conda 环境中测试其使用的情况。如果你希望会话在 conda 环境中运行：

```python
@nox.session(venv_backend="conda")
def test(session):
    ...
```

使用 conda 安装软件包：

```python
session.conda_install("pytest")
```

可以用 pip 安装软件包进 conda 环境中，但是最好的实践是仅使用`--no-deps` 选项安装。这样可以避免 pip 安装的包与 conda 安装的包不兼容，防止 pip 破坏 conda 环境。

```python
session.install("contexter", "--no-deps")
session.install("-e", ".", "--no-deps")
```

## 参数化

就像 Nox 可以控制运行多个解释器一样，它也可以使用[nox.parametrize()](https://nox.thea.codes/en/stable/config.html%23nox.parametrize#nox.parametrize)装饰器，来处理带有一系列不同参数的会话。  

这是一个简短示例，使用参数化对两个不同版本的 Django 进行测试：

```python
@nox.session
@nox.parametrize("django", ["1.9", "2.0"])
def test(session, django):
    session.install(f"django=={django}")
    session.run("pytest")
```

如果运行`nox --list` ，你将会看到 Nox 把一个会话扩展为了多个会话。每个会话将获得你想传递给它的一个参数值：   

```python
Sessions defined in noxfile.py:

* test(django='1.9')
* test(django='2.0')
```

nox.parametrize() 的接口和用法特意类似于[pytest的parametrize](https://pytest.org/latest/parametrize.html%23_pytest.python.Metafunc.parametrize#_pytest.python.Metafunc.parametrize)。这是 Nox 的一项极其强大的功能。你可以在[参数化会话上](https://nox.thea.codes/en/stable/config.html%23parametrized#parametrized)，阅读更多有关参数化的信息与示例。

## 下一步

看看你！你现在基本上是一个 Nox 专家啦！✨

到了这一步，你还可以：

- 阅读更多文档，例如[命令行用法](https://nox.thea.codes/en/stable/usage.html)和[配置＆API](https://nox.thea.codes/en/stable/config.html)。   
- 给我们反馈或作贡献，请参阅[贡献](https://nox.thea.codes/en/stable/CONTRIBUTING.html)。

玩得开心！💜



### 相关链接：

[1] nox tutorial: https://nox.thea.codes/en/stable/tutorial.html

[2] pip: https://pip.readthedocs.org/

[3] 用户站点: https://packaging.python.org/tutorials/installing-packages/%23installing-to-the-user-site#installing-to-the-user-site

[4] pipx: https://packaging.python.org/guides/installing-stand-alone-command-line-tools/

[5] docker: https://www.docker.com/

[6] thekevjames/nox镜像: https://hub.docker.com/r/thekevjames/nox

[7] GitHub Actions中: https://github.com/features/actions

[8] Activatedleigh/setup-nox action: https://github.com/marketplace/actions/setup-nox

[9] flake8: http://flake8.pycqa.org/en/latest/

[10] 命令行用法: https://nox.thea.codes/en/stable/usage.html

[11] 配置＆API: https://nox.thea.codes/en/stable/config.html

[12] nox.sessions.Session.run(): https://nox.thea.codes/en/stable/config.html%23nox.sessions.Session.run#nox.sessions.Session.run

[13] 会话的virtualenv配置: https://nox.thea.codes/en/stable/config.html%23virtualenv-config#virtualenv-config

[14] nox.parametrize(): https://nox.thea.codes/en/stable/config.html%23nox.parametrize#nox.parametrize

[15] pytest的parametrize: https://pytest.org/latest/parametrize.html%23_pytest.python.Metafunc.parametrize#_pytest.python.Metafunc.parametrize

[16] 参数化会话上: https://nox.thea.codes/en/stable/config.html%23parametrized#parametrized

[17] 贡献: https://nox.thea.codes/en/stable/CONTRIBUTING.html

