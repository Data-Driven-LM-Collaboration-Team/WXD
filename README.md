 异常图片生成

![](Snipaste_2025-10-16_13-25-14.jpg)
 
![](Snipaste_2025-08-06_13-15-02.jpg)
### **现有问题**  

1. **真实异常样本稀缺**：工业场景中产品缺陷罕见，导致训练数据严重不平衡（大量正常样本 vs 极少异常样本）。  
2. **现有异常生成方法效果有限**：
   - **模型无关方法**（如 Crop&Paste）：简单拼接异常纹理到正常图像，生成结果缺乏真实性与上下文一致性（见图1）。
   - **GAN-based 方法**（如 DRAEM）：在异常定位上表现有限，且无法支持异常分类任务；生成质量受限于 GAN 训练不稳定性和模式崩溃问题。  
3. **下游任务受限**：由于缺乏高质量异常数据，现有方法难以同时支持检测、定位和分类三大任务。

---

### **提出的方法**  

AnomalyDiffusion 是一个**条件扩散模型**，其核心流程如下：

1. **输入**：一张正常图像$`I_{\text{norm}}`$ + 一张异常掩码$`M`$（指示异常位置与形状）。
2. **掩码生成模块**（基于 Textual Inversion）：
   - 利用少量真实异常掩码学习一个可泛化的“掩码嵌入”；
   - 生成大量多样化的合成掩码，提升异常形态多样性。
3. **扩散生成主干**（U-Net 架构）：
   - 引入 **空间异常嵌入（SAE）**：将掩码编码为位置感知的条件信号；
   - 采用 **掩码扩散损失（Masked Diffusion Loss）**：仅在异常区域计算重建损失；
   - 设计 **自适应注意力重加权机制（AAR）**：利用交叉注意力图与掩码对齐，增强空间一致性。$$
4. **输出**：合成异常图像$`I_{\text{anom}} = I_{\text{norm}} + \text{realistic anomaly at } M`$。

> **图示参考**：论文图1（Bottom）展示了 AnomalyDiffusion 在 hazelnut-crack 和 capsule-squeeze 类别上生成的异常图像，明显比 Crop&Paste 和 DRAEM 更真实、边界更自然。

---

### **方法如何解决问题**  

- **解决数据稀缺**：通过扩散模型强大的生成能力，从 few-shot 异常样本中学习分布，生成大量训练数据。  
- **提升生成真实性**：
  - SAE 和 AAR 确保异常出现在掩码指定位置，且与背景纹理融合自然；
  - 掩码扩散损失聚焦异常区域细节，避免全局模糊。  
- **支持多任务**：生成的图像包含明确异常类型与位置，可直接用于训练分类器（AC）、检测器（AD）和定位模型（AL）。  
- **避免 GAN 缺陷**：扩散模型训练更稳定，生成多样性更高，不易模式崩溃。

---

### **实验结果**  

1. **生成质量评估**：
   - 在 MVTec AD 数据集上，AnomalyDiffusion 在 IS（Inception Score）和 IC-LPIPS（多样性）指标上显著优于 DRAEM、Crop&Paste 等基线。
   - 人工评估显示生成异常更逼真、边界更清晰（见图1）。

2. **下游任务性能**：
   - **异常分类（AC）**：使用生成数据训练 ResNet-34，平均准确率达 **66.09%**，远超 DFMGAN（49.61%）。
   - **异常检测与定位（AD/AL）**：
     - 仅用一个简单 U-Net 在生成数据上训练，在 MVTec 上达到 **95.3% 像素级 AUROC**；
     - 性能媲美或超越当前 SOTA 无监督方法（如 PatchCore、CFLOW），证明生成数据高度可用。

3. **消融实验**：验证了 SAE、AAR 和掩码损失各自对生成质量与任务性能的贡献。

---

综上，AnomalyDiffusion 通过创新的扩散模型架构与条件机制，有效解决了工业异常数据稀缺问题，为少样本异常检测提供了高质量数据增强方案。



### MVTec 数据集

- MVTec数据集包含5354张不同目标和纹理类型的高分辨彩色图像。它包含用于训练的正常（即不包含缺陷）的图像，以及用于测试的异常图像。异常有70种不同类型的缺陷，例如划痕、凹痕、污染和不同结构变化。

- 此外，本文为异常提供了像素级精确的标签。

- 本文还对目前最先进的基于深度结构的无监督异常检测方法进行了评估，例如卷积自动编码CAE、生成对抗网络GAN、基于预训练卷积神经网络的特征描述符以及传统的计算机视觉方法。

MVTec数据集是第一个全面的、多目标、多缺陷并且提供像素级精确标签的异常检测数据集，关注于真实世界应用。




