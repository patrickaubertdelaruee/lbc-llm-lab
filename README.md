# lbc selfmanaged llm lab

Docker compose setup for ollama vector embedding lab

## connect
ssh to your vm with key 
```
ssh -i yourkey.pem username@ip
```	
or password
```
ssh -i yourkey.pem username@ip
```	

## verify your vm
```
lspci | grep -i NVIDIA
```

## install hw drivers
```
sudo apt update && sudo apt install -y ubuntu-drivers-common git
sudo ubuntu-drivers install
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb
sudo apt install -y ./cuda-keyring_1.1-1_all.deb
sudo apt update
sudo apt -y install cuda-toolkit-12-5
sudo reboot
```

## verify hw support
ssh back in and verify
```
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```
## install docker-compose
ubuntu 20.04 only provides docker-compose v1.25.0, we need >= 1.28.0 for hw support in compose so wer work around this:
```
wget https://github.com/docker/compose/releases/download/1.29.0/docker-compose-Linux-x86_64
sudo mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
sudo nvidia-ctk runtime configure --runtime=docker && \
sudo systemctl restart docker
```

## get repositories and start containers
```
cd ~
git clone https://github.com/patrickaubertdelaruee/ollama_weaviate.git
git clone https://github.com/patrickaubertdelaruee/rag_fca
docker logs ollama 2>&1 | grep V100
docker logs ollama_weaviate_nvidia-test_1
```

### configure through ui
ollama web ui: http://ip:8080/
ollama api: http://ip:11434/
jupyter notebook http://ip:8888

load the following models in ollama web ui:
* mistral
* nomic-embed-text

put one or more pdf files into rag_fca/PDF/ by scp from your local machine:
```
scp filename.pdf username@ip:/home/username/rag_fca/PDF/
```

load FullyLocal.ipynb notebook in jupyter and execute
