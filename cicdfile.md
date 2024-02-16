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