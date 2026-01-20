# Attention

Transformer模型中的Q、K、V是自注意力机制的核心组成部分，下面我将通过一个具体示例详细解释它们的计算过程。

## 一、基本概念与计算流程

**Q（Query）**：查询向量，代表当前需要关注其他位置信息的"提问者"
 **K（Key）**：键向量，提供"匹配标签"，用于与查询向量比较
 **V（Value）**：值向量，承载"核心信息"，表示被关注token的实际内容

计算过程分为四步：**生成QKV → 计算注意力分数 → 归一化权重 → 加权求和**。

## 二、具体计算示例

假设输入序列为"我爱吃酸菜鱼"，经过词嵌入后得到5个词向量（为简化计算，假设每个词向量维度为4）：

**输入矩阵 X**（5×4）：

```markdown
[ [0.1, 0.2, 0.3, 0.4],  # "我"
  [0.5, 0.6, 0.7, 0.8],  # "爱"
  [0.9, 1.0, 1.1, 1.2],  # "吃"
  [1.3, 1.4, 1.5, 1.6],  # "酸"
  [1.7, 1.8, 1.9, 2.0] ] # "菜"
```

### 1. 生成Q、K、V向量

通过三个独立的线性变换矩阵 **Wq**、**Wk**、**Wv**（均为4×4）计算：

**计算公式**：

- Q = X × Wq
- K = X × Wk
- V = X × Wv

假设变换后得到：

- **Q**（5×4）：[[0.2, 0.3, 0.4, 0.5], [0.6, 0.7, 0.8, 0.9], [1.0, 1.1, 1.2, 1.3], [1.4, 1.5, 1.6, 1.7], [1.8, 1.9, 2.0, 2.1]]
- **K**（5×4）：[[0.1, 0.2, 0.3, 0.4], [0.5, 0.6, 0.7, 0.8], [0.9, 1.0, 1.1, 1.2], [1.3, 1.4, 1.5, 1.6], [1.7, 1.8, 1.9, 2.0]]
- **V**（5×4）：[[0.3, 0.4, 0.5, 0.6], [0.7, 0.8, 0.9, 1.0], [1.1, 1.2, 1.3, 1.4], [1.5, 1.6, 1.7, 1.8], [1.9, 2.0, 2.1, 2.2]]

> **关键点**：Q、K、V都来自**同一输入序列**的线性变换，但使用不同的权重矩阵。

### 2. 计算注意力分数

**计算公式**：注意力分数 = Q · KT / √dk

- **dk** 是K的维度（本例中为4），√dk ≈ 2
- **Q · KT**：5×4矩阵与4×5矩阵相乘，得到5×5的注意力分数矩阵

**计算过程**：

- "我"的Q向量与所有K向量点积：
  [0.2, 0.3, 0.4, 0.5] · [0.1, 0.2, 0.3, 0.4] = 0.4
  [0.2, 0.3, 0.4, 0.5] · [0.5, 0.6, 0.7, 0.8] = 1.2
  [0.2, 0.3, 0.4, 0.5] · [0.9, 1.0, 1.1, 1.2] = 2.0
  [0.2, 0.3, 0.4, 0.5] · [1.3, 1.4, 1.5, 1.6] = 2.8
  [0.2, 0.3, 0.4, 0.5] · [1.7, 1.8, 1.9, 2.0] = 3.6
- 将所有分数除以√dk（≈2）进行缩放，得到：[0.2, 0.6, 1.0, 1.4, 1.8]

> **为什么需要缩放**？防止点积过大导致Softmax梯度消失。

### 3. Softmax归一化

将缩放后的注意力分数通过Softmax函数转换为概率分布：

**计算公式**：α = Softmax(注意力分数)

- Softmax确保每行分数和为1
- 例如第一行分数[0.2, 0.6, 1.0, 1.4, 1.8] → 归一化后[0.08, 0.12, 0.18, 0.24, 0.38]

> **Softmax的作用**：将原始分数转换为可解释的注意力权重，保证非负性并增加非线性。

### 4. 加权求和得到输出

**计算公式**：Z = α × V

- 用归一化后的注意力权重α对V进行加权求和
- 例如"我"的输出向量：
  0.08×[0.3, 0.4, 0.5, 0.6] + 0.12×[0.7, 0.8, 0.9, 1.0] + 0.18×[1.1, 1.2, 1.3, 1.4] + 0.24×[1.5, 1.6, 1.7, 1.8] + 0.38×[1.9, 2.0, 2.1, 2.2] = [1.45, 1.55, 1.65, 1.75]

> **关键点**：输出向量Z既包含"我"自身的特征，又融合了其他词的上下文信息。

## 三、核心要素总结

1. **Q、K、V 的来源**：通过输入 X*X* 与三个不同的可学习权重矩阵相乘得到，分别表示**查询、键、值**。
2. **计算目的**：
   - Q*Q*：代表当前词要**查询什么信息**。
   - K*K*：代表每个词能**提供什么信息标识**。
   - V*V*：代表每个词**实际携带的信息内容**。
3. **注意力机制**：通过计算 Q*Q* 与 K*K* 的点积，得到每个 Query 对所有 Key 的**相关性分数**，再对 V*V* 加权求和，从而让每个词都能**聚合全局信息**。
4. **多头注意力**：在实际 Transformer 中，上述过程会并行进行多次（多个头），每个头使用不同的 *W**Q*,*W**K*,*W**V*，最后拼接所有头的输出并通过一个线性层融合。

这种设计让 Transformer 能够**动态地、有区分地**关注序列中不同位置的信息，从而有效捕捉长距离依赖关系。

# MLP

Transformer中**MLP（多层感知机）的核心作用是为模型提供非线性变换能力，对注意力机制融合后的上下文信息进行深度加工和语义提炼**，使模型能够学习更复杂的语言模式和知识表示。

## 一、MLP在Transformer中的关键作用

### 1. **提供非线性变换能力**

- 注意力机制本质上是**线性操作**（QK点积+加权求和），虽然能有效融合上下文信息，但无法捕捉复杂的语义关系
- MLP通过**线性变换+激活函数（如GELU/ReLU）**引入非线性，使模型能够学习更复杂的函数关系
- **没有MLP，Transformer将退化为纯线性模型**，表达能力大幅下降，无法处理复杂的语言任务

### 2. **对每个token进行独立语义加工**

- 经过注意力机制后，每个token的向量已包含全局上下文信息（如"吃"知道主语是"猫"）
- MLP作为**position-wise（逐位置）的前馈网络**，对每个token的向量进行独立处理：
  - **过滤无关噪声**，保留关键语义信息
  - **放大重要特征**，增强语义表示
  - **映射到更适合预测的语义空间**
- 这种处理使模型能够**更精准地预测下一个词**，如将"Michael Jordan"的向量映射到"basketball"方向

### 3. **存储和应用语言知识**

- 研究表明，**MLP层是模型存储大量"知识"和"事实"的主要场所**（如记住"刘德华"这个实体）
- 在GPT-3等大模型中，**MLP层占据了约2/3的参数量**，意味着模型的大部分"知识"和"推理能力"藏在MLP里
- MLP通过高维向量空间的非线性变换，能够**同时存储多种语义关系**（如性别、地域、职业等）

### 4. **增强模型的表达能力**

- MLP通过**扩大-压缩**的特征处理流程（通常将维度扩大4倍再压缩回原尺寸）
- 这种设计使模型能够在**高维空间中学习更复杂的特征表示**，同时保持输入输出维度的一致性
- 实验表明，当隐藏维度低于输入维度3倍时，模型难以学习复杂模式；而超过5倍则会导致过拟合

## 二、MLP与注意力机制的协同工作

Transformer模型的工作流程可概括为：**Embedding → Attention → MLP → Unembedding**

