# Workshop environment resource


The ``WorkshopEnvironment`` custom resource defines a workshop environment.

The raw custom resource definition <!-- Shorten to |CRD| after first use. -->for the ``WorkshopEnvironment`` custom resource can be<!-- Consider switching to active voice. --> viewed at:

- [https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/workshop-environment.yaml](https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/workshop-environment.yaml) <!-- Type |in GitHub| somewhere in the cross-reference sentence. -->

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Specifying the workshop definition

The creation of a workshop environment is performed as a separate step to loading the workshop definition. This is to allow multiple distinct workshop environments using the same workshop definition to be created if necessary.

To specify<!-- Use as a general term to instruct users when they have several selections to make. Do not use in procedures when a more specific term, such as enter, select, or click, is appropriate. --> which workshop definition is to be used for a workshop environment, set the ``workshop.name`` field of the specification for the workshop environment.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopEnvironment
metadata:
  name: lab-markdown-sample
spec:
  workshop:
    name: lab-markdown-sample
```

The ``name`` of the workshop environment specified in the ``metadata`` of the workshop environment does not need to<!-- |must| is preferred. --> be the same, and has to be different if creating multiple workshop environments from the same workshop definition.

When the workshop environment is created, the namespace created for the workshop environment will<!-- Avoid |will|: present tense is preferred. --> use the ``name`` of the workshop environment specified in the ``metadata``. This name will<!-- Avoid |will|: present tense is preferred. --> also be used in the unique names of each workshop instance created under the workshop environment.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Overriding environment variables

A workshop definition may<!-- |can| usually works better. Use |might| to convey possibility. --> specify<!-- Use as a general term to instruct users when they have several selections to make. Do not use in procedures when a more specific term, such as enter, select, or click, is appropriate. --> a list of environment variables that need<!-- Avoid anthropomorphizing: |requires| might be better here. --> to<!-- |must| is preferred. --> be set for all workshop instances. If you need to<!-- Consider replacing with just |To|. --><!-- |must| is preferred. --> override an environment variable specified in the workshop definition, or one which is defined in the container image, you can supply a list of environment variables as ``session.env<!-- |environment| is preferred -->``.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopEnvironment
metadata:
  name: lab-markdown-sample
spec:
  workshop:
    name: lab-markdown-sample
  session:
    env:
    - name: REPOSITORY_URL
      value: https://github.com/eduk8s/lab-markdown-sample
``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->

You might use this to set the location of a backend<!-- |back end| is preferred. --> service, such as an image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. -->, to be used by<!-- Active voice is preferred. --> the workshop.

Values of fields in the list of resource objects can reference a number of pre-defined parameters. The available parameters are:

* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`session_id`` - A unique ID for the workshop instance within the workshop environment.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`session_namespace`` - The namespace created for and bound to the workshop instance. This is the namespace unique to the session and where a workshop can create their own resources.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`environment_name`` - The name of the workshop environment. For now this is the same as the name of the namespace for the workshop environment. Don't rely on them being the same, and use the most appropriate to cope with any future<!-- Only document what exists. There are legal ramifications to making promises. --> change.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`workshop_namespace`` - The namespace for the workshop environment. This is the namespace where all deployments of the workshop instances are created, and where the service account that the workshop instance runs as exists.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`service_account`` - The name of the service account the workshop instance runs as, and which has access to the namespace created for that workshop instance.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`ingress_domain`` - The host domain under which hostnames<!-- |host names| is preferred. --> can be<!-- Consider switching to active voice. --> created when creating ingress routes.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`ingress_protocol`` - The protocol (http/https) that is used for ingress routes which are created for workshops.

The syntax for referencing one of the parameters is ``$(parameter_name)``.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Overriding the ingress domain

