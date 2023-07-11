# 模型微调指南 
目录
- [开始一个新的项目](#开始新项目的指导)
   - [选择模型架构](#选择模型架构)
   - [选择优化器](#选择优化器)
   - [选择batch size](#选择batch-size)
   - [选择初始配置](#选择初始配置)

- [提升模型表现的科学方法](#提升模型表现的科学方法)
- [决定每次训练的step](#决定每次训练的step)
- [训练pipeline额外的指导](#训练pipeline的额外指导)

[原文github地址](https://github.com/google-research/tuning_playbook)
## 开始新项目的指导
### 选择模型架构
当开始一个新项目时尝试使用一个已经work的模型。
### 选择优化器
选择解决当前问题最热门的优化器，建议优化器[SGD with momentum](https://github.com/google-research/tuning_playbook/blob/main/README.md#what-are-the-update-rules-for-all-the-popular-optimization-algorithms), [Adam and NAdam](https://github.com/google-research/tuning_playbook/blob/main/README.md#what-are-the-update-rules-for-all-the-popular-optimization-algorithms)
### 选择batch size
在硬件支持的基础之上选择最大的batch size。
#### 确定可行的batch size大小，并估计吞吐量。
 <details><summary><em>[点击进行扩展]</em></summary>

 <br>
  
  -  在给定模型和优化器时，可用资源通常有不同的batch size可以选择。batch size往往受限于加速器的内存大小。  
  -  在模型没有跑起来时很难确定batch size消耗的对应的内存大小。  
  -  最简单的方法是在不同的batch size上训练一些小的step,直到超过了可用的内存。  
  -  对于每一个batch size,我们需要训练足够长的时间去得到一个可靠的吞吐量。  
  <p align="center">训练吞吐量 = （每秒处理的样例数量），或者处理一个step消耗的时间</p>
  <p align="center">time per step = (batch size) / (训练吞吐量)  </p>

  -  当加速器尚未饱和时，如果batch size变为两倍，那么训练的吞吐量也应该变为两倍（或者接近两倍）,训练每个step的时长也应该伴随的batch size增长。  
  -  如果情况并非如此，那么训练的pipeline可能存在瓶颈，比如节点之间I/O或者同步，在继续之前这些情况需要被诊断或者修复。  
  -  如果训练的吞吐量仅支持到某些最大的batch size,那么我们应该选择这些batch size,即使机器支持更大的batch size。  
  -  每次更改模型或者优化器之前，可能需要重复以上步骤。

 </details>

#### 选择batch size, 去最小化训练时长
<details><summary><em>[点击进行扩展]</em></summary>

<br>

<p align="center">训练时间 = （每个step的训练时间）*（总的steps）</p>

 -  我们通常认为对所有可行的batch size, 每个step的训练时长近似恒定，在实践中增加batch size大小通常会带来一些额外开销。
 -  伴随着batch size的大小增加，

<details>

#### 选择batch size, 去最小化训练资源
#### 更改batch size, 需要调整大多数超参数
#### batch norm如何影响batch size
### 选择初始配置
## 提升模型表现的科学方法
## 决定每次训练的step
## 训练pipeline的额外指导