1. **注意力机制**：负责"看懂上下文"，让每个token获取其他token的信息
2. **MLP层**：负责"想明白意思"，对融合后的信息进行深度加工
3. **残差连接**：确保信息在多层网络中稳定传递，防止梯度消失

> 💡 **形象比喻**：如果将Transformer比作一个写作团队，注意力机制是负责收集资料的研究员，而MLP则是负责整理资料、提炼观点并撰写文章的作家。研究员收集的信息再全面，也需要作家进行深度思考和创作才能产出有价值的内容。

## 三、MLP的结构特点

Transformer中的MLP通常包含：

- **两个线性变换层**：第一个将输入维度扩大（通常扩大4倍），第二个将维度还原
- **一个激活函数**：如GELU或ReLU，提供非线性
- **残差连接**：将MLP输出与原始输入相加，保持信息完整性
- **层归一化**：确保训练稳定性

数学表达式为：FFN(x) = max(0, xW₁ + b₁)W₂ + b₂ + x（含残差连接）

## 四、实验验证

最新研究表明，**MLP在Transformer中不可或缺**：

- 去掉MLP层会导致模型性能显著下降，甚至无法正常工作
- 在某些场景下，精心设计的MLP可以**部分替代注意力机制**，但完全去除MLP会导致模型崩溃
- 在时序预测任务中，纯MLP模型（如TimeMixer）在某些场景下能**超越Transformer**，证明了MLP的强大潜力

总之，**注意力机制负责"理解上下文"，而MLP负责"深度思考和表达"**。两者协同工作，使Transformer模型能够处理复杂的语言理解和生成任务。

# Ollama

```bash
[root@server01 deepseek]#docker run -itd --name=ollama -v /software/deepseek:/root/.ollama -p 1234:11434 ollama/ollama:latest
[root@server01 deepseek]# docker exec -it 9b1bd1dc1a0a /bin/bash
root@9b1bd1dc1a0a:/# ollama pull deepseek-r1:32b
```

# 环境

## GPU驱动

```markdown
https://www.nvidia.cn/drivers/lookup/

Data Center / Tesla
A-Series
NVIDIA A100
Linux 64-bit
12.4
Chinese (Simplified)
```

## Docker

- buildkit

  ```bash
  # 二级制安装的docker，在打镜像的时候如果想用--mount=type=cache功能的话，则必须进行以下设置
  # https://github.com/docker/buildx?tab=readme-ov-file
  [root@server02 vllm]# cp buildx-v0.23.0.linux-amd64  /root/.docker/cli-plugins/
  [root@server02 vllm]# cd /root/..docker/cli-plugins/
  [root@server02 vllm]# mv buildx-v0.23.0.linux-amd64 docker-buildx
  [root@server02 vllm]# chmod +x /root/.docker/cli-plugins/docker-buildx
  [root@server02 vllm]# systemctl restart docker 
  [root@server02 vllm]# vi /etc/docker/daemon.json
  {
    "features": {
      "buildkit": true
    }
  }
  ```

- 配置生产仓库

  ```bash
  # https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html
  #curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -#o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  #  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-#container-toolkit.list | \
  #    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-#keyring.gpg] https://#g' | \
  #    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
  
  root@763ac07c2af3:/vllm-workspace# curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  >   && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  >     sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  >     sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
  ```

  

- 离线下载 nvidia-container-toolkit 组件

  ```bash
  #apt-get install --reinstall --download-only
  #功能
  #下载指定包及其所有依赖项，但不会安装或重新安装。
  #--reinstall 表示强制重新下载（即使本地已安装或存在缓存）。
  #--download-only 表示仅下载，不执行安装。
  
  #特点
  #递归下载所有依赖：会自动下载目标包及其依赖的所有包（包括间接依赖）。
  #需要 root 权限：必须使用 sudo 执行。
  #保存路径：下载的 .deb 文件默认存储在 /var/cache/apt/archives/。
  #适用场景：为离线安装准备完整的包和依赖项。
  
  #apt-cache depends
  #分步解析
  #核心命令：查询包的依赖关系。
  #--recurse：递归查询所有层级的依赖（直接依赖、间接依赖的依赖等）。
  #
  #过滤无关信息
  #--no-recommends：排除“推荐”安装的包（非必需）。
  #--no-suggests：排除“建议”安装的包（非必需）。
  #--no-conflicts：忽略冲突包（避免干扰）。
  #--no-breaks：忽略破坏性依赖（避免干扰）。
  #--no-replaces：忽略替换性依赖（避免干扰）。
  #--no-enhances：忽略增强性依赖（非必需）。
  #
  #目标包名
  #nvidia-container-toolkit：要分析的包。
  #
  #grep "^\w"
  #过滤输出：仅保留以字母/数字（即依赖包名）开头的行。
  #例如，过滤掉 Depends:、PreDepends: 等描述性行。
  #
  #sort -u
  #排序并去重：确保依赖包列表唯一且有序。
  
  root@763ac07c2af3:/vllm-workspace# apt-get install --reinstall --download-only $(apt-cache depends --recurse --no-recommends --no-suggests --no-conflicts --no-breaks --no-replaces --no-enhances nvidia-container-toolkit | grep "^\w" | sort -u)
  ```

  ```bash
  # centos
  [root@server01 ~]# curl -s -L https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo |   sudo tee /etc/yum.repos.d/nvidia-container-toolkit.repo
  [root@server01 ~]# yumdownloader --resolve --destdir=/software/nvidia-container-toolkit  nvidia-container-toolkit
  ```

- 离线安装nvidia-container-toolkit 

  ```bash
  root@763ac07c2af3:/vllm-workspace# dpkg -i /home/nfs-pkgs/*.deb
  ```

  ```bash
  [root@server01 ~]# rpm -ivh *
  ```

- docker 配置使用 nvidia-runtime

  新版本

  ```bash
  [root@server01 deepseek]# nvidia-ctk runtime configure --runtime=docker
  [root@server01 deepseek]# systemctl restart docker
  ```

  老版本

  需要手动在 /etc/docker/daemon.json 中增加配置，指定使用 nvidia 的 runtime。

  ```json
  "default-runtime": "nvidia",
  "runtimes": {
      "nvidia": {
          "path": "/usr/bin/nvidia-container-runtime",
          "runtimeArgs": []
      }
  }
  ```

- 错误解析

  ```bash
  # 1、安装驱动时提示：WARNING: The Nouveau kernel driver is currently in use by your system.  This driver is incompatible with the NVIDIA driver，and must be disabled before proceeding.（警告：您的系统当前正在使用Nouveau内核驱动程序。此驱动程序与NVIDIA驱动程序不兼容，必须先禁用才能继续。）
  #在 CentOS 系统上，如果你想要禁用 Nouveau 驱动（这是一个开源的 NVIDIA 驱动，通常用于 Nouveau 显卡），你可以通过以下几种方法来实现：
  # 1.1 临时禁用：
  [root@server01 deepseek]# lsmod | grep nouveau
  [root@server01 deepseek]# rmmod nouveau
  [root@server01 deepseek]# lsmod | grep nouveau
  # 1.2 使用黑名单模块
  [root@server01 deepseek]# vim /etc/modprobe.d/blacklist.conf
  blacklist nouveau
  options nouveau modeset=0
  [root@server01 deepseek]# dracut --force
  [root@server01 deepseek]# reboot
  [root@server01 deepseek]# lsmod | grep nouveau
  # 2、在安装完驱动、docker以及nvidia-container-toolkit后，用docker创建容器时出现如下报错,原因是gpu的持久模式(nvidia-persistenced daemon)并未开启
  [root@server01 deepseek]# docker: Error response from daemon: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: error during container init: error running hook #0: error running hook: exit status 1, stdout: , stderr: Auto-detected mode as 'legacy' nvidia-container-cli: initialization error: driver rpc error: timed out: unknown.
  # 可以用nvidia-smi -a查询自己的 Persistence Mode 是否开启、同时也可以用nvidia-smi
  # 解决方案：使用root权限执行如下命令：
  [root@server01 deepseek]# nvidia-smi -pm ENABLED
  
  # 3、nvidia-docker Failed to initialize NVML: Unknown Error
  [root@server01 deepseek]# docker run --rm --runtime=nvidia --gpus all centos nvidia-smi
  [root@server01 deepseek]# Failed to initialize NVML: Unknown Error
  # 解决方案：
  [root@server01 deepseek]# vim /etc/nvidia-container-runtime/config.toml
  # 设置 no-cgroups = false，然后保存
  [root@server01 deepseek]# systemctl restart docker
  ```

