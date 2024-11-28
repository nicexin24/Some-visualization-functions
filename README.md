# 置信区间与裂缝蝌蚪图

## 简介

本项目包含两个 Python 函数，用于绘制置信区间图和裂缝蝌蚪图。这些函数可以帮助研究人员和数据科学家可视化模型的预测结果及其不确定性，以及地质裂缝的分布情况。

## 目录

- [简介](#简介)
- [目录](#目录)
- [安装依赖](#安装依赖)
- [函数说明](#函数说明)
  - [create_formatter](#create_formatter)
  - [view_conf_interval](#view_conf_interval)
  - [CrackPlotter 类](#crackplotter-类)
- [示例用法](#示例用法)
- [贡献](#贡献)
- [许可证](#许可证)

## 安装依赖

在运行本项目的 Jupyter Notebook 文件之前，请确保已经安装了以下依赖包：

```bash
pip install numpy matplotlib
```

## 函数说明

### create_formatter

`create_formatter` 函数创建一个格式化器，用于格式化刻度标签。

#### 参数

- `n` (int): 小数位数

#### 返回值

- `FuncFormatter` 对象

#### 示例

```python
formatter = create_formatter(2)
print(formatter(3.14159, 0))  # 输出: 3.14
```

### view_conf_interval

`view_conf_interval` 函数用于绘制置信区间图，支持多个模型的对比。

#### 参数

- `time` (array-like): 时间数组
- `mmap` (array-like): 预测模型值数组，形状为 (时间点数, 模型数)
- `std` (array-like): 标准差数组，形状为 (时间点数, 模型数)
- `true_data` (array-like, optional): 真实数据数组，形状为 (时间点数, 模型数)
- `prior_model` (array-like, optional): 先验模型值数组，形状为 (时间点数, 模型数)
- `cal_model_name` (list, optional): 模型名称列表，长度为模型数
- `ylabel` (str, optional): Y轴标签，默认为 "Time(s)"
- `pic_name` (str, optional): 图片保存名称
- `save_type` (bool, optional): 是否保存图片，默认为 `False`

#### 返回值

- `fig` (matplotlib.figure.Figure): 图形对象
- `axs` (list of matplotlib.axes.Axes): 子图对象

#### 示例

```python
import numpy as np

# 示例数据
time = np.linspace(0, 10, 100)
mmap = np.random.rand(100, 2)
std = np.random.rand(100, 2) * 0.1
true_data = np.random.rand(100, 2)
prior_model = np.random.rand(100, 2)
cal_model_name = ["Model A", "Model B"]

# 调用函数
view_conf_interval(time, mmap, std, true_data=true_data, prior_model=prior_model, cal_model_name=cal_model_name, pic_name="conf_interval", save_type=True)
```

### CrackPlotter 类

`CrackPlotter` 类用于绘制裂缝蝌蚪图。

#### 初始化方法

```python
def __init__(self, inline_set, xline_set, e, data, axis_fontdict, font_name):
    """
    初始化CrackPlotter实例。
    
    :param inline_set: 行内距离数组
    :param xline_set: 交叉线距离数组
    :param e: 裂缝密度分布矩阵
    :param data: 包含裂缝角度等信息的数据集
    :param axis_fontdict: 轴标签字体字典
    :param font_name: 刻度标签字体样式
    """
```

#### 内部方法

- `_compute_angles_and_lengths`：计算裂缝的角度和长度。
- `create_formatter`：创建一个用于格式化颜色条标签的函数。
- `plot_main_figure`：绘制主图，包括背景色图和颜色条。
- `plot_crack_tadpole`：绘制裂缝蝌蚪图。
- `setup_plot`：设置图形的基本属性。
- `show_plot`：显示图形。

#### 示例

```python
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.ticker import FuncFormatter

class CrackPlotter:
    def __init__(self, inline_set, xline_set, e, data, axis_fontdict, font_name):
        """
        初始化CrackPlotter实例。
        
        :param inline_set: 行内距离数组
        :param xline_set: 交叉线距离数组
        :param e: 裂缝密度分布矩阵
        :param data: 包含裂缝角度等信息的数据集
        :param axis_fontdict: 轴标签字体字典
        :param font_name: 刻度标签字体样式
        """
        self.inline_set = inline_set
        self.xline_set = xline_set
        self.e = e
        self.data = data
        self.axis_fontdict = axis_fontdict
        self.font_name = font_name
        self.fig, self.ax = plt.subplots(1, 1)
        self.fig.set_size_inches(4, 4)
        self._compute_angles_and_lengths()

    def _compute_angles_and_lengths(self):
        """计算裂缝的角度和长度"""
        self.angles = self.data[:, :, -1]
        self.lengths = (self.e - np.min(self.e)) / (np.max(self.e) - np.min(self.e)) * 60

    def create_formatter(self, decimals):
        """创建一个用于格式化颜色条标签的函数"""
        return FuncFormatter(lambda x, pos: f"{x:.{decimals}f}")

    def plot_main_figure(self):
        """绘制主图，包括背景色图和颜色条"""
        Ax = self.ax.pcolormesh(self.inline_set, self.xline_set, self.e, cmap='jet', alpha=0.5)
        self.ax.set_xlabel('Distance(m)', fontdict=self.axis_fontdict)
        self.ax.set_ylabel('Distance(m)', fontdict=self.axis_fontdict)
        cb = plt.colorbar(Ax, ax=self.ax, shrink=0.8, pad=0.02, location='right', label='$e$')
        cb.ax.yaxis.set_major_formatter(self.create_formatter(2))
        cb.ax.tick_params(labelsize=10)
        plt.setp(cb.ax.get_yticklabels(), fontproperties=self.font_name)

    def plot_crack_tadpole(self):
        """绘制裂缝蝌蚪图"""
        x_step = 6
        y_step = x_step
        for i in range(0, len(self.inline_set), x_step):
            for j in range(0, len(self.xline_set), y_step):
                angle_rad = np.radians(self.angles[j, i])
                x = [self.inline_set[i], self.inline_set[i] + self.lengths[j, i] * np.cos(angle_rad)]
                y = [self.xline_set[j], self.xline_set[j] + self.lengths[j, i] * np.sin(angle_rad)]
                self.ax.plot(x, y, 'k')  # 绘制裂缝线段
                self.ax.plot(self.inline_set[i], self.xline_set[j], marker='o', markersize=2, color='g')  # 绘制起点圆点

    def setup_plot(self):
        """设置图形的基本属性"""
        self.ax.set_ylim([self.xline_set[0], self.xline_set[-1]])
        self.ax.set_xlim([self.inline_set[0], self.inline_set[-1]])
        plt.xticks(fontproperties=self.font_name, fontsize=10)
        plt.yticks(fontproperties=self.font_name, fontsize=10)
        self.ax.plot(1881, 737, marker='*', markersize=10, color='red')  # 绘制特定位置的标记

    def show_plot(self):
        """显示图形"""
        self.plot_main_figure()
        self.plot_crack_tadpole()
        self.setup_plot()
        plt.show()

# 示例数据
inline_set = np.linspace(0, 1000, 100)  # 行内距离
xline_set = np.linspace(0, 1000, 100)  # 交叉线距离
e = np.random.rand(100, 100)  # 裂缝密度分布
data = np.random.rand(100, 100, 3)  # 包含裂缝角度等信息的数据集
axis_fontdict = {'size': 12}
font_name = {'family': 'serif'}

# 创建CrackPlotter对象并显示图形
crack_plotter = CrackPlotter(inline_set, xline_set, e, data, axis_fontdict, font_name)
crack_plotter.show_plot()
```

## 示例用法

### 运行 Jupyter Notebook

1. 克隆仓库：

    ```bash
    git clone https://github.com/nicexin24/Some-visualization-functions.git
    cd yourrepository
    ```

2. 启动 Jupyter Notebook：

    ```bash
    jupyter notebook
    ```

3. 打开 `visualization_functions.ipynb.ipynb` 文件并运行所有单元格。

### 直接运行 Python 脚本

1. 将函数保存到一个 Python 文件中，例如 `confidence_interval.py` 和 `crack_plotter.py`。
2. 在另一个 Python 脚本中导入并使用这些函数。

```python
from confidence_interval import create_formatter, view_conf_interval
from crack_plotter import CrackPlotter

# 示例数据
time = np.linspace(0, 10, 100)
mmap = np.random.rand(100, 2)
std = np.random.rand(100, 2) * 0.1
true_data = np.random.rand(100, 2)
prior_model = np.random.rand(100, 2)
cal_model_name = ["Model A", "Model B"]

# 调用置信区间图函数
view_conf_interval(time, mmap, std, true_data=true_data, prior_model=prior_model, cal_model_name=cal_model_name, pic_name="conf_interval", save_type=True)

# 裂缝蝌蚪图示例
inline_set = np.linspace(0, 1000, 100)  # 行内距离
xline_set = np.linspace(0, 1000, 100)  # 交叉线距离
e = np.random.rand(100, 100)  # 裂缝密度分布
data = np.random.rand(100, 100, 3)  # 包含裂缝角度等信息的数据集
axis_fontdict = {'size': 12}
font_name = {'family': 'serif'}

# 创建CrackPlotter对象并显示图形
crack_plotter = CrackPlotter(inline_set, xline_set, e, data, axis_fontdict, font_name)
crack_plotter.show_plot()
```


## 许可证

本项目采用 MIT 许可证。详情请参见 [LICENSE](LICENSE) 文件。
