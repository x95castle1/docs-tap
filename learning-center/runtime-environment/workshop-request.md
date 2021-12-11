# Workshop request resource

The ``WorkshopRequest`` custom resource defines a workshop request.

The raw CRD <!-- Shorten to |CRD| after first use. -->for the ``WorkshopRequest`` custom resource can be<!-- Consider switching to active voice. --> viewed at:

- [https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/workshop-request.yaml](https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/workshop-request.yaml) <!-- Type |in GitHub| somewhere in the cross-reference sentence. -->

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Specifying workshop environment

The ``WorkshopRequest`` custom resource is only used to request a workshop instance. It does not specify<!-- Use as a general term to instruct users when they have several selections to make. Do not use in procedures when a more specific term, such as enter, select, or click, is appropriate. --> actual details needed to perform the deployment of the workshop instance. That information is instead sourced by the Learning Center operator from the ``WorkshopEnvironment`` and ``Workshop`` custom resources.

The minimum required information in the workshop request is therefore just<!-- Avoid uses that suggest a task is simple. --> the name of the workshop environment. This is supplied by<!-- Active voice is preferred. --> setting the ``environment.name`` field.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopRequest
metadata:
  name: lab-markdown-sample
spec:
  environment:
    name: lab-markdown-sample
```

A request will<!-- Avoid |will|: present tense is preferred. --> only be successful if the ability to<!-- |can| is shorter. Avoid nounification of verbs where possible. --> request a workshop instance for a workshop environment has been<!-- Consider changing to |is| or |has| or rewrite for active voice. --> enabled for that workshop. Enabling of requests needs to<!-- |must| is preferred. --> have been<!-- Consider replacing with |were| or shifting to present tense. --> specified in the ``WorkshopEnvironment`` custom resource for the workshop environment.

If multiple workshop requests, whether for the same workshop environment or different ones, are created in the same namespace, the ``name`` defined in the ``metadata`` for the workshop request must be different for each. The value of this name is not important and is not used in naming of workshop instances. A user will<!-- Avoid |will|: present tense is preferred. --> need to<!-- |must| is preferred. --> remember it if they want<!-- Avoid anthropomorphizing by writing that software |wants| anything. --> to delete the workshop instance, which is done by deleting the workshop request.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Specifying required access token

Where a workshop environment has been<!-- Consider changing to |is| or |has| or rewrite for active voice. --> configured to require an access token when making workshop request against that environment, it can be<!-- Consider switching to active voice. --> specified by<!-- Active voice is preferred. --> setting the ``environment.token`` field.

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

Even with the token, if the workshop environment has restricted the namespaces a workshop request has been<!-- Consider changing to |is| or |has| or rewrite for active voice. --> made from, and the workshop request was not created in one of the white listed namespaces, the request will<!-- Avoid |will|: present tense is preferred. --> fail.