- 启动容器时增加 --gpu 参数



# 传统大模型训练流程

## Pre training

预训练：Base model

预训练利用大量无标签或弱标签的数据，通过某种算法模型进行训练，得到一个初步具备通用知识或能力的模型。

## Supervised fine tuning

监督式微调：Instruct model

尽管预训练模型已经在大规模数据集上学到了丰富的通用特征和先验知识，但这些特征和知识可能并不完全适用于特定的目标任务。微调通过在新任务的少量标注数据上进一步训练预训练模型，使模型能够学习到与目标任务相关的特定特征和规律，从而更好地适应新任务。

## Preference alignment

偏好对齐：chat model

在很多应用场景下有监督微调就已经够用了，但对于一些面向用户的公众模型，偏好对齐还是很有必要的（不然模型说了什么不该说的话可能这个产品甚至公司都要完蛋）。比较经典的偏好对齐的做法就是基于人类反馈的强化学习（Reinforcement Learning from Human Feedback, RLHF）那一套，根据人类偏好/反馈数据训练一个“奖励模型”，并使用该模型作为强化学习中的奖励函数，再通过类似PPO之类的强化学习算法来优化大语言模型的输出。不过对于大多数非公司级的大语言模型来说，不愿意折腾RLHF，毕竟偏好数据不好收集、还要额外训一个奖励模型、还要搞训练不稳定的强化学习。这些成本都是很高的。

## TRL

TRL（Transformer Reinforcement Learning）训练的三个阶段：监督微调（SFT）、奖励模型训练（Reward Modeling）和强化学习微调（PPO）。这三个阶段是逐步递进的关系，共同完成基于人类反馈的强化学习（RLHF）过程。

|        阶段         |                             目标                             |            输入            |       输出       |      所需模型/组件       |
| :-----------------: | :----------------------------------------------------------: | :------------------------: | :--------------: | :----------------------: |
|   监督微调 (SFT)    | 让预训练模型适应特定任务或领域，使其能根据给定的指令生成合理的内容 |      指令-输出对数据       | 微调后的基础模型 |       基础语言模型       |
|  奖励模型训练 (RM)  | 将人类偏好转化为可量化的奖励信号，使强化学习可行（人类无法实时反馈） | 成对的候选回答（人类偏好） |     奖励模型     |    SFT 模型 + RM 模型    |
| 强化学习微调 (RLHF) | 在RM的指导下，通过试错不断改进策略，生成更符合人类偏好的回答。 |       提示（prompt）       |  最终 RLHF 模型  | RM 模型 + PPO/DPO 等算法 |

1. 监督微调（Supervised Fine-Tuning, SFT）

   使用高质量的标注数据（通常是人工编写的回答）对预训练语言模型进行微调，使模型初步具备完成特定任务（如对话、问答）的能力。SFT是RLHF的起点。它为后续的奖励模型训练和PPO提供了一个基础模型。没有SFT，直接进行强化学习微调可能会因为模型行为过于随机而难以训练。

   例子：假设我们想训练一个帮助写代码的AI助手。我们收集了这样的数据：
   输入（Prompt）: "用Python写一个快速排序函数。"
   输出（Response）: "def quicksort(arr): ... " （人工编写的代码）
   用这些数据微调预训练模型（如Llama-3），使模型学会如何根据问题生成代码。

2. 奖励模型训练（Reward Modeling）

   训练一个能够根据人类偏好对模型生成的回答进行打分的模型。这个奖励模型将替代人类，在强化学习阶段提供即时反馈。奖励模型是连接人类偏好和强化学习的桥梁。它通过学习人类对多个回答的偏好（例如，哪个回答更好），从而能够为任何生成的回答打分。这个奖励模型将在PPO阶段作为评判标准，指导策略模型（即我们要优化的模型）的学习方向。

   例子：继续代码助手的例子；我们收集偏好数据：对于同一个问题（Prompt），我们让基础模型（SFT后的模型）生成多个回答，然后让人工标注哪个回答更好。例如：
   Prompt: "用Python写一个快速排序函数。"
   回答A: 一个正确但效率不高的实现（冒泡排序）
   回答B: 一个正确且高效的快速排序实现
   人工标注：回答B更好。
   奖励模型训练的目标是学习一个函数：RM(Prompt, Response) → 分数（标量）。它要能够学会：对于同一个Prompt，给回答B的打分高于回答A。

3. 强化学习微调（PPO）

   使用强化学习（具体是PPO算法）优化策略模型（即SFT后的模型），使其生成的回答在奖励模型看来得分更高，从而更符合人类偏好。PPO阶段将SFT模型作为初始策略，同时固定奖励模型作为环境反馈。通过不断生成回答、获得奖励、更新策略，模型逐步学习生成更高质量的回答。为了防止模型过度优化（只迎合奖励模型而偏离正常语言能力），我们通常保留一个参考模型（即未经过PPO训练的SFT模型），通过KL散度惩罚来约束策略模型不要偏离参考模型太远。

   例子：继续代码助手。
   我们有一个Prompt: "用Python写一个快速排序函数。"
   PPO训练步骤：
   1. 策略模型根据这个Prompt生成一个回答（比如，生成了一个正确的快速排序函数，但变量命名不规范）。
   2. 我们将这个回答输入奖励模型，得到一个分数（比如7分）。
   3. 同时，参考模型（SFT模型）也会生成一个回答（比如，变量命名规范但算法效率一般）。
   4. 计算策略模型生成回答的奖励（包括奖励模型分数和KL惩罚，例如：最终奖励 = 7 - β * KL(策略模型||参考模型)）。
   5. 使用PPO算法更新策略模型，使得它未来更可能生成高奖励的回答（即同时满足：高质量代码、且不过于偏离参考模型）。

监督微调(SFT) → 使用高质量的指令数据微调一个预训练的语言模型；奖励模型训练(RM) ← 使用人类标注的偏好数据（如两个回答之间选择更好的一个）训练一个奖励模型；强化学习微调(PPO) → 使用近端策略优化（PPO）等算法，利用奖励模型来优化SFT模型。

```mermaid
graph LR
A[预训练大模型] --> B(监督微调 SFT)
B --> C{生成响应样本}
C --> D[人工偏好标注]
D --> E(奖励模型训练 RM)
E --> F(强化学习微调 PPO)
F --> G[优化后的安全模型]
```

- SFT 就像学生学习课本知识，知道该怎么回答问题；
- RM 就像老师评分系统，知道什么样的答案更好；
- RLHF 就是学生根据老师的反馈不断改进答题能力，直到得到高分。

# PEFT

Parameter-Efficient Fine-Tuning

# training

```python
# Step1 导入相关模块
from datasets import load_dataset
from transformers import AutoTokenizer, AutoModelForCausalLM, DataCollatorForSeq2Seq, TrainingArguments, Trainer

import warnings
warnings.filterwarnings('ignore')

#import datasets
#datasets.__version__
#'3.6.0'
#import transformers
#transformers.__version
#'4.51.1'
```

## DeepSpeed VS Megatron-LM

