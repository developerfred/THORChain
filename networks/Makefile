########################################
### These targets were broken out of the main Makefile to enable easy setup of testnets.
### They use a form of terraform + ansible to build full nodes in AWS.
### The shell scripts in this folder are example uses of the targets.

# Name of the testnet. Used in chain-id.
TESTNET_NAME?=remotenet

# Name of the servers grouped together for management purposes. Used in tagging the servers in the cloud.
CLUSTER_NAME?=$(TESTNET_NAME)

# Number of servers to put in one availability zone in AWS.
SERVERS?=1

# Maximum number of availability zones to use.
MAX_ZONES?=4

# Number of regions to use in AWS. One region usually contains 2-3 availability zones.
REGION_LIMIT?=1

# Path to thorchaind for deployment. Must be a Linux binary.
BINARY?=$(CURDIR)/../build/thorchaind
CLI_BINARY?=$(CURDIR)/../build/thorchaincli
SPAM_BINARY?=$(CURDIR)/../build/thorchainspam

# Path to the genesis.json and config.toml files to deploy on full nodes.
GENESISFILE?=$(CURDIR)/../build/genesis.json
CONFIGFILE?=$(CURDIR)/../build/config.toml

# Name of application for app deployments
APP_NAME ?= faucettestnet1
# Region to deploy VPC and application in AWS
REGION ?= us-east-2

# Name of application for app deployments
SPAM_NAME ?= $(CLUSTER_NAME)_spammer

all:
	@echo "There is no all. Only sum of the ones."

disclaimer:
	@echo "WARNING: These are example network configuration scripts only and have not undergone security review. They should not be used for production deployments."

########################################
### Extract genesis.json and config.toml from a node in a cluster

extract-config: disclaimer
	#Make sure you have AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY or your IAM roles set for AWS API access.
	@if ! [ -f $(HOME)/.ssh/id_rsa.pub ]; then ssh-keygen ; fi
	cd remote/ansible && \
	ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook \
		-i inventory/ec2.py \
		-l "tag_Environment_$(CLUSTER_NAME)" \
		-b -u centos \
		-e TESTNET_NAME="$(TESTNET_NAME)" \
		-e GENESISFILE="$(GENESISFILE)" \
		-e CONFIGFILE="$(CONFIGFILE)" \
		extract-config.yml


########################################
### Remote validator nodes using terraform and ansible in AWS

validators-start: disclaimer
	#Make sure you have AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY or your IAM roles set for AWS API access.
	@if ! [ -f $(HOME)/.ssh/id_rsa.pub ]; then ssh-keygen ; fi
	@if [ -z "`file $(BINARY) | grep 'ELF 64-bit'`" ]; then echo "Please build a linux binary using 'make build-linux'." ; false ; fi
	cd remote/terraform-aws && terraform init && (terraform workspace new "$(CLUSTER_NAME)" || terraform workspace select "$(CLUSTER_NAME)") && terraform apply -auto-approve -var SSH_PUBLIC_FILE="$(HOME)/.ssh/id_rsa.pub" -var SSH_PRIVATE_FILE="$(HOME)/.ssh/id_rsa" -var TESTNET_NAME="$(CLUSTER_NAME)" -var SERVERS="$(SERVERS)" -var MAX_ZONES="$(MAX_ZONES)" -var REGION_LIMIT="$(REGION_LIMIT)"
	cd remote/ansible && ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory/ec2.py -l "tag_Environment_$(CLUSTER_NAME)" -u centos -b -e BINARY=$(BINARY) -e TESTNET_NAME="$(TESTNET_NAME)" setup-validators.yml
	cd remote/ansible && ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory/ec2.py -l "tag_Environment_$(CLUSTER_NAME)" -u centos -b setup-journald.yml
	cd remote/ansible && ansible-playbook -i inventory/ec2.py -l "tag_Environment_$(CLUSTER_NAME)" -u centos -b start.yml

validators-stop: disclaimer
	cd remote/terraform-aws && terraform workspace select "$(CLUSTER_NAME)" && terraform destroy -force -var SSH_PUBLIC_FILE="$(HOME)/.ssh/id_rsa.pub" -var SSH_PRIVATE_FILE="$(HOME)/.ssh/id_rsa" && terraform workspace select default && terraform workspace delete "$(CLUSTER_NAME)"
	rm -rf remote/ansible/keys/ remote/ansible/files/

validators-status: disclaimer
	cd remote/ansible && ansible-playbook -i inventory/ec2.py -l "tag_Environment_$(CLUSTER_NAME)" status.yml

validators-clear:
	cd remote/ansible && ansible-playbook -i inventory/ec2.py -l "tag_Environment_$(CLUSTER_NAME)" -u centos -b clear-config.yml


