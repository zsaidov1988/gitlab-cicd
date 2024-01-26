## Helm deploy
```
.deploy_template:
  stage: deploy
  image: gitlab.zafarsaidov.uz:5050/devops/docker:dind
  script:
    - echo $K8SCONFIGJSON > tmp
    - yq -P tmp > /root/.kube/config
    - chmod 600 /root/.kube/config
    - DEPLOYMENT=$(echo $CI_PROJECT_NAME | sed s/_/-/g | sed s/$CI_PROJECT_NAMESPACE-//g)
    - helm repo add --username $HELM_REGISTRY_USERNAME --password $HELM_REGISTRY_PASSWORD $HELM_REPO_NAME $HELM_REGISTRY_PATH
    - helm upgrade --install $DEPLOYMENT $HELM_REPO_NAME/$HELM_CHART_NAME --set=image.tag=$CI_PIPELINE_IID --values $VALUES_FILE -n $NAMESPACE
```
```
build-staging:
  stage: build
  extends: .build_template
  variables:
    ENV_TAG: test
    DOCKERFILE: Dockerfile
  only:
    - staging
```
## Helm rollback
```
.rollback_template:
  stage: rollback
  image: gitlab.cloudgate.uz:5050/devops/docker:dind
  script:
    - echo "$KUBECONFIGJSON" | yq -P - > ~/.kube/config
    - kubectl config set-context --current --namespace=$NAMESPACE
    - DEPLOYMENT=$(echo $CI_PROJECT_NAME | sed s/_/-/g | sed s/$CI_PROJECT_NAMESPACE-//g)
    - helm rollback $DEPLOYMENT
```
```
rollback-staging:
  stage: rollback
  extends: .rollback_template
  variables:
    NAMESPACE: test
    K8SCONFIGJSON: $KUBECONFIG
  when: manual
```
## Kubectl deploy
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test-app
  name: test-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test-app
  template:
    metadata:
      labels:
        app: test-app
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: apps
                    operator: In
                    values:
                      - "true"
      imagePullSecrets:
        - name: gitlab-registry
      containers:
          name: test-app
          imagePullPolicy: Always
          image: gitlab.zafarsaidov.uz:5050/test/app:latest
      restartPolicy: Always
---
apiVersion: v1
kind: Service
metadata:
  name: test-app
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: test-app
```
.gitlab-ci.yml
```
variables:
  NAMESPACE: "default"
  DEPLOYMENT: test

before_script:
  ## docker login
  - docker login $CI_REGISTRY --username $CI_REGISTRY_USER --password $CI_REGISTRY_PASSWORD
  ## install dependencies
  - apk update && apk --no-cache --update add build-base openssh curl

stages:
  - build
  - deploy

build_image_prod:
  stage: build
  script:
    - make build-image TAG=$CI_PIPELINE_IID PROJECT_NAME=$CI_PROJECT_NAMESPACE APP=$CI_PROJECT_NAME REGISTRY=$CI_REGISTRY ENV_TAG=latest
    - make push-image TAG=$CI_PIPELINE_IID PROJECT_NAME=$CI_PROJECT_NAMESPACE APP=$CI_PROJECT_NAME REGISTRY=$CI_REGISTRY ENV_TAG=latest
  only:
    - master

deploy_to_prod:
  stage: deploy
  script:
    - curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
    - chmod +x ./kubectl && mkdir -p ~/.kube && mv ./kubectl ~/.kube && export PATH=$PATH:$HOME/.kube
    - cp $KUBECONFIG ~/.kube/config
    - kubectl apply -f .kube/prod/
    - kubectl set image -n $NAMESPACE deployment/$DEPLOYMENT $DEPLOYMENT=$CI_REGISTRY/$CI_PROJECT_NAMESPACE/$CI_PROJECT_NAME:$CI_PIPELINE_IID
    - rm -rf ~/.kube
  only:
    - master
```
Makefile
```
CURRENT_DIR=$(shell pwd)
APP=$(shell basename ${CURRENT_DIR})
REGISTRY=gitlab.zafarsaidov.uz:5050
TAG=latest
ENV_TAG=latest

build-image:
	docker build --rm -t ${REGISTRY}/${PROJECT_NAME}/${APP}:${TAG} .
	docker tag ${REGISTRY}/${PROJECT_NAME}/${APP}:${TAG} ${REGISTRY}/${PROJECT_NAME}/${APP}:${ENV_TAG}

push-image:
	docker push ${REGISTRY}/${PROJECT_NAME}/${APP}:${TAG}
	docker push ${REGISTRY}/${PROJECT_NAME}/${APP}:${ENV_TAG}
```