DeepSpeed和Megatron-LM都是用于大规模模型训练的框架，但它们的侧重点和功能有所不同。

### 1. 背景
- **DeepSpeed**:
- 由Microsoft开发，是一个专注于**优化训练过程**的库，特别是通过减少显存占用和加速通信来提高训练效率。
- 核心创新：ZeRO（Zero Redundancy Optimizer）技术，通过消除数据并行中的冗余状态（参数、梯度、优化器状态）来节省显存。
- 支持多种训练优化技术，如混合精度训练、梯度累积、模型并行（与Megatron-LM集成）、CPU/NVMe offload等。
- 目标：让大规模模型训练更加高效和可扩展，同时降低硬件门槛。
- **Megatron-LM**:
- 由NVIDIA开发，是一个专注于**模型并行**（特别是Tensor Parallelism和Pipeline Parallelism）的框架。
- 提供了高效的模型并行实现，尤其是针对Transformer层内的张量切分（Tensor Parallelism）和模型层的流水线切分（Pipeline Parallelism）。
- 目标：通过模型并行技术解决单个设备无法容纳超大模型的问题，并利用高效的并行计算来加速训练。
### 2. 核心技术
- **DeepSpeed的核心技术**:
- **ZeRO**：分为三个阶段（Stage 1, 2, 3），通过在不同程度上分割优化器状态、梯度和参数来减少显存占用。
- **Offload**：将优化器状态、梯度或参数卸载到CPU或NVMe，进一步节省GPU显存。
- **3D并行**：集成了数据并行（ZeRO）、模型并行（Tensor Parallelism，需要与Megatron-LM集成）和流水线并行（Pipeline Parallelism，通过DeepSpeed的PipelineEngine）。
- **其他优化**：如激活检查点（activation checkpointing）、通信优化（如压缩梯度）、大规模优化器（如AdamW、LAMB）等。
- **Megatron-LM的核心技术**:
- **Tensor Parallelism (TP)**：将Transformer层的矩阵运算（如线性层、自注意力）切分到多个GPU上，实现模型层内的并行。
- **Pipeline Parallelism (PP)**：将模型按层切分到多个GPU上，形成流水线，不同GPU处理不同的层。
- **Sequence Parallelism**：进一步将序列维度切分，用于处理长序列。
- **高效的CUDA核**：针对并行计算优化了Transformer层的实现。
### 3. 使用方式
- **DeepSpeed**:
- 可以作为PyTorch的一个扩展库使用，通过提供配置文件和简单的API集成到现有训练脚本中。
- 通常与Hugging Face Transformers等库结合使用，只需修改少量代码即可启用ZeRO、Offload等功能。
- 对于模型并行，需要与Megatron-LM集成（或使用DeepSpeed自带的实验性模型并行功能）。
- **Megatron-LM**:
- 是一个独立的训练框架，需要按照其特定的方式构建模型和训练流程。
- 提供了自己的模型实现（如GPT-2、GPT-3、BERT等），用户需要基于这些实现来构建模型。
- 通常需要更深入的理解来配置并行策略（TP、PP等）。
### 4. 集成与协作
- **DeepSpeed和Megatron-LM的集成**:
- 两者可以结合使用，形成“3D并行”（数据并行、张量并行、流水线并行）的解决方案。
- 在这种集成中，DeepSpeed负责数据并行（通过ZeRO）和可能的Offload，而Megatron-LM负责模型内部的张量并行和层间的流水线并行。
- 例如，在训练千亿级模型时，通常会同时使用这两种技术。
### 5. 适用场景
- **DeepSpeed**:
- 当你希望使用更少的资源（如显存）训练大模型时。
- 当你已经有一个PyTorch模型，并希望以最小的改动来加速训练和减少显存占用。
- 需要Offload到CPU/NVMe的场景。
- **Megatron-LM**:
- 当模型大到单个GPU无法容纳一层（如百亿、千亿参数模型）时，必须使用模型并行。
- 当你需要最先进的模型并行实现（特别是Tensor Parallelism）以获得最佳性能时。
- 当训练超大规模模型且硬件资源充足（如多节点多GPU）时。

# 分词

| 维度     | 词级分词算法（FMM/HMM/BiLSTM+CRF）      | 子词级分词算法（BPE/Wordpiece/Unigram）           |
| -------- | --------------------------------------- | ------------------------------------------------- |
| 核心目标 | 把文本切分为**具有独立语义的词单元**    | 构建**子词词汇表**，解决 OOV 和词汇表膨胀问题     |
| 处理对象 | 中文：连续汉字序列；英文：连续字母序列  | 英文：单词；中文：字 / 词（因中文无天然单词边界） |
| 应用阶段 | NLP 下游任务前置步骤（如文本分类、NER） | 预训练语言模型训练前的词汇表构建阶段              |
| 典型产出 | 分词结果：`信用卡/账单/逾期`            | 子词拆分结果：`unhappiness`→`un+happy+ness`       |

子词级分词算法的本质与应用场景

这类算法诞生的核心诉求是**解决预训练语言模型的两大痛点**：

1. **OOV（未登录词）问题**：英文中存在大量生僻词、专业术语（如金融领域的`cryptocurrency`），直接纳入词汇表会导致词汇表无限膨胀；
2. **词汇表效率问题**：如果词汇表只包含完整单词，会出现大量重复字符组合，浪费存储和计算资源。

子词级分词算法的核心逻辑是：**将单词拆分为更小的子词单元，让模型学习子词的语义，从而覆盖更多未见过的词**。

这类算法的核心是 “切出有语义的词”；为NLP下游任务服务

# tokenizer

编码过程：

大模型采用了这样一个映射表

```python
def bytes_to_unicode():
    """
    生成字节到Unicode字符的正向映射表
    返回字典：{byte_value: unicode_char}
    """
    # 原始保留的字节范围
    bs = (
        list(range(ord("!"), ord("~") + 1)) +          # ASCII可打印字符（33-126）
        list(range(ord("¡"), ord("¬") + 1)) +          # 西班牙语特殊字符（161-172）
        list(range(ord("®"), ord("ÿ") + 1))            # 其他扩展字符（174-255）
    )
    
    cs = bs.copy()  # 初始字符列表
    n = 0
    
    # 遍历所有可能的字节（0-255）
    for b in range(2**8):
        if b not in bs:
            bs.append(b)
            cs.append(2**8 + n)  # 超出原始范围的字节映射到更高Unicode码位
            n += 1
    
    # 将码位转换为Unicode字符
    cs = [chr(code) for code in cs]
    
    return dict(zip(bs, cs))
```

对照表（如果有问题可以按照上述代码生成一个映射表查看）