########################################
### Remote full nodes using terraform and ansible in Amazon AWS

fullnodes-start: disclaimer
	#Make sure you have AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY or your IAM roles set for AWS API access.
	@if ! [ -f $(HOME)/.ssh/id_rsa.pub ]; then ssh-keygen ; fi
	@if [ -z "`file $(BINARY) | grep 'ELF 64-bit'`" ]; then echo "Please build a linux binary using 'make build-linux'." ; false ; fi
	cd remote/terraform-aws && terraform init && (terraform workspace new "$(CLUSTER_NAME)" || terraform workspace select "$(CLUSTER_NAME)") && terraform apply -auto-approve -var SSH_PUBLIC_FILE="$(HOME)/.ssh/id_rsa.pub" -var SSH_PRIVATE_FILE="$(HOME)/.ssh/id_rsa" -var TESTNET_NAME="$(CLUSTER_NAME)" -var SERVERS="$(SERVERS)" -var MAX_ZONES="$(MAX_ZONES)" -var REGION_LIMIT="$(REGION_LIMIT)"
	cd remote/ansible && ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory/ec2.py -l "tag_Environment_$(CLUSTER_NAME)" -u centos -b -e BINARY=$(BINARY) -e TESTNET_NAME="$(TESTNET_NAME)" -e GENESISFILE="$(GENESISFILE)" -e CONFIGFILE="$(CONFIGFILE)" setup-fullnodes.yml
	cd remote/ansible && ansible-playbook -i inventory/ec2.py -l "tag_Environment_$(CLUSTER_NAME)" -u centos -b start.yml

fullnodes-stop: disclaimer
	cd remote/terraform-aws && terraform workspace select "$(CLUSTER_NAME)" && terraform destroy -force -var SSH_PUBLIC_FILE="$(HOME)/.ssh/id_rsa.pub" -var SSH_PRIVATE_FILE="$(HOME)/.ssh/id_rsa" && terraform workspace select default && terraform workspace delete "$(CLUSTER_NAME)"
	rm -rf remote/ansible/keys/ remote/ansible/files/

fullnodes-status: disclaimer
	cd remote/ansible && ansible-playbook -i inventory/ec2.py -l "tag_Environment_$(CLUSTER_NAME)" status.yml

########################################
### Other calls

UNSAFE_RESET_ALL?=no
upgrade-thorchaind: disclaimer
	#Make sure you have AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY or your IAM roles set for AWS API access.
	@if ! [ -f $(HOME)/.ssh/id_rsa.pub ]; then ssh-keygen ; fi
	@if [ -z "`file $(BINARY) | grep 'ELF 64-bit'`" ]; then echo "Please build a linux binary using 'make build-linux'." ; false ; fi
	cd remote/ansible && ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory/ec2.py -l "tag_Environment_$(CLUSTER_NAME)" -u centos -b -e BINARY=$(BINARY) -e GENESISFILE="$(GENESISFILE)" -e UNSAFE_RESET_ALL=$(UNSAFE_RESET_ALL) upgrade-thorchaind.yml

upgrade-seeds: disclaimer
	#Make sure you have AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY or your IAM roles set for AWS API access.
	@if ! [ -f $(HOME)/.ssh/id_rsa.pub ]; then ssh-keygen ; fi
	@if [ -z "`file $(BINARY) | grep 'ELF 64-bit'`" ]; then echo "Please build a linux binary using 'make build-linux'." ; false ; fi
	@if [ -z "`file $(CLI_BINARY) | grep 'ELF 64-bit'`" ]; then echo "Please build a linux binary using 'make build-linux'." ; false ; fi
	cd remote/ansible && ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory/ec2.py -l "tag_Environment_$(CLUSTER_NAME)" -u centos -b -e BINARY=$(BINARY) -e CLI_BINARY=$(CLI_BINARY) -e UNSAFE_RESET_ALL=$(UNSAFE_RESET_ALL) upgrade-thorchain.yml

list:
	remote/ansible/inventory/ec2.py | python -c 'import json,sys ; print "\n".join(json.loads("".join(sys.stdin.readlines()))["tag_Environment_$(CLUSTER_NAME)"])'

install-datadog: disclaimer
	#Make sure you have AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY or your IAM roles set for AWS API access.
	@if [ -z "$(DD_API_KEY)" ]; then echo "DD_API_KEY environment variable not set." ; false ; fi
	cd remote/ansible && ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory/ec2.py -l "tag_Environment_$(CLUSTER_NAME)" -u centos -b -e DD_API_KEY="$(DD_API_KEY)" -e TESTNET_NAME="$(TESTNET_NAME)" -e CLUSTER_NAME="$(CLUSTER_NAME)" install-datadog-agent.yml

