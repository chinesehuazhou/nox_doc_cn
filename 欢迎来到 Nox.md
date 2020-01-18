# Nox 欢迎你

英文原文 | [Welcome to Nox](https://nox.thea.codes/en/stable/)

nox 是一个命令行工具，可以在多个 Python 环境中作自动化测试，类似于[tox](https://tox.readthedocs.org/)。但与 tox 不同，nox 使用标准的 Python 文件进行配置。  

使用[pip](https://pip.readthedocs.org/)安装 nox ： 

```python
pip install --user --upgrade nox
```

通过项目目录中的 noxfile.py 文件配置 nox 。这是一个运行 lint 和一些测试的简单 noxfile：

```python
import nox

@nox.session
def tests(session):
    session.install('pytest')
    session.run('pytest')

@nox.session
def lint(session):
    session.install('flake8')
    session.run('flake8', '--import-order-style', 'google')
```

要运行这两个会话，只需运行：

```python
nox
```

对于每个会话，nox 将使用适当的解释器自动创建[virtualenv](https://virtualenv.readthedocs.org/)，安装指定的依赖项，然后按顺序运行命令。  

要了解如何安装和使用 nox，请参阅[教程](https://github.com/chinesehuazhou/nox_doc_cn/blob/master/Nox%20%E6%95%99%E7%A8%8B.md)。有关配置会话的文档，请参阅《[配置与API](https://github.com/chinesehuazhou/nox_doc_cn/blob/master/Nox%20%E7%9A%84%E9%85%8D%E7%BD%AE%E4%B8%8E%20API.md)》。有关运行 nox 的文档，请参阅 [命令行用法](https://github.com/chinesehuazhou/nox_doc_cn/blob/master/Nox%20%E5%91%BD%E4%BB%A4%E8%A1%8C%E7%94%A8%E6%B3%95.md) 。    

## 使用 nox 的项目

nox 很幸运，有几个出色的项目使用了它，并提供了反馈和贡献。

- [Bezier](https://github.com/dhermes/bezier)
- [gapic-generator-python](https://github.com/googleapis/gapic-generator-python)
- [gdbgui](https://github.com/cs01/gdbgui)
- [Google Assistant SDK](https://github.com/googlesamples/assistant-sdk-python)
- [google-cloud-python](https://github.com/googlecloudplatform/google-cloud-python)
- [google-resumable-media-python](https://github.com/GoogleCloudPlatform/google-resumable-media-python)
- [OpenCensus Python](https://github.com/census-instrumentation/opencensus-python)
- [packaging.python.org](https://github.com/pypa/python-packaging-user-guide/)
- [pipx](https://github.com/pipxproject/pipx/)
- [Salt](https://github.com/saltstack/salt)
- [Subpar](https://github.com/google/subpar)
- [Urllib3](https://github.com/urllib3/urllib3)
- [Zazo](https://github.com/pradyunsg/zazo)

## 其它有用的项目

nox 不是同类工具中唯一的一个。如果 nox 不太满足你的需求，或者你想进行更多研究，建议你使用以下工具：

- [tox](https://tox.readthedocs.org/)是用于管理多个 Python 测试环境的事实标准，并且是 nox 的直接精神祖先。 
- [Invoke](https://www.pyinvoke.org/)是通用性的任务执行库，类似于 Make。可以认为 nox 就像是 Invoke 专门为 Python 测试而量身定制的，对于超出 nox 的特性的脚本来说，Invoke 是一个很好的选择。

## 维护者和贡献者

nox 是个免费的开源软件，由社区维护者和贡献者成就。

我们的维护者为（按字母顺序）：

- [Chris Wilcox](https://github.com/crwilcox)
- [Danny Hermes](https://github.com/dhermes)
- [Luke Sneeringer](https://github.com/lukesneeringer)
- [Santos Gallegos](https://github.com/stsewd)
- [Thea Flowers](https://github.com/theacodes)

由于[社区](https://github.com/theacodes/nox/graphs/contributors)提供了各种补丁和贡献工作，因此 nox 才存在。如果你想参与其中，请参阅[贡献](https://nox.thea.codes/en/stable/CONTRIBUTING.html) 部分。我们使用[Open Collective](https://opencollective.com/python-nox)向我们的贡献者支付报酬。

 