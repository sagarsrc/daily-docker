# 1. Log in to Docker Hub
sudo docker login

# 2. Build your CUDA image directly
sudo docker build -t sagx/nvidia-cuda-ai-dev:latest -f Dockerfile .

# 3. Push to Docker Hub
sudo docker push sagx/nvidia-cuda-ai-dev:latest