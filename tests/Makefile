
# Copyright (c) 2017 Intel Corporation
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at

#    http://www.apache.org/licenses/LICENSE-2.0

# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.


#----------------------------------------------------------------------------------------------------------------------
# Configs
#----------------------------------------------------------------------------------------------------------------------
SHELL:=/bin/bash

MAKEFILE_DIR := $(abspath $(lastword $(MAKEFILE_LIST)))
CURRENT_DIR  :=  $(patsubst %/,%,$(dir $(MAKEFILE_DIR)))

USERNAME = "oisp@testing.com"
PASSWORD = "OispTesting1"
ROLE = "admin"

USERNAME2 = "oisp2@testing.com"
PASSWORD2 = "OispTesting2"
ROLE2 = "admin"

NAMESPACE?=oisp
FRONTEND_POD:=$(shell kubectl -n $(NAMESPACE) get pods -o custom-columns=:metadata.name | grep frontend | head -n 1)

npm-install:
	@$(call msg,"Installing npm packages ...");
	@cd $(CURRENT_DIR) && npm install > /dev/null

test: npm-install
	@export OISP_GRAFANA_PASSWORD=$$(kubectl -n $(NAMESPACE) get -o yaml configmaps oisp-config | shyaml get-value data.grafana | jq -r .adminPassword) && \
	cat test-config-template.json | envsubst > test-config.json
	@$(call msg,"Resetting database ...");
	@kubectl exec -i $(FRONTEND_POD) -c frontend --  node /app/admin resetDB
	@$(call msg,"Adding a user for testing ...");
	@kubectl exec -i $(FRONTEND_POD) -c frontend --  node /app/admin addUser $(USERNAME) $(PASSWORD) $(ROLE) > /dev/null
	@kubectl exec -i $(FRONTEND_POD) -c frontend --  node /app/admin addUser $(USERNAME2) $(PASSWORD2) $(ROLE2) > /dev/null

ifndef TEST_PREP_ONLY
	@$(call msg,"Starting the e2e testing ...");
	@env http_proxy="" https_proxy="" USERNAME=$(USERNAME) PASSWORD=$(PASSWORD) USERNAME2=$(USERNAME2) PASSWORD2=$(PASSWORD2) LOG_LEVEL="error" NODE_ENV="local" npm test
else
	@$(call msg,"Starting 3rd party test preparation ...");
	@env http_proxy="" https_proxy="" USERNAME=$(USERNAME) PASSWORD=$(PASSWORD) LOG_LEVEL="error" NODE_ENV="local" npm run-script prep-only
endif

clean:
	@$(call msg,"Cleaning ...");

distclean: clean
	@$(call msg,"Dist cleaning ...");
	@rm -rf $(CURRENT_DIR)/iotkit-agent

#----------------------------------------------------------------------------------------------------------------------
# helper functions
#----------------------------------------------------------------------------------------------------------------------
define msg
	tput setaf 2 && \
	for i in $(shell seq 1 120 ); do echo -n "-"; done; echo "" && \
	echo -e "\t"$1 && \
	for i in $(shell seq 1 120 ); do echo -n "-"; done; echo "" && \
	tput sgr0
endef
