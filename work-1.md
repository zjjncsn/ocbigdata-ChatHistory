# 初始化 L2-FDL 预研项目

> Continuing from [编写实验方案](chatgpt-conversation://6af97b5b-167c-83ec-a854-ebf258fd62f2): 在用户的 Windows 电脑上直接初始化并验证 L2-FDL 预研实验项目，项目根目录为 F:\ocbigdata。先检查目录是否存在及已有内容，避免覆盖；检查 Windows、Python、uv 和可用编译环境。优先使用 Windows 原生 Python 3.12 + uv + FAISS CPU，不安装 WSL。创建清晰的项目结构、pyproject.toml/uv.lock、README 和最小可运行的 HNSW 冒烟实验：生成小规模合成 float32 向量，构建 FAISS IndexHNSWFlat（Squared L2），计算精确 Ground Truth 与 Recall@10，测试多个 efSearch，并验证 SearchParametersHNSW 能为单次查询独立设置 efSearch。保存基准结果和日志。若 Windows 原生 FAISS 安装失败，先调查兼容性与可行替代方案，不要擅自安装 WSL 或更改系统级配置；必要时向用户说明并询问。所有操作限定在 F:\ocbigdata 和项目虚拟环境内，不删除用户现有文件，不下载完整 SIFT1M，不执行长时间正式实验。

<details><summary>8 previous messages</summary>

> 我会先做只读检查，确认 `F:\ocbigdata` 的现有内容和本机工具链，再在不覆盖已有文件的前提下初始化项目。随后会运行一个短时 HNSW 冒烟实验，并把环境、结果和日志都留在项目目录内。
>
> <details><summary>Ran a command</summary>
>
> - Ran `$ErrorActionPreference='Continue'; Write-Output '--- ROOT ---'; if (Test-Path -LiteralPath 'F:\ocbigdata') { Get-Item -LiteralPath 'F:\ocbigdata' | Select-Object FullName,Attributes,CreationTime,LastWriteTime | Format-List; Write-Output '--- CONTENTS ---'; Get-ChildItem -LiteralPath 'F:\ocbigdata' -Force | Select-Object Mode,Length,LastWriteTime,Name | Format-Table -AutoSize } else { Write-Output 'MISSING' }; Write-Output '--- INSTRUCTIONS ---'; Get-ChildItem -LiteralPath 'F:\ocbigdata' -Force -File -Filter 'AGENTS.md' -ErrorAction SilentlyContinue | ForEach-Object { Write-Output $_.FullName; Get-Content -LiteralPath $_.FullName -Raw }; Write-Output '--- OS ---'; Get-CimInstance Win32_OperatingSystem | Select-Object Caption,Version,OSArchitecture,BuildNumber | Format-List; Write-Output '--- TOOLS ---'; foreach ($cmd in @('python','py','uv','cl','gcc','clang','cmake','ninja','git')) { $found=Get-Command $cmd -ErrorAction SilentlyContinue; if ($found) { Write-Output ("{0}: {1}" -f $cmd,$found.Source) } else { Write-Output ("{0}: NOT FOUND" -f $cmd) } }; Write-Output '--- VERSIONS ---'; python --version 2>&1; py -0p 2>&1; uv --version 2>&1; git --version 2>&1; cmake --version 2>&1; ninja --version 2>&1`
>
> </details>
>
> 初检结果很好：目标目录存在且当前为空；已安装原生 Python 3.12.4、uv 0.11.29 和 Git。系统路径里没有 C/C++ 编译器、CMake 或 Ninja，不过 FAISS CPU 若有匹配的预编译包就不需要本地编译；我会先按这个路径验证。Windows 系统信息接口被权限限制，但不影响实验，后续会用更轻量的方式记录系统版本。
>
> <details><summary>Created 6 files, ran a command</summary>
>
> - Wrote `F:/ocbigdata/.gitignore` (+10 -0)
> - Wrote `F:/ocbigdata/README.md` (+55 -0)
> - Wrote `F:/ocbigdata/logs/.gitkeep` (+1 -0)
> - Wrote `F:/ocbigdata/pyproject.toml` (+14 -0)
> - Wrote `F:/ocbigdata/results/.gitkeep` (+1 -0)
> - Wrote `F:/ocbigdata/scripts/hnsw_smoke.py` (+218 -0)
> - Ran `$ErrorActionPreference='Stop'; uv lock; uv sync --locked`
>
> </details>
>
> uv 首次运行遇到的是缓存目录权限限制，不是 FAISS 兼容性问题。为保持所有写入都在项目内，我会把 uv 缓存也固定到 `F:\ocbigdata\.uv-cache`，然后继续解析和安装依赖；不会改动用户级或系统级设置。
>
> <details><summary>Ran 3 commands</summary>
>
> - Ran `$ErrorActionPreference='Stop'; $env:UV_CACHE_DIR='F:\ocbigdata\.uv-cache'; uv lock; uv sync --locked`
> - Ran `$ErrorActionPreference='Stop'; $env:UV_CACHE_DIR='F:\ocbigdata\.uv-cache'; uv lock; uv sync --locked`
> - Ran `$ErrorActionPreference='Stop'; $env:UV_CACHE_DIR='F:\ocbigdata\.uv-cache'; uv run --locked python scripts/hnsw_smoke.py`
>
> </details>
>
> 原生 FAISS 已成功安装并完成首轮实验：Python 3.12 对应的预编译 CPU 包可用，无需编译器或 WSL。Recall@10 随 `efSearch` 从 8 到 128 由 0.4985 提升到 0.9835；单次查询参数 7/91 也分别生效，同时索引全局值始终保持为哨兵值 123。接下来我会做最后的文件与可复现性检查，并补齐项目内缓存配置。
>
> <details><summary>Created a file, edited 3 files, ran 2 commands</summary>
>
> - Wrote `F:/ocbigdata/.gitignore` (+2 -0)
> - Wrote `F:/ocbigdata/README.md` (+1 -0)
> - Wrote `F:/ocbigdata/uv.toml` (+3 -0)
> - Ran `$ErrorActionPreference='Stop'; Write-Output '--- UV CACHE ---'; uv cache dir; Write-Output '--- LOCK CHECK ---'; uv lock --check; Write-Output '--- COMPILE CHECK ---'; uv run --locked python -m compileall -q scripts; Write-Output '--- RESULT CHECK ---'; uv run --locked python -c "import csv,json,pathlib; root=pathlib.Path(r'F:\ocbigdata'); data=json.loads((root/'results/hnsw_smoke_latest.json').read_text(encoding='utf-8')); rows=list(csv.DictReader((root/'results/hnsw_smoke_latest.csv').open(encoding='utf-8'))); assert data['status']=='passed'; assert data['search_parameters_hnsw_isolated'] is True; assert len(rows)==7; print('status=',data['status'],'rows=',len(rows),'faiss=',data['environment']['faiss'])"; Write-Output '--- FILES ---'; Get-ChildItem -LiteralPath 'F:\ocbigdata' -Recurse -Force -File | Where-Object { $_.FullName -notlike 'F:\ocbigdata\.venv\*' -and $_.FullName -notlike 'F:\ocbigdata\.uv-cache\*' } | Select-Object FullName,Length | Format-Table -AutoSize`
> - Wrote `F:/ocbigdata/scripts/hnsw_smoke.py` (+8 -0)
> - Ran `$ErrorActionPreference='Stop'; uv run --locked python scripts/hnsw_smoke.py; if (-not (Test-Path -LiteralPath 'F:\ocbigdata\.git')) { git init -b main }; git status --short --ignored`
>
> </details>

</details>

已在 `F:\ocbigdata` 完成 Windows 原生初始化和验证，全程未安装或使用 WSL，也未修改系统级配置。

验证结果：

- Windows 11，Python 3.12.4，uv 0.11.29
- 安装并锁定 `faiss-cpu 1.16.0`、`numpy 2.5.4`
- 使用 FAISS 预编译 CPU 包，无需本地 C/C++ 编译环境
- 精确 Ground Truth 已与 `IndexFlatL2` 交叉验证
- HNSW Recall@10：
  - `efSearch=8`：0.4985
  - `efSearch=16`：0.6645
  - `efSearch=32`：0.8210
  - `efSearch=64`：0.9395
  - `efSearch=128`：0.9835
- `SearchParametersHNSW` 单次查询参数验证通过：分别使用 7 和 91 时生效，索引全局 `efSearch` 始终保持为 123
- 已初始化空的 Git 仓库，未创建提交
- uv 缓存、虚拟环境及所有输出均限制在 `F:\ocbigdata` 内

主要文件：

- [README.md](F:/ocbigdata/README.md)
- [pyproject.toml](F:/ocbigdata/pyproject.toml)
- [uv.lock](F:/ocbigdata/uv.lock)
- [hnsw_smoke.py](F:/ocbigdata/scripts/hnsw_smoke.py)
- [完整实验结果](F:/ocbigdata/results/hnsw_smoke_latest.json)
- [CSV 基准结果](F:/ocbigdata/results/hnsw_smoke_latest.csv)
- [运行日志](F:/ocbigdata/logs/hnsw_smoke_latest.log)

复现命令：

```powershell
cd F:\ocbigdata
uv sync --locked
uv run python scripts/hnsw_smoke.py
```

当前数据规模仅为 5,000 个数据库向量和 200 个查询，适合功能冒烟验证，结果不应视为正式性能基准。

> 之前的 5000 条向量是使用标准正态分布 `np.random.randn()` 生成的吗？

基本等价，但没有直接调用 `np.random.randn()`。

代码使用的是：

```python
rng = np.random.default_rng(20261115)
database = rng.standard_normal((5000, 64), dtype=np.float32)
```

因此这 5,000 条、每条 64 维的向量均来自标准正态分布 \(N(0,1)\)，并直接生成为 `float32`。

它与 `np.random.randn(5000, 64)` 的主要区别是：

- 使用较新的 `Generator` 随机数接口；
- 固定种子为 `20261115`，便于复现；
- 直接生成 `float32`；`randn()` 默认生成 `float64`；
- 随机数算法不同，所以即使设置相同数值的种子，具体向量也不会与旧接口完全相同。

对应代码在 [hnsw_smoke.py](F:/ocbigdata/scripts/hnsw_smoke.py) 中。

> 很好，这样我们就可以直接复用之前的合成数据生成方案，不需要重新设计随机数生成逻辑。
>
> 而且，这个数据集有一个很好的特点：它恰好对应 L2-FDL 最简单、最容易得到精确理论解的情况。
>
> ## 1. 现有数据对应的数学模型
>
> 你的数据库向量满足：
>
> \\[ X\sim N(0,I\_{64}) \\]
>
> 也就是 64 维各向同性标准高斯分布。
>
> 对于一条固定查询向量 \\(q\\)，定义平方欧氏距离：
>
> \\[ D_q=\\|q-X\\|^2 \\]
>
> 那么有一个精确结论：
>
> \\[ \boxed{ D_q\sim\chi'^2\left(64,\lambda=\\|q\\|^2\right) } \\]
>
> 这里的 \\(\chi'^2\\) 是非中心卡方分布。
>
> 也就是说，我们不需要先使用正态近似，也不需要复杂的广义非中心卡方数值计算，就能得到理论上的真实距离分布。
>
> 其理论均值：
>
> \\[ E[D_q]=64+\\|q\\|^2 \\]
>
> 理论方差：
>
> \\[ Var(D_q)=128+4\\|q\\|^2 \\]
>
> 这非常适合作为第一轮实验的理论基准。
>
> ## 2. 下一步具体怎么做？
>
> 我建议直接复用原来的 5000 条数据库向量，增加一个独立的 FDL 分布验证脚本。
>
> 实验 1：标准高斯数据的 L2-FDL 验证
>
> 不需要重新建 HNSW
>
> |     |
> | --- |
>
> 参数
>
> |     |
> | --- |
>
> 设置
>
> |     |
> | --- |
>
> 数据库向量
>
> |     |
> | --- |
>
> 5000
>
> |     |
> | --- |
>
> 向量维度
>
> |     |
> | --- |
>
> 64
>
> |     |
> | --- |
>
> 数据类型
>
> |     |
> | --- |
>
> float32
>
> |     |
> | --- |
>
> 查询数量
>
> |     |
> | --- |
>
> 原测试查询
>
> |     |
> | --- |
>
> 距离
>
> |     |
> | --- |
>
> Squared L2
>
> |     |
> | --- |
>
> 理论分布
>
> |     |
> | --- |
>
> Noncentral Chi-Square
>
> 每条查询分别执行：
>
> 1. 计算与 5000 条向量的真实平方 L2 距离。
> 2. 根据查询向量计算非中心参数 \\(\lambda\\)。
> 3. 计算理论非中心卡方分布。
> 4. 比较真实距离分布和理论分布。
> 5. 记录低分位数预测误差。
>
> 主要观察：
>
> - KS Statistic
> - Wasserstein Distance
> - 1%、5%、10%、50% 分位数误差
> - 理论均值与实际均值的偏差
> - 理论方差与实际方差的偏差
>
> ## 3. 还有一个值得注意的细节
>
> 如果你的查询向量也是独立生成的标准正态向量：
>
> \\[ q\sim N(0,I\_{64}) \\]
>
> 那么：
>
> - 对于每条固定查询，距离服从非中心卡方分布。
> - 如果把所有查询和数据库向量之间的距离混在一起统计，则有：
>
> \\[ D_q\sim 2\chi^2\_{64} \\]
>
> 这两个分布不能混淆。
>
> 我们真正需要研究的是第一种：每条查询各自对应的 FDL 分布，而不是把所有查询的距离混合起来。
>
> ## 4. 后续实验路线
>
> 1. 标准高斯
>
>    已有数据，验证非中心卡方分布。
>
>    可以直接开始
>
> 2. 各向异性高斯
>
>    改变各维方差，验证广义非中心卡方分布。
>
> 3. 相关高斯
>
>    引入维度之间的相关性，验证协方差矩阵的作用。
>
> 4. 高斯混合分布
>
>    检验模型在非高斯数据上的适用边界。
>
> 5. 真实 SIFT1M
>
>    验证理论模型能否迁移到真实向量数据。
>
> 我的建议：下一步先不要扩大数据规模。 先用现有 5000×64 数据把非中心卡方验证做出来。
>
> 另外，我想确认一个小细节：原来 200 条查询向量是不是也通过 `rng.standard_normal()` 独立生成的？如果是，我们就能直接建立完整的理论对照，不需要额外准备数据。

是的。原来的 200 条查询向量也是通过同一个 `Generator` 独立生成的标准正态向量：

```python
rng = np.random.default_rng(20261115)

database = rng.standard_normal((5000, 64), dtype=np.float32)
queries = rng.standard_normal((200, 64), dtype=np.float32)
```

因此：

- 数据库向量 \(X_i\overset{iid}{\sim}N(0,I_{64})\)
- 查询向量 \(q_j\overset{iid}{\sim}N(0,I_{64})\)
- 查询与数据库使用同一随机数生成器顺序生成，但对应互不重叠的伪随机样本，可以按独立样本处理
- 固定某条查询 \(q_j\) 后，5000 个距离的理论分布为
  \[
  \|q_j-X_i\|^2\sim\chi'^2_{64}(\lambda=\|q_j\|^2)
  \]
- 混合所有查询—数据库距离时，边际分布为
  \[
  \|q-X\|^2\sim2\chi^2_{64}
  \]

需要注意：上次脚本没有把生成的向量保存成 `.npy` 文件，但固定了种子、生成顺序、形状和 `float32` 类型，所以可以逐元素重新生成完全相同的数据。你的 FDL 验证脚本应复用现有的 `make_data()`，或者将数据生成逻辑提取为共享模块，以防后续两个实验的生成顺序出现分歧。
