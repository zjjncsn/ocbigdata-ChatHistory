# 添加 L2-FDL 数学验证实验

> Continuing from [浏览海洋大数据文件夹](chatgpt-conversation://6af96936-60c4-83e8-84fa-5ae45b707307): 继续 F:\ocbigdata 中的 L2-FDL 预研项目。先检查现有文件、README、pyproject.toml、uv.lock 和 scripts/hnsw_smoke.py，确认现有 make_data() 或实际生成逻辑；不得覆盖或破坏已有冒烟实验。基于用户确认的数据生成方式：同一个 np.random.default_rng(20261115)，先生成 database = rng.standard_normal((5000,64), dtype=np.float32)，再生成 queries = rng.standard_normal((200,64), dtype=np.float32)。在项目内新增独立的 FDL 数学验证实验，复用或提取共享 make_data()，保证数据逐元素与原冒烟实验一致。对每条固定查询，计算 5000 个平方 L2 距离的经验分布，并与理论非中心卡方 chi-square(df=64, nc=||q||²) 比较；同时验证混合查询-数据库距离的边际理论分布 2*chi-square(df=64)。比较均值、方差、多个分位数（0.1%、1%、5%、10%、50%、90%、99%）、KS/Wasserstein 等指标，生成 CSV/JSON、图表和日志。保留 float32 原始数据语义，统计计算适当用 float64；分批处理避免大内存占用。新增脚本和文档，不下载 SIFT1M、不运行长时间正式实验、不使用 WSL、不改系统配置，所有操作限定 F:\ocbigdata 及其虚拟环境。完成后运行测试并报告实际文件与结果。

<details><summary>12 previous messages</summary>

> 我会先完整核对项目约束、现有数据生成逻辑和冒烟脚本，再把共享数据生成与独立 FDL 验证接进去；现有冒烟实验会保持可运行、输出互不覆盖。随后我会运行项目测试和一次规模受控的验证，并汇报实际产物与数值结果。
>
> <details><summary>Ran 2 commands</summary>
>
> - Ran `$ErrorActionPreference='Stop'; Write-Output '--- files ---'; rg --files -g 'AGENTS.md' -g '!**/.venv/**' -g '!**/.git/**'; rg --files -g '!**/.venv/**' -g '!**/.git/**' | Select-Object -First 200; Write-Output '--- git status ---'; git status --short; Write-Output '--- root ---'; Get-ChildItem -Force | Select-Object Name,Length,Mode`
> - Ran `$ErrorActionPreference='Stop'; Write-Output '--- README.md ---'; Get-Content -Raw README.md; Write-Output '--- pyproject.toml ---'; Get-Content -Raw pyproject.toml; Write-Output '--- uv.lock head ---'; Get-Content uv.lock -TotalCount 160; Write-Output '--- scripts/hnsw_smoke.py ---'; Get-Content -Raw scripts/hnsw_smoke.py; Write-Output '--- existing outputs ---'; Get-ChildItem logs,results -Recurse -File | Select-Object FullName,Length,LastWriteTime`
>
> </details>
>
> 已确认原冒烟脚本的生成顺序与您给出的定义完全一致：同一 `Generator`，先数据库、后查询，且直接生成 `float32`。现有项目没有测试框架，也没有统计分布与绘图库；我会先检查虚拟环境里是否已有可用组件，再决定最小依赖变更，避免无关下载。
>
> <details><summary>Created 3 files, edited 3 files, ran a command</summary>
>
> - Ran `$ErrorActionPreference='Stop'; .\.venv\Scripts\python.exe -c "import sys; print(sys.executable); import numpy, faiss; print('numpy',numpy.__version__,'faiss',faiss.__version__); import importlib.util as u; print('scipy',bool(u.find_spec('scipy')),'matplotlib',bool(u.find_spec('matplotlib')),'pytest',bool(u.find_spec('pytest')))"; uv --version; Get-Content -Raw .gitignore; Get-Content -Raw uv.toml`
> - Wrote `F:/ocbigdata/pyproject.toml` (+2 -0)
> - Wrote `F:/ocbigdata/scripts/data.py` (+23 -0)
> - Wrote `F:/ocbigdata/scripts/hnsw_smoke.py` (+2 -7)
> - Wrote `F:/ocbigdata/README.md` (+42 -1)
> - Wrote `F:/ocbigdata/scripts/fdl_distribution_validation.py` (+264 -0)
> - Wrote `F:/ocbigdata/tests/test_experiments.py` (+45 -0)
>
> </details>
>
> 共享生成逻辑、独立验证脚本、回归测试和文档已加入。验证会逐查询批处理距离，只保留约 100 万个标量距离用于全局分位数与作图，不会构造巨大的三维差值数组；混合 KS 的非独立性也已在结果中明确标注。现在开始锁定依赖并实际运行。
>
> <details><summary>Ran 3 commands</summary>
>
> - Ran `$ErrorActionPreference='Stop'; uv lock --offline; uv sync --locked --offline`
> - Ran `uv lock; uv sync --locked`
> - Ran `$ErrorActionPreference='Stop'; uv run python -m unittest discover -s tests -v; uv run python -m compileall -q scripts tests; uv run python scripts/fdl_distribution_validation.py`
>
> </details>
>
> 回归测试之类的没必要。
>
> 明白。我会去掉专门新增的回归测试文件和 README 中的测试说明，只保留验证脚本自身的运行校验；不会为了测试而扩展项目结构。当前数学验证已启动，我会让它完成并检查产物。
>
> <details><summary>Edited a file, deleted a file, ran a command</summary>
>
> - Wrote `F:/ocbigdata/README.md` (+0 -8)
> - Deleted `F:/ocbigdata/tests/test_experiments.py` (+0 -45)
> - Ran `$ErrorActionPreference='Stop'; Write-Output '--- result json ---'; Get-Content -Raw results\fdl_validation_latest.json; Write-Output '--- summary csv ---'; Get-Content -Raw results\fdl_validation_latest_summary.csv; Write-Output '--- per-query head ---'; Get-Content results\fdl_validation_latest_per_query.csv -TotalCount 4; Write-Output '--- artifacts ---'; Get-ChildItem results\fdl_validation_latest*,logs\fdl_validation_latest.log | Select-Object FullName,Length,LastWriteTime; Write-Output '--- diff ---'; git diff -- README.md pyproject.toml scripts/hnsw_smoke.py scripts/data.py scripts/fdl_distribution_validation.py; Write-Output '--- status ---'; git status --short`
>
> </details>
>
> Viewed an image
>
> 数学验证已成功完成，图表也检查过，理论曲线与经验分布吻合。按你的要求，新增的测试文件和测试文档已撤掉；我再做一次轻量的入口检查与文件清点（不运行、也不覆盖原 HNSW 冒烟结果）。
>
> <details><summary>Ran 2 commands</summary>
>
> - Ran `$ErrorActionPreference='Stop'; $target = [System.IO.Path]::GetFullPath('F:\ocbigdata\tests'); if ($target -ne 'F:\ocbigdata\tests') { throw 'Unexpected test path' }; if (Test-Path -LiteralPath $target) { Remove-Item -LiteralPath $target -Recurse -Force }; uv run python scripts\hnsw_smoke.py --help | Select-Object -First 8; uv run python -c "import sys; sys.path.insert(0, 'scripts'); from data import make_data; import numpy as np; db,q=make_data(); rng=np.random.default_rng(20261115); print('shared_data_exact=', np.array_equal(db,rng.standard_normal((5000,64),dtype=np.float32)) and np.array_equal(q,rng.standard_normal((200,64),dtype=np.float32)), 'dtype=',db.dtype,q.dtype)"; Write-Output '--- HNSW outputs preserved ---'; Get-ChildItem results\hnsw_smoke_latest.*,logs\hnsw_smoke_latest.log | Select-Object Name,Length,LastWriteTime; Write-Output '--- project files ---'; rg --files -g '!**/.venv/**' -g '!**/.git/**' -g '!**/.uv-cache/**' | Sort-Object; Write-Output '--- status ---'; git status --short`
> - Ran `uv run python -c "import csv,math; rows=list(csv.DictReader(open('results/fdl_validation_latest_per_query.csv',encoding='utf-8'))); threshold=1.36/math.sqrt(5000); vals=[float(r['ks_statistic']) for r in rows]; print('ks_threshold',threshold,'above',sum(v>threshold for v in vals),'of',len(vals)); print('mean_abs_mean_error',sum(abs(float(r['mean_error'])) for r in rows)/len(rows)); print('mean_abs_variance_error',sum(abs(float(r['variance_error'])) for r in rows)/len(rows))"`
>
> </details>

</details>

已完成独立 FDL 数学验证实验，原 HNSW 冒烟实验及其结果未被覆盖。

主要变更：

- 提取共享 [data.py](F:/ocbigdata/scripts/data.py)，两个实验共用 `make_data()`。
- 更新 [hnsw_smoke.py](F:/ocbigdata/scripts/hnsw_smoke.py)，仅改为导入共享生成逻辑。
- 新增 [fdl_distribution_validation.py](F:/ocbigdata/scripts/fdl_distribution_validation.py)。
- 更新 [README.md](F:/ocbigdata/README.md) 和依赖锁文件。
- 按你的要求，没有保留回归测试目录或测试说明。

数据一致性检查通过：共享逻辑生成的数据与原始代码逐元素完全相同，数据库和查询均保持 `float32`；距离与统计使用 `float64`，每批处理 10 条查询。

实际结果：

- 条件分布 KS：均值 `0.012551`，中位数 `0.012248`，最大值 `0.025929`
- 条件 Wasserstein：均值 `0.345189`
- 200 条查询中，13 条超过单次 5% KS 渐近阈值，接近理论随机波动范围
- 混合距离均值：`127.5094`，理论值 `128`
- 混合距离方差：`507.0483`，理论值 `512`
- 混合 KS：`0.006939`
- 混合 Wasserstein：`0.490621`
- 总运行时间约 `5.64` 秒

结果文件：

- [完整 JSON](F:/ocbigdata/results/fdl_validation_latest.json)
- [逐查询 CSV](F:/ocbigdata/results/fdl_validation_latest_per_query.csv)
- [混合分布汇总 CSV](F:/ocbigdata/results/fdl_validation_latest_summary.csv)
- [验证图表](F:/ocbigdata/results/fdl_validation_latest.png)
- [运行日志](F:/ocbigdata/logs/fdl_validation_latest.log)

![FDL 分布验证图](F:/ocbigdata/results/fdl_validation_latest.png)

原 `hnsw_smoke_latest` 三个文件仍保持 17:36:46 的原时间戳，未重新运行或覆盖。整体结果支持条件非中心卡方及混合 `2χ²₆₄` 理论。混合 KS 的 p 值因距离对共享数据库/查询向量，仅作描述性指标。

> # 继续 L2-FDL 预研：各向异性高斯验证与 HNSW 搜索难度关联分析
>
> 请继续操作现有项目 `F:\ocbigdata`。这是已有项目的后续实验，不要重新初始化项目。
>
> ## 一、当前研究进度
>
> 已经完成：
>
> 1. Windows 11 原生 Python 3.12 + uv 环境。
> 2. FAISS CPU 1.16.0 安装与 HNSW 冒烟测试。
> 3. 验证了 `SearchParametersHNSW` 可以为单次查询独立设置 efSearch。
> 4. 完成标准高斯数据上的 L2-FDL 验证。
>
> 现有标准高斯实验配置：
>
> ```ini
> rng = np.random.default_rng(20261115)
> database = rng.standard_normal((5000, 64), dtype=np.float32)
> queries = rng.standard_normal((200, 64), dtype=np.float32)
> ```
>
> 理论模型：
>
> 对于固定查询 q：
>
> [\
> |q-X|^2\sim\chi'^2(64,\lambda=|q|^2)\
> ]
>
> 混合所有查询与数据库向量的距离：
>
> [\
> |q-X|^2\sim 2\chi^2(64)\
> ]
>
> 已完成的实验结果：
>
> - 混合距离实测均值：127.509，理论值 128。
> - 混合距离实测方差：507.048，理论值 512。
> - 条件分布平均 KS：0.01255。
> - 条件分布平均 Wasserstein：0.3452。
> - 实验状态：passed。
>
> 结果文件位于：
>
> - `results/fdl_validation_latest.json`
> - `results/fdl_validation_latest_per_query.csv`
> - `results/fdl_validation_latest_summary.csv`
> - `results/fdl_validation_latest.png`
> - `logs/fdl_validation_latest.log`
>
> 请先阅读这些文件和已有代码，确认实验数据、参数及结果，不要重复已经完成的实验。
>
> ## 二、实验 2A：各向异性高斯分布验证
>
> 研究目标：
>
> 验证当数据库向量服从一般多元高斯分布时，平方 L2 距离是否符合广义非中心卡方分布。
>
> 设：
>
> [\
> X\sim N(\mu,\Sigma)\
> ]
>
> 对于固定查询 q：
>
> [\
> D_q=|q-X|^2\
> ]
>
> 对协方差矩阵进行特征值分解：
>
> [\
> \Sigma=U\Lambda U^T\
> ]
>
> 令：
>
> [\
> b=U^T(q-\mu)\
> ]
>
> 则：
>
> [\
> D_q\overset{d}{=}\
> \sum\_{i=1}^{d}\lambda_i\
> \chi_1'^2\left(\frac{b_i^2}{\lambda_i}\right)\
> ]
>
> 其中 (\lambda_i) 为协方差矩阵的特征值。
>
> ### 实验数据
>
> 生成一组与原标准高斯数据规模相同的合成数据：
>
> - 数据库：5000 × 64。
> - 查询：200 × 64。
> - 数据类型：float32。
> - 使用固定随机种子。
> - 数据库和查询使用同一个 Generator，按固定顺序生成。
> - 不改变原有标准高斯数据。
>
> 设计至少三种协方差结构：
>
> 1. 对角各向异性：不同维度具有不同方差。
> 2. 相关高斯：维度之间存在相关性。
> 3. 不同特征值谱：比较协方差矩阵条件数对距离分布的影响。
>
> 注意：各实验使用独立、固定的随机种子，避免破坏原有数据的可复现性。
>
> ### 实验内容
>
> 对于每条查询：
>
> 1. 计算真实平方 L2 距离。
> 2. 计算理论均值和方差。
> 3. 根据广义非中心卡方模型估计理论分布。
> 4. 比较理论分布与实测分布。
> 5. 比较正态近似和广义非中心卡方模型的效果。
>
> 理论均值：
>
> [\
> E[D_q]=|q-\mu|^2+\operatorname{tr}(\Sigma)\
> ]
>
> 理论方差：
>
> [\
> Var(D_q)=2\operatorname{tr}(\Sigma^2)\
> +4(q-\mu)^T\Sigma(q-\mu)\
> ]
>
> 如果没有适用的广义非中心卡方数值计算库，不要直接假装使用了精确 CDF。可以通过独立 Monte Carlo 样本近似理论分布，并明确标注数值误差和采样规模。
>
> 重点比较：
>
> - 均值、方差误差。
> - P1、P5、P10、P50、P90、P99 分位数误差。
> - KS Statistic。
> - Wasserstein Distance。
>
> ## 三、实验 2B：FDL 特征与 HNSW 搜索难度的关联
>
> 这是本次任务中非常重要的部分。
>
> 研究问题：
>
> **仅通过一条查询对应的 FDL 统计特征，能否在一定程度上预测 HNSW 达到目标 Recall 所需要的 efSearch？**
>
> 优先复用原来标准高斯实验的 5000 条数据库向量和 200 条查询。
>
> 保持原始数据完全一致，不要重新生成。
>
> ### 实验方法
>
> 1. 对每条查询计算真实 FDL 的均值、方差、偏度及多个分位数。
> 2. 使用已有 FAISS HNSW 配置建立索引。
> 3. 保持原来的建图参数、随机种子和 Ground Truth 计算方式。
> 4. 使用 `SearchParametersHNSW` 为每条查询独立设置 efSearch。
> 5. 扫描多个 efSearch 档位，至少覆盖原实验使用的 8、16、32、64、128。
> 6. 记录每条查询在各 efSearch 下的 Recall@10。
> 7. 找出每条查询首次达到 Recall@10 ≥ 0.95 的搜索预算。
>
> ### 分析内容
>
> 计算：
>
> - FDL 均值与实际所需 efSearch 的 Spearman 相关系数。
> - FDL 方差与实际所需 efSearch 的 Spearman 相关系数。
> - FDL 低分位数与实际所需 efSearch 的相关系数。
> - 不同 FDL 特征之间的相关性。
> - 不同查询的搜索难度分布。
>
> 绘制：
>
> 1. FDL 特征与所需 efSearch 的散点图。
> 2. 不同 efSearch 下的逐查询 Recall 分布。
> 3. FDL 特征与搜索难度的相关性热图。
>
> 特别注意：
>
> 目前使用的是完整 FDL，因此这只是理论可预测性实验，并不代表在线算法已经能够免费获得完整距离列表。
>
> 后续还需要研究如何从少量初期搜索距离中估计这些统计特征。
>
> 如果发现 FDL 特征与 HNSW 搜索难度相关性很弱，也请如实报告，不要人为制造正相关结论。
>
> ## 四、工程要求
>
> 1. 所有操作限定在 `F:\ocbigdata`。
> 2. 使用现有 uv 虚拟环境。
> 3. 优先使用 Windows 原生 Python，不使用 WSL。
> 4. 不修改系统级配置。
> 5. 不删除或覆盖已有实验结果。
> 6. 不修改已经通过的 `hnsw_smoke.py` 和第一轮 FDL 实验脚本，除非确有必要；如需修改，先保留原版本。
> 7. 控制内存使用，避免构造巨大的三维距离数组。
> 8. 不下载完整 SIFT1M，不运行长时间正式实验。
> 9. 新实验应有独立的配置、输出目录和日志。
>
> ## 五、预期交付物
>
> 新增：
>
> - 各向异性高斯数据生成脚本。
> - 广义非中心卡方分布模型实现。
> - 各向异性 FDL 验证脚本。
> - FDL—HNSW 搜索难度关联实验脚本。
> - CSV、JSON 实验结果。
> - 分布对比图、分位数误差图、相关性分析图。
> - 实验说明文档。
>
> ## 六、最终需要回答的问题
>
> 请在实验结束后给出一份研究分析报告，回答：
>
> 1. 广义非中心卡方模型是否能够准确描述各向异性高斯数据的 L2 距离分布？
> 2. 正态近似与广义非中心卡方模型相比，分别有什么优缺点？
> 3. 协方差矩阵的特征值分布和条件数会如何影响模型表现？
> 4. FDL 的哪些统计特征与 HNSW 搜索难度具有较强相关性？
> 5. FDL 是否足以预测 efSearch，还是必须引入图结构特征？
> 6. 是否值得继续开发 L2-Ada-ef？
>
> 请区分理论结论、实验观察和研究推测，不要将相关性描述为因果关系。
>
> 最后提供实际运行的命令、文件清单、实验耗时和关键数值。不要仅仅创建代码而不运行验证。

<details><summary>13 previous messages</summary>

> 我会把这次工作拆成两个完全独立的新实验目录：2A 采用解析矩公式加独立 Monte Carlo 广义非中心卡方近似，并同时对比正态近似；2B 严格复用现有标准高斯 `make_data()`、原 HNSW 参数和 Ground Truth 方法。已有标准高斯脚本、冒烟脚本及结果只读检查，不会重跑或覆盖。先核对当前文件和第一轮结果，然后直接实现并运行两组实验。
>
> <details><summary>Ran 3 commands</summary>
>
> - Ran `$ErrorActionPreference='Stop'; Write-Output '--- current files ---'; rg --files -g '!**/.venv/**' -g '!**/.uv-cache/**' -g '!**/.git/**' | Sort-Object; Write-Output '--- pyproject ---'; Get-Content -Raw pyproject.toml; Write-Output '--- shared data ---'; Get-Content -Raw scripts\data.py; Write-Output '--- original HNSW config/functions ---'; Select-String -Path scripts\hnsw_smoke.py -Pattern 'def exact_ground_truth','def make_data','--m','--ef-construction','--ef-search','--seed' -Context 0,5; Write-Output '--- first FDL result ---'; Get-Content -Raw results\fdl_validation_latest.json; Write-Output '--- HNSW result ---'; Get-Content -Raw results\hnsw_smoke_latest.json; Write-Output '--- timestamps ---'; Get-ChildItem results\hnsw_smoke_latest.*,results\fdl_validation_latest*,logs\hnsw_smoke_latest.log,logs\fdl_validation_latest.log | Select-Object Name,Length,LastWriteTime`
> - Ran `uv run python -c "import faiss; x=faiss.IndexHNSWFlat(64,16); print([n for n in dir(x.hnsw) if 'rng' in n.lower() or 'seed' in n.lower()]); print('threads',faiss.omp_get_max_threads())"`
> - Ran `uv run python -c "import faiss; x=faiss.IndexHNSWFlat(64,16); print(type(x.hnsw.rng)); print([n for n in dir(x.hnsw.rng) if not n.startswith('_')])"`
>
> </details>
>
> 方案已确定：2A 的三种协方差都归一化为 `tr(Σ)=64`，这样条件数和谱形变化更容易横向解释；查询与数据库在每个场景中来自同一个各自独立的生成器，并严格按“数据库先、查询后”生成。广义模型使用每场景 50,000 个独立标准正态样本，通过特征空间的二次型恒等式批量复用，最坏点位的 Monte Carlo CDF 标准误差约为 `0.00224`，不会冒充精确 CDF。
>
> <details><summary>Created 4 files</summary>
>
> - Wrote `F:/ocbigdata/scripts/anisotropic_data.py` (+97 -0)
> - Wrote `F:/ocbigdata/scripts/generalized_chi_square.py` (+49 -0)
> - Wrote `F:/ocbigdata/scripts/anisotropic_fdl_validation.py` (+262 -0)
> - Wrote `F:/ocbigdata/scripts/fdl_hnsw_correlation.py` (+262 -0)
>
> </details>
>
> 两组脚本已经就位，且没有改动 `hnsw_smoke.py` 或第一轮 FDL 脚本。2B 额外扫描到 `efSearch=512`，因为单查询 `Recall@10 ≥ 0.95` 在离散的 10 个近邻上实际等价于 10/10 全命中；这能减少“最高档仍未达标”的删失样本，同时仍是小规模实验。接下来先做语法检查，再运行完整 2A 和 2B。
>
> <details><summary>Ran 3 commands</summary>
>
> - Ran `$ErrorActionPreference='Stop'; uv run python -m compileall -q scripts; uv run python scripts\anisotropic_fdl_validation.py --help | Select-Object -First 12; uv run python scripts\fdl_hnsw_correlation.py --help | Select-Object -First 14`
> - Ran `uv run python scripts\anisotropic_fdl_validation.py`
> - Ran `uv run python scripts\fdl_hnsw_correlation.py`
>
> </details>
>
> 两组完整实验均已跑通。2A 中广义模型在三种协方差下的平均 KS 约 `0.0133–0.0139`，明显优于正态近似的 `0.0306–0.0562`；2B 的各档平均 Recall 与原冒烟结果逐项一致，说明复用数据和 HNSW 构建配置成功。当前有 2/200 条查询到 `ef=512` 仍未达到 10/10，全量分析会明确按右删失处理，避免把它们伪装成真实的 `ef=1024` 观测。现在检查图表和相关系数，再形成研究报告。
>
> <details><summary>Ran a command</summary>
>
> - Ran `$ErrorActionPreference='Stop'; Write-Output '--- 2A JSON ---'; Get-Content -Raw results\anisotropic_fdl\result.json; Write-Output '--- 2A summary ---'; Get-Content -Raw results\anisotropic_fdl\summary.csv; Write-Output '--- 2B JSON ---'; Get-Content -Raw results\fdl_hnsw_correlation\result.json; Write-Output '--- 2B correlations ---'; Get-Content -Raw results\fdl_hnsw_correlation\spearman_correlations.csv; Write-Output '--- output files ---'; Get-ChildItem results\anisotropic_fdl,results\fdl_hnsw_correlation,logs\anisotropic_fdl,logs\fdl_hnsw_correlation -Recurse -File | Select-Object FullName,Length,LastWriteTime`
>
> </details>
>
> Viewed 6 images
>
> 图表检查通过，且出现了一个值得保留的结论：条件数并非决定正态近似误差的唯一因素。条件数约 78 的 AR(1) 场景比条件数 100 的旋转谱更难近似，因为它的 `tr(Σ²)` 更大、谱质量更集中在少数大特征值上。我会把这个“谱形优于单一条件数”的观察写进报告，并把 2B 的弱相关结论和两条右删失查询明确分开陈述。
>
> <details><summary>Created a file, edited 4 files, ran 3 commands</summary>
>
> - Wrote `F:/ocbigdata/README.md` (+22 -0)
> - Wrote `F:/ocbigdata/docs/experiment_2.md` (+119 -0)
> - Wrote `F:/ocbigdata/scripts/anisotropic_fdl_validation.py` (+6 -3)
> - Wrote `F:/ocbigdata/scripts/fdl_hnsw_correlation.py` (+3 -1)
> - Ran `$ErrorActionPreference='Stop'; uv run python scripts\anisotropic_fdl_validation.py; uv run python scripts\fdl_hnsw_correlation.py`
> - Ran `$ErrorActionPreference='Stop'; $a=Get-Content -Raw results\anisotropic_fdl\result.json | ConvertFrom-Json; $b=Get-Content -Raw results\fdl_hnsw_correlation\result.json | ConvertFrom-Json; Write-Output ('2A elapsed=' + $a.elapsed_seconds); Write-Output ('2B elapsed=' + $b.timings_seconds.total + ' build=' + $b.timings_seconds.index_build); Write-Output '--- existing result timestamps unchanged ---'; Get-ChildItem results\hnsw_smoke_latest.*,results\fdl_validation_latest*,logs\hnsw_smoke_latest.log,logs\fdl_validation_latest.log | Select-Object Name,Length,LastWriteTime; Write-Output '--- files ---'; rg --files -g '!**/.venv/**' -g '!**/.uv-cache/**' -g '!**/.git/**' | Sort-Object; Write-Output '--- git status ---'; git status --short`
> - Wrote `F:/ocbigdata/docs/experiment_2.md` (+2 -2)
> - Ran `uv run python -c "vals=[74.2407236970324,281.67901234568296,152.44585438673215]; print([4096/v for v in vals])"`
> - Wrote `F:/ocbigdata/docs/experiment_2.md` (+5 -5)
>
> </details>
>
> 最终复核将只检查产物完整性、行数、状态字段和旧结果时间戳，不再重跑已有实验。研究报告已经写入项目，包含理论结论、实验观察、研究推测、限制、命令和耗时。
>
> <details><summary>Ran a command</summary>
>
> - Ran `$ErrorActionPreference='Stop'; uv run python -c "import csv,json,pathlib; root=pathlib.Path('.'); a=json.load(open(root/'results/anisotropic_fdl/result.json',encoding='utf-8')); b=json.load(open(root/'results/fdl_hnsw_correlation/result.json',encoding='utf-8')); ar=list(csv.DictReader(open(root/'results/anisotropic_fdl/per_query.csv',encoding='utf-8'))); br=list(csv.DictReader(open(root/'results/fdl_hnsw_correlation/per_query.csv',encoding='utf-8'))); assert a['status']==b['status']=='passed'; assert len(ar)==600 and len(br)==200; assert all((root/'results/anisotropic_fdl'/p).exists() for p in a['outputs']); assert all((root/'results/fdl_hnsw_correlation'/p).exists() for p in b['outputs']); print('artifact_validation=passed anisotropic_rows=',len(ar),'hnsw_rows=',len(br),'censored=',b['censored_query_count'])"; Get-ChildItem scripts\anisotropic_data.py,scripts\generalized_chi_square.py,scripts\anisotropic_fdl_validation.py,scripts\fdl_hnsw_correlation.py,docs\experiment_2.md | Select-Object FullName,Length; Get-ChildItem results\hnsw_smoke_latest.json,results\fdl_validation_latest.json | Select-Object Name,LastWriteTime`
>
> </details>

</details>

已完成实验 2A、2B 的完整实现与实际运行，两项状态均为 `passed`。原 `hnsw_smoke.py`、第一轮 FDL 脚本及其结果均未修改或覆盖。

完整研究报告：[experiment_2.md](F:/ocbigdata/docs/experiment_2.md)

## 核心结论

### 1. 广义非中心卡方模型是否准确？

准确。三种协方差结构下，基于 50,000 个独立 Monte Carlo 样本的广义模型平均 KS 均约为 `0.013`：

| 协方差结构     | 条件数 | 广义模型 KS | 正态 KS | 广义 W1 | 正态 W1 |
| -------------- | ------:| -----------:| -------:| -------:| -------:|
| 对角各向异性   | 4.00   | 0.01389     | 0.03062 | 0.4498  | 1.2431  |
| AR(1) 相关高斯 | 78.05  | 0.01326     | 0.05620 | 0.8018  | 5.2580  |
| 旋转病态谱     | 100.00 | 0.01326     | 0.04465 | 0.5934  | 2.7269  |

理论分布明确标记为 Monte Carlo 近似，而非精确 CDF。最坏点位 CDF 标准误差约 `0.00224`。

### 2. 正态近似与广义模型

- 广义模型保留完整特征值谱、偏度和尾部形状，整体 KS、Wasserstein 和尾部分位数明显更好。
- 正态近似只需解析均值、方差，计算便宜，但无法准确表达偏度，谱集中时 P1/P99 误差很大。
- 例如 AR(1) 场景的 P1 平均绝对误差：广义模型 `0.808`，正态近似 `23.848`。
- 正态近似还可能给本应非负的平方距离分配负值概率。

### 3. 特征值谱与条件数

条件数不是充分指标：

- 对角谱有效维数：`55.17`
- AR(1) 有效维数：`14.54`
- 旋转病态谱有效维数：`26.87`

AR(1) 条件数低于旋转谱，但 `tr(Σ²)` 更大、有效维数更低，因此正态近似反而更差。谱集中程度、最大特征值占比和有效维数比单独使用条件数更有解释力。

### 4. FDL 与 HNSW 搜索难度

最强特征是低尾 P0.1，但仍然只是弱关联：

| FDL 特征 | 所需 efSearch 的 Spearman ρ |
| -------- | ---------------------------:|
| P0.1     | 0.2266                      |
| P1       | 0.2120                      |
| P99      | 0.2040                      |
| P10      | 0.1997                      |
| P5       | 0.1992                      |
| 均值     | 0.1977                      |
| 方差     | 0.1878                      |
| 偏度     | 0.1217，不显著              |

均值、方差和多数分位数之间高度共线，通常 `ρ≈0.97–1.00`，所以增加多个此类特征并没有带来很多独立信息。

搜索难度分布：

- ef=16：4 条查询首次全命中
- ef=32：21 条
- ef=64：93 条
- ef=128：53 条
- ef=256：22 条
- ef=512：5 条
- 超过 512：2 条右删失查询

由于单查询 Recall@10 只能取 0.1 的倍数，目标 `≥0.95` 实际等价于 10/10 全命中。

### 5. FDL 是否足以预测 efSearch？

当前证据不支持仅靠完整 FDL 稳定预测 efSearch。散点在各预算档位大量重叠，最强单特征相关也只有 `0.227`。

更可能需要联合：

- 入口点和局部图连通性
- 初期候选距离与收缩速度
- 已访问节点数
- 候选队列变化
- 最近邻距离间隔
- 搜索过程中的局部密度信号

这些是研究推测，不是因果结论。

### 6. 是否值得继续 L2-Ada-ef？

值得继续，但建议定位为“FDL 低尾估计 + 在线图搜索信号”的联合策略，而不是纯 FDL 阈值规则。下一阶段应检验在线早期信号能否在固定 Recall 下减少平均访问量或延迟。

## 实际运行

```powershell
uv run python scripts/anisotropic_fdl_validation.py
uv run python scripts/fdl_hnsw_correlation.py
```

- 实验 2A：`12.563` 秒
- 实验 2B：`1.572` 秒，其中建图 `0.204` 秒
- 2A 验证了 `600` 条逐查询记录
- 2B 验证了 `200` 条逐查询记录
- 前五档平均 Recall 与原冒烟实验逐项完全一致

## 新增文件

代码：

- [anisotropic_data.py](F:/ocbigdata/scripts/anisotropic_data.py)
- [generalized_chi_square.py](F:/ocbigdata/scripts/generalized_chi_square.py)
- [anisotropic_fdl_validation.py](F:/ocbigdata/scripts/anisotropic_fdl_validation.py)
- [fdl_hnsw_correlation.py](F:/ocbigdata/scripts/fdl_hnsw_correlation.py)

2A 结果：

- [result.json](F:/ocbigdata/results/anisotropic_fdl/result.json)
- [summary.csv](F:/ocbigdata/results/anisotropic_fdl/summary.csv)
- [per_query.csv](F:/ocbigdata/results/anisotropic_fdl/per_query.csv)
- [分布对比图](F:/ocbigdata/results/anisotropic_fdl/distribution_comparison.png)
- [分位数误差图](F:/ocbigdata/results/anisotropic_fdl/quantile_errors.png)
- [协方差谱图](F:/ocbigdata/results/anisotropic_fdl/covariance_spectra.png)

2B 结果：

- [result.json](F:/ocbigdata/results/fdl_hnsw_correlation/result.json)
- [per_query.csv](F:/ocbigdata/results/fdl_hnsw_correlation/per_query.csv)
- [Spearman 结果](F:/ocbigdata/results/fdl_hnsw_correlation/spearman_correlations.csv)
- [特征—预算散点图](F:/ocbigdata/results/fdl_hnsw_correlation/feature_vs_required_ef.png)
- [Recall 与难度分布](F:/ocbigdata/results/fdl_hnsw_correlation/recall_and_difficulty.png)
- [相关性热图](F:/ocbigdata/results/fdl_hnsw_correlation/correlation_heatmap.png)

![FDL 与 HNSW 搜索预算](F:/ocbigdata/results/fdl_hnsw_correlation/feature_vs_required_ef.png)

> # 实验 3A：HNSW 在线搜索特征与 FDL 联合分析
>
> 请在当前项目基础上继续开展实验 3A。
>
> 本次不需要重新介绍项目背景、重新初始化环境或重复之前的实验。请直接检查现有项目代码和实验结果，重点阅读：
>
> - `scripts/fdl_hnsw_correlation.py`
> - `scripts/data.py`
> - 已有 HNSW 建图和查询代码
> - `results/fdl_hnsw_correlation/`
> - 前一轮实验报告 `experiment_2.md`
>
> 本次研究重点是：
>
> **在已有 FDL 统计特征的基础上，引入 HNSW 搜索过程中的图结构特征，研究它们能否更好地解释和预测单条查询的搜索难度。**
>
> ## 一、实验目标
>
> 前一轮实验发现：
>
> - FDL 的 P0.1 与实际所需 efSearch 的 Spearman 相关系数为 0.2266。
> - FDL 均值、方差的相关性也较弱。
> - 单纯使用完整 FDL 难以稳定预测 HNSW 搜索难度。
>
> 本次希望验证：
>
> 1. HNSW 搜索过程中有哪些可观测的在线特征？
> 2. 这些特征与实际搜索难度之间有什么关系？
> 3. 图搜索特征是否比 FDL 特征更有解释力？
> 4. FDL 与图搜索特征结合后，是否能提供更多独立信息？
>
> 本次只进行特征采集和统计分析，暂时不实现预测模型或自适应算法。
>
> ## 二、第一步：调查 FAISS 的特征采集能力
>
> 先检查当前安装的 FAISS 1.16.0 Python API 和 HNSW 实现。
>
> 重点调查能否获取以下信息：
>
> | 特征                           | 含义                     | 优先级 |
> | ------------------------------ | ------------------------ | ------ |
> | ndis                           | 距离计算次数             | 高     |
> | nhops                          | 图搜索跳转次数           | 高     |
> | visited nodes                  | 实际访问节点数           | 高     |
> | candidate queue size           | 候选队列大小             | 高     |
> | best distance progression      | 搜索过程中最优距离的变化 | 高     |
> | candidate distance gap         | 最优与次优候选的距离间隔 | 中     |
> | search termination information | 搜索终止条件或终止状态   | 中     |
>
> 首先检查 FAISS 已有的 `HNSWStats`、Python bindings 及其他可用接口。
>
> 如果统计信息是全局计数器，采用单线程、逐查询重置和读取增量的方式，确保能够获得每条查询独立的统计结果。
>
> 如果某些特征无法通过现有 Python API 获取：
>
> - 明确说明缺少哪些信息。
> - 调查是否需要修改 FAISS C++ 源码。
> - 评估修改和编译的可行性。
> - 不要伪造或通过推测生成这些特征。
> - 第一轮优先使用现有 API 能可靠获取的指标。
>
> 暂时不要为了获取候选队列内部状态而贸然替换 FAISS 或修改系统环境。
>
> ## 三、第二步：采集在线搜索特征
>
> 严格复用上一轮的标准高斯数据：
>
> - 数据库：5000 × 64。
> - 查询：200 × 64。
> - 随机种子：20261115。
> - 数据库先生成，查询后生成。
> - float32。
> - 不改变原始数据。
>
> 沿用现有 HNSW 索引配置，不要擅自改变建图参数。
>
> 对每条查询，在以下 efSearch 档位进行独立测试：
>
> `16、32、64、128`
>
> 如果资源允许，再测试 256 和 512。
>
> 重点记录：
>
> 1. 每次查询的距离计算次数。
> 2. 每次查询的图搜索跳转次数。
> 3. 实际访问节点数（如果能够可靠获得）。
> 4. 搜索过程中可获得的候选集合统计。
> 5. 不同 efSearch 下上述特征的变化。
>
> 特别注意：
>
> 本次希望研究的是能够在搜索早期获得的信号。
>
> 因此，请尽可能区分：
>
> - 搜索结束后才能获得的完整统计量。
> - 搜索过程中可以实时获得的统计量。
> - 在 efSearch 较小时就能获得的早期特征。
>
> 这三类信息的实际用途不同，不能混为一谈。
>
> ## 四、第三步：与 FDL 特征进行联合分析
>
> 复用前一轮已经计算的 FDL 特征，不必重复计算完整距离分布。
>
> 至少包括：
>
> - FDL 均值。
> - FDL 方差。
> - FDL 偏度。
> - P0.1、P1、P5、P10。
> - P50、P90、P99。
>
> 将每条查询的 FDL 特征与对应的 HNSW 在线搜索特征关联起来。
>
> 分析：
>
> 1. 各特征与实际所需 efSearch 的 Spearman 相关系数。
> 2. 各图搜索特征与 Recall@10 的关系。
> 3. 图搜索特征之间的相关性。
> 4. FDL 特征与图搜索特征之间的相关性。
> 5. 不同 efSearch 档位下特征相关性的变化。
>
> 特别关注：
>
> - ndis 是否比 FDL 统计量更能解释搜索难度？
> - nhops 是否提供了 ndis 无法提供的信息？
> - 早期搜索的统计量是否能够区分简单查询和困难查询？
> - FDL 特征是否能够补充图搜索特征？
>
> 建议绘制：
>
> - 特征相关性热图。
> - FDL 特征与图搜索特征的散点图。
> - 不同 efSearch 下 ndis、nhops 的分布图。
> - 搜索难度与早期搜索特征的关系图。
>
> 对于显著性检验，应考虑多重比较问题，必要时使用 Benjamini-Hochberg 方法校正 p 值，并报告效应大小，不要仅依据 p 值判断特征价值。
>
> ## 五、第四步：完善搜索难度标签
>
> 上一轮使用的目标是：
>
> `Recall@10 >= 0.95`
>
> 由于 Recall@10 只能以 0.1 为步长变化，这实际上要求 10 个近邻全部命中。
>
> 本次建议同时保留两个目标：
>
> - Recall@10 ≥ 0.9。
> - Recall@10 ≥ 0.95。
>
> 扫描 efSearch 时，如果 512 仍未达到目标，可以继续测试 1024、2048，直到达到目标或达到合理上限。
>
> 注意：
>
> 如果查询仍未达到目标，应标记为右删失数据，不得将任意指定的 ef 数值当作真实所需预算。
>
> 分别报告：
>
> - 实际观测到的最小 efSearch。
> - 未达到目标的查询数量。
> - 右删失比例。
>
> 必要时进行删失数据敏感性分析。
>
> ## 六、第五步：初步判断特征的独立贡献
>
> 本次暂时不训练复杂机器学习模型。
>
> 先完成三组特征对比：
>
> A. 仅使用 FDL 特征。
>
> B. 仅使用 HNSW 在线搜索特征。
>
> C. FDL + HNSW 在线搜索特征。
>
> 采用相关性分析和简单的统计比较，判断三组特征分别能提供什么信息。
>
> 可以尝试对高度相关的特征进行去冗余分析，但不要直接把相关性解释成因果关系。
>
> 如果数据足够支持，可以补充一个简单的交叉验证预测基线，用于初步比较三组特征的增量价值；但不要把这一步扩展成完整的实验 3B。
>
> ## 七、工程要求
>
> 1. 所有操作限定在 `F:\ocbigdata`。
> 2. 使用现有 uv 虚拟环境。
> 3. 使用 Windows 原生 Python，不使用 WSL。
> 4. 不修改系统级配置。
> 5. 不删除或覆盖前两轮实验结果。
> 6. 不修改已经验证通过的原始实验脚本。
> 7. 新增脚本和结果使用独立目录。
> 8. 保持实验可复现，记录随机种子、索引参数、线程数和软件版本。
> 9. 不运行长时间正式实验，优先完成当前 200 条查询的分析。
> 10. 对 FAISS 无法直接提供的特征，如实记录技术限制。
>
> ## 八、预期交付物
>
> 在项目内新增：
>
> - HNSW 搜索特征采集脚本。
> - 特征关联分析脚本。
> - 每条查询的详细 CSV。
> - 特征相关性汇总 CSV。
> - JSON 实验配置与汇总。
> - 特征相关性热图。
> - 不同 efSearch 下的搜索统计图。
> - 实验运行日志。
> - `experiment_3a.md` 研究分析报告。
>
> ## 九、最终需要回答的问题
>
> 请重点回答：
>
> 1. 当前 FAISS 1.16.0 能够可靠采集哪些 HNSW 内部搜索特征？
> 2. ndis、nhops 等指标与实际搜索难度的关系如何？
> 3. 哪些特征在搜索早期就能获得？
> 4. 图搜索特征是否明显优于 FDL 特征？
> 5. FDL 与图搜索特征结合后是否提供了额外信息？
> 6. 哪些特征适合未来用于在线 efSearch 自适应控制？
> 7. 是否有必要进一步修改 FAISS 底层代码以采集候选队列和搜索过程信息？
>
> 请严格区分：
>
> - 实验实际测得的结果。
> - 根据结果作出的统计解释。
> - 尚未验证的研究推测。
>
> 最后给出下一阶段实验建议。
>
> 本次实验的核心不是尽可能多地采集指标，而是找到**少量、可靠、能够在搜索早期获得，并且对查询难度具有解释力的特征**。

<details><summary>12 previous messages</summary>

> 我会先只做本地能力调查：检查 `faiss.cvar.hnsw_stats` 的字段、重置行为、单查询增量，以及虚拟环境中随包提供的 HNSW 头文件/绑定定义。只有能在 Python API 中逐查询可靠复现的指标才会进入实验；候选队列、访问节点集合和最优距离轨迹如果不可见，会明确列为缺失能力，不做替代推测。随后复用上一轮 CSV 中的 FDL 与难度标签，新增独立的 `results/experiment_3a/` 和日志目录。
>
> <details><summary>Ran 2 commands</summary>
>
> - Ran `$ErrorActionPreference='Stop'; Write-Output '--- requested sources ---'; Get-Content -Raw scripts\fdl_hnsw_correlation.py; Write-Output '--- data ---'; Get-Content -Raw scripts\data.py; Write-Output '--- report ---'; Get-Content -Raw docs\experiment_2.md; Write-Output '--- prior output headers/counts ---'; Get-Content results\fdl_hnsw_correlation\per_query.csv -TotalCount 2; (Import-Csv results\fdl_hnsw_correlation\per_query.csv).Count; Write-Output '--- FAISS stats API ---'; uv run python -c "import faiss; print('version',faiss.__version__); print('has cvar',hasattr(faiss,'cvar')); print('hnsw_stats',faiss.cvar.hnsw_stats); s=faiss.cvar.hnsw_stats; print(type(s)); print([x for x in dir(s) if not x.startswith('_')]); print({x:getattr(s,x) for x in ['n1','n2','ndis','nhops'] if hasattr(s,x)}); print('reset',hasattr(s,'reset'))"; Write-Output '--- package files ---'; uv run python -c "import faiss,pathlib; p=pathlib.Path(faiss.__file__).resolve(); print(p); print(p.parent)"`
> - Ran `$ErrorActionPreference='Stop'; $site='F:\ocbigdata\.venv\Lib\site-packages'; Write-Output '--- HNSWStats definitions/references ---'; rg -n -i "HNSWStats|hnsw_stats|nhops|ndis" $site\faiss -g '*.py' -g '*.i' -g '*.h' -g '*.cpp' | Select-Object -First 200; Write-Output '--- package tree hints ---'; Get-ChildItem $site\faiss -Recurse -File -Include '*.h','*.i','*.cpp' | Select-Object -First 30 FullName; Write-Output '--- binding class excerpt ---'; rg -n -A45 -B10 "class HNSWStats" $site\faiss\swigfaiss.py`
>
> </details>
>
> 本地绑定确认了四个全局计数器 `n1/n2/ndis/nhops` 和 `reset()`，但生成的 Python 文档字符串存在明显错位：`ndis` 被标成候选耗尽、`nhops` 被标成距离次数，而 `reset()` 又被标成 hop。为避免误读，我会对照 FAISS 1.16.0 官方源码定义和实际单查询计数行为，再确定字段语义。
>
> <details><summary>Ran a command, searched the web</summary>
>
> Searched the web for `site:github.com/facebookresearch/faiss v1.16.0 HNSWStats n1 n2 ndis nhops HNSW.h ...`
>
> - Ran `$ErrorActionPreference='Stop'; Get-Content F:\ocbigdata\.venv\Lib\site-packages\faiss\swigfaiss.py | Select-Object -Skip 1060 -First 35; uv run python -c "import faiss,sys,numpy as np; sys.path.insert(0,'scripts'); from data import make_data; db,q=make_data(); faiss.omp_set_num_threads(1); idx=faiss.IndexHNSWFlat(64,16,faiss.METRIC_L2); idx.hnsw.efConstruction=80; idx.add(db); s=faiss.cvar.hnsw_stats; print('before',s.n1,s.n2,s.ndis,s.nhops); 
>   for ef in [16,32,64,128]:
>    p=faiss.SearchParametersHNSW(); p.efSearch=ef; s.reset(); idx.search(q[:1],10,params=p); print(ef,s.n1,s.n2,s.ndis,s.nhops)
>   s.reset(); idx.search(q[:2],10,params=p); print('batch2',s.n1,s.n2,s.ndis,s.nhops)"`
>
> Searched the web for `https://raw.githubusercontent.com/facebookresearch/faiss/v1.16.0/faiss/impl/HNSW.h`
>
> Searched the web for `'struct HNSWStats'`
>
> </details>
>
> 调查结论已经足够明确，可以进入采集：
>
> - 可靠可取：`ndis`（实际距离计算次数）、`nhops`（候选展开/图遍历步数）、`n1`（搜索查询数）、`n2`（候选列表耗尽的查询数）。
> - 采集方法：单线程、每条查询前 `reset()`、搜索后立即读取；官方说明全局统计只有在非并发搜索时才保证准确。
> - 不可直接取：唯一访问节点集合/数量、候选队列当前或峰值大小、最优距离演进、完整终止原因和可恢复搜索状态。
> - 可补充但不冒充内部队列：每次搜索返回的 top-10 距离、首二名间隔、结果跨度等“搜索完成后输出特征”。
>
> 需要特别区分：Python 的 `index.search()` 是阻塞调用，所以 `ndis/nhops` 虽在 C++ 搜索过程中累积，Python 只能在一次搜索结束后读取。低 `ef=16/32` 的统计可作为“早期探测结束后的信号”，但再次提高 ef 会从头搜索，当前 API 不能续跑。
>
> <details><summary>Created 2 files, ran 3 commands</summary>
>
> - Wrote `F:/ocbigdata/scripts/hnsw_search_feature_collection.py` (+234 -0)
> - Wrote `F:/ocbigdata/scripts/hnsw_feature_analysis.py` (+342 -0)
> - Ran `$ErrorActionPreference='Stop'; uv run python -m compileall -q scripts; uv run python scripts\hnsw_search_feature_collection.py --help | Select-Object -First 15; uv run python scripts\hnsw_feature_analysis.py --help | Select-Object -First 10`
> - Ran `uv run python scripts\hnsw_search_feature_collection.py`
> - Ran `uv run python scripts\hnsw_feature_analysis.py`
>
> </details>
>
> 采集与联合分析已完成。所有查询在 `ef=1024` 前都达到两个 Recall 目标，因此本轮难度标签没有右删失；`ef=2048` 作为上限验证也全部满召回。一个重要的 API 行为是：`n2` 在 `ef≥16` 时对 200 条查询全部为 1，几乎没有区分力；可靠不等于有预测价值。下面我会读取经 BH 校正后的效应、偏相关和图表，判断 `ndis/nhops` 是否真的比 FDL 更有信息。
>
> <details><summary>Ran 2 commands</summary>
>
> - Ran `$ErrorActionPreference='Stop'; Write-Output '--- collection summary ---'; Get-Content -Raw results\experiment_3a\collection_summary.json; Write-Output '--- analysis summary ---'; Get-Content -Raw results\experiment_3a\analysis_summary.json; Write-Output '--- partial correlations ---'; Get-Content -Raw results\experiment_3a\partial_correlations.csv; Write-Output '--- selected correlations ---'; Import-Csv results\experiment_3a\correlations.csv | Where-Object { ($_.analysis -eq 'online_vs_difficulty' -and $_.ef_search -in @('8','16','32') -and $_.target -eq 'required_ef_95') -or ($_.analysis -eq 'online_vs_recall' -and $_.ef_search -in @('16','32') -and $_.feature -match 'ndis|nhops') -or ($_.analysis -eq 'fdl_vs_difficulty' -and $_.target -eq 'required_ef_95') } | Format-Table analysis,feature,target,ef_search,spearman_rho,pvalue,bh_qvalue,sample_size -AutoSize | Out-String -Width 240; Write-Output '--- labels ---'; Import-Csv results\experiment_3a\search_features.csv | Group-Object required_ef_90 | Select-Object Name,Count; Import-Csv results\experiment_3a\search_features.csv | Group-Object required_ef_95 | Select-Object Name,Count; Write-Output '--- files ---'; Get-ChildItem results\experiment_3a,logs\experiment_3a -Recurse -File | Select-Object FullName,Length,LastWriteTime`
> - Ran ``$ErrorActionPreference='Stop'; $c=Import-Csv results\experiment_3a\correlations.csv; Write-Output '--- top online vs required 0.90 ---'; $c | Where-Object { $_.analysis -eq 'online_vs_difficulty' -and $_.target -eq 'required_ef_90' -and $_.spearman_rho -ne 'nan' } | Sort-Object {[math]::Abs([double]$_.spearman_rho)} -Descending | Select-Object -First 15 analysis,feature,ef_search,spearman_rho,pvalue,bh_qvalue | Format-Table -AutoSize | Out-String -Width 220; Write-Output '--- ndis/nhops vs difficulty across ef ---'; $c | Where-Object { $_.analysis -eq 'online_vs_difficulty' -and $_.feature -match '^(ndis|nhops)_ef_' } | Select-Object feature,target,ef_search,spearman_rho,pvalue,bh_qvalue | Format-Table -AutoSize | Out-String -Width 220; Write-Output '--- ndis vs nhops ---'; $c | Where-Object { $_.analysis -eq 'online_internal' } | Select-Object feature,target,ef_search,spearman_rho,pvalue,bh_qvalue | Format-Table -AutoSize | Out-String -Width 220; Write-Output '--- FDL vs online 16/32 ---'; $c | Where-Object { $_.analysis -eq 'fdl_vs_online' -and $_.ef_search -in @('16','32') } | Select-Object feature,target,ef_search,spearman_rho,pvalue,bh_qvalue | Format-Table -AutoSize | Out-String -Width 220; Write-Output '--- exact file list ---'; Get-ChildItem results\experiment_3a,logs\experiment_3a -Recurse -File | ForEach-Object { '{0}`t{1}' -f $_.FullName,$_.Length }``
>
> </details>
>
> Viewed 5 images
>
> <details><summary>Ran a command</summary>
>
> - Ran `$ErrorActionPreference='Stop'; $c=Import-Csv results\experiment_3a\correlations.csv; Write-Output '--- FDL required 0.90 ---'; $c | Where-Object { $_.analysis -eq 'fdl_vs_difficulty' -and $_.target -eq 'required_ef_90' } | Sort-Object {[math]::Abs([double]$_.spearman_rho)} -Descending | Format-Table feature,spearman_rho,pvalue,bh_qvalue -AutoSize | Out-String -Width 200; Write-Output '--- online vs recall 32 strongest ---'; $c | Where-Object { $_.analysis -eq 'online_vs_recall' -and $_.ef_search -eq '32' -and $_.spearman_rho -ne 'nan' } | Sort-Object {[math]::Abs([double]$_.spearman_rho)} -Descending | Format-Table feature,spearman_rho,pvalue,bh_qvalue -AutoSize | Out-String -Width 200`
>
> </details>
>
> 统计结果给出了很清晰的负面结论：`ndis/nhops` 在这个标准高斯、固定图上的早期值并没有明显优于 FDL。对 0.95 难度标签，`ef=16` 的相关分别只有 `-0.080/-0.063`，BH 校正后均不显著；两者自身却高度相关（`ρ=0.883`），因此 `nhops` 几乎没有额外信息。相反，低预算搜索返回的最佳距离有弱关联，但它与 FDL P0.1 高度相关（`ρ=0.89`），偏相关后也不再提供独立贡献。这一结果会原样写入报告，不把“可采集”误写成“有预测力”。
>
> <details><summary>Created a file, edited a file, ran a command</summary>
>
> - Wrote `F:/ocbigdata/README.md` (+15 -1)
> - Wrote `F:/ocbigdata/docs/experiment_3a.md` (+170 -0)
> - Ran `$ErrorActionPreference='Stop'; uv run python -c "import csv,json,pathlib,math; root=pathlib.Path('.'); c=json.load(open(root/'results/experiment_3a/collection_summary.json',encoding='utf-8')); a=json.load(open(root/'results/experiment_3a/analysis_summary.json',encoding='utf-8')); s=list(csv.DictReader(open(root/'results/experiment_3a/search_features.csv',encoding='utf-8'))); j=list(csv.DictReader(open(root/'results/experiment_3a/joint_features.csv',encoding='utf-8'))); corr=list(csv.DictReader(open(root/'results/experiment_3a/correlations.csv',encoding='utf-8'))); assert c['status']==a['status']=='passed'; assert len(s)==len(j)==200; assert all(r['required_ef_90'] and r['required_ef_95'] for r in s); assert c['difficulty_targets']['90']['censored_count']==c['difficulty_targets']['95']['censored_count']==0; assert len(corr)>200; print('artifact_validation=passed queries=200 correlations=',len(corr),'censored=0')"; Write-Output '--- original artifacts preserved ---'; Get-ChildItem results\hnsw_smoke_latest.json,results\fdl_validation_latest.json,results\fdl_hnsw_correlation\result.json,scripts\hnsw_smoke.py,scripts\fdl_distribution_validation.py,scripts\fdl_hnsw_correlation.py | Select-Object Name,Length,LastWriteTime; Write-Output '--- experiment 3A files ---'; Get-ChildItem scripts\hnsw_search_feature_collection.py,scripts\hnsw_feature_analysis.py,docs\experiment_3a.md,results\experiment_3a\*,logs\experiment_3a\* -File | Select-Object FullName,Length,LastWriteTime; Write-Output '--- status ---'; git status --short`
>
> </details>

</details>

实验 3A 已完成并实际运行，状态 `passed`。完整报告见 [experiment_3a.md](F:/ocbigdata/docs/experiment_3a.md)。前两轮脚本和结果均未覆盖。

## 主要结论

### 1. FAISS 1.16.0 能可靠采集什么？

通过单线程、逐查询重置全局统计，可以可靠获得：

- `ndis`：距离计算次数
- `nhops`：图遍历/候选展开步数
- `n1`：搜索查询数，本实验始终验证为 1
- `n2`：搜索结束时候选列表耗尽的查询数

字段语义已依据 [FAISS 1.16.0 HNSW.h](https://github.com/facebookresearch/faiss/blob/v1.16.0/faiss/impl/HNSW.h) 和 [HNSW.cpp](https://github.com/facebookresearch/faiss/blob/v1.16.0/faiss/impl/HNSW.cpp) 核对；官方也说明这些全局统计仅在非并发搜索时保证准确。[FAISS implementation notes](https://github.com/facebookresearch/faiss/wiki/Implementation-notes#statistics-for-non-exhaustive-search)

当前 Python API 不能直接获得：

- 唯一访问节点数或节点 ID
- 候选队列当前值、峰值和演化
- 最优距离 progression
- 完整终止原因
- 可恢复的搜索状态

### 2. ndis、nhops 与搜索难度

它们的关联很弱：

| 早期特征 | 0.95 难度 Spearman ρ | BH q  |
| -------- | --------------------:| -----:|
| ndis@16  | -0.0798              | 0.434 |
| nhops@16 | -0.0627              | 0.551 |
| ndis@32  | -0.1272              | 0.210 |
| nhops@32 | -0.1364              | 0.170 |

`ndis` 与 `nhops` 自身高度冗余：

- ef=16：`ρ=0.883`
- ef=32：`ρ=0.749`

因此 `nhops` 没有表现出 ndis 之外的独立信息。

`n2` 在 ef≥16 时对全部 200 条查询均为 1，虽然采集可靠，但完全失去区分力。ef=8 时仍有变化，并与 0.95 难度呈弱相关 `ρ=-0.188`。

### 3. 哪些特征能在早期获得？

一次低预算搜索完成后，可以获得：

- ndis、nhops、n2
- 返回的最佳距离
- top-10 距离跨度和间隔

其中返回最佳距离表现最好：

- best distance@16 与所需 ef：`ρ=0.1885`
- best distance@32 与所需 ef：`ρ=0.1941`
- best distance@32 与同档 Recall：`ρ=-0.2551`，BH `q=0.00483`

但 Python 的 `search()` 是阻塞调用，这些特征只能在低 ef 探测完成后读取。再次提高 ef 会从头搜索，不能续跑。

### 4. 图搜索特征是否优于 FDL？

没有。

FDL P0.1 与 0.95 难度的相关为 `ρ=0.2266`，仍高于早期 ndis、nhops 和最佳返回距离。

最佳返回距离并非独立的图结构信息：它与 P0.1 的相关高达 `0.89`，本质上更像完整 FDL 低尾的廉价代理。

### 5. 联合后有没有增量？

对 0.95 难度，没有观察到明显增量：

| 偏相关                                   | ρpartial |
| ---------------------------------------- | --------:|
| P0.1，控制 ndis@16、nhops@16             | 0.2294   |
| ndis@16，控制 P0.1                       | -0.0894  |
| nhops@16，控制 P0.1、ndis@16             | 0.0024   |
| best distance@16，控制 P0.1、ndis、nhops | -0.0293  |

也就是说，加入早期计数器后，P0.1 的效应基本不变；反过来，计数器的独立效应接近零。

所有 290 项相关性检验统一使用了 Benjamini–Hochberg 校正。

### 6. 未来哪些特征值得用于自适应控制？

当前最适合继续验证的是：

- 低 ef 返回的最佳距离
- ef=8 的 candidate-exhausted 标志
- 低成本的 FDL 低尾估计

`ndis/nhops` 更适合作为成本遥测指标，而不是单独的难度预测器。

尚未验证、但更有潜力的是动态信号：

- 单位 ndis 带来的最佳距离改善率
- 候选阈值收缩速度
- 队列停滞长度
- 唯一访问节点增长率
- 局部度数和重复访问比例

### 7. 是否需要修改 FAISS 底层？

不建议仅为了增加更多最终计数器立即重编译。

如果下一阶段要实现真正可续跑的自适应搜索，则有必要修改底层，一次性加入：

- 固定步数搜索快照
- 唯一访问节点计数
- 候选队列大小和峰值
- best-distance progression
- 明确终止原因
- 候选队列与 visited table 的保存/恢复

这需要修改 HNSW C++ 实现、扩展 SWIG 绑定并构建项目内自定义 wheel，技术可行，但工程成本中等偏高。

## 难度标签

扫描到 ef=2048 后，两个目标均无右删失：

- Recall≥0.9：最晚在 ef=256 达标
- Recall≥0.95：198 条在 ef≤512 达标，剩余 2 条在 ef=1024 达标
- 右删失比例：两个目标均为 `0%`

## 运行信息

```powershell
uv run python scripts/hnsw_search_feature_collection.py
uv run python scripts/hnsw_feature_analysis.py
```

- 采集耗时：`1.655` 秒
- HNSW 建图：`0.221` 秒
- 联合分析：`1.600` 秒
- 查询数：200
- ef 档位：8–2048
- 线程数：1

## 交付文件

代码：

- [hnsw_search_feature_collection.py](F:/ocbigdata/scripts/hnsw_search_feature_collection.py)
- [hnsw_feature_analysis.py](F:/ocbigdata/scripts/hnsw_feature_analysis.py)

数据与汇总：

- [search_features.csv](F:/ocbigdata/results/experiment_3a/search_features.csv)
- [joint_features.csv](F:/ocbigdata/results/experiment_3a/joint_features.csv)
- [correlations.csv](F:/ocbigdata/results/experiment_3a/correlations.csv)
- [partial_correlations.csv](F:/ocbigdata/results/experiment_3a/partial_correlations.csv)
- [collection_summary.json](F:/ocbigdata/results/experiment_3a/collection_summary.json)
- [analysis_summary.json](F:/ocbigdata/results/experiment_3a/analysis_summary.json)

图表：

- [联合相关性热图](F:/ocbigdata/results/experiment_3a/correlation_heatmap.png)
- [不同 ef 的 ndis/nhops 分布](F:/ocbigdata/results/experiment_3a/online_stats_by_ef.png)
- [早期特征与难度](F:/ocbigdata/results/experiment_3a/early_features_vs_difficulty.png)
- [FDL 与在线统计](F:/ocbigdata/results/experiment_3a/fdl_vs_online.png)
- [不同 ef 下相关性变化](F:/ocbigdata/results/experiment_3a/correlation_by_ef.png)

![实验 3A 联合相关性热图](F:/ocbigdata/results/experiment_3a/correlation_heatmap.png)

下一阶段建议优先跨各向异性数据和多个 HNSW 图实例重复 3A；若弱相关结论仍稳定，再进入带搜索快照和可续跑状态的 C++ instrumentation。

> # 最终验证实验：L2-FDL 与 HNSW 搜索难度关联的稳健性
>
> 请继续当前项目，开展最后一轮验证实验。
>
> 本次任务的目标是检验前几轮实验得到的结论是否具有一定稳健性，排除特定数据分布、HNSW 参数和构图随机性造成的偶然结果。
>
> 本次是整个预研项目的收尾实验。完成后冻结实验范围，不再开发新算法或扩展研究方向。
>
> ## 一、开始前检查
>
> 先检查已有实验 2A、2B、3A 的数据、代码和结果。
>
> 重点确认：
>
> 1. 哪些数据集已经生成并保存，可以直接复用。
> 2. 哪些数据集需要按照原有生成脚本重新生成。
> 3. 哪些实验已经完成，哪些验证尚未完成。
> 4. 原有结果是否存在右删失、数据泄漏或统计分析方面的问题。
>
> 不要重复运行已经完成且结果可靠的实验。
>
> 如果发现原实验存在问题，优先补充必要的分析，不要无故重做全部实验。
>
> ## 二、验证 A：不同数据分布下的 FDL 预测能力
>
> 使用实验 2A 已有的各向异性和相关高斯数据。
>
> 至少选择：
>
> - 对角各向异性高斯。
> - AR(1) 相关高斯。
> - 旋转病态协方差高斯。
>
> 对每种数据分别：
>
> 1. 建立 HNSW 索引。
> 2. 使用对应数据集的查询计算 FDL 特征。
> 3. 扫描 efSearch。
> 4. 记录每条查询的 Recall@10。
> 5. 计算达到 Recall@10 ≥ 0.9 和 ≥ 0.95 所需的最小 efSearch。
> 6. 分析 FDL 特征与实际搜索预算之间的相关性。
>
> 重点关注 P0.1、P1、P5、均值、方差和偏度。
>
> 与标准高斯实验结果进行比较，判断之前观察到的弱相关性是否具有跨分布一致性。
>
> 如果不同分布下结果差异明显，请分析可能的原因，不要强行得出统一结论。
>
> ## 三、验证 B：HNSW 图结构敏感性
>
> 研究同一批数据库向量和查询，在不同 HNSW 图配置下是否会产生不同的搜索难度。
>
> 采用有限规模的参数对照：
>
> | 参数           | 测试值                  |
> | -------------- | ----------------------- |
> | M              | 8、16、32               |
> | efConstruction | 40、80、160             |
> | 构图随机种子   | 尽可能使用 3 个独立种子 |
>
> 优先使用已有基准配置，合理控制实验组合数量。
>
> 如果 FAISS 当前版本不能方便地控制构图随机种子，应先调查实际 API 能力，并如实记录，不要假定某个参数一定有效。
>
> 每种配置使用相同的数据库向量和查询向量。
>
> 比较：
>
> 1. 不同图配置下的 Recall。
> 2. 不同图配置下的 ndis 和 nhops。
> 3. 每条查询达到目标 Recall 所需的 efSearch。
> 4. FDL 特征与所需 efSearch 的 Spearman 相关系数。
> 5. 同一查询在不同图实例中的搜索难度变化。
>
> 重点回答：
>
> - 改变 M 是否会改变 FDL 与搜索难度之间的关系？
> - 增加 efConstruction 是否能够改善这种关系？
> - 构图随机性是否会显著影响查询难度？
> - 同一查询在不同图实例中是否可能表现出明显不同的搜索成本？
>
> ## 四、统计分析要求
>
> 统一使用两个目标：
>
> - Recall@10 ≥ 0.9。
> - Recall@10 ≥ 0.95。
>
> 搜索预算应覆盖足够大的范围，直到达到目标或达到合理上限。
>
> 未达到目标的查询必须标记为右删失，不得随意赋予一个虚构的 efSearch 数值。
>
> 统计分析至少包括：
>
> 1. Spearman 相关系数。
> 2. Bootstrap 置信区间。
> 3. Benjamini-Hochberg 多重检验校正。
> 4. 各实验配置下的查询难度分布。
> 5. 不同配置之间的相关性差异。
>
> 注意，同一查询在多个图配置下重复出现，不能简单当成完全独立的样本。Bootstrap 应尽量以查询为抽样单位，并保留不同图配置之间的配对关系。
>
> 对于只有 200 条查询的实验，明确说明统计功效和结论适用范围。
>
> ## 五、最终综合判断
>
> 将实验 1、2A、2B、3A 和本轮验证的结果统一整理。
>
> 重点回答：
>
> 1. L2-FDL 的数学模型在不同高斯分布下是否成立？
> 2. 广义非中心卡方模型相较正态近似是否具有稳定优势？
> 3. FDL 特征与 HNSW 搜索难度的关系是否跨数据分布成立？
> 4. 不同 HNSW 图配置是否会显著改变这种关系？
> 5. 之前发现的弱相关性是否可能只是特定图实例造成的？
> 6. 仅依靠 FDL 特征进行 efSearch 自适应是否具有充分的实验依据？
>
> 最终结论分为：
>
> - 已经得到实验支持的结论。
> - 当前实验不支持的假设。
> - 因样本规模或实验范围而尚不能判断的问题。
>
> 不要把统计不显著直接解释为完全不存在关系，也不要把相关性解释为因果关系。
>
> ## 六、最终报告与项目整理
>
> 完成后生成最终研究报告：
>
> `F:\ocbigdata\experiment_final.md`
>
> 报告应包括：
>
> 1. 研究问题与实验范围。
> 2. 实验设计及参数。
> 3. 各轮实验的主要结果。
> 4. 不同数据分布的对照。
> 5. 不同 HNSW 图配置的对照。
> 6. 统计分析与稳健性检验。
> 7. 研究假设验证结果。
> 8. 研究局限性。
> 9. 最终结论。
>
> 同时整理必要的图表和汇总 CSV、JSON。
>
> 不要删除或覆盖原始实验数据和结果。
>
> 最后提供：
>
> - 新增及修改的文件清单。
> - 实际运行的实验命令。
> - 各实验的运行耗时。
> - 核心统计结果。
> - 最终研究结论。
>
> ## 七、停止条件
>
> 本轮实验完成后，正式结束预研项目。
>
> 不再开展以下工作：
>
> - 不开发新的 L2-Ada-ef 算法。
> - 不修改 FAISS C++ 底层。
> - 不增加动态搜索反馈机制。
> - 不继续寻找新的搜索特征。
> - 不为了得到正面结果而无限增加实验。
>
> 如果最终证明 FDL 可以准确描述距离分布，但不足以预测 HNSW 搜索难度，这完全可以作为一项有价值的否定性研究结论。
>
> **本次工作的首要目标是验证结论的可靠性，而不是追求算法性能提升。**

<details><summary>15 previous messages</summary>

> 我先做收尾审计，不重跑已完成实验：确认各轮数据是否落盘、右删失处理、FDL 复用边界，以及 FAISS 是否能真实控制 HNSW 构图 RNG。随后采用有限而成体系的验证矩阵：三种各向异性分布；标准高斯下分离比较 M、efConstruction 与可控构图种子；所有 Bootstrap 以查询为抽样单位并保持配置配对。新结果会放入独立的 `results/final_validation/`，最终报告写到项目根目录 `experiment_final.md`。
>
> <details><summary>Ran 2 commands</summary>
>
> - Ran `$ErrorActionPreference='Stop'; Write-Output '--- project files ---'; rg --files -g '!**/.venv/**' -g '!**/.uv-cache/**' -g '!**/.git/**' | Sort-Object; Write-Output '--- data/result artifacts ---'; Get-ChildItem results -Recurse -File | Select-Object FullName,Length,LastWriteTime; Write-Output '--- anisotropic generator ---'; Get-Content -Raw scripts\anisotropic_data.py; Write-Output '--- 2A result status/config ---'; $a=Get-Content -Raw results\anisotropic_fdl\result.json | ConvertFrom-Json; $a.status; $a.config; Write-Output '--- 2B censor/config ---'; $b=Get-Content -Raw results\fdl_hnsw_correlation\result.json | ConvertFrom-Json; $b.status; $b.censored_query_count; $b.config; Write-Output '--- 3A censor/config ---'; $c=Get-Content -Raw results\experiment_3a\collection_summary.json | ConvertFrom-Json; $c.status; $c.difficulty_targets | ConvertTo-Json -Depth 8; Write-Output '--- RNG binding ---'; uv run python -c "import faiss; x=faiss.IndexHNSWFlat(8,4); print(type(x.hnsw.rng)); print('rng property in HNSW=', 'rng' in dir(x.hnsw)); import inspect; print([line for line in open(faiss.__path__[0]+'/swigfaiss.py',encoding='utf-8') if line.strip().startswith('rng = property')][:3])"`
> - Ran `$ErrorActionPreference='Stop'; uv run python -c "import faiss,numpy as np; rng=np.random.default_rng(7); x=rng.standard_normal((300,16),dtype=np.float32); holders=[]; sigs=[]; 
>   for seed in [12345,54321,99999,12345]:
>    idx=faiss.IndexHNSWFlat(16,8); rg=faiss.RandomGenerator(seed); holders.append(rg); idx.hnsw.rng=rg; idx.hnsw.efConstruction=40; idx.add(x); levels=faiss.vector_to_array(idx.hnsw.levels); neigh=faiss.vector_to_array(idx.hnsw.neighbors); sigs.append((seed,hash(levels.tobytes()),hash(neigh.tobytes()),levels[:12].tolist()))
>   print(*sigs,sep='\n'); print('same seed identical',sigs[0][1:3]==sigs[3][1:3]); print('different seed differs',len(set(s[1:3] for s in sigs[:3]))==3)"`
>
> </details>
>
> 审计结果如下：
>
> - 标准高斯和三种各向异性数据都未保存为 `.npy`；但生成脚本、种子、顺序和 `float32` 变换完整，可逐元素重建。标准高斯 FDL 已保存，可直接复用；各向异性结果缺少偏度，因此需要按原脚本重建数据并计算本轮特征，但不会重做 2A 的分布理论验证。
> - 2B 的 0.95 标签曾有 2 条在 ef=512 右删失；3A 扫到 ef=1024 后已全部观测，因此本轮不会沿用虚构的 1024 编码。
> - 3A 的解释性分析没有训练/测试泄漏问题，但完整 FDL 是离线全距离特征，不能当作在线免费特征。
> - FAISS Python 绑定确实允许给 `index.hnsw.rng` 赋新的 `RandomGenerator(seed)`。小规模验证显示同种子图结构逐元素一致、不同种子 levels/neighbors 不同，因此本轮可以真实测试三个构图种子，而不是假定。
>
> 图配置采用 7 个单因素对照：基线，加 M=8/32，efConstruction=40/160，以及两个额外种子；这样覆盖所有指定值并避免无必要的 27 组全因子扩张。
>
> <details><summary>Created a file, ran a command</summary>
>
> - Wrote `F:/ocbigdata/scripts/final_robustness_validation.py` (+636 -0)
> - Ran `$ErrorActionPreference='Stop'; uv run python -m compileall -q scripts; uv run python scripts\final_robustness_validation.py --help | Select-Object -First 16`
>
> </details>
>
> 最终验证脚本已经完成语法检查。它将运行三种分布基线和七个单因素图配置，扫描到 ef=2048；对每个相关系数做 2,000 次查询级 Bootstrap，并对配置差异使用保持查询配对的 Bootstrap。现在开始实际运行，预计仍是当前 200 查询的小规模分钟内实验。
>
> <details><summary>Ran a command</summary>
>
> - Ran `uv run python scripts\final_robustness_validation.py`
>
> </details>
>
> 完整运行已通过，总耗时约 17.9 秒，所有 10 个数据/图条件均完成。下一步只做结果审计：检查右删失、跨分布相关方向和置信区间、不同 M/efConstruction/种子的配对差异，以及同一查询跨种子的难度波动；然后据此写根目录最终报告并冻结范围。
>
> <details><summary>Ran a command</summary>
>
> - Ran ``$ErrorActionPreference='Stop'; Write-Output '--- result ---'; Get-Content -Raw results\final_validation\result.json; Write-Output '--- distribution correlations ---'; Import-Csv results\final_validation\distribution_correlations.csv | Sort-Object condition,target,feature | Format-Table condition,feature,target,spearman_rho,bootstrap_ci_low,bootstrap_ci_high,pvalue,bh_qvalue,censored_count -AutoSize | Out-String -Width 260; Write-Output '--- graph correlations ---'; Import-Csv results\final_validation\graph_correlations.csv | Sort-Object condition,target,feature | Format-Table condition,feature,target,spearman_rho,bootstrap_ci_low,bootstrap_ci_high,bh_qvalue,censored_count -AutoSize | Out-String -Width 250; Write-Output '--- graph summary ---'; Import-Csv results\final_validation\graph_summary.csv | Select-Object config,m,ef_construction,graph_seed,index_build_seconds,mean_recall_ef_16,mean_recall_ef_32,mean_recall_ef_64,mean_recall_ef_128,mean_ndis_ef_64,mean_nhops_ef_64,required_ef_90_median,required_ef_95_median,required_ef_95_censored | Format-Table -AutoSize | Out-String -Width 260; Write-Output '--- distribution summary ---'; Import-Csv results\final_validation\distribution_summary.csv | Select-Object dataset,condition_number,index_build_seconds,mean_recall_ef_16,mean_recall_ef_32,mean_recall_ef_64,mean_recall_ef_128,required_ef_90_median,required_ef_95_median,required_ef_95_censored | Format-Table -AutoSize | Out-String -Width 220; Write-Output '--- files ---'; Get-ChildItem results\final_validation,logs\final_validation -Recurse -File | ForEach-Object { '{0}`t{1}' -f $_.FullName,$_.Length }``
>
> </details>
>
> 结果审计发现一个需要保留而非掩盖的边界：M=8 时仍有 5/200 条查询在 ef=2048 未达到 0.95，已按右删失处理；其他分布和图配置无删失。相关分析只使用观测到最小 ef 的查询。为保证这 195 条样本的 Bootstrap 仍是严格均匀的查询级重采样，我会修正删失子集的重采样索引后重跑本轮脚本；不改实验设计，也不扩大 ef 上限来强行消除删失。
>
> <details><summary>Edited a file, ran 5 commands</summary>
>
> - Wrote `F:/ocbigdata/scripts/final_robustness_validation.py` (+10 -4)
> - Ran `uv run python scripts\final_robustness_validation.py`
> - Ran `$ErrorActionPreference='Stop'; $d=Import-Csv results\final_validation\paired_correlation_differences.csv; Write-Output '--- paired differences P0.1 target95 ---'; $d | Where-Object { $_.feature -eq 'p0_1' -and $_.target -eq 'required_ef_95' } | Format-Table config,rho_config,rho_baseline,rho_difference,paired_bootstrap_ci_low,paired_bootstrap_ci_high,bootstrap_pvalue,bh_qvalue,sample_size -AutoSize | Out-String -Width 260; Write-Output '--- significant paired differences after BH ---'; $d | Where-Object { [double]$_.bh_qvalue -lt 0.05 } | Sort-Object {[double]$_.bh_qvalue} | Format-Table config,feature,target,rho_difference,paired_bootstrap_ci_low,paired_bootstrap_ci_high,bh_qvalue,sample_size -AutoSize | Out-String -Width 250; $p=Import-Csv results\final_validation\graph_difficulty_pairwise.csv; Write-Output '--- seed pairwise ---'; $p | Where-Object { $_.config_left -match 'c80_s(12345|54321|99999)' -and $_.config_right -match 'c80_s(12345|54321|99999)' } | Format-Table config_left,config_right,target,spearman_rho,bootstrap_ci_low,bootstrap_ci_high,median_absolute_log2_ef_difference,fraction_same_required_ef -AutoSize | Out-String -Width 260; Write-Output '--- factor pairwise vs baseline ---'; $p | Where-Object { $_.config_left -eq 'm16_c80_s12345' -or $_.config_right -eq 'm16_c80_s12345' } | Format-Table config_left,config_right,target,spearman_rho,bootstrap_ci_low,bootstrap_ci_high,median_absolute_log2_ef_difference,fraction_same_required_ef -AutoSize | Out-String -Width 260; Write-Output '--- seed variability ---'; $v=Import-Csv results\final_validation\seed_query_variability.csv; 'same95=' + (($v | Measure-Object -Property same_across_seeds_95 -Average).Average); 'range>=1=' + (($v | Where-Object {[double]$_.log2_range_95 -ge 1}).Count/200); 'range>=2=' + (($v | Where-Object {[double]$_.log2_range_95 -ge 2}).Count/200); 'range>=3=' + (($v | Where-Object {[double]$_.log2_range_95 -ge 3}).Count/200); Write-Output '--- current result timing ---'; $r=Get-Content -Raw results\final_validation\result.json | ConvertFrom-Json; $r.timings_seconds | ConvertTo-Json -Depth 8`
> - Ran ``$ErrorActionPreference='Stop'; Write-Output '--- graph difficulty counts ---'; $g=Import-Csv results\final_validation\graph_summary.csv; foreach($row in $g){ $c90=@(); $c95=@(); foreach($ef in 8,16,32,64,128,256,512,1024,2048){$n90=$row."required_ef_90_count_$ef";if([int]$n90 -gt 0){$c90+="$ef`:$n90"};$n95=$row."required_ef_95_count_$ef";if([int]$n95 -gt 0){$c95+="$ef`:$n95"}}; [pscustomobject]@{config=$row.config;median90=$row.required_ef_90_median;median95=$row.required_ef_95_median;censored95=$row.required_ef_95_censored;counts90=($c90 -join ',');counts95=($c95 -join ',')}} | Format-Table -AutoSize | Out-String -Width 300}; Write-Output '--- distribution difficulty counts ---'; $g=Import-Csv results\final_validation\distribution_summary.csv; foreach($row in $g){ $c90=@(); $c95=@(); foreach($ef in 8,16,32,64,128,256,512,1024,2048){$n90=$row."required_ef_90_count_$ef";if([int]$n90 -gt 0){$c90+="$ef`:$n90"};$n95=$row."required_ef_95_count_$ef";if([int]$n95 -gt 0){$c95+="$ef`:$n95"}}; [pscustomobject]@{dataset=$row.dataset;median90=$row.required_ef_90_median;median95=$row.required_ef_95_median;censored95=$row.required_ef_95_censored;counts90=($c90 -join ',');counts95=($c95 -join ',')}} | Format-Table -AutoSize | Out-String -Width 300}``
> - Ran `uv run python -c "import csv,json
>   for fn,key in [('graph_summary.csv','config'),('distribution_summary.csv','dataset')]:
>    print('---',fn,'---')
>    for r in csv.DictReader(open('results/final_validation/'+fn,encoding='utf-8')):
>   c90={ef:int(r[f'required_ef_90_count_{ef}']) for ef in [8,16,32,64,128,256,512,1024,2048] if int(r[f'required_ef_90_count_{ef}'])}
>   c95={ef:int(r[f'required_ef_95_count_{ef}']) for ef in [8,16,32,64,128,256,512,1024,2048] if int(r[f'required_ef_95_count_{ef}'])}
>   print(r[key],'median',r['required_ef_90_median'],r['required_ef_95_median'],'censored95',r['required_ef_95_censored'],'counts90',c90,'counts95',c95)"`
> - Ran `uv run python -c "import csv
>   for r in csv.DictReader(open('results/final_validation/graph_summary.csv',encoding='utf-8')): print(r['config'],'R64',round(float(r['mean_recall_ef_64']),4),'ndis64',round(float(r['mean_ndis_ef_64']),1),'nhops64',round(float(r['mean_nhops_ef_64']),1),'R128',round(float(r['mean_recall_ef_128']),4))"`
>
> </details>
>
> Viewed 5 images
>
> 稳健性结果已经足够形成最终判断：
>
> - 跨分布不一致：P0.1–难度在对角各向异性上较明显（0.95 目标 `ρ=0.302`），但 AR(1) 为 `0.111`、旋转病态谱为 `0.086`，后两者置信区间跨 0。
> - 跨图配置仍普遍是弱正相关：标准高斯 P0.1 在七个图上的 `ρ` 约 `0.151–0.310`；相对基线的配对 Bootstrap 差异全部未通过 BH 校正。
> - 构图随机性显著影响单查询标签：三个种子下只有 24% 查询所需 ef 完全一致，30% 查询相差至少两个 ef 倍增档；但聚合层面的 FDL 相关差异并不显著。
> - M 和 efConstruction 显著改变 Recall/成本与难度分布，却没有把 FDL–难度关系稳定地增强。
>
> 这支持一个有价值的收尾结论：FDL 数学模型稳健，但作为 HNSW 难度预测器不具跨分布充分性。现在写最终报告并标记项目范围冻结。
>
> <details><summary>Created a file, edited a file, ran a command</summary>
>
> - Wrote `F:/ocbigdata/README.md` (+12 -0)
> - Wrote `F:/ocbigdata/experiment_final.md` (+234 -0)
> - Ran ``$ErrorActionPreference='Stop'; $r=Get-Content -Raw results\final_validation\result.json | ConvertFrom-Json; Write-Output ('outputs count=' + $r.outputs.Count + ' unique=' + ($r.outputs | Sort-Object -Unique).Count); $r.outputs; uv run python -c "import csv,json,pathlib; root=pathlib.Path('.'); r=json.load(open(root/'results/final_validation/result.json',encoding='utf-8')); dr=list(csv.DictReader(open(root/'results/final_validation/distribution_per_query.csv',encoding='utf-8'))); gr=list(csv.DictReader(open(root/'results/final_validation/graph_per_query.csv',encoding='utf-8'))); dc=list(csv.DictReader(open(root/'results/final_validation/distribution_correlations.csv',encoding='utf-8'))); gc=list(csv.DictReader(open(root/'results/final_validation/graph_correlations.csv',encoding='utf-8'))); assert r['status']=='passed'; assert len(dr)==600 and len(gr)==1400 and len(dc)==36 and len(gc)==84; assert len(set(x['graph_signature_sha256'] for x in csv.DictReader(open(root/'results/final_validation/graph_summary.csv',encoding='utf-8'))))==7; print('artifact_validation=passed distribution_rows=600 graph_rows=1400 correlations=120')"; Write-Output '--- original timestamps ---'; Get-ChildItem results\hnsw_smoke_latest.json,results\fdl_validation_latest.json,results\anisotropic_fdl\result.json,results\fdl_hnsw_correlation\result.json,results\experiment_3a\analysis_summary.json | Select-Object FullName,LastWriteTime; Write-Output '--- final files ---'; Get-ChildItem scripts\final_robustness_validation.py,experiment_final.md,results\final_validation\*,logs\final_validation\run.log -File | ForEach-Object { '{0}`t{1}' -f $_.FullName,$_.Length }; Write-Output '--- status ---'; git status --short``
>
> </details>
>
> 最终产物校验已通过：600 条跨分布查询、1400 条跨图查询、120 个带 Bootstrap 的相关检验，七张图签名均不同。发现的唯一整理问题是 JSON 产物清单中 `result.json` 因二次运行被重复列出；我会只修正清单生成和当前 JSON，不重跑实验、不改变任何数值结果。
>
> <details><summary>Edited 2 files, ran a command</summary>
>
> - Wrote `F:/ocbigdata/results/final_validation/result.json` (+0 -1)
> - Wrote `F:/ocbigdata/scripts/final_robustness_validation.py` (+1 -1)
> - Ran `$ErrorActionPreference='Stop'; uv run python -m compileall -q scripts\final_robustness_validation.py; uv run python -c "import json; r=json.load(open('results/final_validation/result.json',encoding='utf-8')); assert len(r['outputs'])==len(set(r['outputs']))==15; assert r['status']=='passed'; print('final_validation=passed outputs=',len(r['outputs']),'elapsed=',r['timings_seconds']['total'])"; Get-Item experiment_final.md,scripts\final_robustness_validation.py,results\final_validation\result.json | Select-Object FullName,Length,LastWriteTime`
>
> </details>

</details>

最终稳健性验证已完成，状态 `passed`，预研项目正式冻结。

完整报告：[experiment_final.md](F:/ocbigdata/experiment_final.md)

## 最终结论

**L2-FDL 能准确、稳健地描述高斯距离分布，但现有证据不足以支持仅依靠 FDL 预测 HNSW 所需 efSearch。**

### 跨数据分布

P0.1 与 Recall≥0.95 所需预算的相关：

| 分布         | Spearman ρ | 95% Bootstrap CI | BH q    |
| ------------ | ----------:| ----------------:| -------:|
| 对角各向异性 | 0.302      | [0.159, 0.430]   | 0.00050 |
| AR(1) 相关   | 0.111      | [-0.015, 0.252]  | 0.407   |
| 旋转病态谱   | 0.086      | [-0.056, 0.224]  | 0.583   |

弱正相关只在标准高斯和对角各向异性中较明确，没有跨一般高斯分布稳定复现。

### HNSW 参数影响

| 配置               | Recall@64 | ndis@64 | 0.95 中位所需 ef |
| ------------------ | ---------:| -------:| ----------------:|
| M=8                | 0.8000    | 694.1   | 256              |
| M=16               | 0.9395    | 1228.8  | 64               |
| M=32               | 0.9815    | 1726.8  | 64               |
| efConstruction=40  | 0.9075    | 1122.1  | 128              |
| efConstruction=160 | 0.9530    | 1288.0  | 64               |

M 和 efConstruction 明显改变 Recall、计算成本及难度分布，但没有稳定增强 FDL–难度相关性。

七个图配置中，P0.1 的相关约为 `0.151–0.310`，均属于弱相关。相对基线的配对 Bootstrap 差异全部未通过 BH 校正。

### 构图随机性

三个构图种子生成的图签名均不同。同一查询的 0.95 难度：

- 仅 24% 在三个图中需要相同 ef
- 76% 至少相差一个倍增档
- 30% 至少相差两个倍增档
- 9% 至少相差三个倍增档
- 种子间难度排序相关仅 `0.369–0.489`

因此，弱聚合相关不是单张图的偶然产物，但单查询预算明显依赖具体图实例。

### 右删失

- 三种分布：无删失
- 六个图配置：无删失
- M=8：5/200 条查询在 ef=2048 后仍未达到 0.95

这 5 条查询保持右删失，相关分析仅使用其余 195 条，没有赋予虚构预算。

## 研究假设判断

已获得支持：

- 标准及一般高斯的 L2-FDL 数学模型成立。
- 广义非中心卡方模型稳定优于正态近似。
- HNSW 难度显著依赖 M、efConstruction 和构图实例。
- 部分分布中 FDL 低尾与搜索难度存在弱关联。

当前不支持：

- FDL–难度关系能够跨数据分布稳定成立。
- 调整 HNSW 参数能稳定强化该关系。
- ndis、nhops 能提供明显独立信息。
- 仅凭 FDL 足以进行可靠的 efSearch 自适应。
- 当前证据足以支持开发纯 FDL 驱动的 L2-Ada-ef。

尚不能判断：

- 真实嵌入、百万级数据和更高维场景。
- 动态候选队列或距离改善率是否有效。
- 更大查询样本下是否存在小幅可利用增量。

## 统计方法

- 2000 次查询级 Bootstrap
- 图配置比较保持同一查询配对
- Benjamini–Hochberg 多重比较校正
- 两个难度目标：Recall≥0.9、Recall≥0.95
- 未把重复图配置下的查询当作独立样本
- 完整 FDL 明确视为离线解释特征，而非在线免费特征

## 实际运行

```powershell
uv run python scripts/final_robustness_validation.py
```

最终运行耗时：

- 三种分布：每种 `1.593–1.763` 秒
- 七个图配置：每个 `1.443–1.707` 秒
- 全部实验、Bootstrap、汇总与绘图：`17.931` 秒

验证规模：

- 跨分布逐查询记录：600
- 跨图逐查询记录：1400
- 带 Bootstrap 的相关检验：120
- 七个图结构签名全部不同

## 新增及修改文件

新增：

- [final_robustness_validation.py](F:/ocbigdata/scripts/final_robustness_validation.py)
- [experiment_final.md](F:/ocbigdata/experiment_final.md)
- [最终结果 JSON](F:/ocbigdata/results/final_validation/result.json)
- [跨分布相关 CSV](F:/ocbigdata/results/final_validation/distribution_correlations.csv)
- [跨图相关 CSV](F:/ocbigdata/results/final_validation/graph_correlations.csv)
- [配对相关差异](F:/ocbigdata/results/final_validation/paired_correlation_differences.csv)
- [图难度配对稳定性](F:/ocbigdata/results/final_validation/graph_difficulty_pairwise.csv)
- [种子逐查询波动](F:/ocbigdata/results/final_validation/seed_query_variability.csv)
- [运行日志](F:/ocbigdata/logs/final_validation/run.log)

修改：

- [README.md](F:/ocbigdata/README.md)，加入最终实验入口与冻结声明。

主要图表：

- [跨分布相关性](F:/ocbigdata/results/final_validation/distribution_correlations.png)
- [跨图相关性](F:/ocbigdata/results/final_validation/graph_correlations.png)
- [Recall 与成本曲线](F:/ocbigdata/results/final_validation/graph_recall_cost_curves.png)
- [难度分布](F:/ocbigdata/results/final_validation/difficulty_distributions.png)
- [构图种子稳定性](F:/ocbigdata/results/final_validation/seed_stability.png)

![跨分布稳健性](F:/ocbigdata/results/final_validation/distribution_correlations.png)

本轮完成后，项目按要求停止：不开发新算法、不修改 FAISS 底层、不扩展动态特征，也不继续增加实验以追求正面结果。
