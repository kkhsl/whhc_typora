**https://www.bilibili.com/video/BV1qW4y1a7fU**(黑马程序员)

# 1.安装：linux

### 方法 1：使用 `yum` 安装 Python 3

1. **更新包管理器**：

   ```
   sudo yum update
   ```

2. **安装 EPEL 仓库**（Extra Packages for Enterprise Linux）：

   ```
   sudo yum install epel-release
   ```

3. **安装 Python 3**：

   ```
   sudo yum install python3
   ```

4. **验证安装**：

   ```
   python3 --version
   ```

5. **安装 `pip3`**：

   ```
   sudo yum install python3-pip
   ```

6. **验证 `pip3` 安装**：

   ```
   pip3 --version
   ```



### 方法 2：从源代码编译安装

1. **安装必要的依赖**：

   ```
   sudo yum groupinstall -y "Development Tools"
   sudo yum install -y openssl-devel bzip2-devel libffi-devel zlib-devel
   ```

2. **下载 Python 源代码**：

   ```
   cd /usr/src
   sudo wget https://www.python.org/ftp/python/3.10.0/Python-3.10.0.tgz
   sudo tar xzf Python-3.10.0.tgz
   cd Python-3.10.0
   ```

3. **配置和编译 Python**：

   ```
   sudo ./configure --enable-optimizations
   sudo make altinstall
   ```

   注意：使用 `altinstall` 而不是 `install` 以避免覆盖系统默认的 Python 版本。

4. **验证安装**：

   ```
   python3.10 --version
   ```

5. **安装 `pip`**（如果未随 Python 一起安装）：

   ```
   sudo /usr/local/bin/python3.10 -m ensurepip
   sudo /usr/local/bin/python3.10 -m pip install --upgrade pip
   ```

   

学习 Python 的路线可以根据你的目标和背景有所不同，但通常可以分为以下几个阶段：基础知识、进阶技能、专业领域和项目实践。下面是一个详细的 Python 学习路线图，适用于初学者和有一定编程基础的人。

.

### 阶段 1：基础知识

1. **Python 环境配置**
   - 安装 Python（最新版本）
   - 配置 IDE（例如 VS Code、PyCharm 或 Jupyter Notebook）
2. **Python 基础语法**
   - 变量和数据类型（整数、浮点数、字符串、布尔值）
   - 基本输入输出（`input()` 和 `print()`）
   - 运算符（算术运算符、比较运算符、逻辑运算符）
   - 控制结构（条件语句 `if-elif-else`、循环语句 `for` 和 `while`）
3. **数据结构**
   - 列表（List）
   - 元组（Tuple）
   - 字典（Dictionary）
   - 集合（Set）
4. **函数**
   - 定义和调用函数
   - 参数传递（位置参数、关键字参数、默认参数）
   - 返回值
5. **文件操作**
   - 读取和写入文件（文本文件和二进制文件）
6. **错误和异常处理**
   - 异常类型
   - `try-except` 块
   - 自定义异常

### 阶段 2：进阶技能

1. **模块和包**
   - 导入模块和包
   - 创建自己的模块和包
2. **面向对象编程（OOP）**
   - 类和对象
   - 属性和方法
   - 继承和多态
   - 封装和私有属性
   - 魔术方法（例如 `__init__`、`__str__`）
3. **标准库**
   - `datetime` 模块
   - `os` 和 `sys` 模块
   - `math` 和 `random` 模块
   - `collections` 模块
   - `itertools` 模块
4. **第三方库**
   - 安装第三方库（使用 `pip`）
   - 常用第三方库（如 `requests`、`beautifulsoup4`、`numpy`、`pandas`）

### 阶段 3：专业领域

1. **数据科学与分析**
   - `numpy`：科学计算
   - `pandas`：数据处理和分析
   - `matplotlib` 和 `seaborn`：数据可视化
   - `scikit-learn`：机器学习
2. **Web 开发**
   - `Flask` 或 `Django`：Web 框架
   - 前端基础（HTML、CSS、JavaScript）
   - RESTful API 开发
3. **自动化与脚本编写**
   - 使用 `selenium` 进行网页自动化
   - 使用 `openpyxl` 或 `pandas` 操作 Excel
   - 自动化文件和系统任务
4. **人工智能与机器学习**
   - `tensorflow` 和 `keras`：深度学习
   - `pytorch`：深度学习
   - 经典机器学习算法（回归、分类、聚类）
