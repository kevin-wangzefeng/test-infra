# Copyright 2017 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

PROJECT ?= krzyzacy-k8s-test
ZONE ?= us-central1-f
CLUSTER ?= boskos-test

TAG = $(shell date +v%Y%m%d)-$(shell git describe --tags --always --dirty)

boskos:
	go build k8s.io/test-infra/boskos/

client:
	go build -o client/client k8s.io/test-infra/boskos/client/

reaper:
	go build -o reaper/reaper k8s.io/test-infra/boskos/reaper/

server-image:
	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o boskos k8s.io/test-infra/boskos/
	docker build -t "gcr.io/k8s-testimages/boskos:$(TAG)" .
	gcloud docker -- push "gcr.io/k8s-testimages/boskos:$(TAG)"
	rm boskos

reaper-image:
	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o reaper/reaper k8s.io/test-infra/boskos/reaper/
	docker build -t "gcr.io/k8s-testimages/reaper:$(TAG)" reaper
	gcloud docker -- push "gcr.io/k8s-testimages/reaper:$(TAG)"
	rm reaper/reaper

server-deployment:
	kubectl apply -f deployment.yaml

reaper-deployment:
	kubectl apply -f reaper/deployment.yaml

service:
	kubectl apply -f service.yaml

update-config: get-cluster-credentials
	kubectl create configmap resources --from-file=config=resources.json --dry-run -o yaml | kubectl replace configmap resources -f -

get-cluster-credentials:
	gcloud container clusters get-credentials "$(CLUSTER)" --project="$(PROJECT)" --zone="$(ZONE)"

.PHONY: boskos client reaper server-image reaper-image server-deployment service update-config get-cluster-credentials
