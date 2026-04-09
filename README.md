# NS-LSTM-GDM — Installation Guide
 
## FourCastNet Workshop Environment Setup
> ทดสอบแล้วบน Ubuntu + NVIDIA GPU (CUDA 11.6) | Python 3.10
 
---
 
## Prerequisites
 
- Miniconda / Anaconda
- NVIDIA GPU + Driver ≥ 11060 (CUDA 11.6)
- RAM ≥ 18 GB (สำหรับ full ERA5 dataset)
 
---
 
## Step 1 — Create Conda Environment
 
```bash
conda create -n ns-lstm-gdm python=3.10 -y
conda activate ns-lstm-gdm
```
 
---
 
## Step 2 — Clone FourCastNet Repository
 
```bash
git clone https://github.com/kiattikunc/FourCastNet.git
cd FourCastNet
```
 
---
 
## Step 3 — Install PyTorch (CUDA 11.6)
 
> ⚠️ ต้องใช้ PyTorch 1.13.1+cu116 เพราะ NVIDIA driver version 11060
 
```bash
pip install torch==1.13.1+cu116 torchvision==0.14.1+cu116 \
  --index-url https://download.pytorch.org/whl/cu116
```
 
ตรวจสอบ:
```bash
python -c "import torch; print(torch.__version__); print('CUDA:', torch.cuda.is_available())"
# Expected:
# 1.13.1+cu116
# CUDA: True
```
 
---
 
## Step 4 — Install Core Packages
 
```bash
pip install ruamel.yaml einops
pip install pandas scikit-learn openpyxl matplotlib tqdm
pip install torch-geometric
```
 
---
 
## Step 5 — Install timm (version เก่าที่ compatible)
 
```bash
pip install "timm==0.6.13"
```
 
---
 
## Step 6 — Install wandb (version ที่ไม่ต้อง Go binary)
 
```bash
pip install "wandb==0.16.6"
```
 
---
 
## Step 7 — Fix NumPy Version
 
> ⚠️ NumPy 2.0 ลบ `np.float_` ออก ต้อง downgrade
 
```bash
pip install "numpy==1.24.4"
```
 
---
 
## Step 8 — Fix h5py (reinstall ให้ตรงกับ NumPy 1.24.4)
 
```bash
conda install -c conda-forge "numpy=1.24.4" h5py --force-reinstall -y
```
 
---
 
## Step 9 — Fix timm import wandb error
 
> timm พยายาม import wandb ใน summary.py ทำให้ error เพราะ numpy ไม่ compatible
 
```bash
sed -i 's/import wandb/pass  # wandb disabled/' \
  ~/miniconda3/envs/ns-lstm-gdm/lib/python3.10/site-packages/timm/utils/summary.py
```
 
---
 
## Step 10 — Download ERA5 Demo Data
 
```bash
# Full dataset (RAM > 18 GB)
wget https://collab.hii.or.th/s/RcrTZtXsnbjsA56/download/ccai_demo.rar
 
# แตกไฟล์ด้วย p7zip
conda install -c conda-forge p7zip -y
7z x ccai_demo.rar
```
 
โครงสร้างที่ได้:
```
ccai_demo/
├── data/FCN_ERA5_data_v0/out_of_sample/2018.h5
├── model_weights/FCN_weights_v0/backbone.ckpt
└── additional/stats_v0/
    ├── global_means.npy
    ├── global_stds.npy
    ├── time_means.npy
    └── land_sea_mask.npy
```
 
---
 
## Step 11 — Final Verification
 
```bash
python -c "
import torch, numpy, h5py, timm, einops
from timm.models.layers import DropPath, trunc_normal_
print('PyTorch  :', torch.__version__)
print('CUDA     :', torch.cuda.is_available())
print('GPU      :', torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU')
print('NumPy    :', numpy.__version__)
print('h5py     :', h5py.__version__)
print('timm     :', timm.__version__)
print('einops   :', einops.__version__)
print('ALL OK')
"
```
 
Expected output:
```
PyTorch  : 1.13.1+cu116
CUDA     : True
GPU      : NVIDIA GeForce ...
NumPy    : 1.24.4
h5py     : 3.x.x
timm     : 0.6.13
einops   : x.x.x
ALL OK
```
 
---
 
## Quick Reference — All Commands
 
```bash
# 1. สร้าง environment
conda create -n ns-lstm-gdm python=3.10 -y
conda activate ns-lstm-gdm
 
# 2. Clone repo
git clone https://github.com/kiattikunc/FourCastNet.git
cd FourCastNet
 
# 3. PyTorch CUDA 11.6
pip install torch==1.13.1+cu116 torchvision==0.14.1+cu116 \
  --index-url https://download.pytorch.org/whl/cu116
 
# 4. Core packages
pip install ruamel.yaml einops "timm==0.6.13" "wandb==0.16.6"
pip install pandas scikit-learn openpyxl matplotlib tqdm torch-geometric
 
# 5. Fix NumPy + h5py
conda install -c conda-forge "numpy=1.24.4" h5py --force-reinstall -y
 
# 6. Fix timm wandb import
sed -i 's/import wandb/pass  # wandb disabled/' \
  ~/miniconda3/envs/ns-lstm-gdm/lib/python3.10/site-packages/timm/utils/summary.py
 
# 7. Download data
wget https://collab.hii.or.th/s/RcrTZtXsnbjsA56/download/ccai_demo.rar
conda install -c conda-forge p7zip -y
7z x ccai_demo.rar
```
 
---
 
## Troubleshooting
 
| Error | สาเหตุ | วิธีแก้ |
|-------|--------|---------|
| `PackagesNotFoundError: 3.10` | space หลัง `=` | ใช้ `python=3.10` ไม่มี space |
| `wandb build failed (Go binary)` | wandb ใหม่ต้อง Go | `pip install "wandb==0.16.6"` |
| `np.float_ removed` | NumPy 2.0 | `pip install "numpy==1.24.4"` |
| `h5py multiarray failed` | numpy/h5py ไม่ตรงกัน | `conda install numpy=1.24.4 h5py --force-reinstall` |
| `timm import wandb error` | timm เรียก wandb ที่ conflict | `sed -i` patch summary.py |
| `CUDA: False` | PyTorch ใหม่เกินกว่า driver | ใช้ `torch==1.13.1+cu116` |
| `IndexError: index 10 out of bounds` | `t_max > prediction_length` | `t_max = min(14, pred_msl.shape[0])` |
 
---
 
*สร้างขึ้นสำหรับโปรเจค NS-LSTM-GDM | KKU Workshop 2026*
 
