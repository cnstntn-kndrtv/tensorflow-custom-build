# Сборка TensorFlow под процессор
Зачем? Для инференса моделек на CPU. Работает быстрее чем то что в публичном репозитории TF.  

Пример для такого конфига:  
  
OS: Centos  
CPU: Intel Xenon E5645  
Python: 3.6  
TensorFlow: 1.13  


## Pre

```sh
# python
sudo yum install python3-dev python3-pip patch

python3 -m venv env
source ./env/bin/activate

pip install -r requirements.txt

# bazel == 0.21.0s
sudo yum install wget
wget https://github.com/bazelbuild/bazel/releases/download/0.21.0/bazel-0.21.0-installer-linux-x86_64.sh
chmod +x ./bazel-0.21.0-installer-linux-x86_64.sh
sudo ./bazel-0.21.0-installer-linux-x86_64.sh

# TF
git clone https://github.com/tensorflow/tensorflow.git
cd ./tensorflow
git checkout r1.13


./configure
```

Параметры:    
```

(env) [...]$ ./configure
WARNING: --batch mode is deprecated. Please instead explicitly shut down your Bazel server using the command "bazel shutdown".
INFO: Invocation ID: bc5d4c66-7507-4a5a-a3e4-c02edaadb1dd
You have bazel 0.21.0 installed.
Please specify the location of python. [Default is ...]:


Found possible Python library paths:
  /home/trinidata/tensorflow-for-XenonE5645/env/lib/python3.6/site-packages
  /home/trinidata/tensorflow-for-XenonE5645/env/lib64/python3.6/site-packages
Please input the desired Python library path to use.  Default is [...]

#### ??? #accelerated linear algebra - во втором варинте добавил
Do you wish to build TensorFlow with XLA JIT support? [Y/n]: n 
No XLA JIT support will be enabled for TensorFlow.

#### альтернатива CUDA, может поддерживать GPU, CPU (!) любых производителей. Пишут что или Cuda или MKL - выбрал MKL, убрал OpenCL
Do you wish to build TensorFlow with OpenCL SYCL support? [y/N]: n 
No OpenCL SYCL support will be enabled for TensorFlow.

#### AMD
Do you wish to build TensorFlow with ROCm support? [y/N]: n 
No ROCm support will be enabled for TensorFlow.

### GPU
Do you wish to build TensorFlow with CUDA support? [y/N]: n
No CUDA support will be enabled for TensorFlow.


Do you wish to download a fresh release of clang? (Experimental) [y/N]: n 
Clang will not be downloaded.


#### Message Passing Interface - для параллельной работы 
Do you wish to build TensorFlow with MPI support? [y/N]: n ##
No MPI support will be enabled for TensorFlow.

#### native = optimized for local CPU
Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native -Wno-sign-compare]: ↩︎ 

Would you like to interactively configure ./WORKSPACE for Android builds? [y/N]: n
Not configuring the WORKSPACE for Android builds.

Preconfigured Bazel build configs. You can use any of the below by adding "--config=<>" to your build command. See .bazelrc for more details.
	--config=mkl         	# Build with MKL support.
	--config=monolithic  	# Config for mostly static monolithic build.
	--config=gdr         	# Build with GDR support.
	--config=verbs       	# Build with libverbs support.
	--config=ngraph      	# Build with Intel nGraph support.
	--config=dynamic_kernels	# (Experimental) Build kernels into separate shared objects.
Preconfigured Bazel build configs to DISABLE default on features:
	--config=noaws       	# Disable AWS S3 filesystem support.
	--config=nogcp       	# Disable GCP support.
	--config=nohdfs      	# Disable HDFS support.
	--config=noignite    	# Disable Apacha Ignite support.
	--config=nokafka     	# Disable Apache Kafka support.
	--config=nonccl      	# Disable NVIDIA NCCL support.
Configuration finished
```

## cpu-only  
```sh
bazel build --config=opt //tensorflow/tools/pip_package:build_pip_package
```

Если не хватает места - можно перенести папку с кэшем bazel ключом `--output_user_root=/path/to/directory`
сейчас - ~/.cache/bazel  
```
bazel --output_user_root=<bazel-cache-dir> build --config=opt //tensorflow/tools/pip_package:build_pip_package
```
можно удалить `<bazel-cache-dir>` после всех установок  

### mkl
ключ `--config=mkl`
```
bazel --output_user_root=<bazel-cache-dir> build --config=mkl --config=opt //tensorflow/tools/pip_package:build_pip_package
```

Собрать MKL?  
https://github.com/intel/mkl-dnn/blob/master/doc/build/build.md
нужен этот ключ `DNN_ARCH_OPT_FLAGS=-xSSE4.2 make -j`
  
возможно пригодится:  
```sh
yum group install "Development Tools"
```

Проверка после установки:
```python
import tensorflow as tf
tf.pywrap_tensorflow.IsMklEnabled()
```
  
## build pip package  
```sh
./bazel-bin/tensorflow/tools/pip_package/build_pip_package ../tensorflow_pkg
# ./bazel-bin/tensorflow/tools/pip_package/build_pip_package ../tensorflow_pkg_mkl
```

## use in app
```sh
pip install ~/tensorflow-for-XenonE5645/tensorflow_pkg/tensorflow-1.13.2-cp36-cp36m-linux_x86_64.whl
```


 =======
 = MKL =
 =======
github...intel..mkl-dnn/docs/build
DNN_ARCH_OPT_FLAGS=-xSSE4.2 make -j