| DEC(十进制) | OCT(八进制) | HEX(十六进制) | BIN(二进制) | 映射字符 |
| ----------- | ----------- | ------------- | ----------- | -------- |
| 0           | 000         | 00            | 00000000    | Ā        |
| 1           | 001         | 01            | 00000001    | ā        |
| 2           | 002         | 02            | 00000010    | Ă        |
| 3           | 003         | 03            | 00000011    | ă        |
| 4           | 004         | 04            | 00000100    | Ą        |
| 5           | 005         | 05            | 00000101    | ą        |
| 6           | 006         | 06            | 00000110    | Ć        |
| 7           | 007         | 07            | 00000111    | ć        |
| 8           | 010         | 08            | 00001000    | Ĉ        |
| 9           | 011         | 09            | 00001001    | ĉ        |
| 10          | 012         | 0A            | 00001010    | Ċ        |
| 11          | 013         | 0B            | 00001011    | ċ        |
| 12          | 014         | 0C            | 00001100    | Č        |
| 13          | 015         | 0D            | 00001101    | č        |
| 14          | 016         | 0E            | 00001110    | Ď        |
| 15          | 017         | 0F            | 00001111    | ď        |
| 16          | 020         | 10            | 00010000    | Đ        |
| 17          | 021         | 11            | 00010001    | đ        |
| 18          | 022         | 12            | 00010010    | Ē        |
| 19          | 023         | 13            | 00010011    | ē        |
| 20          | 024         | 14            | 00010100    | Ĕ        |
| 21          | 025         | 15            | 00010101    | ĕ        |
| 22          | 026         | 16            | 00010110    | Ė        |
| 23          | 027         | 17            | 00010111    | Ė        |
| 24          | 030         | 18            | 00011000    | ė        |
| 25          | 031         | 19            | 00011001    | Ę        |
| 26          | 032         | 1A            | 00011010    | ę        |
| 27          | 033         | 1B            | 00011011    | ě        |
| 28          | 034         | 1C            | 00011100    | Ĝ        |
| 29          | 035         | 1D            | 00011101    | ĝ        |
| 30          | 036         | 1E            | 00011110    | Ğ        |
| 31          | 037         | 1F            | 00011111    | ğ        |
| 32          | 040         | 20            | 00100000    | Ġ        |
| 33          | 041         | 21            | 00100001    | !        |
| 34          | 042         | 22            | 00100010    | \        |
| 35          | 043         | 23            | 00100011    | #        |
| 36          | 044         | 24            | 00100100    | $        |
| 37          | 045         | 25            | 00100101    | %        |
| 38          | 046         | 26            | 00100110    | &        |
| 39          | 047         | 27            | 00100111    | ’        |
| 40          | 050         | 28            | 00101000    | (        |
| 41          | 051         | 29            | 00101001    | )        |
| 42          | 052         | 2A            | 00101010    | *        |
| 43          | 053         | 2B            | 00101011    | +        |
| 44          | 054         | 2C            | 00101100    | ,        |
| 45          | 055         | 2D            | 00101101    | -        |
| 46          | 056         | 2E            | 00101110    | .        |
| 47          | 057         | 2F            | 00101111    | /        |
| 48          | 060         | 30            | 00110000    | 0        |
| 49          | 061         | 31            | 00110001    | 1        |
| 50          | 062         | 32            | 00110010    | 2        |
| 51          | 063         | 33            | 00110011    | 3        |
| 52          | 064         | 34            | 00110100    | 4        |
| 53          | 065         | 35            | 00110101    | 5        |
| 54          | 066         | 36            | 00110110    | 6        |
| 55          | 067         | 37            | 00110111    | 7        |
| 56          | 070         | 38            | 00111000    | 8        |
| 57          | 071         | 39            | 00111001    | 9        |
| 58          | 072         | 3A            | 00111010    | :        |
| 59          | 073         | 3B            | 00111011    | ;        |
| 60          | 074         | 3C            | 00111100    | <        |
| 61          | 075         | 3D            | 00111101    | =        |
| 62          | 076         | 3E            | 00111110    | >        |
| 63          | 077         | 3F            | 00111111    | ?        |
| 64          | 100         | 40            | 01000000    | @        |
| 65          | 101         | 41            | 01000001    | A        |
| 66          | 102         | 42            | 01000010    | B        |
| 67          | 103         | 43            | 01000011    | C        |
| 68          | 104         | 44            | 01000100    | D        |
| 69          | 105         | 45            | 01000101    | E        |
| 70          | 106         | 46            | 01000110    | F        |
| 71          | 107         | 47            | 01000111    | G        |
| 72          | 110         | 48            | 01001000    | H        |
| 73          | 111         | 49            | 01001001    | I        |
| 74          | 112         | 4A            | 01001010    | J        |
| 75          | 113         | 4B            | 01001011    | K        |
| 76          | 114         | 4C            | 01001100    | L        |
| 77          | 115         | 4D            | 01001101    | M        |
| 78          | 116         | 4E            | 01001110    | N        |
| 79          | 117         | 4F            | 01001111    | O        |
| 80          | 120         | 50            | 01010000    | P        |
| 81          | 121         | 51            | 01010001    | Q        |
| 82          | 122         | 52            | 01010010    | R        |
| 83          | 123         | 53            | 01010011    | S        |
| 84          | 124         | 54            | 01010100    | T        |
| 85          | 125         | 55            | 01010101    | U        |
| 86          | 126         | 56            | 01010110    | V        |
| 87          | 127         | 57            | 01010111    | W        |
| 88          | 130         | 58            | 01011000    | X        |
| 89          | 131         | 59            | 01011001    | Y        |
| 90          | 132         | 5A            | 01011010    | Z        |
| 91          | 133         | 5B            | 01011011    | [        |
| 92          | 134         | 5C            | 01011100    | \\\\     |
| 93          | 135         | 5D            | 01011101    | ]        |
| 94          | 136         | 5E            | 01011110    | ^        |
| 95          | 137         | 5F            | 01011111    | _        |
| 96          | 140         | 60            | 01100000    | `        |
| 97          | 141         | 61            | 01100001    | a        |
| 98          | 142         | 62            | 01100010    | b        |
| 99          | 143         | 63            | 01100011    | c        |
| 100         | 144         | 64            | 01100100    | d        |
| 101         | 145         | 65            | 01100101    | e        |
| 102         | 146         | 66            | 01100110    | f        |
| 103         | 147         | 67            | 01100111    | g        |
| 104         | 150         | 68            | 01101000    | h        |
| 105         | 151         | 69            | 01101001    | i        |
| 106         | 152         | 6A            | 01101010    | j        |
| 107         | 153         | 6B            | 01101011    | k        |
| 108         | 154         | 6C            | 01101100    | l        |
| 109         | 155         | 6D            | 01101101    | m        |
| 110         | 156         | 6E            | 01101110    | n        |
| 111         | 157         | 6F            | 01101111    | o        |
| 112         | 160         | 70            | 01110000    | p        |
| 113         | 161         | 71            | 01110001    | q        |
| 114         | 162         | 72            | 01110010    | r        |
| 115         | 163         | 73            | 01110011    | s        |
| 116         | 164         | 74            | 01110100    | t        |
| 117         | 165         | 75            | 01110101    | u        |
| 118         | 166         | 76            | 01110110    | v        |
| 119         | 167         | 77            | 01110111    | w        |
| 120         | 170         | 78            | 01111000    | x        |
| 121         | 171         | 79            | 01111001    | y        |
| 122         | 172         | 7A            | 01111010    | z        |
| 123         | 173         | 7B            | 01111011    | {        |
| 124         | 174         | 7C            | 01111100    | \|       |
| 125         | 175         | 7D            | 01111101    | }        |
| 126         | 176         | 7E            | 01111110    | ~        |
| 127         | 177         | 7F            | 01111111    | ġ        |
| 128         | 200         | 80            | 10000000    | Ģ        |
| 129         | 201         | 81            | 10000001    | ģ        |
| 130         | 202         | 82            | 10000010    | Ĥ        |
| 131         | 203         | 83            | 10000011    | ĥ        |
| 132         | 204         | 84            | 10000100    | Ħ        |
| 133         | 205         | 85            | 10000101    | ħ        |
| 134         | 206         | 86            | 10000110    | Ĩ        |
| 135         | 207         | 87            | 10000111    | ĩ        |
| 136         | 210         | 88            | 10001000    | Ī        |
| 137         | 211         | 89            | 10001001    | ī        |
| 138         | 212         | 8A            | 10001010    | Ĭ        |
| 139         | 213         | 8B            | 10001011    | ĭ        |
| 140         | 214         | 8C            | 10001100    | Į        |
| 141         | 215         | 8D            | 10001101    | į        |
| 142         | 216         | 8E            | 10001110    | İ        |
| 143         | 217         | 8F            | 10001111    | ı        |
| 144         | 220         | 90            | 10010000    | Ĳ        |
| 145         | 221         | 91            | 10010001    | ĳ        |
| 146         | 222         | 92            | 10010010    | Ĵ        |
| 147         | 223         | 93            | 10010011    | ĵ        |
| 148         | 224         | 94            | 10010100    | Ķ        |
| 149         | 225         | 95            | 10010101    | ķ        |
| 150         | 226         | 96            | 10010110    | ĸ        |
| 151         | 227         | 97            | 10010111    | Ĺ        |
| 152         | 230         | 98            | 10011000    | ĺ        |
| 153         | 231         | 99            | 10011001    | Ļ        |
| 154         | 232         | 9A            | 10011010    | ļ        |
| 155         | 233         | 9B            | 10011011    | Ľ        |
| 156         | 234         | 9C            | 10011100    | ľ        |
| 157         | 235         | 9D            | 10011101    | Ŀ        |
| 158         | 236         | 9E            | 10011110    | ŀ        |
| 159         | 237         | 9F            | 10011111    | Ł        |
| 160         | 240         | A0            | 10100000    | ł        |
| 161         | 241         | A1            | 10100001    | ¡        |
| 162         | 242         | A2            | 10100010    | ¢        |
| 163         | 243         | A3            | 10100011    | £        |
| 164         | 244         | A4            | 10100100    | ¤        |
| 165         | 245         | A5            | 10100101    | ¥        |
| 166         | 246         | A6            | 10100110    | ¦        |
| 167         | 247         | A7            | 10100111    | §        |
| 168         | 250         | A8            | 10101000    | ¨        |
| 169         | 251         | A9            | 10101001    | ©        |
| 170         | 252         | AA            | 10101010    | ª        |
| 171         | 253         | AB            | 10101011    | «        |
| 172         | 254         | AC            | 10101100    | ¬        |
| 173         | 255         | AD            | 10101101    | Ń        |
| 174         | 256         | AE            | 10101110    | ®        |
| 175         | 257         | AF            | 10101111    | ¯        |
| 176         | 260         | B0            | 10110000    | °        |
| 177         | 261         | B1            | 10110001    | ±        |
| 178         | 262         | B2            | 10110010    | ²        |
| 179         | 263         | B3            | 10110011    | ³        |
| 180         | 264         | B4            | 10110100    | ´        |
| 181         | 265         | B5            | 10110101    | µ        |
| 182         | 266         | B6            | 10110110    | ¶        |
| 183         | 267         | B7            | 10110111    | ·        |
| 184         | 270         | B8            | 10111000    | ¸        |
| 185         | 271         | B9            | 10111001    | ¹        |
| 186         | 272         | BA            | 10111010    | º        |
| 187         | 273         | BB            | 10111011    | »        |
| 188         | 274         | BC            | 10111100    | ¼        |
| 189         | 275         | BD            | 10111101    | ½        |
| 190         | 276         | BE            | 10111110    | ¾        |
| 191         | 277         | BF            | 10111111    | ¿        |
| 192         | 300         | C0            | 11000000    | À        |
| 193         | 301         | C1            | 11000001    | Á        |
| 194         | 302         | C2            | 11000010    | Â        |
| 195         | 303         | C3            | 11000011    | Ã        |
| 196         | 304         | C4            | 11000100    | Ä        |
| 197         | 305         | C5            | 11000101    | Å        |
| 198         | 306         | C6            | 11000110    | Æ        |
| 199         | 307         | C7            | 11000111    | Ç        |
| 200         | 310         | C8            | 11001000    | È        |
| 201         | 311         | C9            | 11001001    | É        |
| 202         | 312         | CA            | 11001010    | Ê        |
| 203         | 313         | CB            | 11001011    | Ë        |
| 204         | 314         | CC            | 11001100    | Ì        |
| 205         | 315         | CD            | 11001101    | Í        |
| 206         | 316         | CE            | 11001110    | Î        |
| 207         | 317         | CF            | 11001111    | Ï        |
| 208         | 320         | D0            | 11010000    | Ð        |
| 209         | 321         | D1            | 11010001    | Ñ        |
| 210         | 322         | D2            | 11010010    | Ò        |
| 211         | 323         | D3            | 11010011    | Ó        |
| 212         | 324         | D4            | 11010100    | Ô        |
| 213         | 325         | D5            | 11010101    | Õ        |
| 214         | 326         | D6            | 11010110    | Ö        |
| 215         | 327         | D7            | 11010111    | ×        |
| 216         | 330         | D8            | 11011000    | Ø        |
| 217         | 331         | D9            | 11011001    | Ù        |
| 218         | 332         | DA            | 11011010    | Ú        |
| 219         | 333         | DB            | 11011011    | Û        |
| 220         | 334         | DC            | 11011100    | Ü        |
| 221         | 335         | DD            | 11011101    | Ý        |
| 222         | 336         | DE            | 11011110    | Þ        |
| 223         | 337         | DF            | 11011111    | ß        |
| 224         | 340         | E0            | 11100000    | à        |
| 225         | 341         | E1            | 11100001    | á        |
| 226         | 342         | E2            | 11100010    | â        |
| 227         | 343         | E3            | 11100011    | ã        |
| 228         | 344         | E4            | 11100100    | ä        |
| 229         | 345         | E5            | 11100101    | å        |
| 230         | 346         | E6            | 11100110    | æ        |
| 231         | 347         | E7            | 11100111    | ç        |
| 232         | 350         | E8            | 11101000    | è        |
| 233         | 351         | E9            | 11101001    | é        |
| 234         | 352         | EA            | 11101010    | ê        |
| 235         | 353         | EB            | 11101011    | ë        |
| 236         | 354         | EC            | 11101100    | ì        |
| 237         | 355         | ED            | 11101101    | í        |
| 238         | 356         | EE            | 11101110    | î        |
| 239         | 357         | EF            | 11101111    | ï        |
| 240         | 360         | F0            | 11110000    | ð        |
| 241         | 361         | F1            | 11110001    | ñ        |
| 242         | 362         | F2            | 11110010    | ò        |
| 243         | 363         | F3            | 11110011    | ó        |
| 244         | 364         | F4            | 11110100    | ô        |
| 245         | 365         | F5            | 11110101    | õ        |
| 246         | 366         | F6            | 11110110    | ö        |
| 247         | 367         | F7            | 11110111    | ÷        |
| 248         | 370         | F8            | 11111000    | ø        |
| 249         | 371         | F9            | 11111001    | ù        |
| 250         | 372         | FA            | 11111010    | ú        |
| 251         | 373         | FB            | 11111011    | û        |
| 252         | 374         | FC            | 11111100    | ü        |
| 253         | 375         | FD            | 11111101    | ý        |
| 254         | 376         | FE            | 11111110    | þ        |
| 255         | 377         | FF            | 11111111    | ÿ        |

比如：
```python
# 原始文本 -> UTF-8 编码字节
text = "你好世界"
bytes_data = text.encode('utf-8')
print(bytes_data)
# 此时输出对应的UTF-8 字节为b'\xe4\xbd\xa0\xe5\xa5\xbd\xe4\xb8\x96\xe7\x95\x8c'
# 再按照上面的映射表，将每个字节转换为字符
# \xe4->ä；\xbd->½；\xa0->ł；\xe5->å；\xa5-> ¥； \xbd->½ ； \xe4->ä；\xb8->¸ ； \x96->ĸ； \xe7->ç； \x95->ķ； \x8c->Į；
# 组合即可得到：ä½łå¥½ä¸ĸçķĮ
# 在DeepSeek-V3.2的tokenizer.json中：
# "ä½łå¥½": 30594
# "ä¸ĸçķĮ": 3427
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("./DeepSeek")
print(tokenizer.decode([30594]))
print(tokenizer.decode([3427]))