In order to be able to<!-- |can| is preferred. --> access a workshop instance using a public URL, you will<!-- Avoid |will|: present tense is preferred. --> need to<!-- |must| is preferred. --> specify<!-- Use as a general term to instruct users when they have several selections to make. Do not use in procedures when a more specific term, such as enter, select, or click, is appropriate. --> an ingress domain. If an ingress domain isn't specified, the default ingress domain that the Learning Center operator has been<!-- Consider changing to |is| or |has| or rewrite for active voice. --> configured with will<!-- Avoid |will|: present tense is preferred. --> be used.

When setting a custom domain, DNS must have been<!-- Consider replacing with |were| or shifting to present tense. --> configured with a wildcard domain to forward all requests for sub domains of the custom domain, to the ingress router of the Kubernetes cluster.

To provide the ingress domain, you can set the ``session.ingress.domain`` field.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopEnvironment
metadata:
  name: lab-markdown-sample
spec:
  workshop:
    name: lab-markdown-sample
  session:
    ingress:
      domain: training.eduk8s.io
```

If overriding the domain, by default, the workshop session will<!-- Avoid |will|: present tense is preferred. --> be exposed using a HTTP connection. If you require a secure HTTPS connection, you will<!-- Avoid |will|: present tense is preferred. --> need to<!-- |must| is preferred. --> have access to a wildcard SSL certificate for the domain. A secret of type ``tls`` should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> be created for the certificate in the ``educates`` namespace or the namespace where Learning Center operator is deployed. The name of that secret should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> then be set in the ``session.ingress.secret`` field.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopEnvironment
metadata:
  name: lab-markdown-sample
spec:
  workshop:
    name: lab-markdown-sample
  session:
    ingress:
      domain: training.eduk8s.io
      secret: training.eduk8s.io-tls
```

If HTTPS connections are being terminated using an external load balancer and not by specificying a secret for ingresses managed by the Kubernetes ingress controller, with traffic then routed into the Kubernetes cluster as HTTP connections, you can override the ingress protocol without specifying an ingress secret by setting the ``session.ingress.protocol`` field.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopEnvironment
metadata:
  name: lab-markdown-sample
spec:
  workshop:
    name: lab-markdown-sample
  session:
    ingress:
      domain: training.eduk8s.io
      protocol: https
```

If you need to<!-- Consider replacing with just |To|. --><!-- |must| is preferred. --> override or set the ingress class, which dictates which ingress router is used when more than one option is available, you can add ``session.ingress.class``.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopEnvironment
metadata:
  name: lab-markdown-sample
spec:
  workshop:
    name: lab-markdown-sample
  session:
    ingress:
      domain: training.eduk8s.io
      secret: training.eduk8s.io-tls
      class: nginx
```

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Controlling access to the workshop

By default, the ability to<!-- |can| is shorter. Avoid nounification of verbs where possible. --> request a workshop using the ``WorkshopRequest`` custom resource is disabled<!-- |deactivated| is preferred. --> and so must be enabled for a workshop environment by setting ``request.enabled`` to ``true``.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopEnvironment
metadata:
  name: lab-markdown-sample
spec:
  workshop:
    name: lab-markdown-sample
  request:
    enabled: true
```

With this enabled, anyone able to<!-- |can| is preferred. --> create a ``WorkshopRequest`` custom resource could<!-- |can| or |might| whenever possible is preferred. When providing examples, use simple present tense verbs instead of postulating what someone or something could or would do. --> request the creation of a workshop instance for the workshop environment.

To further control who can request a workshop instance in the workshop environment, you can first set an access token, which a user would<!-- Re-phrase for present tense if possible. --> need to<!-- |must| is preferred. --> know and supply with the workshop request. This can be<!-- Consider switching to active voice. --> done by setting the ``request.token`` field.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopEnvironment
metadata:
  name: lab-markdown-sample
spec:
  workshop:
    name: lab-markdown-sample
  request:
    enabled: true
    token: lab-markdown-sample
```

In this example the same name as the workshop environment is used, which is probably not a good practice. Use a random value instead. The token value can be<!-- Consider switching to active voice. --> multiline if desired<!-- Remove. -->.

