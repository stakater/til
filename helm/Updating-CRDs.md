# Updated CRDs for HelmRelease

Flux old apiVersion `flux.weave.works/v1beta1` is replaced by new version `helm.fluxcd.io/v1` which has additional fixes waiting on pods etc.

## Issue if Old is Deleted

If Old CRD with apiVersion `flux.weave.works/v1beta1` is deleted all the HelmReleases deployed using this CRD will get deleted too.

## How to avoid

To avoid this problem:

1. Create the new CRD
2. Replace apiVersion of every released HelmRelease to new apiVersion.
3. Apply and verify if releases are deployed successfully.
4. Delete old CRD