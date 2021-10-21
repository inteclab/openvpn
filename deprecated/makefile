env_vars=-g
export env_vars

# The binary to build (just the basename).
module := openvpn
version?= 0.0.1

# Where to push the docker image.
docker_registry?= 127.0.0.1:5000

docker_image := $(docker_registry)/$(module)

# This version-strategy uses git tags to set the version string
docker_tag := $(shell git describe --tags --always --dirty)

title_style='\033[3;37;40m'
no_style='\033[0m' # No Color

docker_exec_cmd := @docker container exec -it openvpn

init:
	# initialize the openvpn service the first time
	# Initialize the configuration files and certificates
	@docker-compose run --rm openvpn ovpn_genconfig -u udp://vpn.finclab.com
	@docker-compose run --rm openvpn ovpn_initpki

delete:
	# delete all data
	@sudo rm -rf /lab/data/docker/volume/openvpn-data
	@sudo mkdir -p /lab/data/docker/volume/openvpn-data

start:
	docker-compose up -d openvpn

restart:
	@make stop
	@make start

log:
	# you can access the container logs with
	docker-compose logs -f

stop:
	docker-compose down --remove-orphans

# Define the name of the user to which the VPN cert is generated
client_name?= test
create_ovpn:
	# without a passphrase (not recommended)
	@docker-compose run --rm openvpn easyrsa build-client-full ${client_name} nopass
	$(MAKE) retrieve_ovpn
	@docker-compose run --rm openvpn ovpn_getclient ${client_name} > ${client_name}.ovpn

create_ovpn_with_password:
	# with a passphrase (recommended)
	@docker-compose run --rm openvpn easyrsa build-client-full ${client_name}
	$(MAKE) retrieve_ovpn
	@docker-compose run --rm openvpn ovpn_getclient ${client_name} > ${client_name}.ovpn

retrieve_ovpn:
	@docker-compose run --rm openvpn ovpn_getclient ${client_name} > ${client_name}.ovpn

revoke_ovpn:
	# Remove corresponding crt, key and req files
	@docker-compose run --rm openvpn ovpn_revokeclient ${client_name} remove

shell:
	@${docker_exec_cmd} /bin/bash

exec:
	# Syntax: make exec labenv=prod cmd="touch /tmp/example" 
	@${docker_exec_cmd} ${cmd}

start_swarm_service:
	# Deploy the service in Docker Swarm, service name: i.e. datastore_prod_timescaledb
	# Syntax: make start_swarm_service labenv=prod
	@echo "\n${title_style}Starting OpenVPN Service in Docker Swarm...${no_style}\n"
	docker stack deploy --compose-file ./docker-compose.yml openvpn --with-registry-auth

stop_swarm_service:
	# Stop the service in Docker Swarm, service name: i.e. datastore_prod_datastore
	# Syntax: make stop_swarm_service labenv=prod
	@echo "\n${title_style}Stopping OpenVPN Service in Docker Swarm...${no_style}\n"
	docker stack rm openvpn

