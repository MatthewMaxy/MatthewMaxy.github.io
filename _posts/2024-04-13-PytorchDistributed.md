---
layout: post
title:  "Pytorch模型分布式训练"
date:   2024-04-13 23:44:18 +0800
tags:   Projects Pytorch
color: rgb(32,71,146)
cover: '../assets/Blogs/PytorchDistributed/main.png'
subtitle: 'Pytorch模型的分布式训练方法与代码模板'
---

## 简介

在进行模型训练时，由于训练时间过长需要采用多张 GPU 进行训练，“被迫”进行了相应的学习和探索，对方法及代码模板做一个简单总结，便于下次复用。

## nn.DataParallel

DataParallel 可以帮助我们将模型和数据加载到多个 GPU 中，同时是一个进程控制的，用起来最方便，但是速度比较慢。

**nvidia-smi 进程信息:**

**模型训练代码：**

```python
import torch
import torch.distributed as dist

gpus = [0, 1, 2, 3]
torch.cuda.set_device('cuda:{}'.format(gpus[0]))

train_dataset = ...
train_loader = torch.utils.data.DataLoader(train_dataset, batch_size=...)

model = ...

'''
    只需要用 DataParallel 包裹模型
    重要参数：
        device_ids=gpus, 参与训练的GPU
        output_device=gpus[0], 用于汇总梯度的GPU 
'''

model = nn.DataParallel(model.to(device), device_ids=gpus, output_device=gpus[0])

optimizer = optim.SGD(model.parameters())

for epoch in range(100):
   for batch_idx, (data, target) in enumerate(train_loader):
      images = images.cuda(non_blocking=True)
      target = target.cuda(non_blocking=True)
      ...
      output = model(images)
      loss = criterion(output, target)
      ...
      optimizer.zero_grad()
      loss.backward()
      optimizer.step()
```

**程序执行代码：**

```bash
    python main.py
```

## ⭐️ torch.distributed

与 DataParallel 的单进程控制多 GPU 不同，使用distribud时，torch 会自动将其代码分配给 $n$ 个进程，分别在 $n$ 个 GPU 上运行.

**nvidia-smi 进程信息:**

**模型训练代码：**

代码相对使用 DataParallel 变化较大，主要在【parser】, 【分布式初始化】, 【数据集读取】，【模型加载】，【等待梯度汇总】处需要修改：

```python
import torch
import argparse
import torch.distributed as dist
from torch.utils.data import DataLoader

def parse_arguments():

    ...
    # 注意需要添加 local_rank 
    # 但在程序执行时不需要输入，由程序自行分配
    parser.add_argument(
        "--local_rank", type=int, required=True
    )
    args = parser.parse_args()
    return args

def main():

    args = parse_arguments()

    # 使用 init_process_group 设置GPU 之间通信使用的后端和端口
    dist.init_process_group(backend='nccl')
    torch.cuda.set_device(args.local_rank)

    # 使用 DistributedSampler 对数据集进行划分
    # 将每个 batch 划分成几个 partition，在当前进程中只有和 rank 对应的进行训练
    dataset = ...
    sample = torch.utils.data.distributed.DistributedSampler(dataset)
    dataloader = DataLoader(dataset, batch_size=..., sampler=sample)
    
    # 使用 DistributedDataParallel 包装模型
    # 汇总不同 GPU 计算所得的梯度，并同步计算结果
    model = ...
    model = torch.nn.parallel.DistributedDataParallel(model, device_ids=[local_rank])

    optimizer = optim.SGD(model.parameters())

    for epoch in range(epochs):
        for data_batch, label_batch in tqdm(dataloader):

            data_batch = data_batch.cuda(local_rank, non_blocking=True)
            label_batch = label_batch.cuda(local_rank, non_blocking=True)
            ...
            loss = model(data_batch, label_batch)
            ...
            optimizer.zero_grad()
            loss.backward()
            optimizer.step()

            # 阻塞，等待所有进程结束便与同步汇总
            dist.barrier()
```

**程序执行代码：**

在使用时，调用 torch.distributed.launch 启动器启动：

```bash
    # 以防上次异常退出，端口未释放
    fuser -k -n tcp 14285
    CUDA_VISIBLE_DEVICES=0,1,2,3 \
    python -m torch.distributed.launch \
    --nproc_per_node=4 --master_port 14285 \
    train.py \
    --batch 42 ...
```