# SqoopBigData

### Installing Docker into Machine (Cloud instance or VM)
```
sudo apt-get update
```

```
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

```
sudo mkdir -m 0755 -p /etc/apt/keyrings
```

```
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```
sudo apt-get update
```

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Changing the docker permission
```
sudo usermod -aG docker <username>
```

```
newgrp docker
```

### Run the cloudera image
```
docker run --hostname=quickstart.cloudera --privileged=true -t -i -v /home/ubuntu:/Src --publish-all=true -p 8888 cloudera/quickstart:latest /usr/bin/docker-quickstart
```
