# Custom resources


It is possible<!-- |might| might be better. --> to<!-- Active voice |you can| might be better. --> deploy workshop images directly to a container runtime, but for managing deployments into a Kubernetes cluster,
the Learning Center operator is provided. The operation of the Learning Center operator is controlled through<!-- Do not use as a synonym for |finished| or |done|. Do not use when you mean |by using|. --> a set of
Kubernetes custom resource definitions (CRDs).

Not all possible fields are shown in the examples of each custom resource type below<!-- IX Standards forbids referring to text |below|. Use |following| or |later| or, better, just an anchor. -->. Later documentation will<!-- Avoid |will|: present tense is preferred. --> go into depth o
n all the possible fields that can be<!-- Consider switching to active voice. --> set and what they do.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Workshop definition resource

The ``Workshop`` custom resource defines a workshop. It specifies the title and description of the workshop, the location of the workshop content or container image to be deployed, any resources to be pre-created in the workshop environment, or for each instance of the workshop. You can also define environment variables to be set for the workshop image when deployed, the amount of CPU and memory resources for the workshop instance, and any overall quota to be applied to namespaces created for the user and which would<!-- Re-phrase for present tense if possible. --> be used by<!-- Active voice is preferred. --> the workshop.

A minimal example of the ``Workshop`` custom resource is:

```
apiVersion: training.eduk8s.io/v1alpha2
kind: Workshop
metadata:
  name: lab-markdown-sample
spec:
  title: Markdown Sample
  description: A sample workshop using Markdown
  content:
    files: github.com/eduk8s/lab-markdown-sample
  session:
    namespaces:
      budget: small
    applications:
      console:
        enabled: true
      editor:
        enabled: true
```

The raw custom resource definition <!-- Shorten to |CRD| after first use. -->for the ``Workshop`` custom resource can be<!-- Consider switching to active voice. --> viewed at:

- [https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/workshop.yaml](https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/workshop.yaml) <!-- Type |in GitHub| somewhere in the cross-reference sentence. -->

When an instance of the ``Workshop`` custom resource is created it does not cause any immediate action by the Learning Center operator. This custom resource exists only to define the workshop.

The ``Workshop`` custom resource is created at cluster scope.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Workshop environment resource

In order to deploy instances of a workshop, you first need to<!-- |must| is preferred. --> create a workshop environment. The configuration for the workshop environment, and which workshop definition specifies the details of the workshop to be deployed, is defined the by the ``WorkshopEnvironment`` custom resource.

A minimal example of the ``WorkshopEnvironment`` custom resource is:

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopEnvironment
metadata:
  name: lab-markdown-sample
spec:
  workshop:
    name: lab-markdown-sample
  request:
    token: lab-markdown-sample
  session:
    username: eduk8s
```

When an instance of the ``WorkshopEnvironment`` custom resource is created, the Learning Center operator responds by creating a namespace for hosting the workshop instances defined by the ``Workshop`` resource specified by the ``spec<!-- |specifications| is preferred. -->.workshop.name`` field. The namespace created will<!-- Avoid |will|: present tense is preferred. --> use the same name as specified by the ``metadata.name`` field of the ``WorkshopEnvironment`` resource.

The ``spec<!-- |specifications| is preferred. -->.request.token`` field defines a token which must be supplied with a request to create an instance of a workshop in this workshop environment. If necessary, the namespaces from which a request for a workshop instance can be<!-- Consider switching to active voice. --> initiated can also be specified.

If the ``Workshop`` definition for the workshop to be deployed in this workshop environment defines a set of common resources which must exist for the workshop, these will<!-- Avoid |will|: present tense is preferred. --> be created by<!-- Active voice is preferred. --> the Learning Center operator after the namespace for the workshop environment is created. Where such resources are namespaced, they will<!-- Avoid |will|: present tense is preferred. --> be created in the namespace for the workshop environment. If necessary, these resources can include creation of separate namespaces with specific resources created in those namespaces instead.

The raw custom resource definition <!-- Shorten to |CRD| after first use. -->for the ``WorkshopEnvironment`` custom resource can be<!-- Consider switching to active voice. --> viewed at:

- [https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/workshop-environment.yaml](https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/workshop-environment.yaml) <!-- Type |in GitHub| somewhere in the cross-reference sentence. -->

The ``WorkshopEnvironment`` custom resource is created at cluster scope.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Workshop request resource

To create an instance of the workshop under the workshop environment which was created, the typical path is to create an instance of the ``WorkshopRequest`` custom resource.

The ``WorkshopRequest`` custom resource is namespaced to allow who can create it and so request a workshop instance to be created, to be controlled through<!-- Do not use as a synonym for |finished| or |done|. Do not use when you mean |by using|. --> RBAC. This means it is possible<!-- |might| might be better. --> to<!-- Active voice |you can| might be better. --> allow non privileged users to create workshops, even though the deployment of the workshop instance may<!-- |can| usually works better. Use |might| to convey possibility. --> need<!-- Avoid anthropomorphizing: |require| might be better here. --> elevated privileges.

A minimal example of the ``WorkshopRequest`` custom resource is:

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopRequest
metadata:
  name: lab-markdown-sample
spec:
  environment:
    name: lab-markdown-sample
    token: lab-markdown-sample
```

Apart from needing to have appropriate access through<!-- Do not use as a synonym for |finished| or |done|. Do not use when you mean |by using|. --> RBAC, the only information that the user requesting a workshop instance needs to<!-- |must| is preferred. --> know is the the name of the workshop environment for the workshop, and the secret token which permits workshop requests against that specific workshop environment.

