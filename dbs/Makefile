#
# Copyright SecureKey Technologies Inc. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

SHELL                       := /bin/bash
CONTAINER_CMD         	 	?= docker
USER_ID                          = $(shell id -u)
DOCKER_CMD_RUN_OPTS            ?= -u $(USER_ID)
CONTAINER_CMD_RUN_OPTS         ?= $(if $(findstring docker,$(CONTAINER_CMD)),$(DOCKER_CMD_RUN_OPTS),)

export DEPLOYMENT_ENV	?= local
# space delimited of Key:Value pairs
COMMON_LABELS		:= instance:${DEPLOYMENT_ENV}

OS 				= $(shell uname -s | tr '[:upper:]' '[:lower:]')
ARCH  				= $(shell uname -m | sed 's/x86_64/amd64/')

#IMAGES
MYSQL_IMG		?= docker.io/mysql:8.0.20
COUCH_IMG		?= docker.io/couchdb:3.1
POSTGRES_IMG		?= docker.io/postgres:13

# do not modify
KUSTOMIZE_DIR       	= kustomize/dbs
PREFIX							?=
#KUSTOMIZE_BUILD_OPTS			?= --load-restrictor LoadRestrictionsNone --enable-alpha-plugins
#export KUSTOMIZE_PLUGIN_HOME						= $(abspath .)/kustomize/plugin

.PHONY: all
all: deploy

.PHONY: deploy
deploy: prechecks kustomize kubectl set-images set-labels deploy-dbs

.PHONY: prechecks
prechecks:
ifeq (, $(shell stat ${KUSTOMIZE_DIR}/overlays/${DEPLOYMENT_ENV} 2>/dev/null))
	@echo "Environment not found ${KUSTOMIZE_DIR}/overlays/${DEPLOYMENT_ENV}"
	@exit 1
endif

.PHONY: set-labels
set-labels: kustomize
	@pushd ${KUSTOMIZE_DIR}/overlays/${DEPLOYMENT_ENV} &&\
		${KUSTOMIZE} edit set label ${COMMON_LABELS} &&\
		popd

.PHONY: set-images
set-images: kustomize
	@pushd ${KUSTOMIZE_DIR}/components/couch &&\
	${KUSTOMIZE} edit set image couchdb=${COUCH_IMG} &&\
	popd &&\
	pushd ${KUSTOMIZE_DIR}/components/postgres &&\
    ${KUSTOMIZE} edit set image postgres=${POSTGRES_IMG} &&\
    popd &&\
	pushd ${KUSTOMIZE_DIR}/components/mysql &&\
	${KUSTOMIZE} edit set image mysql=${MYSQL_IMG} &&\
	popd

.PHONY: deploy-dbs
deploy-dbs: prechecks kustomize kubectl
	@docker pull $(COUCH_IMG)
	@minikube image load $(COUCH_IMG)
	@docker pull $(MYSQL_IMG)
	@minikube image load $(MYSQL_IMG)
	@docker pull $(POSTGRES_IMG)
	@minikube image load $(POSTGRES_IMG)
	@docker pull ipfs/go-ipfs:master-2021-04-22-eea198f
	@minikube image load ipfs/go-ipfs:master-2021-04-22-eea198f
	$(KUSTOMIZE) build ${KUSTOMIZE_BUILD_OPTS} \
	${KUSTOMIZE_DIR}/overlays/${DEPLOYMENT_ENV} | $(KUBECTL) apply -f -

.PHONY: undeploy
undeploy: prechecks kustomize kubectl set-images set-labels undeploy-dbs

.PHONY: undeploy-dbs
undeploy-dbs: prechecks kustomize kubectl
	$(KUSTOMIZE) build ${KUSTOMIZE_BUILD_OPTS} \
	${KUSTOMIZE_DIR}/overlays/${DEPLOYMENT_ENV} | $(KUBECTL) delete -f -

.PHONY: kustomize
kustomize:
ifeq (, $(shell which kustomize 2>/dev/null))
	@{ \
	set -e ;\
	mkdir -p bin ;\
	curl -sSLo - https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize/v4.0.4/kustomize_v4.0.4_$(OS)_$(ARCH).tar.gz | tar xzf - -C bin/ ;\
	}
KUSTOMIZE=$(realpath ./bin/kustomize)
else
KUSTOMIZE=$(shell which kustomize)
endif

.PHONY: kubectl
kubectl:
ifeq (, $(shell which kubectl 2>/dev/null))
	@{ \
	set -e ;\
	mkdir -p bin ;\
	curl -sSL https://storage.googleapis.com/kubernetes-release/release/v1.19.6/bin/$(OS)/$(ARCH)/kubectl -o bin/kubectl ;\
  chmod u+x bin/kubectl ;\
	}
KUBECTL=$(realpath ./bin/kubectl)
else
KUBECTL=$(shell which kubectl)
endif