### 如何使用 Transformer 架构实现高分辨率图像合成
![](Snipaste_2025-10-23_13-24-11.jpg)
如何利用 Transformer 架构实现高分辨率图像合成，突破传统卷积生成模型（如 GANs）在建模长程依赖和全局结构方面的局限性。作者系统性地将自注意力机制引入图像生成任务，设计了一种基于 Transformer 的高分辨率图像生成框架。

现有问题: 传统基于 CNN 的生成模型（如 StyleGAN）虽然在图像质量上表现优异，但其局部感受野难以有效建模图像中的全局语义一致性和长距离依赖关系；而早期基于 Transformer 的图像生成方法（如 Image Transformer）受限于计算复杂度，只能在低分辨率（如 32×32 或 64×64）图像上训练，无法直接扩展到高分辨率（如 256×256 或更高）图像合成。

提出的方法: 它提出了一种分层、稀疏或基于向量量化的 Transformer 图像生成架构（具体方法名称未在提供的文本中明确，但可推断为类似 VQ-Transformer 或 Hierarchical Transformer 的设计）。其流程大致包括：  
![](gongshi1.jpg)
![](gongshi2.jpg)
![](gongshi3.jpg)
1. 使用向量量化变分自编码器（VQ-VAE）将高分辨率图像压缩为低维离散潜在表示（token 序列）；  
2. 在离散潜在空间上训练一个自回归 Transformer 模型，以建模 token 之间的全局依赖；  
3. 通过解码器将生成的 token 序列重建为高分辨率图像。  
   （注：文中引用了 Mentzer et al. [47]、Ramesh et al. [59]、Menick & Kalchbrenner [46] 等工作，暗示采用了 VQ + Transformer 范式。）

方法如何解决问题： 该方法通过两阶段策略解决高分辨率生成难题：  

- 第一阶段（VQ-VAE）大幅降低图像空间维度，将像素级建模转化为对紧凑离散 token 序列的建模，缓解了 Transformer 的计算瓶颈；  
- 第二阶段（Transformer）在 token 序列上利用自注意力机制捕获全局上下文信息，从而确保生成图像在语义和结构上的一致性，克服了 CNN 生成器的局部性缺陷。

实验结果： 论文在多个高分辨率图像数据集（如 ImageNet、FFHQ）上验证了所提方法的有效性。实验表明，该方法能够生成细节丰富、结构合理的 256×256 甚至更高分辨率图像，在 FID（Fréchet Inception Distance）和 IS（Inception Score）等指标上优于或媲美当时的 SOTA GAN 模型（如 StyleGAN2），同时展现出更强的多样性与语义可控性。此外，消融实验验证了 Transformer 在建模长程依赖方面的优势。

 <img width="800"  alt="异常点" src="https://github.com/user-attachments/assets/4a17eb38-d530-4934-802f-5db3c20e7600" />
 
 *图1：模型生成不足处*  
  
 模型的缺陷是在不恰当处生成变化，要更改模型结构或者loss来解决。

### 生成模型评估指标： IS, FID, clip-score, pick-score, HPS
通用图像生成（无文本）：可用 FID（主）+ IS（辅）
文生图（Text-to-Image）：可用 clip-score + pick-score（评估对齐与质量），FID（评估整体分布）

测试模型gemini pro3-nano banana

<img width="520" height="131" alt="image" src="https://github.com/user-attachments/assets/6474515e-f6f5-43fa-a947-462b8d4f0d89" />


prompt:
图1是真实有瑕疵的布料图片，图2白色为图1布料瑕疵的位置，图3是完好布料图片，图4是位置遮罩。注意瑕疵是指图1在图2白色位置处的地方。现在请生成一张图片，图3在图4白色位置处生成瑕疵，要求风格类似图1的瑕疵。

<img width="209" height="209" alt="image" src="https://github.com/user-attachments/assets/2600df51-6a9e-4375-9e58-8100088429fa" />

图1是真实的有瑕疵的布料图片，图2白色区域为图1布料瑕疵的位置，图3是完好布料图片，图4是位置遮罩。注意"瑕疵"是指图1在图2白色位置处的布料褶皱。现在请修改图3，让图3在图4白色位置处生成"瑕疵"，要求风格与图1的"瑕疵"风格一致。要求符合实际，符合物理规律。

<img width="310" height="310" alt="image" src="https://github.com/user-attachments/assets/2e76430b-299e-48a1-92ce-afd3a470d2e8" />

### 改进想法：

<img width="685" height="229" alt="image" src="https://github.com/user-attachments/assets/f22a7d3a-41f2-4131-9076-bd5843f79f4d" />

### 两阶段潜空间去噪预览：

#### prompt: minor flaw in bottle，obvious crack
##### 潜空间加噪强度：0.3

