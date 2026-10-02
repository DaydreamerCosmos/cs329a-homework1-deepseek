# CS329A HW1 DeepSeek版

将 Stanford CS329A Homework 1 的模型接口改为 DeepSeek，并提供可以从头练习的空白作业和已完成的参考版本。

本仓库是**非官方参考实现**，基于 [Stanford 官方公开作业](https://github.com/stanford-cs329a/cs329a-homework1-fall2025-public) 的提交 [`6f6eb05`](https://github.com/stanford-cs329a/cs329a-homework1-fall2025-public/tree/6f6eb05)。原题目、TODO 结构和分析要求保留，原 README 保存在 [README_UPSTREAM.md](README_UPSTREAM.md)。这里使用 DeepSeek，实验结果不能视为课程指定模型的结果。

## 从哪个文件开始

- [student_homework1.ipynb](student_homework1.ipynb)：空白作业，保留 **9 处代码 TODO 和 6 处文字 TODO**，没有执行输出。建议先使用这个版本练习。
- [solutions/student_homework1_completed.ipynb](solutions/student_homework1_completed.ipynb)：已完成的参考实现及分析，用于对照思路与代码。
- [solutions/hw1_analysis_results.json](solutions/hw1_analysis_results.json)：一次实验的评分、完整解答和反馈记录。
- [solutions/performance_comparison.png](solutions/performance_comparison.png)：对应实验的性能比较图。

## 安装并打开作业

先安装 Git 和 Conda，然后在终端依次执行：

```bash
git clone https://github.com/DaydreamerCosmos/cs329a-homework1-deepseek.git
cd cs329a-homework1-deepseek
conda create -n cs329a-hw1 python=3.10 -y
conda activate cs329a-hw1
python -m pip install -e .
python -m pip install jupyterlab
python -m jupyterlab student_homework1.ipynb
```

后续也从**仓库根目录**启动 JupyterLab。Notebook 中应选择 `cs329a-hw1` 环境的 Python 内核。

依赖配置固定使用 LiteLLM `1.80.0`，以及相互兼容的 `antlr4-python3-runtime==4.7.2` 和 `latex2sympy2==1.9.1`。包目录补充了 `cs329_hw1/__init__.py`，使 `find_packages()` 能发现项目包；`pip install -e .` 会以可编辑方式安装它。

## 设置自己的 API Key

Notebook 的配置格使用 `getpass` 隐藏输入 DeepSeek API Key。运行到这一格时，输入自己的 Key，输入内容不会显示在输出中。内核重启后需要重新配置。

也可以将根目录的 `.env.example` 复制为 `.env`，填写：

```dotenv
DEEPSEEK_API_KEY=填写自己的DeepSeek_API_Key
HUGGINGFACE_HUB_TOKEN=
```

`HUGGINGFACE_HUB_TOKEN` 是可选项，公共 AIME25 数据集可以匿名下载。Notebook 会读取根目录的 `.env`；已配置 DeepSeek Key 时无需再次输入。`.env` 不属于需要提交的作业文件。

## 按什么顺序完成

1. 从上到下运行导入、配置、数据集加载和验证器初始化格。
2. 阅读每部分说明，在代码 TODO 之间实现单次采样、多数投票、LLM 投票和自我改进。
3. 完成该部分的代码后，再执行对应正式实验，保存准确率及解答记录。
4. 使用已有结果绘图，并填写最后的文字分析 TODO。

**空白版本不能直接成功运行全部代码。** TODO 尚未实现时，后面的实验会失败，这是原作业模板的行为。先完成对应实现，再执行实验；完成参考版则用于检查实现方式。

各部分的主要流程是：

- **Zero-shot**：每道题生成一份解答，使用标准答案评分。
- **Majority voting**：每道题生成 16 份解答，分别用前 1、2、4、8、16 份的最终答案进行投票。
- **LLM voting**：每道题生成 16 份候选解答，再调用模型评审一次，每题共 17 次调用。
- **Self-improvement**：生成初始解答、生成反馈、根据反馈重新解答，每题共 3 次调用。标准答案仅用于评分，不传给生成或反馈模型。

## DeepSeek 接口与实验设置

模型统一使用 `deepseek/deepseek-flash`，API 地址为 `https://api.deepseek.com`。请求关闭 thinking，默认输出上限为 **4096 tokens**；采样温度默认 `0.7`。

默认 `max_workers=8`，表示**同一个采样器最多同时处理 8 个请求**。它控制并发，不改变每道题的采样数。平台允许的并发可能变化，出现 `429` 时可进一步降低这个值。

30 道题全部运行时，多数投票需要 480 次生成请求，LLM 投票需要 480 次生成加 30 次评审，自我改进需要 90 次请求。重新运行这些实验会再次调用 API 并消耗余额。进度达到 `100%` 仅表示请求处理结束；如果日志中存在请求失败，应处理失败后再解释模型准确率。

Windows 下的答案验证器使用直接比较，避免原来的信号超时机制不兼容；其他系统保留原有超时方式。

## 已完成参考版的一次实验结果

数据集共有 30 道 AIME25 题，保存的结果为：

- Zero-shot：20/30，准确率 **66.67%**。
- Majority voting：采样预算为 1、2、4、8 时均为 23/30，**76.67%**；预算为 16 时为 24/30，**80%**。
- LLM voting：23/30，**76.67%**。
- Self-improvement：从 21/30（**70%**）提升至 24/30（**80%**），3 道题由错变对，0 道题由对变错。

不同实验重新采样，因此多数投票中“预算 1”的准确率可以与先前 zero-shot 基线不同。采样具有随机性，模型服务也可能更新，重新执行不保证得到完全相同的数值。

这些评分只检查最终答案。**最终答案正确不表示全部推理正确**；分析时还应检查完整解答、反馈及答案提取行为。例如，原解答文本未完成或最终答案未被正确提取，也可能造成判错。

## 许可

本仓库采用 [MIT 许可](LICENSE)，保留上游许可和版权声明。请同时参阅原作业中的课程说明和要求。