5. **网络爬虫**
   - `requests` 和 `beautifulsoup4`：网页请求和解析
   - `scrapy`：高级网络爬虫框架

### 阶段 4：项目实践

1. **小项目**
   - 数字猜谜游戏
   - 记账本
   - 简单的 Web 应用（例如博客）
2. **中型项目**
   - 数据分析项目（例如分析 COVID-19 数据）
   - Web 爬虫项目（例如抓取电商网站的数据）
   - 自动化项目（例如邮件自动发送）
3. **大型项目**
   - 完整的 Web 应用（例如电商网站）
   - 数据科学项目（例如预测股市价格）
   - 机器学习项目（例如图像分类）

### 学习资源

- **书籍**
  - 《Python 编程：从入门到实践》 (Python Crash Course)
  - 《流畅的 Python》 (Fluent Python)
  - 《Python 数据科学手册》 (Python Data Science Handbook)
  - 《深入理解计算机系统》 (Computer Systems: A Programmer's Perspective)
- **在线课程**
  - Coursera 的 Python for Everybody
  - Udemy 的 Complete Python Bootcamp
  - edX 的 Introduction to Computer Science using Python (MITx 6.00.1x)
- **在线文档**
  - [Python 官方文档](https://docs.python.org/3/)
  - [Real Python](https://realpython.com/)
  - GeeksforGeeks

### 总结

学习 Python 是一个循序渐进的过程，从基础知识开始，逐步深入到进阶技能和专业领域。通过不断实践项目，能加深对 Python 的理解和应用能力。根据个人兴趣和职业发展方向，选择合适的专业领域进行深入学习，将帮助你在 Python 编程之路上不断进步。



# 字面量

Python中常用的6种值(数据)的类型

| 类型             | 描述                                                     | 说明                          |
| ---------------- | -------------------------------------------------------- | ----------------------------- |
| 数字             | 整数(int)、浮点数（flot）、复数（complex）、布尔（bool） | 复数（4+3j），以j结尾表示复数 |
| 字符串           | 描述文本的一种数据类型                                   |                               |
| 列表（List）     | 有序的可变序列                                           |                               |
| 元组(Tuple)      | 有序的不可变序列                                         |                               |
| 集合(Set)        | 无序不重复序列                                           |                               |
| 字典(Dictionary) | 无序key-value集合                                        |                               |

# 1.安装pip

如果 `pip` 没有安装，你可以通过以下步骤进行安装：

#### 方法 1: 使用 `get-pip.py`

1. **下载**:

   - 打开浏览器，访问 

     get-pip.py

      页面，并将内容保存为 

     ```
     get-pip.py
     ```

      文件。你也可以直接使用 

     ```
     curl
     ```

      命令下载：

     ```
     curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
     ```

2. **运行 `get-pip.py`**:

   - 在命令提示符中，导航到你保存 

     ```
     get-pip.py
     ```

      文件的目录，然后运行：

     ```
     python get-pip.py
     ```

   - 这将安装 `pip` 并将其添加到你的 Python 环境中。

3. **验证安装**:

   - 输入以下命令确认 

     ```
     pip
     ```

      已成功安装：

     ```
     pip --version
     ```

#### 方法 2: 使用 Python 安装程序（如果安装 Python 时没有选择安装 `pip`）

1. **重新运行 Python 安装程序**:

   - 找到并运行你用于安装 Python 的安装程序。

2. **选择“修复”**:

   - 选择“修改”或“修复”选项，这将允许你修改现有的 Python 安装。

3. **确保选中 `pip`**:

   - 确保在安装选项中勾选了 `pip`。然后继续安装，安装程序会添加 `pip` 到你的 Python 环境中。

4. **验证安装**:

   - 安装完成后，打开命令提示符，输入以下命令确认 

     ```
     pip
     ```

      已成功安装：

     ```
     pip --version
     ```

### 配置环境变量（如果需要）

如果 `pip` 安装后仍然无法识别，可能是因为其安装目录未添加到系统环境变量中。可以手动添加 `pip` 路径到系统 `PATH` 环境变量中：

1. **找到 Python 安装目录**:
   - 通常，`pip` 安装在 Python 的 `Scripts` 文件夹中，例如 `C:\Python39\Scripts`。
2. **添加到环境变量**:
   - 右键点击桌面上的 `此电脑`（或 `计算机`），选择 `属性`，然后点击 `高级系统设置`。接着点击 `环境变量`。
   - 在“系统变量”部分找到 `Path` 变量，点击“编辑”。
   - 添加 Python `Scripts` 文件夹的路径到 `Path` 变量中，例如 `C:\Python39\Scripts`。
3. **应用更改并重启命令提示符**:
   - 点击“确定”保存更改，并重新启动命令提示符以使更改生效。

### 小结

- **默认安装**：从 Python 3.4 起，`pip` 默认会与 Python 一起安装。
- **手动安装**：如果 `pip` 没有随 Python 安装，你可以使用 `get-pip.py` 脚本来安装它。
- **环境变量**：确保 `pip` 的路径添加到系统环境变量中，以便在命令提示符中使用。

通过这些步骤，你可以在 Windows 上成功安装和配置 `pip`，以便管理 Python 包。

# 2.1.安装Flask(Windows)

在 Windows 上安装 Flask 非常简单。下面是详细的步骤，帮助你从头开始安装和设置 Flask。

#### 步骤 1: 安装 Python 和 pip

1. **下载和安装 Python**:

   - 访问 [Python 官方网站](https://www.python.org/downloads/) 下载最新的 Python 安装程序。
   - 在安装过程中，确保选中“Add Python to PATH”选项，然后点击“Install Now”进行安装。

2. **检查 `pip` 是否已安装**:

   - 安装 Python 时，

     ```
     pip
     ```

     （Python 包管理工具）会自动安装。打开命令提示符，输入以下命令检查 

     ```
     pip
     ```

     ：

     ```
     cmd
     复制代码
     pip --version
     ```

   - 如果未安装 `pip`，可以参考前面的说明来安装它。

#### 步骤 2: 创建虚拟环境（推荐）

使用虚拟环境可以避免包和依赖项冲突，是管理 Python 项目的最佳实践。

1. **打开命令提示符**：

   - 按 `Win + R` 键，输入 `cmd` 并按回车。

2. **创建项目目录**：

   - 创建一个新的目录用于你的 Flask 项目，例如 

     ```
     flask_project
     ```

     ：

     ```
     mkdir flask_project
     cd flask_project
     ```

3. **创建虚拟环境**：

   - 在项目目录中创建一个虚拟环境：

     ```
     python -m venv venv
     ```

4. **激活虚拟环境**：

   - 激活虚拟环境：

     ```
     venv\Scripts\activate
     ```

   - 激活后，你会看到命令提示符前面出现 `(venv)`，表示虚拟环境已激活。

#### 步骤 3: 安装 Flask

1. **使用 `pip` 安装 Flask**：

   - 在虚拟环境中，运行以下命令安装 Flask：

     ```
     pip install flask
     ```

2. **验证安装**：

   - 你可以检查 Flask 版本确认安装成功：

     ```
     python -m flask --version
     ```

   - 应显示 Flask 及其相关版本信息。

#### 步骤 4: 创建一个简单的 Flask 应用

1. **在项目目录中创建应用文件**：

   - 创建一个名为 

     ```
     app.py
     ```

      的文件，并将以下代码粘贴到文件中：

     ```
     from flask import Flask
     
     app = Flask(__name__)
     
     @app.route('/')
     def hello_world():
         return 'Hello, World!'
     
     if __name__ == '__main__':
         app.run(debug=True)
     ```

2. **运行 Flask 应用**：

   - 在命令提示符中，运行以下命令启动 Flask 服务器：

     ```
     python app.py
     ```

   - Flask 服务器将启动并监听默认的 5000 端口，你会看到类似于以下的输出：

     ```
     * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
     ```

3. **访问应用**：

   - 打开浏览器，访问 http://127.0.0.1:5000。
   - 你应该看到页面上显示 `Hello, World!`。

#### 步骤 5: 关闭虚拟环境（可选）

当你完成工作时，你可以退出虚拟环境：

```
deactivate
```

### 小结

- **安装 Python 和 pip**：确保 Python 和 pip 已安装，并可通过命令提示符访问。
- **创建虚拟环境**：使用虚拟环境来管理项目的依赖项。
- **安装 Flask**：在虚拟环境中使用 `pip` 安装 Flask。
- **创建和运行 Flask 应用**：编写一个简单的 Flask 应用并运行它以确保一切正常。

通过这些步骤，你可以在 Windows 上成功安装和运行 Flask。

# 2.2.pymysql

1. pip install pymysql
2. pip show pymysql

# 2.3.django

1. pip install django

# 