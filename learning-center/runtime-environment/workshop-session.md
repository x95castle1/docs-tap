# Workshop session resource <!-- For VMware style, write all headers in sentence case. -->

The ``WorkshopSession`` custom resource defines a workshop session.

The raw custom resource definition <!-- Shorten to |CRD| after first use. -->for the ``WorkshopSession`` custom resource can be<!-- Consider switching to active voice. --> viewed at:

- [https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/workshop-session.yaml](https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/workshop-session.yaml) <!-- Type |in GitHub| somewhere in the cross-reference sentence. -->

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Specifying the session identity

When running training for multiple people, it would<!-- Re-phrase for present tense if possible. --> be more typical to use the ``TrainingPortal`` custom resource to set up a training environment. Alternatively you would<!-- Re-phrase for present tense if possible. --> set up a workshop environment using the ``WorkshopEnvironment`` custom resource, then create requests for workshop instances using the ``WorkshopRequest`` custom resource. If doing the latter and you need<!-- Avoid anthropomorphizing: |require| might be better here. --> more control over how the workshop instances are set up, you can use ``WorkshopSession`` custom resource instead of ``WorkshopRequest``.

To specify<!-- Use as a general term to instruct users when they have several selections to make. Do not use in procedures when a more specific term, such as enter, select, or click, is appropriate. --> the workshop environment the workshop instance is created against, set the ``environment.name`` field of the specification for the workshop session. At the same time, you must specify the session ID for the workshop instance.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopSession
metadata:
  name: lab-markdown-sample-user1
spec:
  environment:
    name: lab-markdown-sample
  session:
    id: user1
```

The ``name`` of the workshop specified in the ``metadata`` of the training environment needs to<!-- |must| is preferred. --> be globally unique for the workshop instance being created. You would<!-- Re-phrase for present tense if possible. --> need to<!-- |must| is preferred. --> create a separate ``WorkshopSession`` custom resource for each workshop instance you want.

The session ID needs to<!-- |must| is preferred. --> be unique within the workshop environment the workshop instance is being created against.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Specifying the login credentials

Access to each workshop instance can be<!-- Consider switching to active voice. --> controlled through<!-- Do not use as a synonym for |finished| or |done|. Do not use when you mean |by using|. --> login credentials. This is so that a workshop attendee cannot interfere with another.

If you want to<!-- Maybe replace with just |To|. --> set login credentials for a workshop instance, you can set the ``session.username<!-- |user name| is preferred. -->`` and ``session.password`` fields.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopSession
metadata:
  name: lab-markdown-sample
spec:
  environment:
    name: lab-markdown-sample-user1
  session:
    username: eduk8s
    password: lab-markdown-sample
```

If you do not specify<!-- Use as a general term to instruct users when they have several selections to make. Do not use in procedures when a more specific term, such as enter, select, or click, is appropriate. --> login credentials, there will<!-- Avoid |will|: present tense is preferred. --> not be any access controls on the workshop instance and anyone will<!-- Avoid |will|: present tense is preferred. --> be able to<!-- |can| is preferred. --> access it.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Specifying the ingress domain

In order to be able to<!-- |can| is preferred. --> access the workshop instance using a public URL, you will<!-- Avoid |will|: present tense is preferred. --> need to<!-- |must| is preferred. --> specify<!-- Use as a general term to instruct users when they have several selections to make. Do not use in procedures when a more specific term, such as enter, select, or click, is appropriate. --> an ingress domain. If an ingress domain isn't specified, the default ingress domain that the Learning Center operator has been<!-- Consider changing to |is| or |has| or rewrite for active voice. --> configured with will<!-- Avoid |will|: present tense is preferred. --> be used.

When setting a custom domain, DNS must have been<!-- Consider replacing with |were| or shifting to present tense. --> configured with a wildcard domain to forward all requests for sub domains of the custom domain, to the ingress router of the Kubernetes cluster.

To provide the ingress domain, you can set the ``session.ingress.domain`` field.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopSession
metadata:
  name: lab-markdown-sample
