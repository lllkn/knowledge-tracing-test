# 鲁凯宁知识追踪测试框架

交付版本 **1.0.0**，日期 **2026 年 10 月 13 日**。本工程完成任务图片中负责人 A 的七项工作：Python 环境、模拟数据、预处理、Mock 模型、统一模型接口、主程序和基础功能测试。当前未收到训练组的真实模型或真实数据文件，默认运行使用明确标记的合成数据与模拟预测。

README 更新于 **2026 年 10 月 14 日**。

## 项目背景与用途

整体大创项目是“基于知识图谱与认知诊断的个性化学习路径动态生成系统”，面向鸿蒙设备，结合知识图谱、知识追踪、遗忘机制和大语言模型生成学习计划。

本仓库实现鲁凯宁负责的 **Python 知识追踪测试框架**：读取学生答题记录，组织可见的历史作答，调用模拟模型或训练组提供的真实模型，再导出统一格式的预测 CSV，供后续评估模块使用。

## 已完成的工作

| 分工任务 | 已实现内容 | 对应文件 |
| --- | --- | --- |
| 1. 环境搭建 | Python 3.12 环境、pandas / NumPy / pytest 依赖、锁定版本与安装脚本 | `requirements*.txt`、`pyproject.toml`、`scripts/setup.sh`、`scripts/setup.bat` |
| 2. 测试数据准备 | 固定随机种子的合成数据、多知识点题目、缺失可选耗时；ASSISTments 与 EdNet KT1 转换 | `sample_data.csv`、`generate_sample_data.py`、`dataset_converters.py`、`examples/` |
| 3. 数据预处理 | CSV 格式检查、缺失值处理、学生分组、时间排序、时区规范化、知识点编号与异常拒收 | `data_loader.py` |
| 4. 模拟模型 | 根据已知历史生成确定性答对概率，支持冷启动；明确声明为 Mock | `mock_model.py` |
| 5. 统一模型接口 | 不可变请求结构、外部工厂加载、统一 predict 调用、按记录 ID 对齐与概率检查 | `contracts.py`、`model_adapter.py`、`examples/` |
| 6. 主测试程序 | 串联读取、历史组织、批量推理、标签关联、CSV 输出和运行摘要；失败保留旧结果 | `main.py`、`pipeline.py`、`io_utils.py` |
| 7. 基础功能测试 | 数据异常、接口异常、因果历史、转换器、CLI、批量一致性与集成验收 | `tests/`、`scripts/verify.sh`、`scripts/acceptance.py` |

## 仓库结构

```text
knowledge_tracing_test/
├── README.md                       # 项目范围、成果、结构与运行说明
├── main.py                         # 命令行入口，默认运行模拟预测
├── pipeline.py                     # 因果历史组织、批量推理和结果导出
├── contracts.py                    # 数据类型、模型协议与异常类型
├── data_loader.py                  # CSV 校验、排序和固定知识点词表
├── model_adapter.py                # 模型工厂接入与预测结果校验
├── mock_model.py                   # 基于历史答题的确定性模拟模型
├── io_utils.py                     # 原子写入与文件 SHA256
├── dataset_converters.py           # ASSISTments / EdNet KT1 数据转换
├── generate_sample_data.py         # 可复现模拟数据生成器
├── sample_data.csv                 # 12 位模拟学生，288 条作答，6 个知识点
├── requirements.txt                # 依赖版本范围
├── requirements-lock.txt           # 验收环境精确依赖版本
├── pyproject.toml                  # Python 版本与代码检查配置
├── pytest.ini                      # 自动测试配置
├── 运行演示.command                # macOS 双击运行入口
├── data/                           # 放置自己的标准输入数据，CSV/JSON 不提交
├── examples/                       # 数据集人工夹具、插件演示与真实模型模板
├── tests/
│   ├── conftest.py                 # 测试数据与公共夹具
│   ├── test_pipeline.py            # 数据、历史、模型接口与输出测试
│   ├── test_datasets.py            # 两个真实数据格式转换器的测试
│   └── test_cli.py                 # 主程序与命令行测试
├── scripts/
│   ├── setup.sh / setup.bat        # 安装环境与锁定依赖
│   ├── run_demo.bat                # Windows 运行入口
│   ├── verify.sh                   # pytest、覆盖率和静态检查
│   ├── acceptance.py               # 端到端集成验收与证据生成
│   └── package_release.py          # 源码 ZIP 打包与逐文件校验
├── docs/
│   ├── RESEARCH.md                 # 项目理解、开发调研与原始来源
│   ├── DATA_FORMAT.md              # 输入输出字段、时间与缺失值规则
│   ├── MODEL_INTERFACE.md          # 训练组模型接入与评估交接契约
│   ├── TEST_REPORT.md              # 实际验收结果和复现方法
│   └── DELIVERY_CHECKLIST.md        # 七项任务成果核对
├── evidence/                       # 模拟运行、转换器、异常及一万条记录验收证据
└── qa/                             # 测试日志、JUnit、覆盖率和发布包检查记录
```