<img width="1988" height="1054" alt="image" src="https://github.com/user-attachments/assets/b0230426-3861-421f-b9b2-017d427c8f2a" />

##### 潜空间加噪强度：0.15

<img width="1992" height="1059" alt="image" src="https://github.com/user-attachments/assets/20ac58ad-ffce-4a7e-96f9-e7ae68ebcfc4" />

##### 潜空间加噪强度：0.05

<img width="1996" height="1054" alt="image" src="https://github.com/user-attachments/assets/655a8242-0a14-4140-a5aa-6931b7ce50b8" />

#### prompt: Astronaut in a jungle, cold color palette, muted colors, detailed, 8k
##### 潜空间加噪强度：0.15

<img width="1998" height="714" alt="image" src="https://github.com/user-attachments/assets/4593349e-74ac-46e5-a315-d349298205fe" />

##### 潜空间加噪强度：0.05

<img width="2001" height="719" alt="image" src="https://github.com/user-attachments/assets/3c7a1700-0ab7-4acb-bb04-3e147f220f3c" />

### 两阶段潜空间去噪架构：

<img width="1571" height="892" alt="image" src="https://github.com/user-attachments/assets/d20ed7cf-b78d-475f-b0d7-6007268a0b74" />
### 两阶段潜空间去噪架构(base+base)：
<img width="1195" height="631" alt="image" src="https://github.com/user-attachments/assets/63006e1e-f1b4-4b4f-9553-cf2abd9521dc" />
固定高低频ratio时：低频部分收敛，高频部分发散。
<img width="1175" height="860" alt="image" src="https://github.com/user-attachments/assets/2334a036-48f0-4944-980e-e74e29656773" />
<img width="1180" height="819" alt="image" src="https://github.com/user-attachments/assets/e4b6b72c-0f1d-43cb-b0b4-0bdc20f33931" />

sigma_max20k后显著增大。
```
训练时：
  sigma_t 已知（采样得到）
  → spectral_gate(H, W, sigma_t) → W_low, W_high
  → GT分解:  gt_low  = IFFT(FFT(z_0) · W_low)
              gt_high = IFFT(FFT(z_0) · W_high)
  → 输出滤波: pred_low  = IFFT(FFT(denoised_base) · W_low)
              pred_high = IFFT(FFT(denoised_refiner) · W_high)
  → loss = w(σ) · ||pred_low - gt_low||² + λ · w(σ) · ||pred_high - gt_high||²
```
<img width="1166" height="584" alt="image" src="https://github.com/user-attachments/assets/b56f33ac-eabc-45ac-b184-4f7b26a9ba44" />
<img width="518" height="260" alt="image" src="https://github.com/user-attachments/assets/1f25da01-0a58-4870-b9c6-ae41f98952c1" />


数据集zju leaper 异常种类中的fabric_pattern17到19，与anomalydiffusion做对比 主实验

| 指标               | FID↓ | IS↑  | LPIPS↓ | SSIM↑ | AUROC-I↑ | AUROC-P↑ |
| ------------------ | ---- | ---- | ------ | ----- | -------- | -------- |
| anomalydiffusion   | -    | -    | -      | -     | -        | -        |
| FreqAD（完整模型） | -    | -    | -      | -     | -        | -        |

- **FID**（Fréchet Inception Distance）：衡量生成图像与真实图像分布的距离，越低越好；
- **IS**（Inception Score）：衡量生成图像的多样性和清晰度，越高越好；
- **LPIPS**（Learned Perceptual Image Patch Similarity）：衡量感知层面的差异，越低表示越逼真；
- **SSIM**（Structural Similarity）：衡量结构相似性，越高表示保留原图结构越好。

| 指标                 | FID↓ | IS↑  | LPIPS↓ | SSIM↑ |
| -------------------- | ---- | ---- | ------ | ----- |
| FreqAD（完整模型）   | -    | -    | -      | -     |
| 移除 NCSG            | -    | -    | -      | -     |
| 移除 FAAM            | -    | -    | -      | -     |
| 移除第二个文本编码器 | -    | -    | -      | -     |



MVTec AD 上的下游异常检测性能。各方法生成的数据用于训练检测模型。最佳结果**加粗**，次佳结果下划线。

| 方法                  | 图像级 AUROC-I↑ | 图像级 AP-I↑ | 图像级 F1max-I↑ | 像素级 AUROC-P↑ | 像素级 AP-P↑ | 像素级 F1max-P↑ | PRO↑ |
| --------------------- | --------------- | ------------ | --------------- | --------------- | ------------ | --------------- | ---- |
| 无数据增强（No Aug.） | -               | -            | -               | -               | -            | -               | -    |
| SDGAN                 | -               | -            | -               | -               | -            | -               | -    |
| ...                   | ...             | ...          | ...             | ...             | ...          | ...             | ...  |
| FreqAD（本文方法）    | -               | -            | -               | -               | -            | -               | -    |


