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
 -  伴随着batch size的大小增加，达到固定性能目标所需要的steps通常会减少。  
    -  将batch size变为两倍，通常会使训练的steps减半，这称之为完美缩放（perfect scaling）。
    -  perfect scaling适用于直到边界batch size的所有batch size，超过该batch size,收益将会减少。
    -  最终，增加批量大小将不再减少batch size。
 -  因此，最小化训练时长的batch size,通常是任然能够减少训练steps的最大batch size。
    -  batch size取决于数据集，模型，优化器，如何计算出batch size依然是一个悬而未决的问题。
    -  当比较batch size的时候，要注意example budget/epoch budget和step budget之间的区别（通过example budget去比较batch size去探索完善的缩放机制，即使更大的batch size能够提供有意义的加速，通过改变steps）。
    -  通常硬件支持的最大的batch size大小小于临界batch size大小，但从经验来说，最好使用最大的batch size。
 -  如果最终会增大batch size，那么使用大batch size将没有意义。

</details>

#### 选择batch size, 去最小化训练资源

<details><summary><em>[点击进行扩展]</em></summary>

<br>

-  当batch size增加时，会有两种类型的资源消耗
   -  前期消耗（例如，购买新的硬件和重写pipeline以实现多gpu或者tpu并行训练）
   -  使用成本（例如，成员使用的资源计费，云服务商计费和电力维护成本）
-  如果增加batch size需要大量的前期消耗，那么最好推迟增加batch size,直到项目成熟，很容易去评估成本，收益消耗。实现多主机并行训练，可能会出现微妙的问题，因此在初始阶段最好使用简单的pipeline进行训练（另一方面，当需要大幅调整实验室，训练时间大幅加速可能在早期非常有益）
-  我们将使用的总成本分为以下几个部分：

<p align="center">资源消耗 = 每个step的资源消耗*（总的steps）</p>

-  增加bath size通常会减少训练消耗的steps,训练的消耗取决于每个step消耗的资源大小。
   -  增加batch size，可能会减少资源消耗，举例来说，如果每个step有更大的batch size和每个batch size有更小的step跑在同一个硬件上，那么每个step的资源消耗可能step的减少所抵消。
   -  增加batch size可能不会改变资源消耗，举例来说如果把batch size变为两倍，使得训练的steps减少一半，把gpu的数量变为两倍，那么使用gpu的时长将不会发生变化。
   -  怎加batch size可能会增加资源消耗，如果增加batch size需要升级硬件，那么每个step增加的资源消耗可能就被训练step的数量减少抵消了。

</details>

#### 更改batch size, 需要调整大多数超参数
#### batch norm如何影响batch size
### 选择初始配置

-  当开始训练调优之前，我们可能要选择开始训练的那个点。这需要明确（1）模型的配置，（2）调优的参数，（3）训练步数的数量
-  决定初始化配置可能需要手动配置训练运行和不断试错
-  我们的指导原则是找到一个简单的，相对快速，相对低资源消耗的配置，包含一个合理的结果。  
   -  简单意味着避免花里胡哨的东西（那些可以在之后被添加的东西），即使华丽花哨的东西被证明在以后可以提供帮助，把他们添加到初始配置中增加了浪费时间去微调无用特征的风险，或者导致一些并发症
      -  举例来说，在引用高级的衰减调度之前保持恒定的学习率
   -  选择更小，资源消耗更少的初始化配置可以使得超参数的微调变得更加高效
      -  例如选择更小的模型
   -  合理的性能取决于问题，但至少意味着经过训练的模型在验证集的表现上优于随机的。


## 提升模型表现的科学方法


## 决定每次训练的step
## 训练pipeline的额外指导