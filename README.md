# jfroghomework

## Ubuntu
```bash
sudo apt-get update
sudo apt-get install nginx
sudo apt-get install docker.io
sudo usermod -aG docker $USER
```

## Start JFrog
```bash
export JFROG_HOME=$(pwd)
mkdir -p $JFROG_HOME/artifactory/var/etc/
cd $JFROG_HOME/artifactory/var/etc/
touch ./system.yaml
chown -R $UID:$GID $JFROG_HOME/artifactory/var
chmod -R 777 $JFROG_HOME/artifactory/var
cd $JFROG_HOME
docker run --name artifactory -v $JFROG_HOME/artifactory/var/:/var/opt/jfrog/artifactory -d -p 8081:8081 -p 8082:8082 releases-docker.jfrog.io/jfrog/artifactory-pro:latest
```

## Build BlueOcean
```bash
docker build -t myjenkins-blueocean:1.1 .
```

## Start Jenkins
```bash
docker network create jenkins

docker run --name jenkins-docker --detach \
  --privileged --network jenkins --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 docker:dind --storage-driver overlay2 --insecure-registry=jfrog.gradle.properties

docker run \
  --name jenkins-blueocean \
  --detach \
  --network jenkins \
  --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client \
  --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 \
  --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:1.1
```