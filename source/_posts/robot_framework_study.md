## 1. 简介
Robot Framework 是一个基于Python的，可扩展的关键字驱动的测试自动化框架。
它可以通过python 或java 编写库文件实现扩展。

注：目前介绍Robot Framework version 为>= 3.2.1

### 1.1. 架构
![robot framework structure](http://robotframework.org/robotframework/latest/images/architecture.png)

1. 测试数据是简单，易于编辑**表格格式**。
2. Robot Framework它会处理测试数据，执行测试用例并生成日志和报告。
3. 核心框架对测试中的目标一无所知，与它的交互由测试库处理。
4. 库可以直接使用应用程序接口，也可以使用低级测试工具作为驱动程序。

### 1.2. 安装，卸载与升级

在linux 环境下，具备pip tool 可以方便使用如下命令：
``` bash
# install
pip install roboframework

# install a specific version
pip install robotframework=2.9.2

# uninstall
pip uninstall roboframework

# upgrade
pip install --upgrade robotframework

# verifying installation, robo process testcase, rebot handle result like *.xml
robo --version

rebot --version
```

如果想使用java 或.net 进行lib 等工具的扩展，可以安装Jython 与 Ironpython。

## 2. 创建测试数据

### 2.1. 测试数据语法
#### 2.1.1. 文件与目录
测试用例是按照分层的形式构建： 
- 测试用例定义在测试文件中
- 包含测试用例的测试文件自动创建一个测试套件(test suite)
- 包含多个测试文件的目录，组成更高层的测试套件
- 测试套件目录也可以包含其他的测试套件目录

另外：
- test libraries 包含the lowest-level keywords
- resource files 包含variables 与 higher-level user keywords
- variable files 包含灵活的方式定义variables

注:
test case files, test suite initialization files, resource files 使用Robot Framework 语法。
test libraries, variable files 使用python, java, .net 等语言语法。

关键字 keywords 的来源就有：
- test libraries
- resource files
- test case files 中 keywords 段

#### 2.1.2. 测试数据段(test data section)
section | used for
:- | :-
Settings | 1) Importing test libraries, resource files and variable files.<br> 2) Defining metadata for test suites and test cases.
Section	Used for
Variables | Defining variables that can be used elsewhere in the test data.
Test Cases | Creating test cases from available keywords.
Tasks | Creating tasks using available keywords. Single file can only contain either tests or tasks.
Keywords | Creating user keywords from existing lower-level keywords
Comments | Additional comments or data. Ignored by Robot Framework.

定义字段推荐使用\*\*, 例如\*\*settings\*\*


#### 2.1.3. 测试文件格式
测试文件扩展名为.robot，目前支持三种格式
- 空格分隔格式
- 管道分隔格式
- reStructuredText格式

`空格分隔格式`
robot framework 以**两个及以上space， 一个及以上table** 作为token 分隔。
注：
如果需要在测试文件中使用多个空格，可以使用"\t", "\xA0", 以及内建的“${SPACE}”，“${EMPTY}”

```txt
*** Settings ***
Documentation     Example using the space separated format.
Library           OperatingSystem

*** Variables ***
${MESSAGE}        Hello, world!

*** Test Cases ***
My Test
    [Documentation]    Example test.
    Log    ${MESSAGE}
    My Keyword    ${CURDIR}

Another Test
    Should Be Equal    ${MESSAGE}    Hello, world!

*** Keywords ***
My Keyword
    [Arguments]    ${path}
    Directory Should Exist    ${path}
```

`管道分隔格式`
```txt
| *** Settings ***   |
| Documentation      | Example using the pipe separated format.
| Library            | OperatingSystem

| *** Variables ***  |
| ${MESSAGE}         | Hello, world!

| *** Test Cases *** |                 |               |
| My Test            | [Documentation] | Example test. |
|                    | Log             | ${MESSAGE}    |
|                    | My Keyword      | ${CURDIR}     |
| Another Test       | Should Be Equal | ${MESSAGE}    | Hello, world!

| *** Keywords ***   |                        |         |
| My Keyword         | [Arguments]            | ${path} |
|                    | Directory Should Exist | ${path} |
```
### 2.2. 创建测试用例（test case）
测试用例使用关键字组成，关键字可以来源于test libraries, resource files 以及测试文件中 Keywords 段。

