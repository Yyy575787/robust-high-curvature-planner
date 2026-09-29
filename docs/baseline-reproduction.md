# CommonRoad Reactive Planner 基线复现说明

## 1. 复现目标

本说明用于复现 https://github.com/CommonRoad/commonroad-reactive-planner 官方示例：

- 场景：`ZAM_Over-1_1`
- 入口：`run_planner.py`
- 基线性质：未修改规划逻辑的官方原始基线
- 验证日期：2026-09-29
- 验证结果：成功到达目标，进程退出码为 0

## 2. 固定版本

| 项目 | 固定值 |
|---|---|
| 操作系统 | Ubuntu 22.04，Linux 6.8.0-138-generic，x86_64 |
| Conda 环境 | `commonroad-py310` |
| Python | 3.10.21 |
| Reactive Planner | 2025.1 |
| Route Planner | 2025.1.0 |
| Drivability Checker | 2025.4.0 |
| CLCS | 2025.2.0 |
| CommonRoad IO | 2024.3 |
| 官方上游提交 | `6f2f0d91d4b9370f8e716349833308271365e7d1` |
| 场景文件 | `baseline/commonroad-reactive-planner/example_scenarios/ZAM_Over-1_1.xml` |
| 场景 SHA-256 | `343ec7f9e8cf0edc15d77bab732babc2b4c97ad6051ca1f171375ec4c24d38b5` |
| 配置 SHA-256 | `a0641bea60c8c2cdd0b3b5eec4085014ac79134b01d804cb98b00f46a9674f65` |

## 3. 创建环境

在项目根目录执行：

```bash
conda env create -f environment/commonroad-py310.yml
conda activate commonroad-py310
```

检查 Python、软件包版本和依赖状态：

```bash
python --version

python -c "import importlib.metadata as m; \
print('reactive-planner:', m.version('commonroad-reactive-planner')); \
print('route-planner:', m.version('commonroad-route-planner')); \
print('drivability-checker:', m.version('commonroad-drivability-checker'))"

python -m pip check
```

预期至少包含：

```text
Python 3.10.21
reactive-planner: 2025.1
route-planner: 2025.1.0
drivability-checker: 2025.4.0
No broken requirements found.
```

## 4. 运行

在commonroad-py310环境中，进入目录：
baseline/commonroad-reactive-planner

运行 python ./run_planner.py (VScode中进入环境直接运行即可)

## 5. 运行结果

| 指标 | 结果 | 解释 |
|---|---:|---|
| 进程退出码 | `0` | 程序正常结束 |
| 到达目标 | 是 | 日志明确报告目标到达 |
| 最终有效性检查 | `True` | 当前场景下通过检查 |
| 规划周期数 | 9 | 每个周期均返回轨迹 |
| 记录状态数 | 28 | 包含初始状态 |
| 仿真时间 | 2.7 s | 从第 0 步推进到第 27 步 |
| 车身中心轨迹长度 | 约 53.65 m | 按离散位置累计 |
| 速度范围 | 19.716～20.026 m/s | 约 71.0～72.1 km/h |
| 规划内部计时平均值 | 69.94 ms | 9 个周期的均值 |
| 规划内部计时 P95 | 155.34 ms | 样本很少，仅作本次描述 |
| 最大单周期内部耗时 | 178.66 ms | 出现在首个周期 |
| 整个进程墙钟耗时 | 约 2.05 s | 包括加载、规划及评估等 |

| 图 | 应观察什么 |
|---|---|
| [轨迹图](../baseline/commonroad-reactive-planner/output/run_analysis/20260929_ZAM_Over-1_1/01_trajectory.png) | 如何绕障、车身占用和目标位置 |
| [加速度一致性](../baseline/commonroad-reactive-planner/output/run_analysis/20260929_ZAM_Over-1_1/02_acceleration_check.png) | 规划加速度与速度差分是否一致 |
| [状态曲线](../baseline/commonroad-reactive-planner/output/run_analysis/20260929_ZAM_Over-1_1/03_states.png) | 位置、转角、速度、航向和横摆角速度 |
| [状态重建误差](../baseline/commonroad-reactive-planner/output/run_analysis/20260929_ZAM_Over-1_1/04_state_errors.png) | 车辆模型能否重建相邻状态转移 |
| [控制输入](../baseline/commonroad-reactive-planner/output/run_analysis/20260929_ZAM_Over-1_1/05_inputs.png) | 转角变化率、纵向加速度及其边界 |


## 6. 已知限制

1. `ZAM_Over-1_1` 是官方基础示例，不属于高曲率 U 型弯场景。
2. 当前结果只能证明官方原始基线已经跑通，不能支持或反驳 H1–H4。
3. `02_acceleration_check.png` 的横轴标签写为秒，但实际传入的是 timestep 索引；应乘以 `dt=0.1` 后再解释为秒。
4. `summary.json` 中的周期耗时来自补充分析运行；正式性能基准以 `run_metrics.json` 中从原始 `run.log` 提取的结果为准。
5. 如果补充分析图和 CSV 的生成脚本尚未进入仓库，不应声称所有分析产物均可一键重建；官方示例运行本身不受此项影响。
