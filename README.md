# FCN-PyTorch

### Requirement

- PyTorch(v0.4.1)
- OpenCV
- Numpy
- tensorboardX

---

### Train

The dataset is in the folder `Dataset` where I want to do a handbag semantic segmentation which is stated as follow:

![task](./images/task.png)

First, you should input the following command in your terminal:

```bash
python FCN.py
```

then the `weight file` will be saved in folder `checkpoints`, just like `fcn_model_xx.pt`. If you want to watch the training process, it is necessary to run:

```bash
tensorboard --logdir=./logs
```

Open your browser and go to ```localhost:6006``` to see the visualization. Left 4 images is the labels, and right 4 images is the predicted images:

![image](./images/image.jpg)

You can also see the loss process:

![loss](./images/loss.jpg)

In `FCN.py`, you can change class `FCNs` into other FCN structures:

```python
class FCN32s(nn.Module):
    def __init__(self, pretrained_net, n_class):
        super().__init__()
        self.n_class = n_class
        self.pretrained_net = pretrained_net
        self.relu = nn.ReLU(inplace=True)
        self.deconv1 = nn.ConvTranspose2d(512, 512, kernel_size=3, stride=2, padding=1, dilation=1, output_padding=1)
        self.bn1 = nn.BatchNorm2d(512)
        self.deconv2 = nn.ConvTranspose2d(512, 256, kernel_size=3, stride=2, padding=1, dilation=1, output_padding=1)
        self.bn2 = nn.BatchNorm2d(256)
        self.deconv3 = nn.ConvTranspose2d(256, 128, kernel_size=3, stride=2, padding=1, dilation=1, output_padding=1)
        self.bn3 = nn.BatchNorm2d(128)
        self.deconv4 = nn.ConvTranspose2d(128, 64, kernel_size=3, stride=2, padding=1, dilation=1, output_padding=1)
        self.bn4 = nn.BatchNorm2d(64)
        self.deconv5 = nn.ConvTranspose2d(64, 32, kernel_size=3, stride=2, padding=1, dilation=1, output_padding=1)
        self.bn5 = nn.BatchNorm2d(32)
        self.classifier = nn.Conv2d(32, n_class, kernel_size=1)

    def forward(self, x):
        output = self.pretrained_net(x)
        x5 = output['x5']  # size = (N, 512, x.H/32, x.W/32)
        score = self.bn1(self.relu(self.deconv1(x5)))  # size = (N, 512, x.H/16, x.W/16)
        score = self.bn2(self.relu(self.deconv2(score)))  # size = (N, 256, x.H/8, x.W/8)
        score = self.bn3(self.relu(self.deconv3(score)))  # size = (N, 128, x.H/4, x.W/4)
        score = self.bn4(self.relu(self.deconv4(score)))  # size = (N, 64, x.H/2, x.W/2)
        score = self.bn5(self.relu(self.deconv5(score)))  # size = (N, 32, x.H, x.W)
        score = self.classifier(score)  # size = (N, n_class, x.H/1, x.W/1)
        return score  # size = (N, n_class, x.H/1, x.W/1)

class FCN16s(nn.Module):
    def __init__(self, pretrained_net, n_class):
        super().__init__()
        self.n_class = n_class
        self.pretrained_net = pretrained_net
        self.relu = nn.ReLU(inplace=True)
        self.deconv1 = nn.ConvTranspose2d(512, 512, kernel_size=3, stride=2, padding=1, dilation=1, output_padding=1)
        self.bn1 = nn.BatchNorm2d(512)
        self.deconv2 = nn.ConvTranspose2d(512, 256, kernel_size=3, stride=2, padding=1, dilation=1, output_padding=1)
        self.bn2 = nn.BatchNorm2d(256)
        self.deconv3 = nn.ConvTranspose2d(256, 128, kernel_size=3, stride=2, padding=1, dilation=1, output_padding=1)
        self.bn3 = nn.BatchNorm2d(128)
        self.deconv4 = nn.ConvTranspose2d(128, 64, kernel_size=3, stride=2, padding=1, dilation=1, output_padding=1)
        self.bn4 = nn.BatchNorm2d(64)
        self.deconv5 = nn.ConvTranspose2d(64, 32, kernel_size=3, stride=2, padding=1, dilation=1, output_padding=1)
        self.bn5 = nn.BatchNorm2d(32)
        self.classifier = nn.Conv2d(32, n_class, kernel_size=1)

    def forward(self, x):
        output = self.pretrained_net(x)
        x5 = output['x5']  # size = (N, 512, x.H/32, x.W/32)
        x4 = output['x4']  # size = (N, 512, x.H/16, x.W/16)

        score = self.relu(self.deconv1(x5))  # size = (N, 512, x.H/16, x.W/16)
        score = self.bn1(score + x4)  # element-wise add, size = (N, 512, x.H/16, x.W/16)
        score = self.bn2(self.relu(self.deconv2(score)))  # size = (N, 256, x.H/8, x.W/8)
        score = self.bn3(self.relu(self.deconv3(score)))  # size = (N, 128, x.H/4, x.W/4)
        score = self.bn4(self.relu(self.deconv4(score)))  # size = (N, 64, x.H/2, x.W/2)
        score = self.bn5(self.relu(self.deconv5(score)))  # size = (N, 32, x.H, x.W)
        score = self.classifier(score)  # size = (N, n_class, x.H/1, x.W/1)
        return score  # size = (N, n_class, x.H/1, x.W/1)

class FCN8s(nn.Module):
    def __init__(self, pretrained_net, n_class):
        super().__init__()
        self.n_class = n_class
        self.pretrained_net = pretrained_net
        self.relu = nn.ReLU(inplace=True)
        self.deconv1 = nn.ConvTranspose2d(512, 512, kernel_size=3, stride=2, padding=1, dilation=1, output_padding=1)
        self.bn1 = nn.BatchNorm2d(512)
        self.deconv2 = nn.ConvTranspose2d(512, 256, kernel_size=3, stride=2, padding=1, dilation=1, output_padding=1)
        self.bn2 = nn.BatchNorm2d(256)
        self.deconv3 = nn.ConvTranspose2d(256, 128, kernel_size=3, stride=2, padding=1, dilation=1, output_padding=1)
        self.bn3 = nn.BatchNorm2d(128)
        self.deconv4 = nn.ConvTranspose2d(128, 64, kernel_size=3, stride=2, padding=1, dilation=1, output_padding=1)
        self.bn4 = nn.BatchNorm2d(64)
        self.deconv5 = nn.ConvTranspose2d(64, 32, kernel_size=3, stride=2, padding=1, dilation=1, output_padding=1)
        self.bn5 = nn.BatchNorm2d(32)
        self.classifier = nn.Conv2d(32, n_class, kernel_size=1)

    def forward(self, x):
        output = self.pretrained_net(x)
        x5 = output['x5']  # size = (N, 512, x.H/32, x.W/32)
        x4 = output['x4']  # size = (N, 512, x.H/16, x.W/16)
        x3 = output['x3']  # size = (N, 256, x.H/8,  x.W/8)
        score = self.relu(self.deconv1(x5))  # size = (N, 512, x.H/16, x.W/16)
        score = self.bn1(score + x4)  # element-wise add, size = (N, 512, x.H/16, x.W/16)
        score = self.relu(self.deconv2(score))  # size = (N, 256, x.H/8, x.W/8)
        score = self.bn2(score + x3)  # element-wise add, size = (N, 256, x.H/8, x.W/8)
        score = self.bn3(self.relu(self.deconv3(score)))  # size = (N, 128, x.H/4, x.W/4)
        score = self.bn4(self.relu(self.deconv4(score)))  # size = (N, 64, x.H/2, x.W/2)
        score = self.bn5(self.relu(self.deconv5(score)))  # size = (N, 32, x.H, x.W)
        score = self.classifier(score)  # size = (N, n_class, x.H/1, x.W/1)
        return score  # size = (N, n_class, x.H/1, x.W/1)
```

---

### Demo

Now, you can execute `demo.py`:

```bash
python demo.py
```

Then `output.jpg` which is the the predicted image will appear.