```

## BPE

BPE算法的主体思想是统计相邻子词的频率，不断融合频率最高的相邻子词

训练过程：

1. 词典构造：训练一个词典大小为8的字典，训练语料为：abcccabcabdef
2. 字符包含a,b,c,d,e,f，首先分别给它们一个id，作为字典的第一个版本：{'a': 0, 'b': 1, 'c': 2, 'd': 3, 'e': 4, 'f': 5}
3. 原始的输入变成id：[0, 1, 2, 2, 2, 0, 1, 2, 0, 1, 3, 4, 5]
4. 第一步更新词典：
   1. 统计相邻的id对出现的频率，得到：{ ('a', 'b'): 3, ('b', 'c'): 2, ('c', 'c'): 2,  ('c', 'a'): 2, ('b', 'd'): 1, ('d', 'e'): 1,  ('e', 'f'): 1 }；
   2. 可以看到，('a', 'b')这个子对出现的频率最高。我们把它们合并起 来并在词典里面加入一个新的id，加入后词典变成，并记录merge 的过程(0, 1)->6，融合后的词表为：{0: 'a', 1: 'b', 2: 'c', 3: 'd', 4: 'e', 5: 'f', 6: 'ab'}；
   3. 原始输入的id里面也把对应的(0,1)替换成6，也就是把(a, b)替换成 ab。替换后，原始输入id从[0, 1, 2, 2, 2, 0, 1, 2, 0, 1, 3, 4, 5]变为：[6, 2, 2, 2, 6, 2, 6, 3, 4, 5]。
5. 继续统计替换后的相邻id出现的频率，得到：{('ab', 'c'): 2, ('c', 'c'): 2, ('c', 'ab'):  2, ('ab', 'd'): 1, ('d', 'e'): 1, ('e', 'f'):  1}
   1. 里面有三个id对出现的频率都是2，任意选择一个进行融合。这 里选择第一个，也就是('ab', 'c')进行融合，并记录merge的过程 为(6, 2)->7，融合后的词表为：{0: 'a', 1: 'b', 2: 'c', 3: 'd', 4: 'e', 5:  'f', 6: 'ab', 7: 'abc'}
   2. 原始输入变为：[7, 2, 2, 7, 6, 3, 4, 5]
6. 继续统计相邻id频率，得到：{('abc', 'c'): 1, ('c', 'c'): 1, ('c', 'abc'):  1, ('abc', 'ab'): 1, ('ab', 'd'): 1, ('d',  'e'): 1, ('e', 'f'): 1}
   1. 继续融合第一个最大的，得到新的字典，记录merge的过程为(7,  2)-> 8，融合后的词表为：{0: 'a', 1: 'b', 2: 'c', 3: 'd', 4: 'e', 5:  'f', 6: 'ab', 7: 'abc', 8: 'abcc'}
7. 词典的大小已经达到我们设定的大小，训练完成

编码过程：

1. 词典：{0: 'a', 1: 'b', 2: 'c',  3: 'd', 4: 'e', 5: 'f',  6: 'ab', 7: 'abc', 8:  'abcc'}
2. merge过程：(0, 1)->6；(6, 2)->7；(7, 2)->8
3. 假设需要编码的输入字符串abc：
   1. 先把单个字符转换成对应的id：a->0, b->1, c->2，得到：[0, 1, 2]
   2. 对[0, 1, 2]进行的merge操作，得到[6,  2]，每一步都选择可以merge的最小 id，继续这个过程，最后得到编码结果为[7]
   3. abc -> [7]

```python
from typing import Dict, Tuple


