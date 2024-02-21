## Stages
```
stages:
  - build
  - migrate
  - deploy
  - rollback
```
## Job
```
deploy_prod:
  stage: deploy
  image: git.zafarsaidov.uz:5050/devops/docker:dind
  script:
    - echo "Hi"
```
## Template
### Define
```
.build_template:
  stage: build
  image: git.zafarsaidov.uz:5050/devops/docker:dind
  before_script:
    - docker login $CI_REGISTRY --username $CI_REGISTRY_USER --password $CI_REGISTRY_PASSWORD
  script:
    - echo $MESSAGE
  tags:
    - qa
    - server
```
### Call
```
build-staging:
  stage: build
  extends: .build_template
  variables:
    MESSAGE: Hello
  only:
    - staging
```
### Include from file
```
include:
  - .gitlab/ci/*.gitlab-ci.yml
```

### Include from repo

```
stages:
  - vm_bootstrap
  - kube_monitoring

variables:
  AWX_INVENTORY_ID: 3
  VM_BOOTSTRAP_LIMIT: ""
  PRE_BOOTSTRAP_LIMIT: ""
  KUBE_MONITORING_LIMIT: "localhost"
  TELEPORT_ROLES_LIMIT: "teleport.prxm.uz"


include:
  - project: 'ci/collections'
    ref: 'main'
    file: 'vm_bootstrap.yml'
  - project: 'ci/collections'
    ref: 'main'
    file: 'kube_monitoring.yml'
```