The raw custom resource definition <!-- Shorten to |CRD| after first use. -->for the ``WorkshopRequest`` custom resource can be<!-- Consider switching to active voice. --> viewed at:

- [https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/workshop-request.yaml](https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/workshop-request.yaml) <!-- Type |in GitHub| somewhere in the cross-reference sentence. -->

Note<!-- Format the note properly. --> <!-- Proper note formatting is |>**Note:**| or |>**Caution:**| or |>**Important:**|. --> that the ``WorkshopRequest`` resource is not used when using the ``TrainingPortal`` resource to provide a web interface for accessing workshops. The ``WorkshopRequest`` resource is only used where you were creating ``WorkshopEnvironment`` resource manually and not using the training portal.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Workshop session resource

Although ``WorkshopRequest`` would<!-- Re-phrase for present tense if possible. --> be the typical way that workshop instances would<!-- Re-phrase for present tense if possible. --> be requested, upon the request being granted, the Learning Center operator will<!-- Avoid |will|: present tense is preferred. --> itself create an instance of a ``WorkshopSession`` custom resource.

The ``WorkshopSession`` custom resource is the expanded definition of what the workshop instance should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> look like<!-- Do not use when citing examples. Use |such as| instead. -->. It combines details from ``Workshop`` and ``WorkshopEnvironment``, and also links back to the ``WorkshopRequest`` resource object which triggered the request. The Learning Center operator reacts to an instance of ``WorkshopSession`` and creates the workshop instance based on that definition.

The raw custom resource definition <!-- Shorten to |CRD| after first use. -->for the ``WorkshopSession`` custom resource can be<!-- Consider switching to active voice. --> viewed at:

- [https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/workshop-session.yaml](https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/workshop-session.yaml) <!-- Type |in GitHub| somewhere in the cross-reference sentence. -->

The ``WorkshopSession`` custom resource is created at cluster scope.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Training portal resource

The ``TrainingPortal`` custom resource provides a high level mechanism for creating a set of workshop environments and populating them with workshop instances.

A minimal example of the ``TrainingPortal`` custom resource is:

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  workshops:
  - name: lab-markdown-sample
    capacity: 1
```

You can set the capacity of the training room and that dictates how many workshop instances are created for each workshop.

The raw custom resource definition <!-- Shorten to |CRD| after first use. -->for the ``TrainingPortal`` custom resource can be<!-- Consider switching to active voice. --> viewed at:

- [https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/training-portal.yaml](https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/training-portal.yaml) <!-- Type |in GitHub| somewhere in the cross-reference sentence. -->

The ``TrainingPortal`` custom resource is created at cluster scope.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->System profile resource

The ``SystemProfile`` custom resources provides a mechanism for configuring the Learning Center operator. This provides additional features above<!-- IX Standards forbids referring to text |above|. Use |earlier| or, better, just an anchor. --> using using environment variables to configure the operator.

A minimal example of the ``SystemProfile`` custom resource is:

```
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  ingress:
    domain: training.eduk8s.io
    secret: training-eduks8-io-tls
    class: nginx
  environment:
    secrets:
      pull:
      - cluster-image-registry-pull
```

The operator by default will<!-- Avoid |will|: present tense is preferred. --> look for a default system profile called ``default-system-profile``. The name of the default can be<!-- Consider switching to active voice. --> overridden globally by setting the ``SYSTEM_PROFILE`` environment variable on the deployment for the operator, or for specific deployments via<!-- |through|, |using| and |by means of| are preferred. --> the ``system.profile`` setting on ``TrainingPortal``, ``WorkshopEnvironment`` or ``WorkshopSession`` custom resources.

As only a global deployment of the operator is supported, the ``SystemProfile`` custom resource is created at cluster scope.

Changes can be<!-- Consider switching to active voice. --> made to instances of the ``SystemProfile`` custom resource and they will<!-- Avoid |will|: present tense is preferred. --> be automatically<!-- Avoid if not definitely useful to mention. --> used by the Learning Center operator without needing to redeploy it.

The raw custom resource definition <!-- Shorten to |CRD| after first use. -->for the ``SystemProfile`` custom resource can be<!-- Consider switching to active voice. --> viewed at:

- [https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/system-profile.yaml](https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/system-profile.yaml) <!-- Type |in GitHub| somewhere in the cross-reference sentence. -->

The ``SystemProfile`` custom resource is created at cluster scope.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Loading the workshop CRDs

The custom resource definitions for the custom resource described above<!-- IX Standards forbids referring to text |above|. Use |earlier| or, better, just an anchor. -->, are created in the Kubernetes cluster when you deploy the Learning Center operator using the Tanzu CLI.

This is because ``v1`` versions of CRDs are only supported from Kubernetes 1.17. If for some reason you need to<!-- Consider replacing with just |To|. --><!-- |must| is preferred. --> use the ``v1`` versions of the CRDs at this time, you will<!-- Avoid |will|: present tense is preferred. --> need to<!-- |must| is preferred. --> create a copy of the Learning Center operator deployment resources and override the configuration so that the ``v1`` versions are used.

The location of the ``v1`` versions of the CRDs is:

* [https://github.com/eduk8s/eduk8s/tree/develop/resources/crds-v1](https://github.com/eduk8s/eduk8s/tree/develop/resources/crds-v1) <!-- Type |in GitHub| somewhere in the cross-reference sentence. -->
