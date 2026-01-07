# xyc-MLRG
MLRG的复现

## 一、环境准备

### 步骤：
**创建新的 Conda 环境**
conda create -n mlrg python=3.9.0
conda activate mlrg

**安装依赖**
下载requirements.txt中的内容
pip install -r requirements.txt

#### **注意**
安装pytorch时需注意版本
pip install torch== 2.3.1 torchvision==0.18.1 --index-url https://download.pytorch.org/whl/cu118

其次，txt中的numpy==2.2.3与他要求的python==3.9冲突
pip install numpy==1.25.2


##  数据集准备

MLRG 使用的是 **MIMIC-CXR** 数据集（公开但需要申请）：

1. 注册 PhysioNet 账号并申请 MIMIC-CXR 数据访问权限
   → [https://physionet.org/content/mimic-cxr/2.0.0/](https://physionet.org/content/mimic-cxr/2.0.0/)

2. 下载并解压：

# 假设下载到 /data/MIMIC-CXR/
tar -xvf mimic-cxr-2.0.0.tar.gz -C /data/MIMIC-CXR/

3. 结构整理（官方 repo 有脚本整理多视图和纵向历史）：


cd MLRG/tools
python prepare_dataset.py --data_root /data/MIMIC-CXR/ --output_root ./data/MIMIC-CXR/


> 这个脚本会生成：
>
> * 多视图训练数据
> * 纵向历史信息（patient_id, study_id）
> * 对应文本报告文件

---

## **3️⃣ 安装依赖**

MLRG 的 `requirements.txt`：


pip install -r requirements.txt


如果遇到 **radgraph 安装报错**，可以尝试：


git clone https://github.com/stanfordmlgroup/RaDGraph.git
cd RaDGraph
pip install -e .


> 如果还有报错，可以先跳过 RaDGraph，复现训练与评估流程后再加。

---

## **4️⃣ 模型训练**

MLRG 的训练分两阶段：

### **阶段 1：对比预训练 (Contrastive Pretraining)**

```bash
cd script/MIMIC-CXR
bash run_cxr_pt_v0906_fs.sh
```

> 这个阶段会：
>
> * 训练图像编码器
> * 学习多视图纵向对比表示
> * 输出 checkpoint，供阶段 2 使用

### **阶段 2：报告生成微调 (Report Generation Finetune)**

```bash
cd script/MIMIC-CXR
bash run_cxr_ft_mlrg_v1011.sh
```

> 这个阶段会：
>
> * 加载阶段 1 的预训练模型
> * 利用历史信息生成报告
> * 输出最终 checkpoint 与生成报告文件

> ⚠️ 注意：脚本里面有 `--batch_size` 和 `--gpu` 等参数，可根据显存调整。

---

## **5️⃣ 模型评估**

MLRG 提供了多种指标：

* **BLEU / METEOR / ROUGE**（文本相似度）
* **RadGraph / CheXbert**（医学诊断指标）

示例：

```bash
python tools/evaluate.py \
    --pred ./output/generation_results.json \
    --gt ./data/MIMIC-CXR/test_reports.json
```

会输出：

```text
BLEU-1: 0.42
BLEU-2: 0.35
RadGraph F1: 0.65
...
```

---

## **6️⃣ 复现要点**

1. **环境冲突**：

   * Python 3.9 + numpy 1.25
   * PyTorch 2.2 + 对应 CUDA

2. **数据路径**：

   * 确保 `prepare_dataset.py` 输出的路径和训练脚本里 `--data_root` 一致

3. **显存问题**：

   * batch_size 可调小（8 → 4）
   * 多视图训练占用大，推荐 24GB GPU 或分布式训练

4. **依赖问题**：

   * radgraph/transformers/torchvision 版本要对应
   * 先保证训练跑通，再处理指标计算

---

我可以帮你整理一个 **可直接运行的复现流程脚本**（从环境搭建、数据处理到训练和评估一条命令运行），这样你直接复制到终端就能复现。

你希望我直接帮你写这个吗？

