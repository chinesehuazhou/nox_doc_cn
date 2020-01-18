# nox 的配置与 API

**英文原文** | [Configuration & API](https://nox.thea.codes/en/stable/config.html)



## Noxfile

Nox 默认在一个名为`noxfile.py`的文件中查找配置。在运行 nox 时，你可以使用 `--noxfile`参数指定其它的文件。

## 定义会话

格式：session(func=None, python=None, py=None, reuse_venv=None, name=None, venv_backend=None)，将被装饰的函数指定为一个会话。


Nox 会话是通过被`@nox.session`装饰的标准 Python 函数来配置的。例如:

```python
import nox

@nox.session
def tests(session):
    session.run('pytest')
```

## 会话描述


你可以使用[文档字符串](https://www.python.org/dev/peps/pep-0257)向会话中添加一个描述。第一行内容会在列出会话时显示。例如:

```python
import nox

@nox.session
def tests(session):
    """Run the test suite."""
    session.run('pytest')
```

`nox --list`命令将显示出:

```python
$ nox --list
Available sessions:
* tests -> Run the test suite.
```

## 会话名称

默认情况下，Nox 使用被装饰函数的名称作为会话的名称。这对于绝大多数项目都非常有效，但是，如果需要，你也可以使用 @nox.session 的 name 参数来自定义会话的名称。例如:

```python
import nox

@nox.session(name="custom-name")
def a_very_long_function_name(session):
    print("Hello!")
```

`nox --list` 命令将显示：

```python
$ nox --list
Available sessions:
* custom-name
```

你可以告诉 nox 使用自定义的名称运行会话:

```python
$ nox --session "custom-name"
Hello!
```

## 配置会话的virtualenv

默认情况下，Nox 在为每个会话创建一个新的 virtualenv 时，会使用 Nox 所用的同一个解释器。如果你使用 Python 3.6 安装了 nox，则 nox 将默认在所有会话中使用 Python 3.6。


通过给 @nox.session 指定 python 参数(或其别名 py)，你可以告诉 nox 使用不同的 Python 解释器/版本:

```python
@nox.session(python='2.7')
def tests(session):
    pass
```

你还可以告诉 Nox 使用多个 Python 解释器运行你的会话。Nox 将为指定的每个解释器创建一个单独的 virtualenv 并运行会话。例如，下面的会话将运行两次——一次使用 Python 2.7，一次使用 Python 3.6:

```python
@nox.session(python=['2.7', '3.6'])
def tests(session):
    pass
```

当你提供一个版本号时，Nox 会自动添加 python 来确定可执行文件的名称。但是，Nox 也可以接受完整的可执行名称。如果你想使用 pypy 来测试，例如:

```python
@nox.session(python=['2.7', '3.6', 'pypy-6.0'])
def tests(session):
    pass
```

当准备你的会话时，Nox 将为每个解释器创建单独的会话。你可以在运行 nox --list 的时候看到这些会话。例如这个 Noxfile:

```python
@nox.session(python=['2.7', '3.5', '3.6', '3.7'])
def tests(session):
    pass
```

将产生这些会话:

```python
* tests-2.7
* tests-3.5
* tests-3.6
* tests-3.7
```

注意，这个扩展发生在参数化之前，所以你仍然可以对多个解释器的会话进行参数化。


如果你想完全禁止创建 virtualenv，你可以设置 python 参数为 False:

```python
@nox.session(python=False)
def tests(session):
    pass
```


最后，你还可以指定每次都重用 virtualenv，而不是重新创建:

```python
@nox.session(
    python=['2.7', '3.6'],
    reuse_venv=True)
def tests(session):
    pass
```

## 将参数传入会话

通常往测试会话中传递参数是很有用的。下面是一个简单示例，演示了如何使用参数对特定文件作测试：

```python
@nox.session
def test(session):
    session.install('pytest')

    if session.posargs:
        test_files = session.posargs
    else:
        test_files = ['test_a.py', 'test_b.py']

    session.run('pytest', *test_files)
```


现在如果你运行:

```python
nox
```

那么 nox 将运行:

```python
pytest test_a.py test_b.py
```


但如果你运行:

```python
nox -- test_c.py
```


那么 nox 将运行:

```python
pytest test_c.py
```

## 参数化会话

会话的参数可以用`nox.parametrize()` 装饰器来作参数化。下面是一个典型的参数化安装 Django 版本的例子：

```python
@nox.session
@nox.parametrize('django', ['1.9', '2.0'])
def tests(session, django):
    session.install(f'django=={django}')
    session.run('pytest')
```

当你运行`nox`时，它会创建两个不同的会话：

```python
$ nox
nox > Running session tests(django='1.9')
nox > pip install django==1.9
...
nox > Running session tests(djano='2.0')
nox > pip install django==2.0
```

`nox.parametrize()` 的接口和用法故意跟[pytest的参数化](https://pytest.org/latest/parametrize.html#_pytest.python.Metafunc.parametrize) 相类似。

格式：parametrize(*arg_names*, *arg_values_list*, *ids=None*)

作用是参数化一个会话。

将 arg_values_list 列表赋给对应的 arg_names，为装饰的会话函数添加新的调用。参数化在会话发现期间执行，每次调用都作为 nox 的单个会话出现。

参数：

- **arg_names** (*Sequence[str]*)——一系列参数名称
- **arg_values_list** (*Sequence[Union[Any, Tuple]]*)——参数值列表决定了使用不同参数值调用会话的频率。如果只指定了一个参数名，那么这就是一个简单的值列表，例如[1,2,3]。如果指定了 N 个参数名，这必须是一个 N 元组的列表，其中每个元素为其各自的参数名指定一个值，例如 [(1,'a'), (2,'b')]。
- **ids** (*Sequence[str]*) ——可选项，一系列测试 id，被参数化的参数使用。

你也可以堆叠装饰器，令其产生组合了参数的会话，例如:

```python
@nox.session
@nox.parametrize('django', ['1.9', '2.0'])
@nox.parametrize('database', ['postgres', 'mysql'])
def tests(session, django, database):
    ...
```

如果运行`nox —list`，你将看到它生成了以下的会话集:

```python
* tests(database='postgres', django='1.9')
* tests(database='mysql', django='1.9')
* tests(database='postgres', django='2.0')
* tests(database='mysql', django='2.0')
```

如果你只想运行一个参数化会话，请参阅"指定参数化会话"部分。

## 为参数化的会话起友好的名称

自动生成的参数化会话的名称，如`tests(django='1.9', database='postgres')`，即使用关键字过滤，也可能很长且很难处理。

在此场景中，可以为参数化会话提供辅助的自定义 id 。这两个例子是等价的:

```python
@nox.session
@nox.parametrize('django',
    ['1.9', '2.0'],
    ids=['old', 'new'])
def tests(session, django):
    ...
```

```python
@nox.session
@nox.parametrize('django', [
    nox.param('1.9', id='old'),
    nox.param('2.0', id='new'),
])
def tests(session, django):
    ...
```

当运行`nox --list`时，你将看到它们的新 id:

```python
* tests(old)
* tests(new)
```

你可以用`nox --sessions "tests(old)"`，以此类推。

这也适用于堆叠参数化。id 是在组合期间组合的。例如:

```python
@nox.session
@nox.parametrize(
    'django',
    ['1.9', '2.0'],
    ids=["old", "new"])
@nox.parametrize(
    'database',
    ['postgres', 'mysql'],
    ids=["psql", "mysql"])
def tests(session, django, database):
    ...
```

运行`nox --list`时会产生这些会话:

```python
* tests(psql, old)
* tests(mysql, old)
* tests(psql, new)
* tests(mysql, new)
```

## 会话对象

Nox 将使用 Session 类的一个实例来调用你的会话函数。

*class Session(runner)* [¶](https://nox.thea.codes/en/stable/config.html#nox.sessions.Session)

会话对象被传递到用户自定义的每个会话函数中。

这是在 Nox 会话中安装软件包和运行命令的主要途径。

- `bin`[¶](https://nox.thea.codes/en/stable/config.html#nox.sessions.Session.bin)——virtualenv 的 bin 目录



- `cd`(*dir*)[¶](https://nox.thea.codes/en/stable/config.html#nox.sessions.Session.cd)——[chdir()](https://nox.thea.codes/en/stable/config.html#nox.sessions.Session.chdir) 的一个别名


- `chdir`(*dir*)[¶](https://nox.thea.codes/en/stable/config.html#nox.sessions.Session.chdir)——更改当前的工作目录


- `conda_install`(*args, **kwargs)[¶](https://nox.thea.codes/en/stable/config.html#nox.sessions.Session.conda_install)

  调用[conda install](https://nox.thea.codes/en/stable/config.html#conda-install)来在会话环境中的安装软件包。

  直接安装软件包:

  ```python
  session.conda_install('pandas')
  session.conda_install('numpy', 'scipy')
  session.conda_install('--channel=conda-forge', 'dask==2.1.0')
  ```

  根据 requirements.txt 文件来安装软件包：

  ```python
  session.conda_install('--file', 'requirements.txt')
  session.conda_install('--file', 'requirements-dev.txt')
  ```

  不破坏 conda 已安装的依赖而安装软件包：

  ```python
  session.install('.', '--no-deps')
  # Install in editable mode.
  session.install('-e', '.', '--no-deps')
  ```

  剩下的关键字参数跟 [run()](https://nox.thea.codes/en/stable/config.html#nox.sessions.Session.run) 相同。

- `env`[¶](https://nox.thea.codes/en/stable/config.html#nox.sessions.Session.env)——一个环境变量的字典，传给所有的命令。


- `error`(*args, **kwargs)[¶](https://nox.thea.codes/en/stable/config.html#nox.sessions.Session.error)——立即中止会话并随意地记录一个错误。


- `install`(*args, **kwargs)[¶](https://nox.thea.codes/en/stable/config.html#nox.sessions.Session.install) ——调用 [pip](https://pip.readthedocs.org/) 在会话的 virtualenv 里安装包。

  直接安装包：

  ```python
  session.install('pytest')
  session.install('requests', 'mock')
  session.install('requests[security]==2.9.1')
  ```

  根据 requirements.txt 文件来安装软件包：

  ```python
  session.install('-r', 'requirements.txt')
  session.install('-r', 'requirements-dev.txt')
  ```

  安装当前的包：

  ```python
  session.install('.')
  # Install in editable mode.
  session.install('-e', '.')
  ```

  剩下的关键字参数跟 run() 相同。

- `interactive`[¶](https://nox.thea.codes/en/stable/config.html#nox.sessions.Session.interactive) ——如果 Nox 在交互式会话中运行，则返回 True，否则返回 False。

- `log`(*args, **kwargs)[¶](https://nox.thea.codes/en/stable/config.html#nox.sessions.Session.log)——在会话期间输出一份日志。


- `notify`(*target*)[¶](https://nox.thea.codes/en/stable/config.html#nox.sessions.Session.notify) ——将给定的会话放在队列的末尾。

  此方法是幂等的；对同一会话的多次通知无效。

  参数：**target** (*Union[str, Callable]*)——需要通知的会话。这可以指定适当的字符串(与`nox -s` 的使用相同)或使用函数对象。

- `posargs`[¶](https://nox.thea.codes/en/stable/config.html#nox.sessions.Session.posargs) ——用于设置从命令行上传给 nox 的额外参数。


- `python`[¶](https://nox.thea.codes/en/stable/config.html#nox.sessions.Session.python) ——传给`@nox.session`的 Python 版本。

- `run`(*args, env=None, kwargs*)[¶](https://nox.thea.codes/en/stable/config.html#nox.sessions.Session.run) ——运行一个命令。

  命令必须安装字符串列表指定，例如：

  ```python
  session.run('pytest', '-k', 'fast', 'tests/')
  session.run('flake8', '--import-order-style=google')
  ```

  你不能把所有东西都当作一个字符串传递。例如，不可以这样：

  ```python
  session.run('pytest -k fast tests/')
  ```

  你可以用`env` 为命令设置环境变量：

  ```python
  session.run(
      'bash', '-c', 'echo $SOME_ENV',
      env={'SOME_ENV': 'Hello'})
  ```

  你还可以使用`success_codes` ，告诉 nox 将非零退出码视为成功。例如，如果你想将 pytest 的“tests discovered, but none selected”错误视为成功：

  ```python
  session.run(
      'pytest', '-k', 'not slow',
      success_codes=[0, 5])
  ```

  在 Windows 上，像`del`这样的内置命令不能直接调用，但是你可以使用`cmd /c` 来调用它们：

  ```python
  session.run('cmd', '/c', 'del', 'docs/modules.rst')
  ```

  参数：

  + **env** (*dict or None*)——用于向命令公开的环境变量字典。默认情况下，传递所有环境变量。
  + **silent** (*bool*) ——静默命令输出，除非命令失败。默认为 False。
  + **success_codes** (*list, tuple, or None*)——一系列被认为是成功的返回码。默认情况下，只有 0 被认为是成功的。
  + **external** (*bool*) ——如果为 False(默认值)，那么不在 virtualenv 路径中的程序将发出告警。如果为 True，则不会发出告警。这些告警可以使用`--error-on-external-run`将其转换为错误。这对没有 virtualenv 的会话没有影响。

- `skip`(*args, **kwargs)[¶](https://nox.thea.codes/en/stable/config.html#nox.sessions.Session.skip) ——立即跳出会话，并随意记录一个告警。

- `virtualenv`[¶](https://nox.thea.codes/en/stable/config.html#nox.sessions.Session.virtualenv) ——运行所有命令的 virtualenv。

## 修改 Noxfile 中的 Nox 行为

Nox 有各种命令行参数，可用于修改其行为。其中一些还可以在 Noxfile 中使用 nox.options 指定。例如，如果你想将 Nox 的 virtualenvs 存储在不同的目录中，而不需要每次都将它传递给 nox：

```python
import nox

nox.options.envdir = ".cache"

@nox.session
def tests(session):
    ...
```

或者，如果你想提供一组默认运行的会话：

```python
import nox

nox.options.sessions = ["lint", "tests-3.6"]

...
```

以下的选项可以在 Noxfile 中指定：

- `nox.options.envdir` 等同于指定 [–envdir](https://nox.thea.codes/en/stable/usage.html#opt-envdir).
- `nox.options.sessions` 等同于指定 [-s or –sessions](https://nox.thea.codes/en/stable/usage.html#opt-sessions-and-keywords).
- `nox.options.keywords` 等同于指定 [-k or –keywords](https://nox.thea.codes/en/stable/usage.html#opt-sessions-and-keywords).
- `nox.options.reuse_existing_virtualenvs` 等同于指定 [–reuse-existing-virtualenvs](https://nox.thea.codes/en/stable/usage.html#opt-reuse-existing-virtualenvs) 。通过在调用时指定 `--no-reuse-existing-virtualenvs` ，你可以强制取消它。
- `nox.options.stop_on_first_error` 等同于指定 [–stop-on-first-error](https://nox.thea.codes/en/stable/usage.html#opt-stop-on-first-error). 通过在调用时指定 `--no-stop-on-first-error`，你可以强制取消它。
- `nox.options.error_on_missing_interpreters` 等同于指定 [–error-on-missing-interpreters](https://nox.thea.codes/en/stable/usage.html#opt-error-on-missing-interpreters) 。通过在调用时指定 `--no-error-on-missing-interpreters` ，你可以强制取消它。
- `nox.options.error_on_external_run` 等同于指定 [–error-on-external-run](https://nox.thea.codes/en/stable/usage.html#opt-error-on-external-run). 通过在调用时指定 `--no-error-on-external-run` ，你可以强制取消它。
- `nox.options.report` 等同于指定 [–report](https://nox.thea.codes/en/stable/usage.html#opt-report)。

在调用 nox 时，命令行上指定的任何选项都优先于 Noxfile 中指定的选项。如果在命令行上指定了`--sessions`或`--keywords`，那么在 Noxfile 中指定的两个选项都将被忽略。



