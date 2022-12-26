



# 【AI】Tunning skills

### 疑问点

 





指导：

self-attention+nsa

抗拟合：adversial training 对抗学习





## 一、Pooling

Bert模型出来很多Token之后如何将其组合成一个向量？

思路：是否需要attention_mask信息，若使用attention_mask信息即

- MeanPooling
- MaxPooling
- MinPooling
- MeanPooling+K_MaxPooling
- WeightedPooling
- AttentionPooling
- Concatenation
- Bottleneck

其中Concatenation就是Flatten，只不过为了维持网络的大小不变，还用了Padding进行补齐

Bottleneck

<img src="https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/%E7%BB%93%E6%9E%9C.jpg" alt="结果" style="zoom:33%;" />

```python
class MeanPooling(nn.Module):
    def __init__(self):
        super(MeanPooling, self).__init__()
        
    def forward(self, last_hidden_state, attention_mask):
        input_mask_expanded = attention_mask.unsqueeze(-1).expand(last_hidden_state.size()).float()
        sum_embeddings = torch.sum(last_hidden_state * input_mask_expanded, 1)
        sum_mask = input_mask_expanded.sum(1)
        sum_mask = torch.clamp(sum_mask, min = 1e-9)
        mean_embeddings = sum_embeddings/sum_mask
        return mean_embeddings
```

```python
class MaxPooling(nn.Module):
    def __init__(self):
        super(MaxPooling, self).__init__()
        
    def forward(self, last_hidden_state, attention_mask):
        input_mask_expanded = attention_mask.unsqueeze(-1).expand(last_hidden_state.size()).float()
        embeddings = last_hidden_state.clone()
        embeddings[input_mask_expanded == 0] = -1e4
        max_embeddings, _ = torch.max(embeddings, dim = 1)
        return max_embeddings
```

```python
class MinPooling(nn.Module):
    def __init__(self):
        super(MinPooling, self).__init__()
        
    def forward(self, last_hidden_state, attention_mask):
        input_mask_expanded = attention_mask.unsqueeze(-1).expand(last_hidden_state.size()).float()
        embeddings = last_hidden_state.clone()
        embeddings[input_mask_expanded == 0] = 1e-4
        min_embeddings, _ = torch.min(embeddings, dim = 1)
        return min_embeddings
```

```python
class Mean_KMaxPooling(nn.Module):
    def __init__(self):
        super(MaxPooling, self).__init__()
        
    def forward(self, last_hidden_state, attention_mask, k):
        input_mask_expanded = attention_mask.unsqueeze(-1).expand(last_hidden_state.size()).float()
        embeddings = last_hidden_state.clone()
        embeddings[input_mask_expanded == 0] = -1e4
        max_embeddings, _ = torch.max(embeddings, dim = 1)
        return max_embeddings
```

```python
class WeightedLayerPooling(nn.Module):
    def __init__(self, num_hidden_layers, layer_start: int = 4, layer_weights = None):
        super(WeightedLayerPooling, self).__init__()
        self.layer_start = layer_start
        self.num_hidden_layers = num_hidden_layers
        self.layer_weights = layer_weights if layer_weights is not None \
            else nn.Parameter(
                torch.tensor([1] * (num_hidden_layers+1 - layer_start), dtype=torch.float)
            )
 
    def forward(self, ft_all_layers):
        all_layer_embedding = torch.stack(ft_all_layers)
        all_layer_embedding = all_layer_embedding[self.layer_start:, :, :, :]
 
        weight_factor = self.layer_weights.unsqueeze(-1).unsqueeze(-1).unsqueeze(-1).expand(all_layer_embedding.size())
        weighted_average = (weight_factor*all_layer_embedding).sum(dim=0) / self.layer_weights.sum()
 
        return weighted_average
```