spec:
  environment:
    name: lab-markdown-sample-user1
  session:
    ingress:
      domain: training.eduk8s.io
```

A full hostname<!-- |host name| is preferred. --> for the session will<!-- Avoid |will|: present tense is preferred. --> be created by<!-- Active voice is preferred. --> prefixing the ingress domain with a hostname<!-- |host name| is preferred. --> constructed from the name of the workshop environment and the session ID.

If overriding the domain, by default, the workshop session will<!-- Avoid |will|: present tense is preferred. --> be exposed using a HTTP connection. If you require a secure HTTPS connection, you will<!-- Avoid |will|: present tense is preferred. --> need to<!-- |must| is preferred. --> have access to a wildcard SSL certificate for the domain. A secret of type ``tls`` should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> be created for the certificate in the ``educates`` namespace or the namespace where Learning Center operator is deployed. The name of that secret should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> then be set in the ``session.ingress.secret`` field.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopSession
metadata:
  name: lab-markdown-sample
spec:
  environment:
    name: lab-markdown-sample-user1
  session:
    ingress:
      domain: training.eduk8s.io
      secret: training.eduk8s.io-tls
```

If HTTPS connections are being terminated using an external load balancer and not by specifying a secret for ingresses managed by the Kubernetes ingress controller, with traffic then routed into the Kubernetes cluster as HTTP connections, you can override the ingress protocol without specifying an ingress secret by setting the ``session.ingress.protocol`` field.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopSession
metadata:
  name: lab-markdown-sample
spec:
  environment:
    name: lab-markdown-sample-user1
  session:
    ingress:
      domain: training.eduk8s.io
      protocol: https
```

If you need to<!-- Consider replacing with just |To|. --><!-- |must| is preferred. --> override or set the ingress class, which dictates which ingress router is used when more than one option is available, you can add ``session.ingress.class``.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopSession
metadata:
  name: lab-markdown-sample
spec:
  environment:
    name: lab-markdown-sample-user1
  session:
    ingress:
      domain: training.eduk8s.io
      secret: training.eduk8s.io-tls
      class: nginx
```

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Setting the environment variables

If you want to<!-- Maybe replace with just |To|. --> set the environment variables for the workshop instance, you can provide the environment variables in the ``session.env<!-- |environment| is preferred -->`` field.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopSession
metadata:
  name: lab-markdown-sample
spec:
  environment:
    name: lab-markdown-sample
  session:
    id: user1
    env:
    - name: REPOSITORY_URL
      value: https://github.com/eduk8s/lab-markdown-sample
``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->

Values of fields in the list of resource objects can reference a number of pre-defined parameters. The available parameters are:

* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`session_id`` - A unique ID for the workshop instance within the workshop environment.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`session_namespace`` - The namespace created for and bound to the workshop instance. This is the namespace unique to the session and where a workshop can create their own resources.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`environment_name`` - The name of the workshop environment. For now this is the same as the name of the namespace for the workshop environment. Don't rely on them being the same, and use the most appropriate to cope with any future<!-- Only document what exists. There are legal ramifications to making promises. --> change.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`workshop_namespace`` - The namespace for the workshop environment. This is the namespace where all deployments of the workshop instances are created, and where the service account that the workshop instance runs as exists.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`service_account`` - The name of the service account the workshop instance runs as, and which has access to the namespace created for that workshop instance.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`ingress_domain`` - The host domain under which hostnames<!-- |host names| is preferred. --> can be<!-- Consider switching to active voice. --> created when creating ingress routes.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`ingress_protocol`` - The protocol (http/https) that is used for ingress routes which are created for workshops.

The syntax for referencing one of the parameters is ``$(parameter_name)``.

Note<!-- Format the note properly. --> that<!-- Notes must be in Note boxes and start with |Note: |. --> if the workshop environment had specified a set of extra environment variables to be set for workshop instances, it is up to you to incorporate those in the set of environment variables you list under ``session.env<!-- |environment| is preferred -->``. That is, anything listed in ``session.env<!-- |environment| is preferred -->`` of the ``WorkshopEnvironment`` custom resource of the workshop environment is ignored.