运行后生成的 `outputs/`、本机 `.venv/`、Python 缓存及系统文件不纳入版本控制。`evidence/` 中提交的数据为合成数据或人工夹具，不含真实学生记录。项目申报表及任务原图保留在原工作目录，不属于此源码仓库。

## 数据处理流程

```text
标准 CSV / 数据集转换结果
  → 校验字段、拒收异常、固定知识点词表
  → 按学生和时间 / 事件顺序组织记录
  → 为每道目标题建立已可见历史
  → ModelAdapter.predict → Mock 或外部模型
  → 检查数量、记录 ID 和概率范围
  → 推理后关联 y_true
  → predictions.csv + 拒收清单 + 词表 + 运行摘要
```

预测请求不包含当前答案或当前作答耗时；历史仅使用同一学生已可见的早先答案，排除同时发生、尚未完成及同题组的答案。模型乱序返回时按 `record_id` 对齐，不按数组位置关联标签。

## 验证结果

- **93 项自动测试通过**，0 失败、0 跳过，所统计模块的代码覆盖率 **92%**。
- **10 个集成命令检查通过**，包含正常运行和预期失败场景。
- 默认程序成功输出 **288 条模拟预测**。
- 两个转换器完成人工夹具的转换到推理全流程。
- **一万条模拟记录**运行成功，输出数量和文件校验值一致。
- 发布包在临时解压目录及新建虚拟环境中验证，默认运行、全部测试和静态检查通过。

这些是 2026 年 10 月 13 日的交付验收记录，不是持续集成状态。完整说明见 [验收报告](docs/TEST_REPORT.md)，机器可读证据见 [集成验收记录](evidence/acceptance.json) 和 [发布包环境检查](qa/release_smoke.json)。

## 快速运行

建议使用 Python 3.12。先克隆并进入工程目录；私有仓库需要拥有访问权限。

```bash
git clone https://github.com/lllkn/knowledge-tracing-test.git knowledge_tracing_test
cd knowledge_tracing_test
```

macOS / Linux：

```bash
bash scripts/setup.sh
source .venv/bin/activate
python main.py
```

macOS 也可双击 `运行演示.command`；缺少环境时会安装锁定依赖，需要联网。仓库不分发本机虚拟环境，其他电脑首次使用需要执行安装步骤。

Windows：

```bat
scripts\setup.bat
.venv\Scripts\python main.py
```

第二次运行到相同目录时，显式允许替换已有结果：

```bash
python main.py --overwrite
```

成功后生成 `outputs/demo/`：

| 文件 | 用途 |
| --- | --- |
| `predictions.csv` | 统一预测结果，交给刘静涵的评估程序 |
| `skill_mapping.json` | 本次知识点编号，0 预留给未知知识点 |
| `rejected_rows.csv` | 拒收记录和原因，无拒收时保留表头 |
| `run_summary.json` | 模型类型、数据统计、配置、依赖版本、耗时与文件 SHA256 |

默认数据包含 12 位模拟学生、288 条打乱顺序的答题记录、6 个知识点、多知识点题目和缺失的可选答题耗时。主程序按学生和时间恢复序列。

## 常用命令

