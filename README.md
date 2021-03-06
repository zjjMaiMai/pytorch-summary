## Keras style `model.summary()` in PyTorch
[![PyPI version](https://badge.fury.io/py/torchsummary.svg)](https://badge.fury.io/py/torchsummary)

Keras has a neat API to view the visualization of the model which is very helpful while debugging your network. Here is a barebone code to try and mimic the same in PyTorch. The aim is to provide information complementary to, what is not provided by `print(your_model)` in PyTorch.

### Usage

- `pip install torchsummary` or 
- `git clone https://github.com/sksq96/pytorch-summary`

```python
from torchsummary import summary
summary(your_model, input_size=(channels, H, W))
```

- Note that the `input_size` is required to make a forward pass through the network.

### Examples

#### CNN for MNSIT

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
from torchsummary import summary

class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.conv1 = nn.Conv2d(1, 10, kernel_size=5)
        self.conv2 = nn.Conv2d(10, 20, kernel_size=5)
        self.conv2_drop = nn.Dropout2d()
        self.fc1 = nn.Linear(320, 50)
        self.fc2 = nn.Linear(50, 10)

    def forward(self, x):
        x = F.relu(F.max_pool2d(self.conv1(x), 2))
        x = F.relu(F.max_pool2d(self.conv2_drop(self.conv2(x)), 2))
        x = x.view(-1, 320)
        x = F.relu(self.fc1(x))
        x = F.dropout(x, training=self.training)
        x = self.fc2(x)
        return F.log_softmax(x, dim=1)

device = torch.device("cuda" if torch.cuda.is_available() else "cpu") # PyTorch v0.4.0
model = Net().to(device)

summary(model, (1, 28, 28))
```

```
--------------------------------------------------------------------------------------------
        Layer (type)               Output Shape         Param #               MACC #
============================================================================================
            Conv2d-1           [-1, 10, 24, 24]             260               29.97%
            Conv2d-2             [-1, 20, 8, 8]           5,020               66.60%
         Dropout2d-3             [-1, 20, 8, 8]               0                    -
            Linear-4                   [-1, 50]          16,050                3.33%
            Linear-5                   [-1, 10]             510                0.10%
============================================================================================
Total params: 21,840
Total MACC: 480,500
Trainable params: 21,840
Non-trainable params: 0
--------------------------------------------------------------------------------------------
Input size (MB): 0.00
Forward/backward pass size (MB): 0.06
Params size (MB): 0.08
Estimated Total Size (MB): 0.15
--------------------------------------------------------------------------------------------
```


#### VGG16


```python
import torch
from torchvision import models
from torchsummary import summary

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
vgg = models.vgg16().to(device)

summary(vgg, (3, 224, 224))
```



```
--------------------------------------------------------------------------------------------
        Layer (type)               Output Shape         Param #               MACC #
============================================================================================
            Conv2d-1         [-1, 64, 224, 224]           1,792                0.56%
              ReLU-2         [-1, 64, 224, 224]               0                    -
            Conv2d-3         [-1, 64, 224, 224]          36,928               11.96%
              ReLU-4         [-1, 64, 224, 224]               0                    -
         MaxPool2d-5         [-1, 64, 112, 112]               0                    -
            Conv2d-6        [-1, 128, 112, 112]          73,856                5.98%
              ReLU-7        [-1, 128, 112, 112]               0                    -
            Conv2d-8        [-1, 128, 112, 112]         147,584               11.96%
              ReLU-9        [-1, 128, 112, 112]               0                    -
        MaxPool2d-10          [-1, 128, 56, 56]               0                    -
           Conv2d-11          [-1, 256, 56, 56]         295,168                5.98%
             ReLU-12          [-1, 256, 56, 56]               0                    -
           Conv2d-13          [-1, 256, 56, 56]         590,080               11.96%
             ReLU-14          [-1, 256, 56, 56]               0                    -
           Conv2d-15          [-1, 256, 56, 56]         590,080               11.96%
             ReLU-16          [-1, 256, 56, 56]               0                    -
        MaxPool2d-17          [-1, 256, 28, 28]               0                    -
           Conv2d-18          [-1, 512, 28, 28]       1,180,160                5.98%
             ReLU-19          [-1, 512, 28, 28]               0                    -
           Conv2d-20          [-1, 512, 28, 28]       2,359,808               11.96%
             ReLU-21          [-1, 512, 28, 28]               0                    -
           Conv2d-22          [-1, 512, 28, 28]       2,359,808               11.96%
             ReLU-23          [-1, 512, 28, 28]               0                    -
        MaxPool2d-24          [-1, 512, 14, 14]               0                    -
           Conv2d-25          [-1, 512, 14, 14]       2,359,808                2.99%
             ReLU-26          [-1, 512, 14, 14]               0                    -
           Conv2d-27          [-1, 512, 14, 14]       2,359,808                2.99%
             ReLU-28          [-1, 512, 14, 14]               0                    -
           Conv2d-29          [-1, 512, 14, 14]       2,359,808                2.99%
             ReLU-30          [-1, 512, 14, 14]               0                    -
        MaxPool2d-31            [-1, 512, 7, 7]               0                    -
           Linear-32                 [-1, 4096]     102,764,544                0.66%
             ReLU-33                 [-1, 4096]               0                    -
          Dropout-34                 [-1, 4096]               0                    -
           Linear-35                 [-1, 4096]      16,781,312                0.11%
             ReLU-36                 [-1, 4096]               0                    -
          Dropout-37                 [-1, 4096]               0                    -
           Linear-38                 [-1, 1000]       4,097,000                0.03%
============================================================================================
Total params: 138,357,544
Total MACC: 15,470,264,320
Trainable params: 138,357,544
Non-trainable params: 0
--------------------------------------------------------------------------------------------
Input size (MB): 0.57
Forward/backward pass size (MB): 218.59
Params size (MB): 527.79
Estimated Total Size (MB): 746.96
--------------------------------------------------------------------------------------------
```


### References

- The idea for this package sparked from [this PyTorch issue](https://github.com/pytorch/pytorch/issues/2001).
- Thanks to @ncullen93 and @HTLife. 
- For Model Size Estimation @jacobkimmel ([details here](https://github.com/sksq96/pytorch-summary/pull/21))
