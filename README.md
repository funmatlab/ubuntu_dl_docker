# Ubuntu Deep Learning Docker Environment

This repository provides a Docker environment for deep learning based on Ubuntu 22.04, integrating CUDA 12.6.0, PyTorch, and TensorFlow. It helps you avoid tedious environment configuration and dependency issues.

## Environment Configuration

This Docker image contains the following main components:

- **Base Image**: nvidia/cuda:12.6.0-cudnn-runtime-ubuntu22.04
- **Python**: 3.10
- Deep Learning Frameworks:
  - PyTorch (latest version with CUDA support)
  - TensorFlow 2.15.0
- Common Scientific Computing Libraries:
  - NumPy
  - SciPy
  - Pandas
  - Scikit-learn
  - Matplotlib
  - And other data processing tools

## Usage

### Prerequisites

- Docker with NVIDIA Docker support installed
- NVIDIA drivers installed

> **Important Note**: Ensure your NVIDIA driver supports CUDA 12.6.0 or higher. If your host NVIDIA driver version is incompatible, you will encounter the following error:
>
> ```
> Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: error during container init: error running hook #0: error running hook: exit status 1, stdout: , stderr: Auto-detected mode as 'legacy'
> nvidia-container-cli: requirement error: unsatisfied condition: cuda>=12.6, please update your driver to a newer version, or use an earlier cuda container: unknown.
> ```
>
> Solutions:
>
> 1. Update your NVIDIA driver to a version that supports CUDA 12.6
> 2. Or modify the base image in the Dockerfile to use a CUDA version compatible with your driver

### Building the Image

You can build the image using the provided Dockerfile:

```bash
docker build -t ubuntu2204_dl:latest -f Dockerfile_Ubuntu22_DeepL.txt .
```

### Running the Container

```bash
# Container with GPU support
docker run --gpus all -it --rm ubuntu2204_dl:latest bash

# Mount local directory to container
docker run --gpus all -it --rm -v /your/local/path:/workspace ubuntu2204_dl:latest bash
```

### Creating Your Own Container

You can create a custom deep learning environment based on this Dockerfile:

1. **Copy and modify the Dockerfile**:

   ```bash
   cp Dockerfile_Ubuntu22_DeepL.txt Dockerfile_Custom.txt
   ```

2. **Edit Dockerfile_Custom.txt** to meet your needs, for example, to add more libraries:

   ```dockerfile
   # Add at the appropriate location in the Dockerfile
   RUN $PIP_INSTALL \
       transformers \
       opencv-python \
       pillow \
       jupyter
   ```

3. **Build your custom image**:

   ```bash
   docker build -t my_custom_dl:v1 -f Dockerfile_Custom.txt .
   ```

4. **Save and share your image**:

   ```bash
   # Save the image as a tar file
   docker save -o my_custom_dl_image.tar my_custom_dl:v1
   
   # Load the image on another machine
   docker load -i my_custom_dl_image.tar
   ```

### Detailed Guide to Directory Mounting

There are several common ways to mount local directories to a Docker container:

1. **Basic mount** (recommended for development):

   ```bash
   docker run --gpus all -it --rm -v /host/path:/container/path ubuntu2204_dl:latest bash
   ```

2. **Read-only mount** (for data security):

   ```bash
   docker run --gpus all -it --rm -v /host/path:/container/path:ro ubuntu2204_dl:latest bash
   ```

3. **Mounting multiple directories**:

   ```bash
   docker run --gpus all -it --rm \
     -v /host/code:/workspace \
     -v /host/data:/data \
     -v /host/output:/output \
     ubuntu2204_dl:latest bash
   ```

4. **Using data volumes** (for data persistence):

   ```bash
   # Create a named volume
   docker volume create dl_data
   
   # Use the volume
   docker run --gpus all -it --rm -v dl_data:/data ubuntu2204_dl:latest bash
   ```

5. **Mounting with specific user permissions** (resolving permission issues):

   ```bash
   docker run --gpus all -it --rm \
     -v /host/path:/container/path \
     --user $(id -u):$(id -g) \
     ubuntu2204_dl:latest bash
   ```

## Accelerating Docker Image Pulling

Due to network limitations, pulling Docker images in mainland China may be slow. You can use the following Docker image acceleration services to improve download speed.

### Configuring Docker Image Acceleration

On Linux systems, edit or create the `/etc/docker/daemon.json` file:

```bash
sudo mkdir -p /etc/docker
sudo vim /etc/docker/daemon.json
```

Copy the following content to the file:

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

Restart the Docker service to apply the configuration:

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## Verifying the Installation

Run the following commands inside the container to verify the environment is correctly installed:

```bash
# Verify PyTorch and CUDA
python -c "import torch; print('PyTorch version:', torch.__version__); print('CUDA available:', torch.cuda.is_available())"

# Verify TensorFlow and GPU support
python -c "import tensorflow as tf; print('TensorFlow version:', tf.__version__); print('GPU devices:', tf.config.list_physical_devices('GPU'))"
```

## Customization and Extension

You can add more libraries and tools by modifying the Dockerfile, for example:

```dockerfile
# Add at the appropriate location in the Dockerfile
RUN $PIP_INSTALL \
    transformers \
    opencv-python \
    pillow \
    jupyter
```

## Troubleshooting

### GPU Not Available

Ensure your system has correctly installed NVIDIA drivers and the NVIDIA Container Toolkit:

```bash
# Install NVIDIA Container Toolkit
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker
```

### Checking NVIDIA Driver and CUDA Compatibility

You can use the following command to check which CUDA version your NVIDIA driver supports:

```bash
nvidia-smi
```

The output will show "CUDA Version" - make sure it's compatible with the CUDA version in your Docker image.

## Additional Resources

For more detailed information, please refer to the following resources:

- **Docker/DockerHub Mirrors in China**: For a comprehensive list of Docker registry mirrors available in China, visit: https://cloud.tencent.com/developer/article/2485043
- **Dockerfile References**: This project was inspired by the deepo project. For more Dockerfile examples and best practices, check: https://github.com/ufoym/deepo/tree/master/docker
- **NVIDIA Driver, CUDA, and cuDNN Compatibility**: For detailed information about kernel version, NVIDIA driver, CUDA, and cuDNN compatibility and installation guides, refer to: https://blog.csdn.net/qq_23076153/article/details/112754824
- **PyTorch Environment Installation**: For version compatibility between PyTorch, torchvision, and Python, as well as environment installation guides, visit: https://blog.csdn.net/trr27/article/details/144162171

## License

This project is licensed under the MIT License. See the LICENSE file for details.