remove-datadog: disclaimer
	#Make sure you have AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY or your IAM roles set for AWS API access.
	cd remote/ansible && ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory/ec2.py -l "tag_Environment_$(CLUSTER_NAME)" -u centos -b remove-datadog-agent.yml


########################################
### Application infrastructure setup

app-start: disclaimer
	#Make sure you have AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY or your IAM roles set for AWS API access.
	@if ! [ -f $(HOME)/.ssh/id_rsa.pub ]; then ssh-keygen ; fi
	@if [ -z "`file $(BINARY) | grep 'ELF 64-bit'`" ]; then echo "Please build a linux binary using 'make build-linux'." ; false ; fi
	cd remote/terraform-app && terraform init && (terraform workspace new "$(APP_NAME)" || terraform workspace select "$(APP_NAME)") && terraform apply -auto-approve -var SSH_PUBLIC_FILE="$(HOME)/.ssh/id_rsa.pub" -var SSH_PRIVATE_FILE="$(HOME)/.ssh/id_rsa" -var APP_NAME="$(APP_NAME)" -var SERVERS="$(SERVERS)" -var MAX_ZONES="$(MAX_ZONES)" -var REGION="$(REGION)"
	cd remote/ansible && ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory/ec2.py -l "tag_Environment_$(APP_NAME)" -u centos -b -e BINARY=$(BINARY) -e TESTNET_NAME="$(TESTNET_NAME)" -e GENESISFILE="$(GENESISFILE)" -e CONFIGFILE="$(CONFIGFILE)" setup-fullnodes.yml
	cd remote/ansible && ansible-playbook -i inventory/ec2.py -l "tag_Environment_$(APP_NAME)" -u centos -b start.yml

app-stop: disclaimer
	cd remote/terraform-app && terraform workspace select "$(APP_NAME)" && terraform destroy -force -var SSH_PUBLIC_FILE="$(HOME)/.ssh/id_rsa.pub" -var SSH_PRIVATE_FILE="$(HOME)/.ssh/id_rsa" -var APP_NAME=$(APP_NAME) && terraform workspace select default && terraform workspace delete "$(APP_NAME)"
	rm -rf remote/ansible/keys/ remote/ansible/files/

########################################
### Spammer infrastructure setup

spam-start: disclaimer
	#Make sure you have AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY or your IAM roles set for AWS API access.
	@if ! [ -f $(HOME)/.ssh/id_rsa.pub ]; then ssh-keygen ; fi
	@if [ -z "`file $(CLI_BINARY) | grep 'ELF 64-bit'`" ]; then echo "Please build a linux binary using 'make build-linux'." ; false ; fi
	@if [ -z "`file $(SPAM_BINARY) | grep 'ELF 64-bit'`" ]; then echo "Please build a linux binary using 'make build-spam-linux'." ; false ; fi
	cd remote/terraform-spam && terraform init && (terraform workspace new "$(SPAM_NAME)" || terraform workspace select "$(SPAM_NAME)") && terraform apply -auto-approve -var SSH_PUBLIC_FILE="$(HOME)/.ssh/id_rsa.pub" -var SSH_PRIVATE_FILE="$(HOME)/.ssh/id_rsa" -var CLUSTER_NAME="$(SPAM_NAME)" -var SERVERS="$(SERVERS)" -var MAX_ZONES="$(MAX_ZONES)" -var REGION="$(REGION)"
	cd remote/ansible && ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory/ec2.py -l "tag_Environment_$(SPAM_NAME)" -u centos -b -e CLI_BINARY=$(CLI_BINARY) -e SPAM_BINARY=$(SPAM_BINARY) setup-spammers.yml

spam-stop: disclaimer
	cd remote/terraform-spam && terraform workspace select "$(SPAM_NAME)" && terraform destroy -force -var SSH_PUBLIC_FILE="$(HOME)/.ssh/id_rsa.pub" -var SSH_PRIVATE_FILE="$(HOME)/.ssh/id_rsa" -var CLUSTER_NAME=$(SPAM_NAME) && terraform workspace select default && terraform workspace delete "$(SPAM_NAME)"
	rm -rf remote/ansible/keys/ remote/ansible/files/

# To avoid unintended conflicts with file names, always add to .PHONY
# unless there is a reason not to.
# https://www.gnu.org/software/make/manual/html_node/Phony-Targets.html
.PHONY: all disclaimer extract-config validators-start validators-stop validators-status \
fullnodes-start fullnodes-stop fullnodes-status \
upgrade-thorchaind upgrade-seeds list install-datadog remove-datadog \
app-start app-stop spam-start spam-stop
