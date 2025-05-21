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
  # 1、在安装完驱动、docker以及nvidia-container-toolkit后，用docker创建容器时出现如下报错,原因是gpu的持久模式(nvidia-persistenced daemon)并未开启
  [root@server01 deepseek]# docker: Error response from daemon: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: error during container init: error running hook #0: error running hook: exit status 1, stdout: , stderr: Auto-detected mode as 'legacy' nvidia-container-cli: initialization error: driver rpc error: timed out: unknown.
  # 可以用nvidia-smi -a查询自己的 Persistence Mode 是否开启、同时也可以用nvidia-smi
  # 解决方案：使用root权限执行如下命令：
  [root@server01 deepseek]# nvidia-smi -pm ENABLED
  
  # 2、nvidia-docker Failed to initialize NVML: Unknown Error
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





