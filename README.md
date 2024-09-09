# lbc selfmanaged llm lab

The purpose of this lab is to set up a self hosted llm with hardware acceleration. In the process you will make yourself familiar with the following core concepts and technologies:  

* Azure Portal and Virtual Machines
* Docker
* The linux command line and basic tools
* LLM
* RAG
* Python & Jupyter Notebook

## Preparation material for basic concepts and context:

### Prio1 (Must view):  
Terminology and context. From AI/ML/DL/GenAI/LLM... https://www.youtube.com/watch?v=qYNweeDHiyU  => this is the referenced video, totally optional: https://www.youtube.com/watch?v=4RixMPF4xis
Intro to LLM: https://www.youtube.com/watch?v=5sLYAQS9sWQ  
CUDA: https://www.youtube.com/watch?v=pPStdjuYzSI  (until 1:43, unless you want a C++ CUDA implementation crash course))
Vector DB & vector embedding: https://www.youtube.com/watch?v=dN0lsF2cvm4  (We will use Weaviate)
RAG: https://www.youtube.com/watch?v=T-D1OfcDW1M  
Explains what we're doing in the supplied jupyter notebook. Follow along. If there are details that you don't understand (e.g. the code) don't worry, try to get the bigger picture. We will also be using mistral and nomic-text-embed. Relevant until 4:35): https://www.youtube.com/watch?v=jENqvjpkwmw   
   
while at it, why not get some understanding of linear regression: https://www.youtube.com/watch?v=qxo8p8PtFeA   

  
### Prio2 (based on interest/background, not required for lab):  
CPU vs GPU (why are we using GPU enabled VMs for this lab?): https://www.youtube.com/watch?v=LfdK-v0SbGI  
container/docker: https://www.youtube.com/watch?v=0qotVMX-J5s  
Open source LLMs (contaxt of llama or mistral): https://www.youtube.com/watch?v=y9k-U9AuDeM  
What makes LLMs expensive: https://www.youtube.com/watch?v=7gMg98Hf3uM  
Git in 2minutes: https://www.youtube.com/watch?v=BZr7oJyh4WM or more depth in 4min: https://www.youtube.com/watch?v=e9lnsKot_SQ  
Vector databases (e.g. weaviate): https://www.youtube.com/watch?v=t9IDoenf-lo  

## Prerequisite for lab  
Bring a computer or tablet to the lab with a browser and ssh client (e.g. Linux, MacOS, Windows with WSL, Windows with Putty...)

# Azure & docker-compose setup for ollama vector embedding lab
## set up your vm
see https://www.youtube.com/watch?v=OCiN37sjXuw  
go to https://portal.azure.com/  
Create a resource, Virtual Machine  
for os pick NVIDIA GPU-Optimized VMI with vGPU driver (Nvidia enabled Ubuntu os) v22.08.0 x64 Gen1  
choose a nvidia enabled machine, any NC* will do, e.g. NC12s in Switzerland North (i.e. NC12s_v3)    
Choose Security Type Standard, Azure-selected Zone, Eviction = Stop
Choose Spot pricing  
When configuring disk space, ensure you reserve 64GB.
set a username/password or download the ssh key. Don't loose file or u/p  
Go to Networking Tab & Create Public IP with default settings
In Networking Tab: open up inbound TCP Ports 8080 & 8888
Hit Review & Create  
  
Goto newly created Resource, copy the IP address to access host via ssh and browser. You'll find the IP under networking as public IP address or in the ressource group in a file called *-ip*
  
If it displays some error above like * virtual machine agent status is not ready then restart the VM with the button on top... be patient :)  
  
Check disksize on newly created VM under Disk  
If disk < 64GB, expand disk with: https://learn.microsoft.com/en-us/azure/virtual-machines/linux/expand-disks?tabs=ubuntu   

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
sudo dpkg --configure -a
sudo apt install -y ubuntu-drivers-common git  
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
cat /proc/driver/nvidia/version
nvcc -V
```

## install docker-compose
ubuntu 20.04 only provides docker-compose v1.25.0, we need >= 1.28.0 for hw support in compose so we work around this:
```
wget https://github.com/docker/compose/releases/download/1.29.0/docker-compose-Linux-x86_64
sudo mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
``` 

## prepare installation of nvidia container toolkit
```
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker && \
sudo systemctl restart docker
```

## get repositories and start containers  
Start of docker container takes up to 5 mins... be patient :)
```
cd ~
git clone https://github.com/patrickaubertdelaruee/lbc-llm-lab.git
git clone https://github.com/patrickaubertdelaruee/rag_fca
cd lbc-llm-lab/  
docker-compose up -d  
docker logs ollama 2>&1 | grep V100
docker logs ollama_weaviate_nvidia-test_1
```

### configure through ui
ollama web ui: http://ip:8080/  
ollama api: http://ip:11434/  
jupyter notebook http://ip:8888  

load the following models in ollama web ui under Head Icon top right/Admin Panel/Settings/Models.  
Models can be found on https://ollama.com/library :  
* mistral  
* nomic-embed-text  

give write access to PDF directory:
```
sudo chmod 777 ~/rag_fca/PDF  
```

on your vm: download, unip and put one or more pdf files into rag_fca/PDF/ by scp from your local machine.
Password is supplied during the workshop   
```
mkdir ~/files
wget -O ~/files/files.zip https://drive.usercontent.google.com/download\?id\=1-FzpTEk4hbhfJlbNq3dogqygWQlIqniJ\&export\=download\&authuser\=1\&confirm\=t\&uuid\=cd9d42bb-2c3e-493e-9647-d04fb2047f33\&at\=AO7h07exuyQMTew243UlnbqKePsk%3A1725883485592
cd ~/files && unzip files.zip
cp FILENAME.pdf ~/rag_fca/PDF/ # replace FILENAME.pdf with an actual filenam
```

load FullyLocal.ipynb notebook in jupyter and execute by running each segment. Verify success and run next. The last command step of the script executes the prompt and outputs the response.   
Congratulations, you've now set up a self hosted open llm with RAG

Hint: Start by loading one PDF file and verifying successfull embedding through a query before loading more files. These operations take time and you will only know once finished loading if the operation was successfull ;)


## Lab exercises   


1. Time vectorizing and query of a document with the default hardware accelerated setup. Disable hardware acceleration and redo timing. How much better or worse did the lab setup perform?  

2. Find out what the machine cost was for the lab (so far). How much would you expect to pay per day in continuous operation?  

3. Query the API from the commandline. Hint: you can send requests to web services with curl. The API is well documented.  

4. Tune the prompt and observe if response quality improves  