CSMFS: **Conditional Spectral Mask with Frequency-domain Soft routing** （条件驱动的频域软路由掩码）

ZJU Leaper Custom

| Category | AnomalyDiffusion | Ours（AnomalyDiffusion+CSMFS） |
| -------- | ---------------- | ------------------------------ |
| fabric   | 43.75%           | 60.42%                         |

MVTec AD

| Category | AnomalyDiffusion | Ours（SDXL+CSMFS） |
| -------- | ---------------- | ------------------ |
| Zipper   | 69.53%           | 64%左右            |
|    ... (待测量)     |                  |                    |


1. 异常可出现在任意位置 → 空间增强 (翻转、旋转) 有效
2. 异常形态与方向无关 → 几何变换保持语义
3. 工业相机光照可变 → ColorJitter 增加鲁棒性
4. 异常区域可能很小 → RandomErasing 模拟遮挡

<img width="1046" height="520" alt="image" src="https://github.com/user-attachments/assets/60cbee53-1361-43c7-bc68-5dc19029ce3b" />


```
trainer.fit(model, data)
  └─ training_step(batch, batch_idx)                 [ddpm.py, PL hook]
       ├─ shared_step(batch)
       │     → get_input(batch, 'image')             [ddpm.py:803]
       │         ├─ encode_first_stage(img)           → z = VAE(img) * scale_factor
       │         ├─ mask = batch['mask']              → 二值异常区域
       │         ├─ c_text = batch['caption']         → 文本条件
       │         └─ mask_cond = img                   → PSP 编码器输入
       │     → return z, c_text, mask, mask_cond, name
       │
       │     → forward(z, c_text, mask=mask, mask_cond=mask_cond, name=name)
       │           ├─ t = randint(0, T, [B])          → 随机时间步
       │           ├─ (CSMFS) get_dual_conditioning() → c_dual [B, 154, 1280]
       │           │  或 (基线) get_learned_conditioning() → c [B, 77, 1280]
       │           └─ p_losses(z, c, t, mask=mask)    → loss, loss_dict
       │
       └─ return loss  → PL 自动 backward + optimizer.step
```
2x2合并，无重叠
<img width="1034" height="260" alt="18bcfaa9ee0e069fcaf5095dfcc76d09" src="https://github.com/user-attachments/assets/37bfa25c-2c8a-4ba5-9420-9a64807d7aca" />
<img width="138" height="36" alt="17eb44f1a541e2cdf632eeacde0c141b" src="https://github.com/user-attachments/assets/88bcd69f-6c63-4a6c-87b1-e060ea1dd9f8" />
3x3
<img width="1034" height="260" alt="image" src="https://github.com/user-attachments/assets/fd1a6f04-cbef-4dfb-b12e-cc2a20525618" />
3x3 + tiled diffusion 推理 pattern 17
<img width="256" height="256" alt="image" src="https://github.com/user-attachments/assets/1ee8b919-bcb2-41b6-a00e-5368b2776c0f" />
<img width="256" height="256" alt="image" src="https://github.com/user-attachments/assets/48c5aacb-9601-467a-b50d-fd76c7731803" />

3x3 分块 重叠50% 使用tile diffusion推理 pattern16 -> 大瑕疵纹理高频互相抵消 导致纯色低频 (cfg=5)
<img width="1034" height="260" alt="2119cf41ce6c8c94a74f5b8e6ff9c713" src="https://github.com/user-attachments/assets/f090ec32-6c87-44d6-ae17-d48f5fe7bb6d" />
原图是：
<img width="1034" height="260" alt="12c9cea2f5eb4f4f183208b2d60e7328" src="https://github.com/user-attachments/assets/adfc47b3-88d2-4b56-a28a-19fee0946ca3" />

推理时  将每一层basicTransformer的自注意力reshape 实现跨tile 共享注意力. 交叉注意力层不作reshape ，即每个块的空间文本向量仍然互不关联(cfg=9)
<img width="1034" height="260" alt="035a5051eae8caa047b65e738eff4efa" src="https://github.com/user-attachments/assets/9dcb6cd6-76d4-4a60-95c5-048457179a84" />
[B*N,H*W,C]->[B,N*H*W,C]
原图是：
<img width="1034" height="260" alt="8cf2edb2e1b6bb1d39ee663c80f50d0e" src="https://github.com/user-attachments/assets/ba334ca8-11cc-41f7-b2d8-9fcd5b9ede2a" />

值得进一步研究的是如何共享交叉注意力，训练时候开启共享自注意力会不会有效果。


