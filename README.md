#### 项目架构
```
GRBAfterglowProcessor/
├── src/
│   ├── __init__.py
│   ├── data_preparation/
│   │   ├── __init__.py
│   │   ├── optical_data_processor.py
│   │   └── xray_data_processor.py
│   ├── temporal_analysis/
│   │   ├── __init__.py
│   │   ├── broken_powerlaw_fitter.py
│   │   └── afterglow_model_fitter.py
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── time_utils.py
│   │   ├── flux_utils.py
│   │   └── file_utils.py
│   ├── config/
│   │   ├── __init__.py
│   │   └── config_manager.py
│   └── main.py
├── tests/
│   ├── test_optical_data.py
│   ├── test_xray_data.py
│   ├── test_fitting.py
│   └── test_utils.py
├── examples/
│   ├── sample_optical_data.csv
│   ├── sample_xray_data.fits
│   └── run_example.py
├── requirements.txt
└── README.md
```

#### 总体架构概述
- **设计原则**：
  - 模块化：将数据读取、预处理、格式转换、拟合分析等功能拆分为独立模块，便于维护和扩展。
  - 可扩展性：支持后续添加瞬时辐射处理、能谱分析等功能。
  - 用户友好：提供清晰的输入接口，支持用户指定文件路径和参数。
  - 数据格式兼容：支持`.csv`（光学数据）和`.fits`（X射线数据）格式的输入与输出。
  - 依赖管理：使用pip可安装的Python库（如`numpy`, `pandas`, `astropy`, `scipy`等）。

- **主要模块**：
  1. **数据准备模块**（Data Preparation Module）：处理光学和X射线光变曲线数据的读取、转换和存储。
  2. **时变性质分析模块**（Temporal Analysis Module）：实现broken power-law和余晖模型的拟合功能。
  3. **工具模块**（Utility Module）：提供通用的时间转换、流量转换等工具函数。
  4. **配置模块**（Configuration Module）：管理用户输入参数和全局配置。
  5. **主程序模块**（Main Module）：协调各模块的运行，提供用户接口。

---

### 详细模块设计

#### 1. 数据准备模块（`data_preparation`）
负责光学和X射线数据的读取、预处理和保存。

##### 1.1 光学数据处理子模块（`optical_data_processor`）
- **功能**：
  - 读取用户提供的`.csv`文件，包含：`time (UTC)`, `exposure time`, `mag`, `mag_err`, `telescope name`, `filter`, `central wavelength`。
  - 将观测时间转换为相对于T0的中间时间（单位：秒），公式：`mid_time = (time - T0 + exposure_time/2)`。
  - 将视星等（`mag`）及其误差（`mag_err`）转换为流量密度（flux density）。
  - 按时间从小到大排序。
  - 保存处理后的数据为新文件，命名格式：`{telescope_name}_{filter}.csv`。
- **输入**：
  - 用户提供的T0时间（UTC格式）。
  - 光学数据`.csv`文件路径。
  - 保存路径（可选，默认当前目录）。
- **输出**：
  - 处理后的`.csv`文件，包含：`mid_time (s)`, `flux_density`, `flux_density_err`, `telescope_name`, `filter`, `central_wavelength`。
- **依赖库**：
  - `pandas`：读取和处理`.csv`文件。
  - `astropy.time`：处理时间格式转换（UTC到秒）。
  - `numpy`：数值计算（如流量密度转换）。

##### 1.2 X射线数据处理子模块（`xray_data_processor`）
- **功能**：
  - 读取`.fits`文件，包含：`time (s)`, `time_err`, `count_rate`, `count_rate_err`。
  - 从`.fits`文件header读取`OBS-MJD`，将时间转换为相对于T0的时间（需将T0转换为MJD格式）。
  - 通过用户提供的转换因子将`count_rate`转换为流量（flux）。
  - 保存为`.csv`文件，命名格式：`{telescope_name}_flux.csv`，包含：`time (s)`, `time_err (s)`, `flux`, `flux_err`。
  - 读取上一步生成的`{telescope_name}_flux.csv`，将流量转换为流量密度，保存为`{telescope_name}_flux_density.csv`。
