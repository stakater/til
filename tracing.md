# I can “get distributed tracing” from the service mesh without having to make any changes to my application?

*NO* Unfortunately, while a service mesh can emit trace spans from the proxies, the application will need to propagate headers in order for these spans to be pulled together into traces–i.e. copy any distributed tracing header that is attached to an incoming request to each resulting outgoing request.

In other words, in contrast to nearly every other service mesh feature, distributed tracing mesh can only work if developers make changes to their code. Note that this isn’t a flaw with Linkerd; this is a function of the sidecar proxy model and is true of any sidecar-based service mesh.