开启交叉注意力共享后，训练后的可视化效果目前很差: 
<img width="1034" height="260" alt="Image_2026-04-30_13-12-06_yjzgry4i rye" src="https://github.com/user-attachments/assets/d03709b5-6805-4e45-9b35-52bfdbd07bda" />

可能的原因是，训练的时候因为共享了注意力导致模型学到的内容变得丰富，cfg需要适当调小才行。
<img width="1034" height="260" alt="b3d92e0f4c34b1d788eb018b0ba1a320" src="https://github.com/user-attachments/assets/6eea02ba-8ae8-4c3f-a1a2-04fa25ed5260" />
<img width="256" height="256" alt="e4414d7fb50dcf50ee0ed53ae5eef5a4" src="https://github.com/user-attachments/assets/f053a301-559d-45d8-989c-0f7256314c21" />
<img width="1034" height="260" alt="cc5d69e65e51bb18016af93be04730d3" src="https://github.com/user-attachments/assets/759980e2-d4bf-4485-af4f-5621d3783483" />

待解决难题：tile间方差巨大，不同tile学习效果悬殊
使用 max为 N 个token 向量学习，软gate机制自动调整token数量
<img width="784" height="784" alt="1240bd9f5aa315438f9cfabdbf691253" src="https://github.com/user-attachments/assets/0baf1ef7-4b3d-4d36-b9f2-ffd296786196" />
<img width="784" height="784" alt="6691a3cc1394b484caf422a8dfb82652" src="https://github.com/user-attachments/assets/92e0e677-8eac-4cc5-a4db-615ed6947cec" />



zju leaper small 数据集
#### 未标注的cfg默认为5,每个异常子类默认训练10k步
| 排名 | 方法 |  Acc |
|:-:|---|:-:|
| **1** | **v13_stageB_cfgA3.0_B1.0**(**+stagaB 各种token**) | **81.25%** ★ |
| 2 | **v11_anomaly_purify**(**+bg token,sub bg token**) | **72.92%** |
| 3 | v11_anomaly_purify_append_token(+bg token,sub bg token,此外第二阶段训练时候冻结stageA训练好的向量仅仅训练残差token) | 70.83% |
| 4 | **v7_stageA_256**(**+tbf mask+sub class token**) | **68.75%** |
| 5 | v7_stageA_256_cfg2(+tbf mask+sub class token)  | 62.50% |
| 6 | cta_lite_adapter（分块降噪并合并） | 52.08% |
| 7 | baseline_512_lpips_v2(+tbf mask) | 45.83% |
| 8 | baseline_512 plain | 43.75% |
| 9 | anomaly diffusion baseline | 43.75% |
| 10 | **v11_anomaly_purify_512**(+ bg token,sub bg token) | **37.50%** |

结论：ldm-1.3B模型不适合文本反推512分辨率的图片。两阶段学习残差最适合文本反推来生成细小瑕疵。

## 三阶段训练：

<img width="1789" height="810" alt="2bb533cd2b5ea573f653d42977596a76" src="https://github.com/user-attachments/assets/497b4c6a-91f6-4a76-8881-92cfb73bd5fe" />

### stage0: 训练背景token ,使用 ssim分块聚类自动计算子类token数量: bg token + bg sub class token
### stageA: 训练异常token ,使用 ssim分块聚类 + tbf mask + pix loss学习精确的异常特征(像素级): class token + sub class token + psp token 
### stageB: 冻结stageA 学习到的条件向量，训练新的异常token学习残差 ，仍然使用 ssim分块聚类 + tbf mask 学习特定区域: class_B token + sub class_B token + psp token 

#### PS: psp token 为anomaly diffusion自带的空间编码器产生的token.

### TBF mask
TBF = Trim + Bilinear + Fill

对应 pipeline 的最后三个关键步骤：

Trim：清零边缘一行/列（消除 VAE padding 伪影）

Bilinear：与双线性下采样的基线 mask 取 max 合并（互补不遗漏）

Fill：迭代填洞（≥3/4 邻居为白则填充，闭合内部孔洞）


如图所示：

#### mask进入vae 后面取得4通道的各个矩阵 使用ostu 计算出最大类间方差，将比例小的视作前景mask,对四通道取得并集得到ostu_mask,此外使用Trim操作：清零边缘一行/列（消除 VAE padding 伪影）

<img width="1036" height="561" alt="image" src="https://github.com/user-attachments/assets/4b8f56a0-2dc8-4ce7-9015-63e34d1d4d38" />

#### Bilinear遮罩与ostu mask 取 max 合并（互补不遗漏）

<img width="351" height="354" alt="image" src="https://github.com/user-attachments/assets/82c28730-d883-44d3-a945-dad602d74463" />

