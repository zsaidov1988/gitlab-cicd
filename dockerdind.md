## .gitlab-ci
```
before_script:
  ## docker login
  - docker login $CI_REGISTRY --username $CI_REGISTRY_USER --password $CI_REGISTRY_PASSWORD

stages:
  - build

build_image_prod:
  stage: build
  image: docker:dind
  script:
    - docker build -t registry-gitlab.kpi.com/docker/docker:dind .
    - docker push registry-gitlab.kpi.com/docker/docker:dind
  only:
    - master
```
## Dockerfile
```
FROM docker:dind
RUN apk update \
    && apk --no-cache --update add wget bash build-base curl openssh-client sshpass
RUN curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
RUN chmod +x ./kubectl \
    && mv kubectl /usr/local/bin/ \
    && mkdir -p ~/.kube \
    && export PATH=$PATH:$HOME/.kube
RUN wget https://get.helm.sh/helm-v3.7.1-linux-amd64.tar.gz \
    && tar -xzvf helm-v3.7.1-linux-amd64.tar.gz
RUN chmod +x linux-amd64/helm \
    && mv linux-amd64/helm /usr/local/bin/ \
    && rm -rf helm-v3.7.1-linux-amd64.tar.gz
RUN wget https://github.com/golang-migrate/migrate/releases/download/v4.14.1/migrate.linux-amd64.tar.gz \
    && tar -xvf migrate.linux-amd64.tar.gz \
    && mv migrate.linux-amd64 migrate \
    && chmod +x migrate \
    && mv migrate /usr/local/bin/ \
    && rm -rf migrate.linux-amd64.tar.gz
RUN wget https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 -O /usr/local/bin/yq &&\
    chmod +x /usr/local/bin/yq
RUN curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin v0.47.0

RUN curl -O https://cdn.teleport.dev/teleport-v14.1.0-linux-amd64-bin.tar.gz && tar -xvf teleport-v14.1.0-linux-amd64-bin.tar.gz
RUN ./teleport/install

CMD ["bash”]
```