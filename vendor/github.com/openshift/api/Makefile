all: build
.PHONY: all

# Ensure update-scripts are run before crd-gen so updates to Godoc are included in CRDs.
update-codegen-crds: update-scripts

# Include the library makefile
include $(addprefix ./vendor/github.com/openshift/build-machinery-go/make/, \
	golang.mk \
	targets/openshift/deps.mk \
	targets/openshift/crd-schema-gen.mk \
)

EXCLUDE_DIRS := _output/ dependencymagnet/ hack/ third_party/ tls/ tools/ vendor/ tests/
GO_PACKAGES :=$(addsuffix ...,$(addprefix ./,$(filter-out $(EXCLUDE_DIRS), $(wildcard */))))
GO_BUILD_PACKAGES :=$(GO_PACKAGES)
GO_BUILD_PACKAGES_EXPANDED :=$(GO_BUILD_PACKAGES)
# LDFLAGS are not needed for dummy builds (saving time on calling git commands)
GO_LD_FLAGS:=
CONTROLLER_GEN_VERSION :=v0.9.2+openshift-0.2

# $1 - target name
# $2 - apis
# $3 - manifests
# $4 - output
$(call add-crd-gen,authorization,./authorization/v1,./authorization/v1,./authorization/v1)
$(call add-crd-gen,apiserver,./apiserver/v1,./apiserver/v1,./apiserver/v1)
$(call add-crd-gen,config,./config/v1,./config/v1,./config/v1)
$(call add-crd-gen,config-alpha,./config/v1alpha1,./config/v1alpha1,./config/v1alpha1)
$(call add-crd-gen-for-featureset,config,./config/v1,./config/v1,./config/v1,Default)
$(call add-crd-gen-for-featureset,config,./config/v1,./config/v1,./config/v1,TechPreviewNoUpgrade)
$(call add-crd-gen,helm,./helm/v1beta1,./helm/v1beta1,./helm/v1beta1)
$(call add-crd-gen,console,./console/...,./console/v1,./console/v1)
$(call add-crd-gen,console-alpha,./console/...,./console/v1alpha1,./console/v1alpha1)
$(call add-crd-gen,example,./example/v1,./example/v1,./example/v1)
$(call add-crd-gen-for-featureset,example,./example/v1,./example/v1,./example/v1,TechPreviewNoUpgrade)
$(call add-crd-gen-for-featureset,example,./example/v1,./example/v1,./example/v1,Default)
$(call add-crd-gen,example-alpha,./example/v1alpha1,./example/v1alpha1,./example/v1alpha1)
$(call add-crd-gen-for-featureset,example-alpha,./example/v1alpha1,./example/v1alpha1,./example/v1alpha1,TechPreviewNoUpgrade)
$(call add-crd-gen,imageregistry,./imageregistry/v1,./imageregistry/v1,./imageregistry/v1)
$(call add-crd-gen,operator,./operator/v1,./operator/v1,./operator/v1)
$(call add-crd-gen,operator-alpha,./operator/v1alpha1,./operator/v1alpha1,./operator/v1alpha1)
$(call add-crd-gen,operatoringress,./operatoringress/v1,./operatoringress/v1,./operatoringress/v1)
$(call add-crd-gen,sharedresource,./sharedresource/v1alpha1,./sharedresource/v1alpha1,./sharedresource/v1alpha1)
$(call add-crd-gen,quota,./quota/v1,./quota/v1,./quota/v1)
$(call add-crd-gen,samples,./samples/v1,./samples/v1,./samples/v1)
$(call add-crd-gen,security,./security/v1,./security/v1,./security/v1)
$(call add-crd-gen,securityinternal,./securityinternal/v1,./securityinternal/v1,./securityinternal/v1)
$(call add-crd-gen,cloudnetwork,./cloudnetwork/v1,./cloudnetwork/v1,./cloudnetwork/v1)
$(call add-crd-gen,network,./network/v1,./network/v1,./network/v1)
$(call add-crd-gen,networkoperator,./networkoperator/v1,./networkoperator/v1,./networkoperator/v1)
$(call add-crd-gen,operatorcontrolplane,./operatorcontrolplane/v1alpha1,./operatorcontrolplane/v1alpha1,./operatorcontrolplane/v1alpha1)
$(call add-crd-gen,machine-beta,./machine/v1beta1,./machine/v1beta1,./machine/v1beta1)
$(call add-crd-gen,machine,./machine/v1,./machine/v1,./machine/v1)
$(call add-crd-gen,monitoring-alpha,./monitoring/v1alpha1,./monitoring/v1alpha1,./monitoring/v1alpha1)
$(call add-crd-gen,route,./route/v1,./route/v1,./route/v1)
$(call add-crd-gen,platform,./platform/v1alpha1,./platform/v1alpha1,./platform/v1alpha1)

RUNTIME ?= podman
RUNTIME_IMAGE_NAME ?= registry.ci.openshift.org/openshift/release:rhel-8-release-golang-1.18-openshift-4.12

verify-scripts:
	bash -x hack/verify-deepcopy.sh
	bash -x hack/verify-openapi.sh
	bash -x hack/verify-protobuf.sh
	bash -x hack/verify-swagger-docs.sh
	hack/verify-crds.sh
	bash -x hack/verify-types.sh
	bash -x hack/verify-compatibility.sh
	bash -x hack/verify-integration-tests.sh

.PHONY: verify-scripts
verify: verify-scripts verify-codegen-crds verify-codegen-TechPreviewNoUpgrade-crds verify-codegen-Default-crds

################################################################################################
#
# BEGIN: Update scripts. Defaults to generating updates for all API packages.
#        Set API_GROUP_VERSIONS to a space separated list of <group>/<version> to limit
#        the scope of the updates. Eg API_GROUP_VERSIONS="apps/v1 build/v1" make update-scripts.
#        Note: Protobuf generation is handled separately, see hack/lib/init.sh.
#
################################################################################################

.PHONY: update-scripts
update-scripts: update-compatibility update-openapi update-deepcopy update-protobuf update-swagger-docs update-codegen-TechPreviewNoUpgrade-crds update-codegen-Default-crds tests-vendor

.PHONY: update-compatibility
update-compatibility:
	hack/update-compatibility.sh

.PHONY: update-openapi
update-openapi:
	hack/update-openapi.sh

.PHONY: update-deepcopy
update-deepcopy:
	hack/update-deepcopy.sh

.PHONY: update-protobuf
update-protobuf:
	hack/update-protobuf.sh

.PHONY: update-swagger-docs
update-swagger-docs:
	hack/update-swagger-docs.sh

#####################
#
# END: Update scripts
#
#####################

verify-with-container:
	$(RUNTIME) run -ti --rm -v $(PWD):/go/src/github.com/openshift/api:z -w /go/src/github.com/openshift/api $(RUNTIME_IMAGE_NAME) make verify

generate-with-container:
	$(RUNTIME) run -ti --rm -v $(PWD):/go/src/github.com/openshift/api:z -w /go/src/github.com/openshift/api $(RUNTIME_IMAGE_NAME) make update

.PHONY: integration
integration:
	make -C tests integration

tests-vendor:
	make -C tests vendor