#### Fill：迭代填洞（≥3/4 邻居为白则填充，闭合内部孔洞）

<img width="787" height="401" alt="image" src="https://github.com/user-attachments/assets/0c95686a-c95e-4c91-9d90-1001e83f1b8c" />


### 双线性插值遮罩：

<img width="527" height="486" alt="image" src="https://github.com/user-attachments/assets/a4ca242a-ea5a-4ad7-8144-8350905a8af8" />


<img width="1006" height="299" alt="Snipaste_2026-06-02_22-23-25" src="https://github.com/user-attachments/assets/2a3fb361-43f7-483f-8a6c-b91e39027baf" />


### tbf遮罩：

<img width="580" height="577" alt="image" src="https://github.com/user-attachments/assets/4a493dd6-1923-4698-aff0-7fab5268d466" />

<img width="990" height="293" alt="image" src="https://github.com/user-attachments/assets/5d9c2570-c0f4-4729-8cf1-e02e6a8d4caa" />

magic 70.83%  ours 81.25%


### pattern 16
#### magic   明显出现类间平滑

<img src="https://github.com/user-attachments/assets/fe3f3664-14e6-4ffd-96ff-2f5b4bc8fd08" alt="image" style="height:516px; width:auto;" />

#### ours:

<img src="https://github.com/user-attachments/assets/69be70c0-f8b1-43eb-b4a9-81875f6f7966" alt="Image_2026-06-09_20-57-35_2al50czt k5b" style="height:516px; width:auto;" />


### 二阶段残差改善可视化：

#### bottle broken  large

#### baselines:

<img src="https://github.com/user-attachments/assets/f1282b5e-55af-4268-b21b-f8f5af021518" alt="Image_2026-06-09_21-00-30_gmspbzx0 e2v" style="height:516px; width:auto;" />


#### ours:

<img src="https://github.com/user-attachments/assets/ea4a340b-5bf3-475a-b263-1e30f0cbb050" alt="Image_2026-06-09_20-59-18_4eub4a44 but" style="height:516px; width:auto;" />


| Category | DualAnoDiff (official code) | DualAnoDiff† (paper values) | MAGIC | **Ours‡ (GPU1 batch)** |
| :--- | ---: | ---: | ---: | ---: |
| bottle | 72.09 | **79.07** | <u>76.74</u> | **79.07** |
| cable | 56.25 | **78.12** | <u>68.75</u> | 65.62 |
| capsule | 48.00 | **70.67** | 58.67 | <u>68.00</u> |
| carpet | <u>70.97</u> | **79.03** | 62.90 | 66.13 |
| grid | 60.00 | **80.00** | 60.00 | **80.00** |
| hazelnut | 85.42 | <u>89.58</u> | **97.92** | <u>89.58</u> |
| leather | 84.13 | **90.48** | <u>85.71</u> | 79.37 |
| metal_nut | 76.56 | <u>89.06</u> | **90.62** | — |
| pill | 33.33 | <u>56.25</u> | **67.71** | — |
| screw | 58.02 | 70.37 | **82.72** | <u>80.25</u> |
| tile | <u>98.25</u> | **100.00** | **100.00** | — |
| transistor | **71.43** | **71.43** | **89.29** | — |
| wood | 71.43 | **85.71** | <u>73.81</u> | — |
| zipper | <u>73.17</u> | <u>75.61</u> | **78.05** | — |
| **Average** | 68.50 | **79.67** | <u>78.06</u> | **76.00**† |


### 📝 中文版初稿：方法论 (Methodology)

**3. 方法论 (Methodology)**

**3.1 总体架构 (Overall Architecture)**
本文提出了一种基于潜在扩���模型（Latent Diffusion Models, LDM）的工业异常生成新范式。为了解决传统文本反演（Textual Inversion）在拟合高度不规则、多峰态工业异常分布时产生的模式坍塌与边缘混叠问题，我们构建了一个端到端的两阶段生成框架。具体而言，该框架包含四个核心模块：(1) 应对长尾分布的子类谱聚类机制；(2) 保证特征对齐的 TBF 掩码下采样策略；(3) 架构核心：噪声空间梯度提升范式；(4) 解决空间混叠的低时间步像素级损失。

**3.2 应对类内方差的子类谱聚类 (Subclass Spectral Clustering)**
工业缺陷在同一类别内往往表现出极大的视觉方差（例如背景光照的剧烈波动或异常形态的多样性）。使用单一条件向量强制对整个类别进行编码，会导致模型在潜空间中执行平滑的“加权平均”，从而丧失生成多样性。为此，我们引入了基于 Tile-SSIM 的谱聚类算法。在训练前，算法将每个类别自适应地划分为 $K$ 个子类，并为每个子类分配独立的全局异常 Token 和背景 Token。这种显式的语义路由机制打破了长尾分布下的条件表达瓶颈，为后续的残差学习提供了纯粹的初始化状态。

