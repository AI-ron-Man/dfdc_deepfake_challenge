ARG PYTORCH="1.10.0"
ARG CUDA="11.3"
ARG CUDNN="8"

FROM pytorch/pytorch:${PYTORCH}-cuda${CUDA}-cudnn${CUDNN}-devel

ENV TORCH_NVCC_FLAGS="-Xfatbin -compress-all"
ENV CMAKE_PREFIX_PATH="$(dirname $(which conda))/../"

# Setting noninteractive build, setting up tzdata and configuring timezones
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=America/Los_Angeles
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Add the NVIDIA GPG key and update packages
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys A4B469963BF863CC
RUN apt-get update && apt-get install -y gnupg2 \
    libglib2.0-0 libsm6 libxrender-dev libxext6 nano mc glances vim git \
    build-essential cmake libopenblas-dev liblapack-dev libx11-dev libgtk-3-dev wget \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# Install Python packages
RUN conda install cython -y && conda clean --all
RUN /opt/conda/bin/pip install --upgrade pip

# Explicitly install compatible torchvision version for PyTorch 1.10.0
RUN /opt/conda/bin/pip install torch==1.10.0+cu113 torchvision==0.11.1+cu113 -f https://download.pytorch.org/whl/torch_stable.html

RUN /opt/conda/bin/pip install packaging dlib facenet-pytorch albumentations==1.0.0 timm==0.4.12 pytorch_toolbelt tensorboardx \
    cython jupyter jupyterlab ipykernel matplotlib tqdm pandas

# Clone and install Apex
RUN git clone https://github.com/NVIDIA/apex
RUN cd apex && /opt/conda/bin/pip install -v --disable-pip-version-check --no-cache-dir --no-build-isolation --config-settings "--build-option=--cpp_ext" --config-settings "--build-option=--cuda_ext" ./

# Download pretrained Imagenet models
RUN wget https://github.com/rwightman/pytorch-image-models/releases/download/v0.1-weights/tf_efficientnet_b7_ns-1dbc32de.pth -P /root/.cache/torch/hub/checkpoints/
RUN wget https://github.com/rwightman/pytorch-image-models/releases/download/v0.1-weights/tf_efficientnet_b5_ns-6f26d0cf.pth -P /root/.cache/torch/hub/checkpoints/

# Setting the working directory
WORKDIR /workspace

# Copying the required codebase
COPY . /workspace

RUN chmod 777 preprocess_data.sh
RUN chmod 777 train.sh
RUN chmod 777 predict_submission.sh

ENV PYTHONPATH=.

CMD ["/bin/bash"]
