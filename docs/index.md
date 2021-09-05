# Welcome to NDD docs

NDD is an opensource [Kubernetes](https://kubernetes.io) add-on that enables platform and application teams to consume network devices in a similar way as other resources are consumed in [Kubernetes](https://kubernetes.io). 

NDD uses a modular approach, through providers, which allows multiple network device types to be supported. NDD allows the network providers to be generated from [YANG data models](https://en.wikipedia.org/wiki/YANG), which enables rapid enablement of multiple network device types. Through YANG we can provider automate input and dependency management between the various resource that are consumed within the device.

An NDD provider represents the device model through various [Customer Resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) within the Kubernetes API in order to provide flexible management of the device resources.

NDD is build on the basis of the [kubebuilder](https://kubebuilder.io) and [operator pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) within kubernetes.

Features:

* Device discovery and Provider registration
* Declaritive CRUD configuration of network devices through [Customer Resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
* Configuration Input Validation:
    - Declarative validation using an OpenAPI v3 schema derived from [YANG](https://en.wikipedia.org/wiki/YANG)
    - Runtime Dependency Management amongst the various resources comsumed within a device (parent dependency management and leaf reference dependency management amont resources)
* Automatic or Operator interacted configuration drift management
* Delete Policy, and Active etc 