第一行为测试用例名称，有效范围到下一个测试用例或者结尾处。
```txt
*** Test Cases ***
Valid Login
    Open Login Page
    Input Username    demo
    Input Password    mode
    Submit Credentials
    Welcome Page Should Be Open

Setting Variables
    Do Something    first argument    second argument
    ${value} =    Get Some Value
    Should Be Equal    ${value}    Expected value
```
在测试用例还可以有自定义的settings
- \[Documentation\]      用于执行时显示信息等内容
- \[Tags\]               用于分类统计
- \[Setup\], \[Teardown\] 测试用例执行前，后的动作
- \[Template\]
- \[Timeout\]

```txt
*** Test Cases ***
Valid Login
    [Documentation]    another test
    [Tags]    dummy owner-john
    [Setup]    Do something
    Open Login Page
    Input Username    demo
    [Teardown]    Do something
```

### 2.3. 创建tasks
与创建测试用例相似，都是使用keywords， 唯一不同的可能就是section的不同。 从\*\* Test Cases \*\* 变为\*\* Tasks \*\*

### 2.4. 使用test libraries
#### 2.4.1. 导入库文件
`using library setting`

```txt
*** Settings ***
Library    OperatingSystem
Library    my.package.TestLibrary
Library    MyLibrary    arg1    arg2
Library    ${LIBRARY}
```

`using Import Library keyword`

```txt
*** Test Cases ***
Example
    Do Something
    Import Library    MyLibrary    arg1    arg2
    KW From MyLibrary
```

`using physical path to libary`
```txt
*** Settings ***
Library    PythonLibrary.py
Library    /absolute/path/JavaLibrary.java
Library    relative/path/PythonDirLib/    possible    arguments
Library    ${RESOURCES}/Example.class
```

