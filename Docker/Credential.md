## Commandline

```bash
# default to https://hub.docker.com/
# cred will be stored at ~/.docker/config.json
$ docker login
$ docker login ur-private-registry.com

# not recommended
$ docker login -u ur-username -p ur-password
```

## Config

```bash
$ ENCODED=`echo -n "ur-username:ur-password" | base64`

$ cat > ~/.docker/config.json << EOF
{
  "auths": {
    "https://index.docker.io/v1/": {
      "auth": "${ENCODED}"
    }
  }
}
EOF

# private reg
$ cat > ~/.docker/config.json << EOF
{
  "auths": {
    "ur-rivate-registry.com": {
      "auth": "${ENCODED}"
    }
  }
}
EOF
```

## ENV

```bash
# private reg
$ export DOCKER_REGISTRY_URL=ur-private-registry.com

$ export DOCKER_USERNAME=ur-username
$ export DOCKER_PASSWORD=ur-password

$ echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
```

## CI

```bash
$ cat > .gitlab-ci.yml << EOF
image: docker:latest

services:
  - docker:dind

variables:
  DOCKER_HOST: tcp://docker:2375/
  DOCKER_TLS_CERTDIR: ""

before_script:
  - docker login -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD" $CI_REGISTRY

build:
  script:
    - docker build -t myimage .
    - docker push myimage
EOF
```



