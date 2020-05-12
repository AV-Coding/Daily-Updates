# Week 1

### Monday April 20th, 2020

## Notes
  - [x] Installed Anchore [link](https://docs.anchore.com/current/docs/engine/engine_installation/docker_compose/)
  - [x] Show Edwin a report log for an analysis of an image
    - [x] Tested analysis using anchore and docker-compose
  - I'm currently using achore-cli which is a command tool for Anchore to run an analysis on an image.
    - [ ] Look into using [Rest API](https://app.swaggerhub.com/apis/anchore/anchore-engine/0.1.13). This may not be used with anchore-cli, so may need to switch approach.
  - [ ] Look into Anchore policy hub, Edwin was interested in this aspect of Anchore.
  - [ ] Would like to install Claire with docker-compose, but if you aren't close with an hour install with whatever approach is easiest.
  - [Image analysis](https://docs.anchore.com/current/docs/engine/engine_installation/docker_compose/) instructions with anchore-cli and docker-compose

## Issues
 - Communicate with Edwin any challenges being faced.

## Code history
- Anchore Installation
      sudo docker pull anchore/anchore-engine
- Adding docker to have sudo priveleges
      sudo groupadd docker
      sudo usermod -aG docker $USER
- Setting up docker compose
      mkdir docker-compose-install
      cd docker-compose-install/
      docker-compose pull
      docker-compose up -d
- Analyzing images with Anchore, anchore-cli and docker-compose
      docker-compose exec engine-api anchore-cli image add cyverse/vibrant:1.0.1
      docker-compose exec engine-api anchore-cli image wait cyverse/vibrant:1.0.1
      docker-compose exec engine-api anchore-cli image content cyverse/vibrant:1.0.1
      docker-compose exec engine-api anchore-cli image vuln cyverse/vibrant:1.0.1 all

- docker-compose commands
  - `docker-compose ps -a` *Lists all containers being run in *`docker-compose-install` folder*
  - `docker-compose --force-restart` *restarts all containers in folder*

---

### Friday April 24th, 2020

## Notes
 - [ ] Maybe look into setting up Swagger UI for Anchore
  - [Here](https://docs.anchore.com/current/docs/faq/#27) is an FAQ. It talks a little bit about how to set up swagger on the browser.
 - Made a list to compare different tools
    - Do I need to rescan the same image to get it's list of vulenrabilites, or can I access the vulnerabilities without re-processing?(I believe this will answer whether the information is cached/persistent)
    - Can I filter the report based on the level of vulnerabilities? preferably [high]
    - What packages are supported in the vulnerability DB?
    - Does it provide a notification system?
    - How long does it take to scan?
    - Can it scan local images (Can a personal image that is not on docker hub be scanned for vulnerabilities)? (Would be nice, but isn't a priority)

## Issues
 - Was having the issue after running the command `docker-compose exec engine-api anchore-cli system status` that some of the services were down
        Service catalog (anchore-quickstart, http://engine-catalog:8228): up
        Service policy_engine (anchore-quickstart, http://engine-policy-engine:8228): down (unavailable)
        Service analyzer (anchore-quickstart, http://engine-analyzer:8228): down (unavailable)
        Service simplequeue (anchore-quickstart, http://engine-simpleq:8228): down (unavailable)
        Service apiext (anchore-quickstart, http://engine-api:8228): down (unavailable)
- restarting it with `docker-compose restart` fixed it.

## Code history

- docker-compose commands
  - `docker-compose ps -a` *Lists all containers being run in *`docker-compose-install` folder*
  - `docker-compose --force-restart` *restarts all containers in folder*

# Week 2
### Friday May 1st, 2020

## Notes
- [x] Helped Nancy run instances for multiple users at once.
  - Setup up personal tokens for users by:
    - emulating user
    - settings->advanced->Personal Token
    - Named it workshop then copied personal token to seperate file for saving
    - cloned john's [repository](https://github.com/cyverse/atmo-workshop-scripts.git)
    - Included history down below
- [x] Installed clair, I'll try analysis testing on Monday.
  - May need to install `Quay`

## Issues

## Code history
- Clair Installation:
      sudo apt-get install rpm
      sudo apt-get install xz-utils
      go get github.com/quay/clair/cmd/clair
      docker run --name postgres -e POSTGRES_HOST_AUTH_METHOD=trust -p 5432:5432 -d postgres # Need to include POSTGRES_HOST_AUTH_METHOD or replace with POSTGRES_PASSWORD and set a password.
      docker run --net=host -d -p 6060-6061:6060-6061 -v /home/arielv/clair_config:/config quay.io/coreos/clair:latest -config=/config/config.yaml
      curl -X GET -I http://128.196.142.30:6061/health

- John Xu's script:
 - git clone https://github.com/cyverse/atmo-workshop-scripts.git
 - Placed csv in this format:
        token,image,image version,instance size
        this_is_an_access_token,https://use.jetstream-cloud.org/application/images/717,1.27,m1.tiny

 - python3 batch_launch_instance.py --token --csv example.csv &

---
# Week 3

### Monday May 4th, 2020

## Notes
- [x] Informed user how to install Irods with sudo priveleges
- [x] Created separate VM for container security
  - Ports were meant for Internet connection were being used by jupyterhub.
  - Could have removed ports, but used this as a good time to seperate the 2 projects.
- While trying to figure out how to launch an analysis with Clair, found a installation [tutorial](https://www.katacoda.com/courses/docker-security/image-scanning-with-clair) that seemed better than my installation since it uses docker_compose.
  - [x] Installed Clair
  - [x] Ran image analysis on two cyverse images `cyverse/vibrant` and `cyverse/rsem-prepare`
  - Both analysis done by Clair had less found vulnerabilities when compared to anchore.

## Issues
  - Try to figure out why the huge discrepancy between the 2 tools.

## Code history

Clair Docker compose installation Assuming **GO, Docker and Docker-Compose** are installed:

    curl -LO https://raw.githubusercontent.com/coreos/clair/05cbf328aa6b00a167124dbdbec229e348d97c04/contrib/compose/docker-compose.yml
    mkdir clair_config && curl -L https://raw.githubusercontent.com/coreos/clair/master/config.yaml.sample -o clair_config/config.yaml
    sed 's/clair-git:latest/clair:v2.0.1/' -i docker-compose.yml &&   sed 's/host=localhost/host=postgres password=password/' -i clair_config/config.yaml
    docker-compose up -d postgres
    curl -LO https://gist.githubusercontent.com/BenHall/34ae4e6129d81f871e353c63b6a869a7/raw/5818fba954b0b00352d07771fabab6b9daba5510/clair.sql
    docker run -it     -v $(pwd):/sql/     --network "${USER}_default"     --link clair_postgres:clair_postgres     postgres:latest         bash -c "PGPASSWORD=password psql -h clair_postgres -U postgres < /sql/clair.sql"
    docker-compose up -d clair
    go get github.com/optiopay/klar
    sudo cp klar /usr/local/bin/
    sudo apt-get install jq

Running Analysis on Clair:

    CLAIR_ADDR=http://localhost:6060 klar cyverse/rsem-prepare

---

# Week 4

### Monday May 11th, 2020
## Notes
- Clair currently just uses Common Vulnerabilities and Exposures(CVE) data
- Anchore uses CVE data and software repositories like Node.js and Ruby Gem.
- [ ] Look into harbor
  - Might add more data to clair.
- [x] Setup Nafigos-cli onto docker vm

## Issues
## Code history
- `docker-compose exec engine-api anchore-cli --debug image vuln [OPTIONS] INPUT_IMAGE [VULN_TYPE]`
- export SERVICE_GATEWAY_URL=$(kubectl --context=kind-service-cluster get po -l istio=ingressgateway -n istio-system -o jsonpath="{.items[0].status.hostIP}"):$(kubectl --context=kind-service-cluster -n istio-system get service istio-ingressgateway -o jsonpath="{.spec.ports[?(@.name==\"http2\")].nodePort}")

- export NAFIGOS_API=$SERVICE_GATEWAY_URL

**Setup for Nafigos-cli (Needs to be setup as root)**
- Install docker (You should know how to do this, but here's a [link](https://docs.docker.com/engine/install/ubuntu/))
- Install go
      wget https://dl.google.com/go/go1.14.2.linux-amd64.tar.gz
      tar -C /usr/local -xzf go1.14.2.linux-amd64.tar.gz
      echo GOPATH='$HOME'/go  >> /etc/bash.bashrc
      echo PATH='$PATH':/usr/local/go/bin:$GOPATH/bin >> /etc/bash.bashrc
      source /etc/bash.bashrc
      mkdir -p /go/src/gitlab.com/cyverse

- Install kind
      GO111MODULE="on" go get sigs.k8s.io/kind@v0.7.0

- Install Istio
      curl -sL https://istio.io/downloadIstioctl | sh -
      echo PATH='$PATH':$HOME/.istioctl/bin

- Install helm
      curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
      chmod 700 get_helm.sh
      ./get_helm.sh

- Install skaffold
      curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64
      chmod +x skaffold
      sudo mv skaffold /usr/local/bin

- Setup kind clusters
      cd /go/src/gitlab.com/cyverse/
      git clone https://gitlab.com/AV-Coding/nafigos.git
      ./tools/kind/create_kind_clusters.sh

- Save $SERVICE_GATEWAY_URL and $NAFIGOS_API
      export SERVICE_GATEWAY_URL=$(kubectl --context=kind-service-cluster get po -l istio=ingressgateway -n istio-system -o jsonpath="{.items[0].status.hostIP}"):$(kubectl --context=kind-service-cluster -n istio-system get service istio-ingressgateway -o jsonpath="{.spec.ports[?(@.name==\"http2\")].nodePort}")
      export NAFIGOS_API=$SERVICE_GATEWAY_URL

- Switch to kind-user-cluster
      kubectl config set-context kind-user-cluster
      export USER_GATEWAY_URL=$(kubectl --context=kind-user-cluster get po -l istio=ingressgateway -n istio-system -o jsonpath="{.items[0].status.hostIP}"):$(kubectl --context=kind-user-cluster -n istio-system get service istio-ingressgateway -o jsonpath="{.spec.ports[?(@.name==\"http2\")].nodePort}")

- Switch back to kind-service-cluster
      kubectl config set-context kind-service-cluster

- launch
      skaffold dev
      then ctrl^z and bg

- setup Nafigos executable
      cd go/src/gitlab.com/cyverse/nafigos/cmd
      go build -o nafigos
      export NAFIGOS_TOKEN=somepass

- May need to setup $SERVICE_GATEWAY_URL and $NAFIGOS_API again if `nafigos --version` doesn't work

### Tuesday May 12th, 2020
## Notes
## Issues
## Code history
