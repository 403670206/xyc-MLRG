# xyc-MLRG
MLRG的复现

## 一、环境准备

**创建新的 Conda 环境**
```
conda create -n mlrg python=3.9.0
conda activate mlrg
```
**安装依赖**

下载requirements.txt中的内容
```
pip install -r requirements.txt
```
#### **注意**

安装pytorch时需注意版本
```
pip install torch== 2.3.1 torchvision==0.18.1 --index-url https://download.pytorch.org/whl/cu118
```

其次，txt中的numpy== 2.2.3与他要求的python==3.9冲突
```
pip install numpy==1.25.2
```

##  二、数据集准备

**MIMIC-CXR** 数据集等需要在 PhysioNet上下载

为此我申请了账号然后还要完成它的培训，贼恶心

之后就是作者给的JSON标记文件等


##  三、模型训练

训练分两阶段：

### **阶段 1：对比预训练 (Contrastive Pretraining)**
```
bash run_cxr_pt_v0906_fs.sh
```

### **阶段 2：报告生成微调 (Report Generation Finetune)**
```
bash run_cxr_ft_mlrg_v1011.sh
```
## 四、模型评估

MLRG 提供了多种指标：

* **BLEU / METEOR / ROUGE**（文本相似度）
* **RadGraph / CheXbert**（医学诊断指标）

运行mlrg的评估工具，输出结果：

| 指标          | 值    |
| ----------- | ---- |
| BLEU-1      | 0.49 |
| BLEU-2      | 0.41 |
| BLEU-3      | 0.37 |
| BLEU-4      | 0.33 |
| METEOR      | 0.42 |
| ROUGE-L     | 0.45 |
| RadGraph F1 | 0.55 |
| CheXbert F1 | 0.50 |

结果可能有浮动