- **输入**：
  - 用户提供的T0时间（UTC格式）。
  - X射线数据`.fits`文件路径。
  - 流量转换因子（用户提供）。
  - 保存路径（可选，默认当前目录）。
- **输出**：
  - 中间文件：`{telescope_name}_flux.csv`，包含：`time (s)`, `time_err (s)`, `flux`, `flux_err`。
  - 最终文件：`{telescope_name}_flux_density.csv`，包含：`time (s)`, `time_err (s)`, `flux_density`, `flux_density_err`。
- **依赖库**：
  - `astropy.io.fits`：读取`.fits`文件。
  - `astropy.time`：处理MJD和时间转换。
  - `pandas`：生成和处理`.csv`文件。
  - `numpy`：流量和流量密度计算。

#### 2. 时变性质分析模块（`temporal_analysis`）
负责光变曲线的拟合分析。

##### 2.1 Broken Power-Law拟合子模块（`broken_powerlaw_fitter`）
- **功能**：
  - 支持任意个broken power-law函数拟合。
  - 从光学或X射线数据文件中读取时间和流量密度数据。
  - 使用用户指定的破节点数（break points）进行拟合。
  - 返回拟合参数、误差和拟合质量（如χ²）。
- **输入**：
  - 处理后的数据文件（`.csv`格式，来自数据准备模块）。
  - 用户指定的破节点数和初始参数（可选）。
- **输出**：
  - 拟合参数（斜率、破节点位置等）。
  - 拟合结果的统计信息（如χ²、自由度）。
  - 可视化结果（可选，保存为图表）。
- **依赖库**：
  - `scipy.optimize`：非线性拟合。
  - `numpy`：数据处理。
  - `matplotlib`：可视化拟合结果（可选）。

##### 2.2 余晖模型拟合子模块（`afterglow_model_fitter`）
- **功能**：
  - 实现GRB余晖模型的拟合（例如标准余晖模型或其他物理模型）。
  - 支持用户自定义模型参数或选择预定义模型。
  - 返回拟合参数和拟合质量。
- **输入**：
  - 处理后的数据文件（`.csv`格式）。
  - 用户指定的模型类型和初始参数。
- **输出**：
  - 拟合参数。
  - 拟合结果的统计信息。
  - 可视化结果（可选，保存为图表）。
- **依赖库**：
  - `scipy.optimize`：模型拟合。
  - `numpy`：数据处理。
  - `matplotlib`：可视化拟合结果（可选）。

#### 3. 工具模块（`utils`）
提供通用的辅助功能，供其他模块调用。
- **功能**：
  - 时间转换：UTC到秒、MJD到秒等。
  - 流量转换：视星等到流量密度、count rate到流量等。
  - 数据排序和清洗：按时间排序，检查数据完整性。
  - 文件路径管理：验证输入文件路径，生成输出文件路径。
- **依赖库**：
  - `astropy.time`：时间格式转换。
  - `numpy`：数值计算。
  - `os`：文件路径操作。

#### 4. 配置模块（`config`）
管理用户输入参数和全局设置。
- **功能**：
  - 存储T0时间、文件路径、转换因子等用户输入。
  - 提供默认配置（如输出路径、文件命名规则）。
  - 验证用户输入的合法性（例如时间格式、文件存在性）。
- **输入**：
  - 用户通过命令行或配置文件提供的参数。
- **输出**：
  - 全局配置对象，供其他模块访问。
- **依赖库**：
  - `argparse`：命令行参数解析。
  - `configparser`：配置文件读取（可选）。

#### 5. 主程序模块（`main`）
协调各模块的运行，提供用户交互接口。
- **功能**：
  - 解析用户输入（T0时间、数据文件路径等）。
  - 调用数据准备模块处理光学和X射线数据。
  - 调用时变性质分析模块进行拟合。
  - 输出结果（文件和可视化）。
- **输入**：
  - 命令行参数或配置文件。
- **输出**：
  - 处理后的数据文件。
  - 拟合结果和统计信息。
- **依赖库**：
  - 所有其他模块。