As a second measure of control, you can specify<!-- Use as a general term to instruct users when they have several selections to make. Do not use in procedures when a more specific term, such as enter, select, or click, is appropriate. --> what namespaces the ``WorkshopRequest`` needs to<!-- |must| is preferred. --> be created in to be successful. This means a user would<!-- Re-phrase for present tense if possible. --> need to<!-- |must| is preferred. --> have the specific ability to create ``WorkshopRequest`` resources in one of those namespaces.

The list of namespaces from which workshop requests for the workshop environment is allowed can be<!-- Consider switching to active voice. --> specified by<!-- Active voice is preferred. --> setting ``request.namespaces``.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopEnvironment
metadata:
  name: lab-markdown-sample
spec:
  workshop:
    name: lab-markdown-sample
  request:
    enabled: true
    token: lab-markdown-sample
    namespaces:
    - default
```

If you want to<!-- Maybe replace with just |To|. --> add the workshop namespace in the list, rather than list the literal name, you can reference a predefined parameter specifying the workshop namespace by including ``$(workshop_namespace)``.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopEnvironment
metadata:
  name: lab-markdown-sample
spec:
  workshop:
    name: lab-markdown-sample
  request:
    enabled: true
    token: lab-markdown-sample
    namespaces:
    - $(workshop_namespace)
```

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Overriding the login credentials

When requesting a workshop using ``WorkshopRequest``, a login prompt<!-- |dialog box| is preferred. --> for the workshop instance will<!-- Avoid |will|: present tense is preferred. --> be presented to a user when the URL for the workshop instance is accessed. By default the username<!-- |user name| is preferred. --> they need to<!-- |must| is preferred. --> use will<!-- Avoid |will|: present tense is preferred. --> be ``eduk8s``. The password will<!-- Avoid |will|: present tense is preferred. --> be a random value which they need to<!-- |must| is preferred. --> query from the ``WorkshopRequest`` status after the custom resource has been<!-- Consider changing to |is| or |has| or rewrite for active voice. --> created.

If you want to<!-- Maybe replace with just |To|. --> override the username<!-- |user name| is preferred. -->, you can specify<!-- Use as a general term to instruct users when they have several selections to make. Do not use in procedures when a more specific term, such as enter, select, or click, is appropriate. --> the ``session.username<!-- |user name| is preferred. -->`` field. If you want to<!-- Maybe replace with just |To|. --> set the same fixed password for all workshop instances, you can specify<!-- Use as a general term to instruct users when they have several selections to make. Do not use in procedures when a more specific term, such as enter, select, or click, is appropriate. --> the ``session.password`` field.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: WorkshopEnvironment
metadata:
  name: lab-markdown-sample
spec:
  workshop:
    name: lab-markdown-sample
  session:
    username: workshop
    password: lab-markdown-sample
