# The following are targers that do not exist in the filesystem as real files and should be always executed by make
.PHONY: default deps base build dev shell start stop test_unit test_integration test image push

# Name of this service/application
SERVICE_NAME := ruby25

# Name of the service used to run the shell (overridable)
SHELL_SERVICE ?= ruby25

# Docker image name for this project
IMAGE_NAME := $(SERVICE_NAME)

# Shell to use for running scripts
SHELL := $(shell which bash)

# Get docker path or an empty string
DOCKER := $(shell command -v docker)

# Get docker-compose path or an empty string
DOCKER_COMPOSE := $(shell command -v docker-compose)

# Get the main unix group for the user running make (to be used by docker-compose later)
GID := $(shell id -g)

# Get the unix user id for the user running make (to be used by docker-compose later)
UID := $(shell id -u)

CREATE_USER_COMMAND =

ifeq ($(PLATFORM), Linux)
    # Get the main unix group for the user running make (to be used by docker-compose later)
    GID := $(shell id -g)

    # Get the unix user id for the user running make (to be used by docker-compose later)
    UID := $(shell id -u)

    # Create user command
    CREATE_USER_COMMAND = --build-arg gid=$(GID) --build-arg uid=$(UID)
endif

# Get the username of theuser running make. On the devbox, we give priority to /etc/username
USERNAME ?= $(shell ( [ -f /etc/username ] && cat /etc/username  ) || whoami)

# Try to detect current branch if not provided from environment
BRANCH ?= $(shell git rev-parse --abbrev-ref HEAD)

# Detect current commit
COMMIT = $(shell git rev-parse --short HEAD)

# Bash history file for container shell
HISTORIES_DIR := ~/.bash_histories
HISTORY_FILE := $(HISTORIES_DIR)/ruby25


# Compose project name
export COMPOSE_PROJECT_NAME = $(SERVICE_NAME)

#Defaults for environment required by docker-compose
export HOST_USER ?= $(USERNAME)
export _VERSION = $(COMMIT)
export AWS_ACCESS_KEY_ID ?= fake-access-key
export AWS_SECRET_ACCESS_KEY ?= fake-secret
export AWS_REGION ?= eu-west-1


# The default action of this Makefile is to build the development docker image
default: build

# Test if the dependencies we need to run this Makefile are installed
deps:
ifndef DOCKER
	@echo "Docker is not available. Please install docker"
	@exit 1
endif
ifndef DOCKER_COMPOSE
	@echo "docker-compose is not available. Please install docker-compose"
	@exit 1
endif

# Build the base docker image which is shared between the development and production images
base: deps
	docker build $(CREATE_USER_COMMAND) -t $(IMAGE_NAME)_base:latest .

# Build the development docker image
build: base
	docker-compose build

# Run the development environment in non-daemonized mode (foreground)
dev: build
	( docker-compose up; \
		docker-compose stop; \
		docker-compose rm -f; )


# Run a shell into the development docker image
shell: build
	-mkdir -p $(HISTORIES_DIR)
	-touch $(HISTORY_FILE)
	docker-compose run --rm --service-ports $(SHELL_SERVICE) /bin/bash

# Run unit tests
test_unit:
		docker-compose run --rm $(SERVICE_NAME)-test \
		bash -c "mamba -f documentation test/unit"


# Run both unit and integration tests
test: build test_unit


# Run the development environment in the background
start: build
	cd environment/dev && docker-compose up -d

# Stop the development environment (background and/or foreground)
stop:
	cd environment/dev && ( \
		docker-compose stop; \
		docker-compose rm -f; \
		)


# Clean project containers
clean:
	docker-compose stop && \
	docker-compose rm --all -f
