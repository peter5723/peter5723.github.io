# transformer


下面的代码是以 NanoGPT 为基础对 GPT 的极简介绍：

相关的博客有：[NanoGPT](https://zhuanlan.zhihu.com/p/678155640)

```python
import torch
import torch.nn as nn
from torch.nn import functional as F

class CausalSelfAttention(nn.Module):
    def __init__(self, n_embd, n_head, block_size, dropout=0.1):
        super().__init__()
        assert n_embd % n_head == 0
        
        # key, query, value projections
        # 我们把 Q, K, V 合并成一个大的 Linear 层，然后切分，这样效率更高
        self.c_attn = nn.Linear(n_embd, 3 * n_embd, bias=False)
        # output projection
        self.c_proj = nn.Linear(n_embd, n_embd, bias=False)
        
        self.n_head = n_head
        self.n_embd = n_embd
        self.dropout = nn.Dropout(dropout)
        
        # 这里的 tril 就是下三角矩阵（mask），用于遮盖未来
        # 1 1 0
        # 1 1 1
        self.register_buffer("tril", torch.tril(torch.ones(block_size, block_size)))

    def forward(self, x):
        B, T, C = x.shape # Batch, Time(Sequence Length), Channels(n_embd)
        
        # 1. 计算 Q, K, V
        # q, k, v 的形状都是 (B, T, C)
        q, k, v  = self.c_attn(x).split(self.n_embd, dim=2)
        
        # 2. 变换形状以适应多头注意力 (Multi-Head)
        # (B, T, C) -> (B, T, n_head, head_size) -> (B, n_head, T, head_size)
        k = k.view(B, T, self.n_head, C // self.n_head).transpose(1, 2)
        q = q.view(B, T, self.n_head, C // self.n_head).transpose(1, 2)
        v = v.view(B, T, self.n_head, C // self.n_head).transpose(1, 2)
        
        # 3. Attention Score (打分) -> QK^T / sqrt(d_k)
        # (B, n_head, T, hs) @ (B, n_head, hs, T) -> (B, n_head, T, T)
        wei = q @ k.transpose(-2, -1) * (1.0 / (k.size(-1)**0.5))
        
        # 4. Mask (掩码) -> 这一步让它成为 "Decoder" (只能看过去)
        wei = wei.masked_fill(self.tril[:T, :T] == 0, float('-inf'))
        
        # 5. Softmax -> 归一化为概率
        wei = F.softmax(wei, dim=-1)
        wei = self.dropout(wei)
        
        # 6. Weighted Sum (加权求和) -> Attention(Q,K,V) = softmax(...) * V
        y = wei @ v # (B, n_head, T, T) @ (B, n_head, T, hs) -> (B, n_head, T, hs)
        
        # 7. 拼接回原来的形状
        y = y.transpose(1, 2).contiguous().view(B, T, C)
        
        return self.c_proj(y)
```


```python
class FeedForward(nn.Module):
    def __init__(self, n_embd, dropout=0.1):
        super().__init__()
        # 通常放大倍数是 4 倍 (4 * n_embd)
        self.net = nn.Sequential(
            nn.Linear(n_embd, 4 * n_embd),
            nn.GELU(), # 现代 LLM 多用 GELU 或 SwiGLU，比 ReLU 更好
            nn.Linear(4 * n_embd, n_embd),
            nn.Dropout(dropout),
        )

    def forward(self, x):
        return self.net(x)
```

```python
class Block(nn.Module):
    def __init__(self, n_embd, n_head, block_size):
        super().__init__()
        # Layer Norm 1
        self.ln1 = nn.LayerNorm(n_embd)
        # Communication (Attention)
        self.sa = CausalSelfAttention(n_embd, n_head, block_size)
        # Layer Norm 2
        self.ln2 = nn.LayerNorm(n_embd)
        # Computation (MLP)
        self.ffwd = FeedForward(n_embd)

    def forward(self, x):
        # 这里的 += 就是残差连接 (Residual Connection)
        x = x + self.sa(self.ln1(x))   # 通信
        x = x + self.ffwd(self.ln2(x)) # 计算
        return x
```

```python
class GPT(nn.Module):
    def __init__(self, vocab_size, n_embd, n_head, n_layer, block_size):
        super().__init__()
        # 1. Token Embedding Table: 把 token ID 变成向量
        self.token_embedding_table = nn.Embedding(vocab_size, n_embd)
        # 2. Position Embedding Table: 学习位置信息
        self.position_embedding_table = nn.Embedding(block_size, n_embd)
        
        # 3. 堆叠 N 层 Block
        self.blocks = nn.Sequential(*[
            Block(n_embd, n_head, block_size) for _ in range(n_layer)
        ])
        
        # 4. 最后的归一化
        self.ln_f = nn.LayerNorm(n_embd)
        # 5. Unembedding Head: 把向量变回 logits (词表大小)
        self.lm_head = nn.Linear(n_embd, vocab_size)

        self.block_size = block_size

    def forward(self, idx, targets=None):
        B, T = idx.shape
        
        # 获取 token embedding 和 position embedding
        tok_emb = self.token_embedding_table(idx) # (B, T, C)
        pos_emb = self.position_embedding_table(torch.arange(T, device=idx.device)) # (T, C)
        
        # x = 内容 + 位置
        x = tok_emb + pos_emb 
        
        # 通过所有 Transformer Blocks
        x = self.blocks(x) 
        
        x = self.ln_f(x)
        logits = self.lm_head(x) # (B, T, vocab_size)
        
        if targets is None:
            loss = None
        else:
            # 计算 Loss (通常是 CrossEntropy)
            B, T, C = logits.shape
            logits = logits.view(B*T, C)
            targets = targets.view(B*T)
            loss = F.cross_entropy(logits, targets)

        return logits, loss
```

```python
# 超参数配置
vocab_size = 1000 # 假设词表大小
n_embd = 64       # 嵌入维度
n_head = 4        # 4个头
n_layer = 3       # 3层 Block
block_size = 128  # 上下文长度

# 实例化模型
model = GPT(vocab_size, n_embd, n_head, n_layer, block_size)
print(f"模型参数量: {sum(p.numel() for p in model.parameters())/1e6:.2f}M")

# 模拟输入数据 (Batch=2, Time=32)
dummy_input = torch.randint(0, vocab_size, (2, 32))

# 前向传播
logits, loss = model(dummy_input, dummy_input) # 这里 target 也是 dummy，仅作演示
print(f"Logits shape: {logits.shape}") # 预期: [64, 1000] (因为 view 展平了) 或者是 [2, 32, 1000]
print(f"Loss: {loss.item()}")
```

![](https://cdn.jsdelivr.net/gh/peter5723/imagehost/img/gpt1.png)