**3.3 潜空间对齐的掩码处理 (Latent-Aligned Mask Processing via TBF)**
在计算掩码损失时，直接对高分辨率像素掩码进行双线性插值或最大池化，会导致微小异常（如裂纹、划痕）在潜空间分辨率下严重萎缩或产生方块化伪影。为了实现精确的空间特征对齐，我们提出 TBF (Trim, Bilinear, Fill) 掩码处理管道。我们创新性地利用预训练 VAE 编码器的多通道响应机制，将二值掩码映射为包含物理投影先验的 4 通道特征，并通过逐通道 Otsu 自适应阈值化与形态学闭合（Fill holes）完成合并。TBF 确保了在 $32 \times 32$ 潜空间中计算的重构误差能够完美贴合真实物理边界。

**3.4 噪声空间梯度提升 (Noise-Space Gradient Boosting)**
真实工业异常的概率密度函数表现出极强的不规则多峰尖峰特性。为了高保真地拟合这一分布，我们借鉴经典机器学习中的梯度提升思想，在扩散模型的噪声预测空间中提出两阶段残差学习架构。
在训练阶段，我们冻结 UNet 主干，并设立两条独立的条件注入路径：Stage-A（冻结的平滑核估计）和 Stage-B（可训练的残差补偿）。给定输入 $x_t$ 与时间步 $t$，模型预测的噪声公式化为：


$$\hat{\boldsymbol{\varepsilon}} = \hat{\boldsymbol{\varepsilon}}_A(x_t, t, c_A) + \hat{\boldsymbol{\varepsilon}}_B(x_t, t, c_B)$$


其中 $\hat{\boldsymbol{\varepsilon}}_A$ 停止梯度回传，Stage-B 路径的最优解被严格约束为拟合目标残差：


$$\hat{\boldsymbol{\varepsilon}}_B^* = \mathbb{E}[\boldsymbol{\varepsilon} - \hat{\boldsymbol{\varepsilon}}_A \mid x_t, c_B]$$


在推理阶段，为了防止残差特征被过度放大而产生彩虹噪声（Rainbow artifacts），我们采用非对称无分类器引导（Asymmetric CFG），为主分布预测 $\hat{\boldsymbol{\varepsilon}}_A$ 施加高引导尺度（$w_A=3$），而对残差预测 $\hat{\boldsymbol{\varepsilon}}_B$ 保持原尺度（$w_B=1$）。

**3.5 低时间步像素级掩码损失 (Low-Timestep Pixel-Space Mask Loss)**
尽管 TBF 策略缓解了掩码的粗粒度错位，VAE 固有的 $8\times$ 下采样依然会在潜空间不可避免地引入高频混叠（Spatial Aliasing），导致生成的异常边界模糊。为此，我们提出了一种动态空间优化的像素级损失 $\mathcal{L}_{\text{pix}}$。
我们观察到，在高噪声阶段（$t \to T$），推导的 $\hat{x}_0$ 缺乏有意义的结构信息；而在低噪声阶段（$t \leq 200$），$\hat{x}_0$ 已初步成型。因此，仅在 $t \leq 200$ 时，我们将预测的 $\hat{x}_0$ 穿透 VAE 解码器映射回像素空间，并仅在异常掩码区域 $\mathbf{m}_{\text{pix}}$ 内计算 $\ell_1$ 损失：


$$\mathcal{L}_{\text{pix}} = \mathbb{E}_{t \leq 200} \left[ \frac{\| \mathbf{m}_{\text{pix}} \odot (\hat{x}_0(\hat{\boldsymbol{\varepsilon}}) - x_0) \|_1}{\|\mathbf{m}_{\text{pix}}\|_1} \right]$$


该损失函数的梯度能够通过冻结的 VAE 解码器和 UNet 主干，精准反向传播至 Stage-B 的文本嵌入参数中，引导模型生成极其锐利且符合物理几何的缺陷边界。

---

### 🎓 英文版学术稿：Methodology (可直接用于 AAAI 投稿)

**3. Methodology**

**3.1 Overall Architecture**
This paper proposes a novel paradigm for industrial anomaly generation based on Latent Diffusion Models (LDMs). To address the issues of mode collapse and boundary aliasing caused by conventional textual inversion when fitting highly irregular, multi-modal industrial anomaly distributions, we construct an end-to-end two-stage generative framework. Specifically, the framework consists of four core modules: (1) a Subclass Spectral Clustering mechanism for modeling intra-class variance; (2) a Latent-Aligned Mask Processing (TBF) strategy to ensure spatial fidelity; (3) the core architecture: Noise-Space Gradient Boosting; and (4) a Low-Timestep Pixel-Space Mask Loss to mitigate spatial aliasing.

