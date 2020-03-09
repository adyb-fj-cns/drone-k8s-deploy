# Drone K8S Deploy Image
Simple docker image for running kubectl in a drone pipeline
## Building
```
docker build -t adybfjcns/drone-k8-deploy .
docker push adybfjcns/drone-k8-deploy
```

## Running in drone
```
global-variables:
  environment: &default_environment
    KUBERNETES_SERVER: 
      from_secret: kubernetes_server
    KUBERNETES_TOKEN:
      from_secret: kubernetes_token
    KUBERNETES_CERT:
      from_secret: kubernetes_cert
    KUBERNETES_USER: default
    KUBERNETES_NAMESPACE: default
    KUBERNETES_CLUSTER: default

kind: pipeline
type: kubernetes
name: default

steps:

- name: test
  image: alpine
  commands:
  - echo Testing

- name: publish 
  image: plugins/docker
  settings:
    username: adybfjcns
    password:
      from_secret: docker_password
    repo: adybfjcns/python-microservice
    tags: [ "${DRONE_COMMIT_SHA:0:7}","latest" ]

- name: deploy
  image: adybfjcns/drone-k8-deploy
  environment:
    <<: *default_environment
  commands:
  - kubectl config set-credentials $KUBERNETES_CLUSTER --token=$KUBERNETES_TOKEN
  - kubectl config set-credentials $KUBERNETES_CLUSTER --token=$KUBERNETES_TOKEN
  - kubectl config set clusters.$KUBERNETES_CLUSTER.certificate-authority-data $KUBERNETES_CERT
  - kubectl config set clusters.$KUBERNETES_CLUSTER.server $KUBERNETES_SERVER
  - kubectl config set-context $KUBERNETES_CLUSTER --cluster=$KUBERNETES_CLUSTER --user=$KUBERNETES_USER
  - kubectl config use-context $KUBERNETES_CLUSTER
  - kubectl get nodes
  - ls
  - kubectl apply -f ./k8s.yml
  - kubectl get pods
```

## Links
Other options using plugins
* https://github.com/honestbee/drone-kubernetes