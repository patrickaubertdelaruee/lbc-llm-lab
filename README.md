# ollama_weaviate

Docker compose setup for ollama vector embedding lab

# ssh to your vm with key...
ssh -i ./mydowloadedkey.pem yourusername@thevmipaddress
# or password
ssh yourusername@thevmipaddress

# verify you have the right vm provisioned
lspci | grep -i NVIDIA

# install hw drivers
sudo apt update && sudo apt install -y ubuntu-drivers-common git
sudo ubuntu-drivers install
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb
sudo apt install -y ./cuda-keyring_1.1-1_all.deb
sudo apt update
sudo apt -y install cuda-toolkit-12-5
sudo reboot

# ssh back in and verify hw support
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# ubuntu 20.04 only provides docker-compose v1.25.0, we need >= 1.28.0 for hw support in compose so wer work around this:
cd /tmp
wget https://github.com/docker/compose/releases/download/1.29.0/docker-compose-Linux-x86_64
sudo mv /tmp/docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version

# configure docker
sudo nvidia-ctk runtime configure --runtime=docker && \
sudo systemctl restart docker

# get repos
cd ~
git clone https://github.com/patrickaubertdelaruee/ollama_weaviate.git
git clone https://github.com/patrickaubertdelaruee/rag_fca

# check hw support in containers
docker logs ollama 2>&1 | grep V100
ollama_weaviate_nvidia-test_1


