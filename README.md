# Gitlab Server Install
## Dependencies
```
sudo apt-get update
sudo apt-get install -y curl openssh-server ca-certificates tzdata perl
sudo apt-get install -y postfix
```

## Set gitlab repository
```
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
```

## Show available-versions
```
apt-cache madison gitlab-ce 
```

Check all ports in use

## Install gitlab-ce package
```
sudo EXTERNAL_URL="https://gitlab.zafarsaidov.uz" apt-get install gitlab-ce
```

## Install Gitlab-runner
```
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash

sudo apt-get install gitlab-runner
```

kubectl create secret docker-registry gitlab-registry  --docker-server=gitlab.cloudgate.uz:5050 --docker-username=unisoft --docker-password=aetheiCahNg9uopu --docker-email=zafar.saidov@shipox.com