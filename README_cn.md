# Ubuntu深度学习Docker环境

本仓库提供了一个基于Ubuntu 22.04的深度学习Docker环境，集成了CUDA 12.6.0、PyTorch和TensorFlow。它可以帮助您避免繁琐的环境配置和依赖问题。

## 环境配置

这个Docker镜像包含以下主要组件：

- **基础镜像**：nvidia/cuda:12.6.0-cudnn-runtime-ubuntu22.04
- **Python**：3.10
- 深度学习框架：
  - PyTorch（支持CUDA的最新版本）
  - TensorFlow 2.15.0
- 常用科学计算库：
  - NumPy
  - SciPy
  - Pandas
  - Scikit-learn
  - Matplotlib
  - 以及其他数据处理工具

## 使用方法

### 前提条件

- 安装了支持NVIDIA Docker的Docker
- 已安装NVIDIA驱动

> **重要提示**：确保您的NVIDIA驱动支持CUDA 12.6.0或更高版本。如果您的主机NVIDIA驱动版本不兼容，您将遇到以下错误：
>
> ```
> Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: error during container init: error running hook #0: error running hook: exit status 1, stdout: , stderr: Auto-detected mode as 'legacy'
> nvidia-container-cli: requirement error: unsatisfied condition: cuda>=12.6, please update your driver to a newer version, or use an earlier cuda container: unknown.
> ```
>
> 解决方案：
>
> 1. 更新您的NVIDIA驱动到支持CUDA 12.6的版本
> 2. 或修改Dockerfile中的基础镜像，使用与您的驱动兼容的CUDA版本

### 构建镜像

您可以使用提供的Dockerfile构建镜像：

```bash
docker build -t ubuntu2204_dl:latest -f Dockerfile_Ubuntu22_DeepL.txt .
```

### 运行容器

```bash
# 带GPU支持的容器
docker run --gpus all -it --rm ubuntu2204_dl:latest bash

# 将本地目录挂载到容器
docker run --gpus all -it --rm -v /your/local/path:/workspace ubuntu2204_dl:latest bash
```

### 创建您自己的容器

您可以基于此Dockerfile创建自定义的深度学习环境：

1. **复制并修改Dockerfile**：

   ```bash
   cp Dockerfile_Ubuntu22_DeepL.txt Dockerfile_Custom.txt
   ```

2. **编辑Dockerfile_Custom.txt**以满足您的需求，例如，添加更多库：

   ```dockerfile
   # 在Dockerfile的适当位置添加
   RUN $PIP_INSTALL \
       transformers \
       opencv-python \
       pillow \
       jupyter
   ```

3. **构建您的自定义镜像**：

   ```bash
   docker build -t my_custom_dl:v1 -f Dockerfile_Custom.txt .
   ```

4. **保存并分享您的镜像**：

   ```bash
   # 将镜像保存为tar文件
   docker save -o my_custom_dl_image.tar my_custom_dl:v1
   
   # 在另一台机器上加载镜像
   docker load -i my_custom_dl_image.tar
   ```

### 目录挂载详细指南

挂载本地目录到Docker容器的几种常见方式：

1. **基本挂载**（推荐用于开发）：

   ```bash
   docker run --gpus all -it --rm -v /host/path:/container/path ubuntu2204_dl:latest bash
   ```

2. **只读挂载**（用于数据安全）：

   ```bash
   docker run --gpus all -it --rm -v /host/path:/container/path:ro ubuntu2204_dl:latest bash
   ```

3. **挂载多个目录**：

   ```bash
   docker run --gpus all -it --rm \
     -v /host/code:/workspace \
     -v /host/data:/data \
     -v /host/output:/output \
     ubuntu2204_dl:latest bash
   ```

4. **使用数据卷**（用于数据持久化）：

   ```bash
   # 创建命名卷
   docker volume create dl_data
   
   # 使用该卷
   docker run --gpus all -it --rm -v dl_data:/data ubuntu2204_dl:latest bash
   ```

5. **使用特定用户权限挂载**（解决权限问题）：

   ```bash
   docker run --gpus all -it --rm \
     -v /host/path:/container/path \
     --user $(id -u):$(id -g) \
     ubuntu2204_dl:latest bash
   ```

## 加速Docker镜像拉取

由于网络限制，在中国大陆拉取Docker镜像可能较慢。您可以使用以下Docker镜像加速服务来提高下载速度。

### 配置Docker镜像加速

在Linux系统上，编辑或创建`/etc/docker/daemon.json`文件：

```bash
sudo mkdir -p /etc/docker
sudo vim /etc/docker/daemon.json
```

将以下内容复制到文件中：

```json
{
    "registry-mirrors": [
        "https://docker.1ms.run",
        "https://docker.xuanyuan.me"
    ],
    "runtimes": {
        "nvidia": {
            "args": [],
            "path": "nvidia-container-runtime"
        }
    }
}
```

重启Docker服务以应用配置：

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 验证安装

在容器内运行以下命令以验证环境是否正确安装：

```bash
# 验证PyTorch和CUDA
python -c "import torch; print('PyTorch version:', torch.__version__); print('CUDA available:', torch.cuda.is_available())"

# 验证TensorFlow和GPU支持
python -c "import tensorflow as tf; print('TensorFlow version:', tf.__version__); print('GPU devices:', tf.config.list_physical_devices('GPU'))"
```

## 自定义和扩展

您可以通过修改Dockerfile添加更多库和工具，例如：

```dockerfile
# 在Dockerfile的适当位置添加
RUN $PIP_INSTALL \
    transformers \
    opencv-python \
    pillow \
    jupyter
```

## 故障排除

### GPU不可用

确保您的系统已正确安装NVIDIA驱动和NVIDIA Container Toolkit：

```bash
# 安装NVIDIA Container Toolkit
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker
```

### 检查NVIDIA驱动和CUDA兼容性

您可以使用以下命令检查您的NVIDIA驱动支持哪个CUDA版本：

```bash
nvidia-smi
```

输出将显示"CUDA Version" - 确保它与您Docker镜像中的CUDA版本兼容。

## 其他资源

有关更详细的信息，请参考以下资源：

- **中国的Docker/DockerHub镜像**：有关中国可用的Docker注册表镜像的完整列表，请访问：https://cloud.tencent.com/developer/article/2485043
- **Dockerfile参考**：本项目受deepo项目启发。有关更多Dockerfile示例和最佳实践，请查看：https://github.com/ufoym/deepo/tree/master/docker
- **NVIDIA驱动、CUDA和cuDNN兼容性**：有关内核版本、NVIDIA驱动、CUDA和cuDNN兼容性以及安装指南的详细信息，请参考：https://blog.csdn.net/qq_23076153/article/details/112754824
- **PyTorch环境安装**：有关PyTorch、torchvision和Python之间的版本兼容性以及环境安装指南，请访问：https://blog.csdn.net/trr27/article/details/144162171

## 许可证

本项目基于MIT许可证。有关详细信息，请参阅LICENSE文件。

## 作者

秦文杰、胡敬平（hujp@hust.edu.cn）