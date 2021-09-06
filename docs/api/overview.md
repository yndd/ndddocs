# API Documentation

The ndd ecosystem contains many CRDs that map to API types represented by
external network device providers. The documentation for these CRDs are
auto-generated on [doc.crds.dev]. To find the CRDs available for providers
maintained by the ndd team, you can search for the Github URL, or
append it in the [doc.crds.dev] URL path.

For instance, to find the CRDs available for [ndd-provider-srl], you would go to:

[doc.crds.dev/github.com/yndd/ndd-provider-srl]

By default, you will be served the latest CRDs on the `master` branch for the
repository. If you prefer to see the CRDs for a specific version, you can append
the git tag for the release:

[doc.crds.dev/github.com/yndd/ndd-provider-srl@v0.1.4]

Ndd repositories that are not providers but do publish CRDs are also
served on [doc.crds.dev]. For instance, the [yndd/ndd-core] repository.

Bugs and feature requests for API documentation should be [opened as issues] on
the open source [doc.crds.dev repo].


[doc.crds.dev]: https://doc.crds.dev/
[ndd-provider-srl]: https://github.com/yndd/ndd-provider-srl
[doc.crds.dev/github.com/yndd/ndd-provider-srl]: https://doc.crds.dev/github.com/yndd/ndd-provider-srl
[doc.crds.dev/github.com/yndd/ndd-provider-srl@v0.1.4]: https://doc.crds.dev/github.com/yndd/ndd-provider-srl@v0.1.4
[yndd/ndd-core]: https://doc.crds.dev/github.com/yndd/ndd-core
[opened as issues]: https://github.com/crdsdev/doc/issues/new
[doc.crds.dev repo]: https://github.com/crdsdev/doc