#### 2.4.2. 可用库
robot framework 包含的标准库
- [BuiltIn](http://robotframework.org/robotframework/latest/libraries/BuiltIn.html)
- [Collections](http://robotframework.org/robotframework/latest/libraries/Collections.html)
- [DateTime](http://robotframework.org/robotframework/latest/libraries/DateTime.html)
- [Dialogs](http://robotframework.org/robotframework/latest/libraries/Dialogs.html)
- [OperatingSystem](http://robotframework.org/robotframework/latest/libraries/OperatingSystem.html)
- [Process](http://robotframework.org/robotframework/latest/libraries/Process.html)
- [Screenshot](http://robotframework.org/robotframework/latest/libraries/Screenshot.html)
- [String](http://robotframework.org/robotframework/latest/libraries/String.html)
- [Telnet](http://robotframework.org/robotframework/latest/libraries/Telnet.html)
- [XML](http://robotframework.org/robotframework/latest/libraries/XML.html)

更多可在robot framework 官网上查看到， 例如external libraries, other libraries。

### 2.5. variables
支持变量类型有：
- scalar, ${SCALAR}
- list, @{LIST}
- dictionary, &{DICT}
- environment variable %{ENV_VAR}

在Variables 段中定义变量：
```txt
*** Variables ***
${NAME}         Robot Framework
@{NAMES}        Matti       Teppo
&{USER 1}       name=Matti    address=xxx         phone=123
```

keyword 也可以返回变量值：
```txt
*** Test Cases ***
Example
    ${x} =    Get X    an argument
    @{list} =    Create List    first    second    third
    &{dict} =    Create Dictionary    first=1    second=${2}    ${3}=third
    {a}    ${b}    ${c} =    Get Three
```

robot framework 也有内建的变量，具体可查看官网[built-in-variables](http://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#built-in-variables)

### 2.7. 创建user keywords
keywords 语法与test case 等大致相同，同样在keyword setting 部分可以进行设定：
- \[Documentation\]
- \[Tags\]
- \[Arguments\] 表明传递给keyword 的参数，也可指定默认值
- \[Return\] keyword 的返回值
- \[Teardown\]
- \[Timeout\]

```txt
*** Keywords ***
Open Login Page
    [Documentation]    One line documentation.
    Open Browser    http://host/login.html
    Title Should Be    Login Page

Title Should Start With
    [Arguments]    ${expected} ${var}=1
    ${title} =    Get Title
    [Return] ${tiltle}

With Teardown
    Do Something
    [Teardown]    Log    keyword teardown    
```

## 3. 执行测试用例

### 3.1. 运行
```bash
robot tests.robot
robot path/to/my_tests/

# filter by tags
#
# And or &
# OR
# NOT
#
# Matches tests containing tags 'foo' and 'bar'.
robot --include fooANDbar path/my_tests/
robot --include foo&bar path/my_tests/
```
### 3.2. 执行结果
一般输出结果到output.xml, report 与LOG 将会输出到HTML文件。

```txt
==============================================================================
Example test suite
==============================================================================
First test :: Possible test documentation                             | PASS |
------------------------------------------------------------------------------
Second test                                                           | FAIL |
Error message is displayed here
==============================================================================
Example test suite                                                    | FAIL |
2 critical tests, 1 passed, 1 failed
2 tests total, 1 passed, 1 failed
==============================================================================
Output:  /path/to/output.xml
Report:  /path/to/report.html
Log:     /path/to/log.html
```

### 3.3. 执行流程
robot framework 从测试套件目录中按照字典排序的顺序依次执行测试文件。 当然我们也可以使用option 进行干预, 例如这些：
1. --include
2. --exclude
3. --randomize

### 3.4. 对输出结果的处理
#### 3.4.1. 使用Rebot

```bash
rebot output.xml

# combining outputs
rebot outputs/*.xml
rebot output1.xml output2.xml
```
### 3.5. 创建输出文件
使用robot 的option 可以设定输出文件格式，目录等。

**Log file**
option --log (-l) 可以指定LOG file。
![log file](http://robotframework.org/robotframework/latest/images/log_passed.png)

**Report file**
option --report (-r)

![report file](http://robotframework.org/robotframework/latest/images/report_passed.png)

## 4. 扩展Robot Framework
支持扩展的语言有：
- python
- java, 当使用Jython 语言框架时
- c, 使用python c API


### 4.1. libraries 中什么将会成为keywords
在Python 中若没有其他限制，除了下划线定义的函数都会判定为keywords， java 中public 函数会成为keywords.

```python
class MyLibrary:

    def my_keyword(self, arg):
        return self._helper_method(arg)

    def _helper_method(self, arg):
        return arg.upper()
```

```java
public class MyLibrary {

    public String myKeyword(String arg) {
        return helperMethod(arg);
    }

    private String helperMethod(String arg) {
        return arg.toUpperCase();
    }
}
```

当然我们可以是用python 中`ROBOT_AUTO_KEYYWORDS = FALSE`, 我们也可以使用装饰器`library`, `keyword`指定为库文件、关键字， `not_keyword`指定为不是关键字

```python
class Example
    ROBOT_AUTO_KEYYWORDS = FALSE

@library
class Example:

    @keyword
    def this_is_keyword(self):
        pass

    @not_keyword
    def this_is_not_keyword():
        pass

def return_two_values():
    return 'first value', 'second value'
```

### 4.2. Remote library interface
Robot Framework 支持使用远程调用库函数。

![](http://robotframework.org/robotframework/latest/images/remote.png)

```txt
*** Settings ***
Library    Remote    http://127.0.0.1:8270       WITH NAME    Example1
Library    Remote    http://example.com:8080/    WITH NAME    Example2
Library    Remote    http://10.0.0.2/example    1 minute    WITH NAME    Example3
```

### 4.3. Listener interface
Robot Framework 支持监听test case何时开始、结束等。

robot 可以从命令行指定监听class。
```bash
robot --listener listener.py arg1 arg2 tests.robot
```

## 参考
[Robot Framework官方教程](https://www.jianshu.com/p/c3a9d20db4e5)
[Robot Framework user guide](http://robotframework.org/robotframework/#user-guide)
[Robot Framework quickstart](https://github.com/robotframework/QuickStartGuide/blob/master/QuickStart.rst)
[Robot Framework org](https://robotframework.org/)
[Robot Framework and RIDE](https://www.cnblogs.com/lsdb/p/10861344.html)
[GITHUB RIDE](https://github.com/robotframework/RIDE)
