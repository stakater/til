# Operator Findings

An Operator is nothing more than a set of application-specific custom controllers. So in our case we needed an application specific controller to manage a new CRD that will act as our templated config.

## How to create one

### The need for an operator toolkit

Operators make it easy to manage complex stateful applications on top of Kubernetes. However writing an operator today can be difficult because of challenges such as using low level APIs, writing boilerplate, and a lack of modularity which leads to duplication.

There's a bunch of operator sdk's / api's that have been written to facilitate operator development. Some of them are listed below:

- [controllerutil](https://github.com/ianlewis/controllerutil)
- [operatorkit](https://github.com/giantswarm/operatorkit)
- [operator-kit](https://github.com/rook/operator-kit)
- [operator-sdk](https://github.com/operator-framework/operator-sdk)

The most popular one being `operator-sdk`. It has by far the best API for creating a controller and can be easily used for creating and managing an operator with a CRD.

## Operator SDK

The Operator SDK is a framework designed to make writing operators easier by providing:

- High level APIs and abstractions to write the operational logic more intuitively
- Tools for scaffolding and code generation to bootstrap a new project fast
- Extensions to cover common operator use cases

The SDK provides the following workflow to develop a new operator:

1. Create a new operator project using the SDK Command Line Interface(CLI)
2. Define new resource APIs by adding Custom Resource Definitions(CRD)
3. Specify resources to watch using the SDK API
4. Define the operator reconciling logic in a designated handler and use the SDK API to interact with resources
5. Use the SDK CLI to build and generate the operator deployment manifests

### Using the operator SDK

First, checkout and install the operator-sdk CLI:

```bash
mkdir -p $GOPATH/src/github.com/operator-framework
cd $GOPATH/src/github.com/operator-framework
git clone https://github.com/operator-framework/operator-sdk
cd operator-sdk
git checkout master
make dep
make install
```

Create an app using the SDK CLI:

```bash
# Create an app-operator project that defines the App CR.
cd $GOPATH/src/github.com/stakater/
operator-sdk new app-operator --api-version=app.example.com/v1alpha1 --kind=App
cd app-operator
```

After that we can add our CRD handling logic inside `$GOPATH/src/github/stakater/app-operator/pkg/stub/handler.go`'s Handle function

```go
func (h *Handler) Handle(ctx types.Context, event types.Event) error {
    switch o := event.Object.(type) {
    case *api.App:
        // Your logic to do with the app here
    }
    return nil
}
```