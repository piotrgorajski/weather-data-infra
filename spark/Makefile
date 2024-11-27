IMAGE_NAME := spark-minio-local
IMAGE_TAG := latest
REGISTRY := piotrgorajski
FULL_IMAGE_NAME := $(REGISTRY)/$(IMAGE_NAME):$(IMAGE_TAG)

.PHONY: build push all

build:
	@echo "Building Docker image..."
	docker build -t $(FULL_IMAGE_NAME) .

push:
	@echo "Pushing Docker image..."
	docker push $(FULL_IMAGE_NAME)

all: build push
	@echo "Build and push complete for $(FULL_IMAGE_NAME)"