class BPETokenizer:

    """
    BPE算法的实现
    """

    def __init__(self):
        """
        初始化BPE分词器
        """
        self.merge = {}
        self.id_2_char = {}
        self.char_2_id = {}

    def train(self, input_texts, vocab_size):

        """
        BPE算法的训练过程

        Args:
            input_texts: str: 输入语料库
            vocab_size: int: 目标构建的词典的大小

        Returns:
            None
        """

        # 1.对输入语料进行切分
        unique_chrs = list(set(list(input_texts)))
        # 2.得到一个初始化的字典
        id_2_char = {idx: char for idx, char in enumerate(unique_chrs)}
        char_2_id = {char: idx for idx, char in enumerate(unique_chrs)}
        # 3.利用字典对输入语料进行id化
        ids = [char_2_id[c] for c in input_texts]

        merge_times = vocab_size - len(unique_chrs)
        vocab_size = len(unique_chrs) - 1
        merge = {}
        # 4.训练，合并子词，直到字典的大小达到val_size
        for i in range(merge_times):
            if len(ids) == 1:
                break
            # 统计相邻子词出现的频率
            stats = self._stats_adjacent_pairs(ids)
            # 找出出现频率最高的相邻子词对
            pair = max(stats, key=stats.get)
            vocab_size += 1
            id_2_char[vocab_size] = id_2_char[pair[0]] + id_2_char[pair[1]]
            char_2_id[id_2_char[pair[0]] + id_2_char[pair[1]]] = vocab_size
            merge[pair] = vocab_size

            # 根据当前的词典，合并ids
            ids = self._merge_ids(ids, pair, vocab_size)

        self.merge = merge
        self.char_2_id = char_2_id
        self.id_2_char = id_2_char

    @staticmethod
    def _stats_adjacent_pairs(ids) -> Dict[Tuple[str, str], int]:


        """
        统计输入列表中相邻元素对的出现频率（次数）

        核心逻辑：通过遍历列表中相邻的元素对（如ids[i]和ids[i+1]），
        记录每一组相邻元素对出现的总次数，最终返回频率统计结果。
        示例：输入[1,2,3,4,1,2]，返回{(1,2):2, (2,3):1, (3,4):1, (4,1):1}

        Args:
            ids (List[int]): 包含整数元素的输入列表，用于提取相邻元素对进行统计，示例：[1,2,3,4,1,2]

        Returns:
            Dict[Tuple[int, int], int]: 相邻元素对频率统计字典
            - 键：Tuple[int, int]，相邻的两个整数组成的元组（相邻元素对）
            - 值：int，对应相邻元素对在输入列表中出现的次数
        """
        # [1,2,3,4,1,2]
        # [1,2,3,4,1]
        # [2,3,4,1,2]
        count = {}
        for item in zip(ids[:-1], ids[1:]):
            count[item] = count.get(item, 0) + 1
        return count

    @staticmethod
    def _merge_ids(ids, pair, idx):

        """
        合并语料库里面的相邻子词对，并更新ids

        Args:
            ids (List[int]): 语料库未更新前的ids
            pair (Tuple[int, int]): 当前待合并的相邻子词对
            idx (int): 当前合并的相邻子词对在词典里的id

        Returns:
            List[int]: 语料库更新后的ids
        """
        new_ids = []
        i = 0
        # ids: [1,2,3,4,5]
        # pair: [3,4]
        while i < len(ids):
            if ids[i] == pair[0] and i < len(ids) - 1 and ids[i + 1] == pair[1]:
                new_ids.append(idx)
                i += 2
            else:
                new_ids.append(ids[i])
                i += 1
        return new_ids

    def encode(self, text):

        """
        将输入文本进行切分并索引化

        Args:
            text (str): 输入的文本

        Returns:
            List(int): 索引化之后的列表
        """

        # 1.对输入文本进行简单切分
        ids = [self.char_2_id[c] for c in text]
        # print(ids)
        # 2 利用merge词典，进行多次合并，得到最终的输出
        while len(ids) >= 2:
            stats = self._stats_adjacent_pairs(ids)
            pair = min(stats, key=lambda p: self.merge.get(p, float('inf')))
            if pair not in self.merge:
                break
            ids = self._merge_ids(ids, pair, self.merge[pair])
        return ids

    def decode(self, ids):

        """
        将索引列表转化为文本

        Args:
            ids (List[int]): 索引列表

        Returns:
            List[str]: 文本列表
        """
        return "".join([self.id_2_char[index] for index in ids])


