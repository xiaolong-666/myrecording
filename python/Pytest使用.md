# Pytest 使用

## 安装

```bash
pip3 install pytest allure-pytest pytest-cov
```



## 小笔记

1. `pytest`使用 `.` 标识测试成功（`PASSED`)
2. `pytest`使用 `F` 标识测试失败 (`FAILED`)
3. `pytest`使用 `s` 标识测试失败 (`SKIPPED`)
4. `pytest`使用 `x` 标识测试失败 (`XFAIL`)
5. 在`pytest`中，`assert` 是编写测试的最基础工具

### 捕获异常

在测试过程中，经常需要测试是否如期抛出预期的异常，以确定异常处理模块生效。在 pytest 中使用 `pytest.raises()` 进行异常捕获：

```python
def test_raise():
    with pytest.raises(TypeError) as e:
        connect('localhost', '6379')
    exec_msg = e.value.args[0]
    assert exec_msg == 'port type must be int'
```

### 参数化

参数化测试，即每组参数都独立执行一次测试。使用的工具就是 `pytest.mark.parametrize(argnames, argvalues)`。

密码长度的测试函数，其中参数名为 `passwd`，其可选列表包含三个值：

```python
@pytest.mark.parametrize('passwd',
                      ['123456',
                       'abcdefdfs',
                       'as52345fasdf4'])
def test_passwd_length(passwd):
    assert len(passwd) >= 8
```

### 固件（Fixture)

固件（Fixture）是一些函数，pytest 会在执行测试函数之前（或之后）加载运行它们。

Pytest 使用 `pytest.fixture()` 定义固件。

更多时候，我们希望一个固件可以在更大程度上复用，这就需要对固件进行集中管理。Pytest 使用文件 `conftest.py` 集中管理固件。

#### 预处理和后处理

很多时候需要在测试前进行预处理（如新建数据库连接），并在测试完成进行清理（关闭数据库连接）。当有大量重复的这类操作，最佳实践是使用固件来自动化所有预处理和后处理。

Pytest 使用 `yield` 关键词将固件分为两部分，`yield` 之前的代码属于预处理，会在测试前执行；`yield` 之后的代码属于后处理，将在测试完成后执行。

```python
@pytest.fixture()
def xiaolong_print():
    print("执行测试用例前，干活啦!")
    yield
    print("执行测试用例结束啦！")

def test_step1(xiaolong_print,monkeypatch):
    load_path = "/home/xiaolong/Downloads/iso/xiaolong-test.iso"
    test_args = ["load", load_path]
    monkeypatch.setattr("sys.stdin", StringIO("Y\n"))
    assert iso_tailor_test(test_args) == ResultCode.OK_CODE

```

执行时使用 `-s` 阻止消息被吞

```bash
platform linux -- Python 3.7.3, pytest-6.2.5, py-1.10.0, pluggy-1.0.0
rootdir: /home/xiaolong/WorkSpace/gerrit/isotailor
plugins: allure-pytest-2.9.45, cov-3.0.0
collected 3 items                                                                                                                 
tests/test_load.py 执行测试用例前，干活啦!
已载入镜像，是否覆盖当前镜像，请输入[Y/N]     正在载入镜像文件...
Parallel unsquashfs: Using 16 processors
...
.执行测试用例结束啦！
```

#### 作用域

固件的作用是为了抽离出重复的工作和方便复用，为了更精细化控制固件（比如只想对数据库访问测试脚本使用自动连接关闭的固件），pytest 使用作用域来进行指定固件的使用范围。

在定义固件时，通过 `scope` 参数声明作用域，可选项有：

- `function`: 函数级(默认)，每个测试函数都会执行一次固件；
- `class`: 类级别，每个测试类执行一次，所有方法都可以使用；
- `module`: 模块级，每个模块执行一次，模块内函数和方法都可使用；
- `session`: 会话级，一次测试只执行一次，所有被找到的函数和方法都可用。

```python
@pytest.fixture(scope='function')
def func_scope():
    pass

@pytest.fixture(scope='module')
def mod_scope():
    pass

@pytest.fixture(scope='session')
def sess_scope():
    pass

@pytest.fixture(scope='class')
def class_scope():
    pass
```

#### 自动执行

如果我们想让固件自动执行，可以在定义时指定 `autouse` 参数

```python
@pytest.fixture(autouse=True)
def timer_function_scope():
    start = time.time()
    yield
    print(' Time cost: {:.3f}s'.format(time.time() - start))
```

#### 使用固件

如果`fixture`有返回值，使用`usefixture`无法获取到返回值，这个是装饰器`usefixture`与用例直接传`fixture`参数的区别。

当`fixture`需要用到`return`出来的参数时，只能将参数名称直接当参数传入，不需要用到`return`出来的参数时，两种方式都可以。

```python
@pytest.fixture()
def test1():
    print("1111")
@pytest.mark.usefixtures('test1')
def test2():
    print("2222")
```

如果一个方法或者一个class用例想要同时调用多个`fixture`，可以使用`@pytest.mark.usefixture()`进行叠加。注意叠加顺序，先执行的放底层，后执行的放上层。

```python
@pytest.mark.usefixtures('test1')
@pytest.mark.usefixtures('test2')
def test3():
    print("333")
```





## 用例中交互如何处理

使用pytest的插件 `monkeypatch`

```python
def test_temp2(monkeypatch):
    import argparse
    from io import StringIO
    args = argparse.Namespace()
    args.loadpath = "/home/xiaolong/Downloads/iso/xiaolong-test.iso"
    args.subComnd = "load"
    monkeypatch.setattr('sys.stdin',StringIO("Y\n"))
    assert load(args) == 0
```



## 参考链接

[learning-pytest](https://learning-pytest.readthedocs.io/zh/latest/doc/intro/getting-started.html)
