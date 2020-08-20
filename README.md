[toc]

# Deep FM详解

## 背景

> CTR预估是目前推荐系统的核心技术，其目标是预估用户点击推荐内容的概率。在CTR预估任务中，用户行为的隐式low-order和high-order特征都起到十分重要。有些特征是易于理解的，可以通过领域专家进行人工特征工程抽取特征。但是对于不易于理解的特征，如“啤酒和尿布”，则只能通过机器学习的方法得到。同样的对于需要特别巨量特征的模型，人工特征工程也是难以接受的，所以特征的自动学习显的十分重要。
>
> 简单线性模型缺乏学习high-order特征的能力，很难从训练样本中学习到从未出现或极少出现的重要特征。FM模型可以通过点积和隐向量的形式学习交叉特征。由于复杂度的约束，FM通常只应用order-2的2重交叉特征。深层模型善于捕捉high-order复杂特征，如CNN/RNN/FNN/PNN/Wide & Deep等，但都有各自的问题。FNN和PNN缺乏low-order特征，Wide & Deep需要人工构造Wide部分交叉特征。
>
> 为了解决以上模型的缺陷，同时学习高阶和低阶的组合特征，避免Wide & Deep的手工构造交叉特征，Deep FM集成了因子分解机（FM）和深度神经网络（DNN）的体系结构。

## 模型结构

> DeepFM模型结构设计如下：

![image](https://github.com/ShaoQiBNU/DeepFM/blob/master/img/1.jpg)

> DeepFM模型包含FM和DNN两部分，FM算法负责对一阶特征以及二阶特征（由一阶特征交叉组合得到的）进行特征的提取；DNN算法负责对高阶特征进行特征的提取，与Wide & Deep不同的地方主要有以下两点：
>
> 1. wide模型部分由LR替换为FM。FM模型具有自动学习交叉特征的能力，避免了原始Wide & Deep模型中浅层部分人工特征工程的工作。
> 2. 共享原始输入特征。DeepFM模型的原始特征将作为FM和Deep模型部分的共同输入，所以模型训练速度很快。



### FM部分

> FM部分主要是对一维特征与二维交叉特征进行建模。对于一维部分进行了Addition运算，对于二维交叉特征主要通过隐向量点积的方法高效的获得2阶特征表示，即使交叉特征在数据集中非常稀疏甚至是从来没出现过，这也是FM的优势所在。

![image](https://github.com/ShaoQiBNU/DeepFM/blob/master/img/2.jpg)

### Deep部分

> Deep部分是一个前向的DNN，用于学习更高阶的交叉特征，原始的高维稀疏特征向量被Embedding层压缩成低维稠密特征向量，然后再送入hidden layer。

![image](https://github.com/ShaoQiBNU/DeepFM/blob/master/img/3.jpg)

**论文直接将FM和DNN进行整体联合训练，从而实现了一个端到端的模型**

## 实验

> 文中通过在Criteo数据集和Company数据集进行大量实验，对比了LR、FM、FNN、IPNN、OPNN、PNN\*，LR&DNN、FM&DNN和DeepFM的效果，证明DeepFM在效果上超越了目前最优的模型，效率上的与当前最优模型相当。实验评价指标采用AUC和Logloss两个离线指标。



### 效率实验

![image](https://github.com/ShaoQiBNU/DeepFM/blob/master/img/3.jpg)

> 实验分别比较了各个模型在CPU和GPU上的运行效率，DeepFM都取得了和最优效率相当的结果。（从图中看OPNN的执行效率也很高，但是OPNN结果效果波动性大，不稳定）

### 效果实验

![image](https://github.com/ShaoQiBNU/DeepFM/blob/master/img/4.jpg)

> 实验结果可以看出，DeepFM模型比其他对比模型AUC提升至少0.37%，logloss减少至少0.42%。同时可以得到以下结论：
>
> 1. 学习特征交互可以改善CTR预估模型。
> 2. 同时学习high-order和low-order特征可以改善CTR模型效果。
> 3. 通过共享输入特征学习high-order和low-order交互特征表示可以提升CTR模型。

### 参数学习实验

> 为了比较实验效果，文中对超参数进行了多组实验，如下：

#### **激活函数**

![image](https://github.com/ShaoQiBNU/DeepFM/blob/master/img/5.jpg)

#### **Dropout**

![image](https://github.com/ShaoQiBNU/DeepFM/blob/master/img/6.jpg)

#### **每层节点数**

![image](https://github.com/ShaoQiBNU/DeepFM/blob/master/img/7.jpg)

#### **隐藏层数**

![image](https://github.com/ShaoQiBNU/DeepFM/blob/master/img/8.jpg)

#### **网络形状**

![image](https://github.com/ShaoQiBNU/DeepFM/blob/master/img/9.jpg)

![image](https://github.com/ShaoQiBNU/DeepFM/blob/master/img/10.jpg)

![image](https://github.com/ShaoQiBNU/DeepFM/blob/master/img/11.jpg)