if __name__ == '__main__':
    t1 = BPETokenizer()
    train_text = """
        小说塑造了一群可歌可泣的英雄群像，展现了这些英雄的悲剧以及其中深刻的意义。
        他们具有强大的不可战胜的精神力量，面对不可抗拒的命运也毅然“亮剑”。
        在作者的笔下，李云龙的形象更为深刻，与以前的英雄形象有着巨大的差别。
        李云龙之死是小说的浓墨重彩之处，这时的李云龙不但具有着超乎常人的英雄意志和英雄业绩，
        同时还具备了大彻大悟的英雄的精神力量，他的死亡是撼人心魄的悲剧，是面对更为强大的外在力量的不屈抗争，
        他以死来肯定生，以他的毅然亮剑来张扬生命的尊严与高贵。作者通过英雄的悲剧表达了一种对文革历史的思考。
        作品通过对造成英雄悲剧的社会环境的描写，为读者重现了十年文革的社会动荡。
        城市的肆意破坏，人民的大量伤亡，英雄的无辜迫害，道德的任意践踏，人沦的丧失殆尽。
        人们在悲叹小说主人公的悲剧命运的同时也不由对那场十年浩劫进行反思。
        作者借赵刚之口表达了自己对“十年文革”的客观审视与冷静反思，
        “革命也许是个中性词，它可以引导人们走向光明， 也可以以革命的名义制造人间灾难。
        革命必须符合普遍的道德准则即人道的原则，如果对个体生命漠视或无动于衷，甚至无端制造流血和死亡，
        所谓革命无论打着怎样好看的旗帜，其性质都是可疑的”。
        作者与英雄的悲剧寓意中蕴含了对人性光辉的追求及对人类美好前景的期望。
        英雄的悲剧使人悲痛，但与悲痛之中我们看到的不仅仅是英雄的被毁灭，更多的是走向毁灭中那种震慑人心的精神。
        这种精神是一种人性中对真的不懈追求，对善的至死不渝，对美的无限向往。
        正是有这种精神的存在，黑暗中人们才不怕迷失方向，无助中人们才能有前行的力量。
        书中的英雄虽然以悲剧收场，但书中的结尾“阿波罗十一号”登月成功，“亚洲四小龙”经济腾飞，李云龙恢复名誉、平反昭雪，这一切预示着：历史是公正的，正义事业势必会顺着历史的潮流向前迈进，人性中的假恶丑也许可以在一时一地得势，阻碍社会的向前发展， 但在人性真善美光辉的普照下，历史的车轮是不可阻挡的，他必将以一种决然的气势荡涤世间的污浊而滚滚前行。
        从《亮剑》中的英雄人物身上可以找到英雄之间的共同点，那就是精神上的至高无上与不可战胜。
        他们在面对强大的外敌时表现出了万众一心，其利断金的伟大决心，还有可歌可泣的生命的高贵与尊严。
         勇往直前、慷慨无畏的亮剑精神，用主人公李云龙的话描述，就是：“面对强大的敌人，明知不敌也要毅然亮剑；即使倒下，也要成为一座山，一道岭。
         ”这种自强不息、奋斗不止、坚忍不拔的亮剑精神，其动机、动力与目的，来自于“人民军队为人民”的大忠大义。民是国之本，为民即为国；“为人民服务”，是军队的宗旨，也是军人的天职；为了人民的根本利益，个人生死荣辱是那样微不足道。
         作者站在人道主义的立场塑造英雄，人道主义简言之就是批判和摒弃无视甚至践踏人的尊严和自由的封建价值观，强调人的尊严、自由和发展。以人道主义的情怀表现战争，不仅写出了战争中双方的各种各样的人物形象，而且写出了人的生命的宝贵、人的尊严的神圣以及战争对人的生命和价值的破坏和毁灭，达到了一种人性的高度，表现了一种进步的现代意义的战争观。
         在一定程度上反映了战争的残酷性和反人类性，这样的战争表现才能更适合人类的精神需要，更符合人类的审美理想。
         《亮剑》43章，加一个尾声，共44个相对独立而完整的故事。
         这44个段落所描写的44个故事情节，从作者设计到实际的叙述，都是以主人公李云龙的行动作为主线。
         李云龙是全部故事的筋。他从来也不可能离开每一个故事，他参与了每一章所描写的情节。
         他不仅是故事的联结者，更是故事的主要参与者。
         从中读者可以看出，李云龙的行动从第一章开始就是不间断地出现的。
         即使某一章，如第六章写楚云飞去偷袭日本鬼子的部队，写得相当精彩而详细，把楚云飞足智多谋、精明强干的性格写得栩栩如生，但在这个人物身上下重笔的时候，李云龙的行动也没有间断，除了楚云飞的行动在逻辑上是为了解救李云龙这个相关的大前提之外，李云龙也在这一章里出现。
         并且，在面对残暴的日本侵略者的时候，这两个既是对手又是朋友的关系，通过两个人的不断的行动得到了加强。
         《亮剑》小说采用了传统的线性叙事结构，从李云龙参加抗日战争开始到文革中的自杀结尾，按照时间顺序将故事娓娓道来。
         如五个章节讲述李云龙围攻平安县城，之前李云龙与日本关东军的白刃战让他受到日军注意，因此山本一木的特种部队才被派遣偷袭独立团；恰巧正值李云龙大婚之际，新娘子秀芹被俘虏了；吃了大亏的李云龙怒发冲冠集结独立团攻打平安县，牵一发而动全身的战争格局竟因为李云龙的这一战取得了积极成果，晋西北战场打成一片。
         在线性叙事结构中，所有事件的发生、发展、高潮、结局有据可循，读者可以根据其叙事内容明确一条或几条线索。
         首先，《亮剑》通过增强传奇偶然性的方式来消解先前的革命历史传奇英雄塑造手法。
         所谓传奇的偶然性在此是指传奇故事情节演进的突变性、非线性与超逻辑性。在先前的革命历史传奇中，英雄传奇故事发生的深层次原因往往是党的使命或人民的需要，而《亮剑》却将偶然的凡俗小事或英雄人物的小过失作为英雄传奇故事发生的诱因和英雄传奇故事演进的内驱力，如《亮剑》中李云龙没有脱离红军仅仅是因为传令兵陷入沼泽淹死了，攻打平安县城的原因竟是“冲冠一怒为红颜”。这种偶然性淡化了政治的严肃性，使英雄传奇故事具有生活的“原生态”特征或不假雕饰的真实性。
         其次，《亮剑》通过增强传奇浪漫色彩的方式来消解先前革命历史传奇英雄塑造手法。
         革命历史传奇塑造的英雄形象大都具有很强的政治隐喻意义，如通过“革命战士”个体的品格隐喻整个共产党阶层的优秀品格与特殊属性，通过“革命战士”的成长经历与过程隐喻历史的演进规律或图解特殊的政治哲学理念等等；这种隐喻手法带来两种结果：一是由政治功利性所致的英雄形象的鲜明政治印记，二是英雄形象的浪漫色彩被削弱。
         《亮剑》的英雄形象塑造并不强调的政治隐喻性，相反，它在一定程度上是以满足大众文化勃兴背景中读者的休闲需求为目的的，正如作者都梁所说：“读者朋友，咱老百姓看书，没别的要求，故事好就行。
         作为作者，我也是这么想。我写出的故事，要能得到您的认可，我就知足了。”因此《亮剑》的英雄传奇更具有浪漫色彩。浪漫色彩增强了传奇的戏剧性和革命历史传奇英雄经历的曲折性。
        """
    t1.train(input_texts=train_text, vocab_size=128)
    t1.id_2_char
    print(t1.decode(t1.encode("英雄人物")))
```

