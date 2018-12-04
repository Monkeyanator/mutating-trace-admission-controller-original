# mutating-trace-admission-controller

Kubernetes [mutating admission webhook](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#mutatingadmissionwebhook) which injects base64 encoded [OpenCensus span context](https://github.com/census-instrumentation/opencensus-specs/blob/master/trace/Span.md#spancontext) into pod objects.

## Purpose

This controller adds trace context to pod objects as an annotation with the key `trace.kubernetes.io/context`. This trace context is used by Kubernetes components to export traces associated with lifecycle events of the object.

For more detailed information on the how and why of adding distributed tracing to Kubernetes, [please refer to this design document](https://docs.google.com/document/d/1cqdw7JfHSovl1E-FoH4rTpI32Xt0saZvdKv6q6-v4uc/edit#heading=h.xgjl2srtytjt).

// TODO(Monkeyanator): Link to KEP rather than design document

## How to run

The structure of this mutating admission controller was informed by the [mutating admission webhook found here](https://github.com/morvencao/kube-mutating-webhook-tutorial). The basic idea is as follows:

1) Create an HTTPS-enabled server that takes Pod json from the API server, inserts encoded span context as an annotation, and returns it (which requires some certificate acrobatics)
2) Run a deployment with this webhook server, and expose it as a service
3) Create a `MutatingWebhookConfiguration` which instructs the API server to send Pod objects to the aforementioned service upon creation

The included `Makefile` makes these steps straightforward and the available commands are as follows:

* `make docker-build`: build local Docker image
* `make docker-release`: build local Docker image, tag with `latest`, and push to specified registry
* `make cluster-up`: apply certificate configuration and deployment configuration to cluster for the mutating webhook
* `make cluster-down`: delete resources associated with the mutating webhook from the active cluster