```python
class AttentionPooling(nn.Module):
    def __init__(self, in_dim):
        super().__init__()
        self.attention = nn.Sequential(
        nn.Linear(in_dim, in_dim),
        nn.LayerNorm(in_dim),
        nn.GELU(),
        nn.Linear(in_dim, 1),
        )
 
    def forward(self, last_hidden_state, attention_mask):
        w = self.attention(last_hidden_state).float()
        w[attention_mask==0]=float('-inf')
        w = torch.softmax(w,1)
        attention_embeddings = torch.sum(w * last_hidden_state, dim=1)
        return attention_embeddings
```



```

```



```

```



## 二、Normaliaztion

二维矩阵[ batch_size ，dim_size ]

|           |                                                    |                        |
| --------- | -------------------------------------------------- | ---------------------- |
| LayerNorm | **横**着归一化，作用于每个样本的所有特征           | norm = nn.LayerNorm(d) |
| BatchNorm | **竖**着归一化，作用于一个batch_size样本的每个特征 | norm = nn.BatchNorm(d) |

三维矩阵[ batch_size，len_size， dim_size ]

|              |                                                 |                            |
| ------------ | ----------------------------------------------- | -------------------------- |
| LayerNorm    | [1，len_size，dim_size]，一个batch的第i个样本   | norm = nn.LayerNorm([l,d]) |
| BatchNorm    | [batch_size，1，dim_size]，所有样本的第i个token | norm = nn.BatchNorm([l])   |
| InstanceNorm | [1，1，dim_size]，每个token的每个特征           | norm=nn.LayerNorm([d])     |

<img src="https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/Norm.drawio.png" alt="Norm.drawio" style="zoom: 33%;" />

二者公式均如下
$$
y=\frac {x-E(x)} {\sqrt {Var(x)+\epsilon }}*\gamma +\beta
$$
LN适合NLP，因为它可抹杀不同样本间大小关系，保留同样本内不同特征大小关系

BN适合CV，因为CV看重不同图像之间的样本特征关系，而同张图内各个特征之间的差异性不是很关注

**Bert**中是使用IN的

用法

```python
# batch_normalizaton
norm = nn.LayerNorm([l,d]).cuda()
output = norm(output)
```





## 三、Optimizer

SGD

Adam

Wadam





## 四、Overfitting



## 五、Regex





## 六、Loss



## 七、Activation

理论上非线性函数都可以做激活函数，但是最终层激活函数是固定的如下

|        |                    |
| ------ | ------------------ |
| 二分类 | Sigmoid or Softmax |
| 多分类 | Softmax            |



在Pytorch中，如果我们使用了nn.CrossEntropyLoss(),其默认会在输出加上一层Softmax，官网的解释如下

![image-20221210164106268](https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/image-20221210164106268.png)

也就是说，我们只要将FNN输出的分类概率和Labels输出到CrossEntropyLoss()中即可，若想自己加激活函数，需使用**nn.BCELoss**()



## 八、Parameter 

### 8.1 learning-rate

使用warm-up对学习率进行优化，其核心思想为：在训练之初采用较小的学习率，训练一段时间之后，采用较大的学习率

**理论分析**

开始阶段，模型的weights是随机初始化的，模型对数据的理解为0，第一个Epoch是模型快速学习阶段，如果lr过大会导致学偏。

中间阶段，模型训练一段时间后，模型对数据有了一定的先验知识，此时较大学习率的模型就不容易学偏。

最后阶段，模型的分布较为稳定了，使用大lr会破坏模型的稳定性，而且不易找到全局最优位置

**具体方法**

**Constant Warm-up**

lr从非常小的数值线性增加到预设值后保持不变

![在这里插入图片描述](https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/20200224100256431.png)

**Linner Warm-up**

lr从非常小的数值线性增加到预设值后，再线性减少

![在这里插入图片描述](https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/2020022410055516.png)

**Cosine Warm-up**

lr从非常小的数值线性增加到预设值后，再cos数值减少

![在这里插入图片描述](https://aaron-images-bed.oss-cn-hangzhou.aliyuncs.com/Typora/20200224100409219.png)



### 8.2 eps

作用：在分母上加上eps，一般为1e-8，一来防止分母为0，二来保证数值的稳定性