**3.2 Subclass Spectral Clustering for Intra-class Variance**
Industrial defects typically exhibit significant visual variance within the same category (e.g., severe fluctuations in background illumination or diverse anomaly morphologies). Forcing a single condition vector to encode an entire category causes the model to perform a smoothed "weighted average" in the latent space, thereby sacrificing generative diversity. To alleviate this, we introduce a Tile-SSIM-based spectral clustering algorithm. Prior to training, this algorithm adaptively partitions each category into $K$ subclasses and assigns independent global anomaly tokens and background tokens to each subclass. This explicit semantic routing mechanism breaks the conditional representation bottleneck under long-tailed distributions, providing a purified initialization state for subsequent residual learning.

**3.3 Latent-Aligned Mask Processing via TBF**
When computing the mask loss, directly applying bilinear interpolation or max-pooling to high-resolution pixel masks causes subtle anomalies (e.g., micro-cracks, scratches) to severely shrink or produce blocky artifacts at the latent resolution. To achieve precise spatial feature alignment, we propose the Trim, Bilinear, Fill (TBF) mask processing pipeline. We innovatively leverage the multi-channel response mechanism of the pre-trained VAE encoder to map binary masks into 4-channel features infused with physical projection priors. This is followed by per-channel Otsu adaptive thresholding and morphological hole-filling. The TBF pipeline ensures that the reconstruction error computed in the $32 \times 32$ latent space perfectly adheres to the authentic physical boundaries of the defects.

**3.4 Noise-Space Gradient Boosting**
The probability density functions of real-world industrial anomalies exhibit highly irregular, multi-modal peak characteristics. To fit this distribution with high fidelity, we draw inspiration from the concept of Gradient Boosting in classical machine learning and propose a two-stage residual learning architecture within the noise prediction space of the diffusion model.
During the training phase, we freeze the UNet backbone and establish two independent conditioning paths: Stage-A (a frozen, smoothed kernel estimator) and Stage-B (a trainable residual compensator). Given the input $x_t$ and timestep $t$, the predicted noise of the model is formulated as:


$$\hat{\boldsymbol{\varepsilon}} = \hat{\boldsymbol{\varepsilon}}_A(x_t, t, c_A) + \hat{\boldsymbol{\varepsilon}}_B(x_t, t, c_B)$$


where the gradient backpropagation for $\hat{\boldsymbol{\varepsilon}}_A$ is stopped. The optimal solution for the Stage-B path is strictly constrained to fit the target residual:


$$\hat{\boldsymbol{\varepsilon}}_B^* = \mathbb{E}[\boldsymbol{\varepsilon} - \hat{\boldsymbol{\varepsilon}}_A \mid x_t, c_B]$$


During the inference phase, to prevent the residual features from being over-amplified—which could generate rainbow artifacts—we employ an Asymmetric Classifier-Free Guidance (CFG). Specifically, we apply a high guidance scale ($w_A=3$) to the main distribution prediction $\hat{\boldsymbol{\varepsilon}}_A$, while maintaining the original scale ($w_B=1$) for the residual prediction $\hat{\boldsymbol{\varepsilon}}_B$.

**3.5 Low-Timestep Pixel-Space Mask Loss**
Although the TBF strategy alleviates coarse-grained mask misalignment, the inherent $8\times$ downsampling of the VAE inevitably introduces spatial aliasing in the latent space, resulting in blurred anomaly boundaries. To address this, we propose $\mathcal{L}_{\text{pix}}$, a pixel-level loss governed by dynamic spatial optimization.
We observe that in the high-noise phase ($t \to T$), the derived $\hat{x}_0$ lacks meaningful structural information; conversely, in the low-noise phase ($t \leq 200$), $\hat{x}_0$ begins to physically materialize. Therefore, strictly when $t \leq 200$, we decode the predicted $\hat{x}_0$ back to the pixel space using the VAE decoder and compute the $\ell_1$ error exclusively within the pixel-space anomaly mask $\mathbf{m}_{\text{pix}}$:


$$\mathcal{L}_{\text{pix}} = \mathbb{E}_{t \leq 200} \left[ \frac{\| \mathbf{m}_{\text{pix}} \odot (\hat{x}_0(\hat{\boldsymbol{\varepsilon}}) - x_0) \|_1}{\|\mathbf{m}_{\text{pix}}\|_1} \right]$$


The gradients of this loss function perform a gradient-through operation bypassing the frozen VAE decoder and UNet backbone, propagating precisely to the text embedding parameters of Stage-B. This explicitly guides the model to synthesize extraordinarily sharp defect boundaries that conform to physical geometries.