```bash
# 自己的标准数据
python main.py --input data/my_records.csv --output-dir outputs/my_run

# 数值 Unix 毫秒时间戳，必须明确单位
python main.py --input data/my_records.csv --timestamp-unit ms --output-dir outputs/ms_run

# 拒收异常行并导出清单，其余数据继续运行
python main.py --input data/my_records.csv --invalid-policy drop --output-dir outputs/clean_run

# 调整批量和历史窗口
python main.py --batch-size 32 --max-history 128 --output-dir outputs/b32

# 重建相同模拟数据；不同 seed 可生成另一份数据
python generate_sample_data.py --output data/new_sample.csv --students 20 --events 30 --seed 7

# 检查自动测试与代码规范
bash scripts/verify.sh

# 重现交付时的集成验收，包括两个转换器与异常流程
python scripts/acceptance.py
```

脚本 `verify.sh` 适用于 macOS / Linux；Windows 可运行相同的 `python -m pytest` 与 `ruff check .` 命令。

## 真实数据与模型接入

ASSISTments 支持含 `user_id, problem_id, skill_id, correct, order_id` 的 CSV。旧版同一次作答的多技能行会合并，修正版 `skill1_skill2` 会转换为标准的分号列表。

```bash
python dataset_converters.py --dataset assistments --input raw/skill_builder_data.csv --output data/assistments.csv
python main.py --input data/assistments.csv --output-dir outputs/assistments
```

只有 `order_id` 时保留顺序，时间间隔为空。原始文件若提供无时区的 `start_time`，需确认真实源时区后指定 `--source-timezone`。使用 `start_time` 时还需 `end_time` 或 `ms_first_response` 确认答案可见时间。其他版本的时间列通过 `--time-column` 显式指定。

EdNet KT1 支持单个 `u1.csv` 或包含多个 `u*.csv` 的目录，并要求配套 `questions.csv`：

```bash
python dataset_converters.py --dataset ednet --input raw/KT1 --questions raw/contents/questions.csv --output data/ednet.csv
python main.py --input data/ednet.csv --output-dir outputs/ednet
```

转换器会关联正确答案和标签，识别原始毫秒时间，并隔离同一 `solving_id` 题组。建议先选取学生子集；该版本在内存中处理输入，未验证 EdNet 全量规模。`examples/` 中的两个数据集样例都是人工构造的测试夹具。

训练组实现 `create_model(config)`，返回含 `name`、`is_mock` 和 `predict(requests)` 的对象。接入时必须使用训练组的固定词表：

```bash
python main.py --model plugin --factory my_training_module:create_model \
  --model-config model/config.json --vocab model/skill_mapping.json \
  --input data/heldout.csv --output-dir outputs/real_model
```

可运行的插件示例：

```bash
python main.py --model plugin --factory examples.constant_plugin:create_model \
  --model-config examples/plugin_config.json \
  --vocab evidence/mock_run/skill_mapping.json --output-dir outputs/plugin_demo
```

该插件仍是模拟模型，`is_mock=True`。真实模型模板 `examples/training_model_template.py` 会明确报错“尚未接入真实模型”，不会伪造预测或自动退回 Mock。

## 设计与交付材料

- `docs/RESEARCH.md`：项目理解、数据集与因果预测调研、选型依据和原始来源。
- `docs/DATA_FORMAT.md`：输入/输出字段、缺失值、时间和词表规则。
- `docs/MODEL_INTERFACE.md`：训练组接入契约及刘静涵的评估对接方式。
- `docs/TEST_REPORT.md`：实际验收结果、复现命令、覆盖范围和未验证事项。
- `docs/DELIVERY_CHECKLIST.md`：任务图片中七项任务对应成果和验收证据。
- `evidence/`：本次实际运行生成的数据转换、预测结果与验收记录。
- `qa/`：pytest 日志、JUnit XML、覆盖率 JSON、静态检查日志。
- `requirements-lock.txt`：本次验证的精确依赖版本；`requirements.txt` 提供允许的版本范围。

当前交付证明测试框架可以运行并正确检查接口与数据；模拟概率不能用于证明 DKT/AKT 的精度、遗忘曲线效果或鸿蒙端侧性能。模型训练、评估指标程序和鸿蒙应用分别属于其他工作环节。
