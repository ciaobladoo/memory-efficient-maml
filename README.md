# Memory Efficient MAML

### Overview

PyTorch implementation of Model-Agnostic Meta-Learning[[1]](#references) with 
gradient checkpointing[[2]](#references). It allows you to perform way (~10-100x) more
MAML steps with the same GPU memory budget. 


### Install

For normal installation, run
```pip install torch_maml```

For development installation, clone a repo and
```python setup.py develop```


### How to use:
See examples in [```example.ipynb```](./example.ipynb)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/dbaranchuk/memory-efficient-maml/blob/master/example.ipynb)


### Tips and tricks
1) Make sure that your model doesn't have implicit parameter updates like 
torch.nn.BatchNorm2d under track_running_stats=True. With gradient checkpointing,
 these updates will be performed twice (once per forward pass). If still want these
 updates, take a look at [```torch_maml.utils.disable_batchnorm_stats```](torch_maml/utils.py#L86-L101).
 Note that we already support this for vanilla BatchNorm{1-3}d.

2) When computing gradients through many MAML steps (e.g. 100 or 1000),
you should care about vanishing and exploding gradients within
optimizers (same as in RNN). This implementation supports gradient clipping 
to avoid the explosive part of the problem.

3) Also, when you deal with a large number of MAML steps, be aware of 
accumulating computational error due to float precision and specifically
CUDNN operations. We recommend you to use 
```torch.backend.cudnn.determistic=True```. The problem appears when
gradients become slightly noisy due to errors, and, 
during backpropagation through MAML steps, the error is likely to increase dramatically.  

4) You could also consider Implicit Gradient MAML [[3]](#references) for memory efficient meta-learning alternatives. While this algorithm requires even less memory, it assumes that your optimization converges to the optimum. Therefore, it is inapplicable if your task does not always converge by the time you start backpropagating. In contrast, our implementation allows you to meta-learn even from a partially converged state. 
 
### References

[1] [Model-Agnostic Meta-Learning for Fast Adaptation of Deep Networks](http://proceedings.mlr.press/v70/finn17a/finn17a.pdf)

[2] [Gradient checkpointing technique (GitHub)](https://github.com/cybertronai/gradient-checkpointing)

[3] [Meta-Learning with Implicit Gradients](https://arxiv.org/pdf/1909.04630.pdf)