```

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Additional workshop resources

The workshop definition defined by the ``Workshop`` custom resource already declares a set of resources to be created with the workshop environment. This could<!-- |can| or |might| whenever possible is preferred. When providing examples, use simple present tense verbs instead of postulating what someone or something could or would do. --> be used when you have shared service applications needed by the workshop, such as an image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. -->, or a Git repository server.

If you need to<!-- Consider replacing with just |To|. --><!-- |must| is preferred. --> deploy additional applications related to a specific workshop environment, you can declare them by adding them into the ``environment.objects`` field of the ``WorkshopEnvironment`` custom resource. You might use this deploy a web application used by attendees of a workshop to access their workshop instance.

For namespaced resources, it is not necessary to specify<!-- Use as a general term to instruct users when they have several selections to make. Do not use in procedures when a more specific term, such as enter, select, or click, is appropriate. --> the ``namespace`` field of the resource ``metadata``. When the ``namespace`` field is not present the resource will<!-- Avoid |will|: present tense is preferred. --> automatically<!-- Avoid if not definitely useful to mention. --> be created within the workshop namespace for that workshop environment.

When resources are created, owner references are added making the ``WorkshopEnvironment`` custom resource corresponding to the workshop environment the owner. This means that when the workshop environment is deleted, any resources will<!-- Avoid |will|: present tense is preferred. --> be automatically<!-- Avoid if not definitely useful to mention. --> deleted.

Values of fields in the list of resource objects can reference a number of pre-defined parameters. The available parameters are:

* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`workshop_name`` - The name of the workshop. This is the name of the ``Workshop`` definition the workshop environment was created against.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`environment_name`` - The name of the workshop environment. For now this is the same as the name of the namespace for the workshop environment. Don't rely on them being the same, and use the most appropriate to cope with any future<!-- Only document what exists. There are legal ramifications to making promises. --> change.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`environment_token`` - The value of the token which needs to<!-- |must| is preferred. --> be used in workshop requests against the workshop environment.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`workshop_namespace`` - The namespace for the workshop environment. This is the namespace where all deployments of the workshop instances, and their service accounts, are created. It is the same namespace that shared workshop resources are created.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`service_account`` - The name of a service account that can be<!-- Consider switching to active voice. --> used when creating deployments in the workshop namespace.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`ingress_domain`` - The host domain under which hostnames<!-- |host names| is preferred. --> can be<!-- Consider switching to active voice. --> created when creating ingress routes.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`ingress_protocol`` - The protocol (http/https) that is used for ingress routes which are created for workshops.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`ingress_secret`` - The name of the ingress secret stored in the workshop namespace when secure ingress is being used.

If you want to<!-- Maybe replace with just |To|. --> create additional namespaces associated with the workshop environment, embed a reference to ``$(workshop_namespace)`` in the name of the additional namespaces, with an appropriate suffix. Be mindful that the suffix doesn't overlap with the range of session IDs for workshop instances.

When creating deployments in the workshop namespace, set the ``serviceAccountName`` of the ``Deployment`` resouce to ``$(service_account)``. This will<!-- Avoid |will|: present tense is preferred. --> ensure the deployment makes use of a special pod<!-- |Pod| is capitalized per the K8s docs style. --> security policy set up by the Learning Center. If this isn't used and the cluster imposes a more strict default pod<!-- |Pod| is capitalized per the K8s docs style. --> security policy, your deployment may<!-- |can| usually works better. Use |might| to convey possibility. --> not work, especially if any image expects to run as ``root``.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Creation of workshop instances

Once<!-- Only use |once| when you mean |one time|, not when you mean |after|. --> a workshop environment has been<!-- Consider changing to |is| or |has| or rewrite for active voice. --> created you can create the workshop instances. A workshop instance can be<!-- Consider switching to active voice. --> requested using the ``WorkshopRequest`` custom resource. This can be<!-- Consider switching to active voice. --> done as a separate step, or you can use the trick of adding them as resources under ``environment.objects``.

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
    namespaces:
    - $(workshop_namespace)
  session:
    username: eduk8s
    password: lab-markdown-sample
  environment:
    objects:
    - apiVersion: training.eduk8s.io/v1alpha1
      kind: WorkshopRequest
      metadata:
        name: user1
      spec:
        environment:
          name: $(environment_name)
          token: $(environment_token)
    - apiVersion: training.eduk8s.io/v1alpha1
      kind: WorkshopRequest
      metadata:
        name: user2
      spec:
        environment:
          name: $(environment_name)
          token: $(environment_token)
```

Using this method, the workshop environment will<!-- Avoid |will|: present tense is preferred. --> be automatically<!-- Avoid if not definitely useful to mention. --> populated with workshop instances. You will<!-- Avoid |will|: present tense is preferred. --> need to<!-- |must| is preferred. --> query the workshop requests from the workshop namespace to determine<!-- |determine| has two meanings. Consider if the univocal |discover| or |verify| would be better. --> the URLs for accessing each, and the password if you didn't set one and a random password was assigned.

If you needed more control over how the workshop instances were created using this method, you could<!-- |can| or |might| whenever possible is preferred. When providing examples, use simple present tense verbs instead of postulating what someone or something could or would do. --> use the ``WorkshopSession`` custom resource instead.
