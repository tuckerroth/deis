include ../includes.mk

# the filepath to this repository, relative to $GOPATH/src
repo_path = github.com/deis/deis/router

GO_FILES = boot.go
GO_PACKAGES = logger
GO_PACKAGES_REPO_PATH = $(addprefix $(repo_path)/,$(GO_PACKAGES))

COMPONENT = $(notdir $(repo_path))
IMAGE = $(IMAGE_PREFIX)$(COMPONENT):$(BUILD_TAG)
DEV_IMAGE = $(DEV_REGISTRY)/$(IMAGE)
BUILD_IMAGE = $(COMPONENT)-build

build: check-docker
	cp -pR ../Godeps .
	docker build -t $(BUILD_IMAGE) .
	docker cp `docker run -d $(BUILD_IMAGE)`:/nginx.tgz image/
	docker cp `docker run -d $(BUILD_IMAGE)`:/go/bin/boot image/bin/
	docker build -t $(IMAGE) image
	rm image/nginx.tgz
	rm image/bin/boot

clean: check-docker check-registry
	rm -rf Godeps
	docker rmi $(IMAGE)

full-clean: check-docker check-registry
	docker images -q $(IMAGE_PREFIX)$(COMPONENT) | xargs docker rmi -f

install: check-deisctl
	deisctl scale $(COMPONENT)=3

uninstall: check-deisctl
	deisctl scale $(COMPONENT)=0

start: check-deisctl
	deisctl start $(COMPONENT)@*

stop: check-deisctl
	deisctl stop $(COMPONENT)@*

restart: stop start

run: install start

dev-release: push set-image

push: check-registry
	docker tag -f $(IMAGE) $(DEV_IMAGE)
	docker push $(DEV_IMAGE)

set-image: check-deisctl
	deisctl config $(COMPONENT) set image=$(DEV_IMAGE)

release:
	docker push $(IMAGE)

deploy: build dev-release restart

test: test-style test-unit test-functional

test-functional:
	@docker history deis/test-etcd >/dev/null 2>&1 || docker pull deis/test-etcd:latest
	GOPATH=`cd ../tests/ && godep path`:$(GOPATH) go test -v ./tests/...

test-style:
# display output, then check
	$(GOFMT) $(GO_PACKAGES) $(GO_FILES)
	@$(GOFMT) $(GO_PACKAGES) $(GO_FILES) | read; if [ $$? == 0 ]; then echo "gofmt check failed."; exit 1; fi
# FIXME: make this mandatory
	-$(GOVET) $(repo_path) $(GO_PACKAGES_REPO_PATH)
# FIXME: make this mandatory
	-$(GOLINT) ./...

test-unit:
	@echo no unit tests
