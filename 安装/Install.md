# Python 相关包



## pysyft



- linux

```bash
conda create -n pysyft python=3.7
conda activate pysyft # some older version of conda require "source activate pysyft" instead.
conda install jupyter notebook
```

```bash
pip install 'syft[udacity]'
```



- windows

```bash
conda create -n pysyft python=3.7
conda activate pysyft
```

```bash
conda install pytorch torchvision cudatoolkit=10.1 -c pytorch # cuda
conda install pytorch torchvision cpuonly -c pytorch # cpu-only
pip install torch==1.4.0+cpu torchvision==0.5.0+cpu -f https://download.pytorch.org/whl/torch_stable.html # 实际使用
```

```bash
git clone https://github.com/OpenMined/PySyft.git
cd PySyft
```

```bash
pip install -r pip-dep/requirements.txt
```

安装tensorflow 1.14.0  和 tf_encrypted

```bash
pip install https://storage.googleapis.com/tensorflow/mac/cpu/tensorflow-1.14.0-py3-none-any.whl
```

安装其他包

```bash
pip install -r pip-dep/requirements_udacity.txt #这个安装的是tf_encrypted
pip install syft[udacity]
```



遇到的问题：

```html
ModuleNotFoundError: No module named '_pywrap_tensorflow_internal'--解决方法
```

tensorflow安装的问题。我的Python==3.8。无法兼容低版本的tensorflow. 换成python=3.7可解决。

