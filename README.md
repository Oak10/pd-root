# Stack docks

## Run Stack
### Get Code
```
# Clone the repo and submodules:
git clone git@github.com:Oak10/pd-root.git --recurse-submodules
# update submodules (master branch)
git submodule foreach 'git fetch && git checkout master && git pull'
```

### Local
```
docker-compose up -d
```
- create realm (demo-real), users (angular-demo, spring-boot-demo), and roles (ROLE_APP for all users) at keycloak
- update token generated in keycloak (spring services - KEYCLOAK_CREDENTIALS_SECRET) || OR || upload realm in keycloak UI (files in keycloak dir)
- Configure smtp (use google) and update variables (mail service)

### APP Links (servers)
```
# base: 
http://{service}.{environment}.pd.com

# Exp (access keycloak):
http://keycloak.dev.pd.com/
# Exp (dev env   - test (recomendation -> kafka -> mailservice -> mail)):
http://recomendation.dev.pd.com/kafka/produce
# Exp (localhost)
http://localhost:8081/kafka/produce


```


# Build Images
### keycloak
```
docker build -t claudiooak/keycloak:21.1 -f DockerfileKeycloak .
```

### mailservice, moviestorageservice, recomendationservice, and frontend:
```
docker build -t claudiooak/mailservice:0.0.1 -f Dockerfile .
docker push claudiooak/mailservice:0.0.1

docker build -t claudiooak/recomendationservice:0.0.1 -f Dockerfile .
docker push claudiooak/recomendationservice:0.0.1

docker build -t claudiooak/moviesstorageservice:0.0.1 -f Dockerfile .
docker push claudiooak/moviesstorageservice:0.0.1

docker build -t claudiooak/frontendpd:0.0.2 -f Dockerfile .
docker push claudiooak/frontendpd:0.0.2
```

# SWARM:
## On a swarm master node
```
docker stack deploy -c swarm-traefik.yml traefik
docker stack rm traefik
CI_COMMIT_REF_SLUG=22 docker stack deploy -c helloworld.yml helloworld22
#remove service:
# docker service rm prd_moviesstorage-service
```
# Ansible
## Requirements:
- https://docs.ansible.com/ansible/latest/collections/community/docker/docker_stack_module.html#ansible-collections-community-docker-docker-stack-module

- install module:
```
ansible-galaxy collection install community.docker
# (destination host)
pip install jsondiff
# (playbook) docker network create --driver=overlay --attachable traefik 
```

## Run playbooks
```
# Install Traefik (Only once for all swarm stacks deployed  - production exp):
ansible-playbook -i inventory/prod/hosts.yml --diff --extra-vars "force_restart=true" playbooks/traefik-swarm.yml

# Install Jaeger, Postgres, Kafka, and Keycloak ("once" for each stack  - production exp)
ansible-playbook -i inventory/prod/hosts.yml --diff playbooks/infra-swarm.yml
```

### install all stack (dev)
```
ansible-playbook -i pd-ansible/inventory/dev/hosts.yml --diff pd-ansible/playbooks/traefik-swarm.yml 
ansible-playbook -i pd-ansible/inventory/dev/hosts.yml --diff pd-ansible/playbooks/infra-swarm.yml 
ansible-playbook -i pd-ansible/inventory/dev/hosts.yml --diff pd-ansible/playbooks/mailservice-swarm.yml 
ansible-playbook -i pd-ansible/inventory/dev/hosts.yml --diff pd-ansible/playbooks/recomendationservice-swarm.yml 
ansible-playbook -i pd-ansible/inventory/dev/hosts.yml --diff pd-ansible/playbooks/moviesstorageservice-swarm.yml 
ansible-playbook -i pd-ansible/inventory/dev/hosts.yml --diff pd-ansible/playbooks/frontendpd-swarm.yml 
```

# Generate account for mail (Google)
```
Login to Gmail 
    -> Manage your Google Account 
        -> Security 
            -> Search on google account "Palavras-passe de apps" || "App Passwords"
                -> Select app with a custom name 
                    -> Click on Generate
```

https://myaccount.google.com/apppasswords?pli=1&rapt=AEjHL4O_UU0o9nn4twF26MyO8bjhrNJJ5FLCNnh1CuVzJILHZl-egbvbM4Eam7Vd2Vd8RJpzr-yGudyB1_45aXhOiF7sODIN7Q



