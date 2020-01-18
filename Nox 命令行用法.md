# Nox 命令行用法

**英文原文** | [Command-line usage](https://nox.thea.codes/en/stable/usage.html)

## 调用方式

nox 通常是在命令行上被调用的：

```python
nox
```

你还可以通过 Python 解释器调用 nox：

```python
python3 -m nox
```

## 列出可用的会话

列出所有可用的会话，包括参数化的会话：

```python
nox -l
nox --list
nox --list-sessions
```

## 运行所有会话

你可以不带任何参数地执行 nox 来运行每个会话：  

```python
nox
```

会话被执行的顺序是它们在 noxfile 中出现的顺序。

## 指定一个或多个会话

默认情况下，nox 将运行在 noxfile 中定义的所有会话。但是，你可以选择使用`--session`、`-s` 或`-e` 运行特定的一组：

```python
nox --session tests
nox -s lint tests
nox -e lint
```

你还可以使用`NOXSESSION`环境变量：  

```python
NOXSESSION=lint nox
NOXSESSION=lint,tests nox
```

nox 将按照指定的顺序运行这些会话。

你还可以使用[pytest-风格的关键字](https://docs.pytest.org/en/latest/usage.html%23specifying-tests-selecting-tests#specifying-tests-selecting-tests)来过滤测试会话：

```python
nox -k "not lint"
nox -k "tests and not lint"
```

## 指定参数化的会话

如果你有参数化的会话，例如：  

```python
@nox.parametrize('django', ['1.9', '2.0'])
def tests(session, django):
    ...
```

那么运行`nox --session tests`，实际上将运行该会话的所有参数化版本。如果你要使用一组特定的参数化参数运行会话，则可以使用会话名称来指定它们：    

```python
nox --session "tests(django='1.9')"
nox --session "tests(django='2.0')"
```

## 重用虚拟环境

默认情况下，nox 在每次运行时都会删除并重新创建虚拟环境（virtualenv）。通常，对于大多数项目和持续集成环境而言，这都是很好的，因为[pip的缓存](https://pip.pypa.io/en/stable/reference/pip_install/%23caching#caching)使得重新安装相当快。但是，在某些情况下，在两次运行之间重用虚拟环境是更有利的。使用`-r`或`--reuse-existing-virtualenvs`：     

```python
nox -r
nox --reuse-existing-virtualenvs
```

如果 noxfile 设置了`nox.options.reuse_existing_virtualenvn`，你可以在命令行使用`--no-reuse-existing-virtualenvs` 覆盖 noxfile 的设置。  

## 如果有会话失败，则停止

默认情况下，即使一个会话失败，nox 也将继续运行所有会话。一旦第一个会话失败，你可以使用`--stop-on-first-error`来使 nox 中止：  

```python
nox --stop-on-first-error
```

如果 noxfile 设置了`nox.options.stop_on_first_error`，你可以在命令行中使用`--no-stop-on-first-error`覆盖 noxfile 的设置。

## 当缺失解释器时令会话失败

默认情况下，nox 将跳过找不到 Python 解释器的会话。如果你希望 nox 将这些会话标记为失败，你可以使用`--error-on-missing-interpreters`： 

```python
nox --error-on-missing-interpreters
```

如果 noxfile 设置了`nox.options.error_on_missing_interpreters`，你可以在命令行中使用`--no-error-on-missing-interpreters`覆盖 noxfile 设置。  

## 禁止外部程序

默认情况下，对于未在会话的虚拟环境中安装的程序，nox 会发出警告，但最终会允许你运行它。如果 nox 在非显式将`external = True` 传递给`session.run` 的情况下，还使用任意外部程序，则你可以使用`--error-on-external-run`来使它失败：

```python
nox --error-on-external-run
```

如果 noxfile 设置了`nox.options.error_on_external_run`，你可以在命令行中使用`--no-error-on-external-run`覆盖 noxfile 设置。  

## 指定其它配置文件

如果由于某种原因你的 noxfile 没有命名为 noxfile.py ，你可以使用`--noxfile` 或`-f` ：

```python
nox --noxfile something.py
nox -f something.py
```

## 将虚拟环境存储在其它目录中

默认情况下，nox 将虚拟环境存储在`./.nox`中，但是，你可以使用`--envdir`进行更改：

```python
nox --envdir /tmp/envs
```

## 跳过除安装命令外的所有内容

在很多情况下，仅需要 nox 运行安装命令，例如准备环境作离线测试，或者重新创建用于测试的虚拟环境。你可以使用`--install-only`跳过 run 命令。      

例如，给定这个 noxfile：

```python
@nox.session
def tests(session):
    session.install("pytest")
    session.install(".")
    session.run("pytest")
```

运行：

```python
nox --install-only
```

将同时运行两个 install 命令，但跳过 run 命令：

```python
nox > Running session tests
nox > Creating virtualenv using python3.7 in ./.nox/tests
nox > pip install pytest
nox > pip install .
nox > Skipping pytest run, as --install-only is set.
nox > Session tests was successful.
```

## 强制非交互行为

[session.interactive](https://nox.thea.codes/en/stable/config.html%23nox.sessions.Session.interactive#nox.sessions.Session.interactive)可用于判断 nox 是在交互式终端（例如一个实际的人在其计算机上运行它）还是在非交互式终端（例如一个连续集成系统）中运行。 

```python
@nox.session
def docs(session):
    ...

    if session.interactive:
        nox.run("sphinx-autobuild", ...)
    else:
        nox.run("sphinx-build", ...)
```

有时，需要强制 nox 将会话视为非交互式的。你可以使用`--non-interactive`参数来执行此操作：

```python
nox --non-interactive
```

这会使得`session.interactive`始终返回 False 。   

## 控制彩色输出

默认情况下，如果你在交互式终端中使用，则 nox 将输出彩色的日志。但是，如果要将`stderr`重定向到文件，或者不使用交互式终端，或者设置了环境变量`NO_COLOR`，则 nox 会以纯文本格式输出。    

你可以使用`--nocolor`和`--forcecolor`标志来手动控制 nox 的输出。    

例如，这将始终输出彩色日志：

```python
nox --forcecolor
```

但是，这将永远不会输出彩色日志：

```python
nox --nocolor
```

## 控制命令的详细程度

默认情况下，nox 仅显示失败的命令的输出，当给命令传递了`silent = False` 时，没有输出。通过将`--verbose`传递给 nox，无论 silent 参数如何，都会显示所有命令的所有输出。

## 输出机器可读的报告

你可以通过指定`--report`以`json`格式输出报告：   

```python
nox --report status.json
```

## Windows

nox 临时性支持在 Windows 上运行。但是，这取决于你的 Windows，Python 和虚拟环境的版本可能会出现问题。有关更多信息，请参见以下内容：

- [tox issue 260](https://github.com/tox-dev/tox/issues/260)
- [Python issue 24493](http://bugs.python.org/issue24493)
- [Virtualenv issue 774](https://github.com/pypa/virtualenv/issues/774)

Windows 上的 Python 二进制文件可通过 Windows 的 Python [启动器](https://docs.python.org/3/using/windows.html%23python-launcher-for-windows#python-launcher-for-windows)（py ）找到。例如，通过确定`py -3.5` 会调用哪个可执行文件，以此来找到 Python 3.5 。如果一个测试需要使用特定的 Python 的 32 位版本，则应使用`X.Y-32` 作为版本。      

## 从 tox 转化

nox 具有将 tox.ini 文件转换为 noxfile.py 文件的实验性支持。它还不支持 tox 的所有功能，仅用于完成过度转换的大部分机械工作，你可能仍需要对转换后的 noxfile.py 作一些修改。     

要使用转换器，请在安装 nox 时附上`tox_to_nox`：    

```python
pip install --upgrade nox[tox_to_nox]
```

然后，只需在 tox.ini 所在的目录中运行`tox-to-nox`：    

```python
tox-to-nox
```

这将基于 tox.ini 中的环境创建一个 noxfile.py。一些注意事项：   

- [生成环境](http://tox.readthedocs.io/en/latest/config.html%23generating-environments-conditional-settings#generating-environments-conditional-settings) 可以工作，但是会被转换为单独的环境。`tox-to-nox`不够聪明，无法将其转换为[参数化的](https://nox.thea.codes/en/stable/usage.html%23running-paramed-sessions#running-paramed-sessions)会话，但是手动提取通用配置以进行参数化应该很简单。     
- 由于 tox 解析其配置的方式，所有[替换项](http://tox.readthedocs.io/en/latest/config.html%23substitutions#substitutions) 会在转换时被引入。这意味着你需要用适当的变量替换 noxfile.py 中的静态字符串。    
- 几种不常用的 tox 选项尚未实现，但有可能实现。如果遇到你认为有用的功能，请提出功能请求（feature request）。

## shell 补齐

将适当的命令添加到 shell 的配置文件中，以便在启动时运行。你可能需要重启或重新登录，才能使自动补齐功能生效。

bash

```
eval "$(register-python-argcomplete nox)"
```

zsh

```python
# To activate completions for zsh you need to have
# bashcompinit enabled in zsh:
autoload -U bashcompinit
bashcompinit

# Afterwards you can enable completion for nox:
eval "$(register-python-argcomplete nox)"
```

tcsh

```python
eval `register-python-argcomplete --shell tcsh nox`
```

fish

```python
register-python-argcomplete --shell fish nox | .
```



 