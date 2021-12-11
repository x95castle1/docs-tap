# Training portal resource

The ``TrainingPortal`` custom resource triggers the deployment of a set of workshop environments and a set number of workshop instances.

The raw custom resource definition <!-- Shorten to |CRD| after first use. -->for the ``TrainingPortal`` custom resource can be<!-- Consider switching to active voice. --> viewed at:

- [https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/training-portal.yaml](https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/training-portal.yaml) <!-- Type |in GitHub| somewhere in the cross-reference sentence. -->

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Specifying the workshop definitions

Running multiple workshop instances to perform training to a group of people can be<!-- Consider switching to active voice. --> done by following the step wise process of creating the workshop environment, and then creating each workshop instance. The ``TrainingPortal`` workshop resource bundles that up as one step.

Before creating the training environment you still need to<!-- |must| is preferred. --> load<!-- Do not use |load| as a synonym for |run| or |set up|. Acceptable for calling installed programs or data. --> the workshop definitions as a separate step.

To specify<!-- Use as a general term to instruct users when they have several selections to make. Do not use in procedures when a more specific term, such as enter, select, or click, is appropriate. --> the names of the workshops to be used for the training, list them under the ``workshops`` field of the training portal specification. Each entry needs to<!-- |must| is preferred. --> define a ``name`` property, matching the name of the ``Workshop`` resource which was created.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: sample-workshops
spec:
  portal:
    sessions:
      maximum: 8
  workshops:
  - name: lab-asciidoc-sample
  - name: lab-markdown-sample
```

When the training portal is created, it will<!-- Avoid |will|: present tense is preferred. --> setup the underlying workshop environments, create any workshop instances required to be created initially for each workshop, and deploy a web portal for attendees of the training to access their workshop instances.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Limiting the number of sessions

When defining the training portal, you can set a limit on the workshop sessions that can be<!-- Consider switching to active voice. --> run concurrently. This is done using the ``portal.sessions.maximum`` property.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: sample-workshops
spec:
  portal:
    sessions:
      maximum: 8
  workshops:
  - name: lab-asciidoc-sample
  - name: lab-markdown-sample
```

When this is specified, the maximum capacity of each workshop will<!-- Avoid |will|: present tense is preferred. --> be set to the same maximum value for the portal as a whole. This means that any one workshop can have as many sessions as specified by the maximum, but to achieve that only instances of that workshops could<!-- |can| or |might| whenever possible is preferred. When providing examples, use simple present tense verbs instead of postulating what someone or something could or would do. --> have been<!-- Consider replacing with |were| or shifting to present tense. --> created. In other words the maximum applies to the total number of workshop instances created across all workshops.

Note<!-- Format the note properly. --> that<!-- Notes must be in Note boxes and start with |Note: |. --> if you do not set ``portal.sessions.maximum``, you must set the capacity for each individual workshop as detailed below<!-- IX Standards forbids referring to text |below|. Use |following| or |later| or, better, just an anchor. -->. In only setting the capacities of each workshop and not an overall maximum for sessions, you can't share the overall capacity of the training portal across multiple workshops.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Capacity of individual workshops

When you have more than one workshop, you may<!-- |can| usually works better. Use |might| to convey possibility. --> want to limit how many instances of each workshop you can have so that they cannot grow<!-- |extend| or |expand| is preferred when referring to disk size. --> to the maximum number of sessions for the whole training portal, but a lessor maximum. This means you can stop one specific workshop taking over all the capacity of the whole training portal. To do this set the ``capacity`` field under the entry for the workshop.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: sample-workshops
spec:
  portal:
    sessions:
      maximum: 8
  workshops:
  - name: lab-asciidoc-sample
    capacity: 4
  - name: lab-markdown-sample
    capacity: 6
```

The value of ``capacity`` caps the number of workshop sessions for the specific workshop at that value. It should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> always be less than or equal to the maximum number of workshops sessions as the latter always sets the absolute cap.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Set reserved workshop instances

By default, one instance of each of the listed workshops will<!-- Avoid |will|: present tense is preferred. --> be created up front so that when the initial user requests that workshop, it is available for use immediately.

When such a reserved instance is allocated to a user, provided that the workshop capacity hasn't been reached a new instance of the workshop will<!-- Avoid |will|: present tense is preferred. --> be created as a reserve ready for the next user. When a user ends a workshop, if the workshop had been<!-- Consider replacing with |was| or shifting to present tense. --> at capacity, when the instance is deleted, then a new reserve will<!-- Avoid |will|: present tense is preferred. --> be created. The total of allocated and reserved sessions for a workshop cannot therefore exceed the capacity for that workshop.

If you want to<!-- Maybe replace with just |To|. --> override for a specific workshop how many reserved instances are kept in standby ready for users, you can set the ``reserved`` setting against the workshop.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: sample-workshops
spec:
  portal:
    sessions:
      maximum: 8
  workshops:
  - name: lab-asciidoc-sample
    capacity: 4
    reserved: 2
  - name: lab-markdown-sample
    capacity: 6
    reserved: 4
```

The value of ``reserved`` can be<!-- Consider switching to active voice. --> set to 0 if you do not ever want any reserved instances for a workshop and you instead only want instances of that workshop created on demand when required for a user. Only creating instances of a workshop on demand can result in a user needing to wait longer to access their workshop session.

Note<!-- Format the note properly. --> that<!-- Notes must be in Note boxes and start with |Note: |. --> in this instance where workshop instances are always created on demand, but also in other cases where reserved instances are tying up capacity which could<!-- |can| or |might| whenever possible is preferred. When providing examples, use simple present tense verbs instead of postulating what someone or something could or would do. --> be used for a new session of another workshop, the oldest reserved instance will<!-- Avoid |will|: present tense is preferred. --> be terminated to allow a new session of the desired<!-- Remove. --> workshop to be created instead. This will<!-- Avoid |will|: present tense is preferred. --> occur so long as any caps for specific workshops are being satisfied.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Override initial number of sessions

The initial number of workshop instances created for each workshop will<!-- Avoid |will|: present tense is preferred. --> be what is specified by<!-- Active voice is preferred. --> ``reserved``, or 1 if the setting wasn't provided.

In the case where ``reserved`` is set in order to keep workshop instances on standby, you can indicate that initially you want more than the reserved number of instances created. This is useful where you are running a workshop for a set period of time. You might create up front instances of the workshop corresponding to 75% of the expected number of attendees, but with a smaller reserve number. With this configuration, new reserve instances would only start to be created when getting close to the 75% and all of the extra instances created up front have been allocated to users. This way you can ensure you have enough instances ready for when most people show up, but then can create others if necessary as people trickle in later.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: kubernetes-fundamentals
spec:
  portal:
    sessions:
      maximum: 100
  workshops:
  - name: lab-kubernetes-fundamentals
    initial: 75
    reserved: 5
```

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Setting defaults for all workshops

If you have a list of workshops and they all need to<!-- |must| is preferred. --> be set with the same values for ``capacity``, ``reserved`` and ``initial``<!-- Insert the Oxford comma if it is missing here. -->, rather than add the settings to each, you can set defaults to apply to each under the ``portal`` section instead.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: sample-workshops
spec:
  portal:
    sessions:
      maximum: 10
    capacity: 6
    reserved: 2
    initial: 4
  workshops:
  - name: lab-asciidoc-sample
  - name: lab-markdown-sample
```

Note<!-- Format the note properly. --> that<!-- Notes must be in Note boxes and start with |Note: |. --> the location of these defaults in the training portal configuration will<!-- Avoid |will|: present tense is preferred. --> most likely change in a future<!-- Only document what exists. There are legal ramifications to making promises. --> version.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Setting caps on individual users

By default a single user can run more than one workshop at a time. You can though cap this if you want to<!-- Maybe replace with just |to|. --> ensure that they can only run one at a time. This avoids the problem of a user wasting resources by starting more than one at the same time, but only proceeding with one, without shutting down the other first.

The setting to apply a limit on how many concurrent workshop sessions a user can start is ``portal.sessions.registered``.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: sample-workshops
spec:
  portal:
    sessions:
      maximum: 8
      registered: 1
  workshops:
  - name: lab-asciidoc-sample
    capacity: 4
    reserved: 2
  - name: lab-markdown-sample
    capacity: 6
    reserved: 4
```

This limit will<!-- Avoid |will|: present tense is preferred. --> also apply to anonymous users when anonymous access is enabled through<!-- Do not use as a synonym for |finished| or |done|. Do not use when you mean |by using|. --> the training portal web interface, or if sessions are being created via<!-- |through|, |using| and |by means of| are preferred. --> the REST API. If you want to<!-- Maybe replace with just |To|. --> set a distinct limit on anonymous users, you can set ``portal.sessions.anonymous`` instead.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: sample-workshops
spec:
  portal:
    sessions:
      maximum: 8
      anonymous: 1
  workshops:
  - name: lab-asciidoc-sample
    capacity: 4
    reserved: 2
  - name: lab-markdown-sample
    capacity: 6
    reserved: 4
```

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Expiring of workshop sessions

Once<!-- Only use |once| when you mean |one time|, not when you mean |after|. --> you reach the maximum capacity, no more workshops sessions can be<!-- Consider switching to active voice. --> created. Once<!-- Only use |once| when you mean |one time|, not when you mean |after|. --> a workshop session has been<!-- Consider changing to |is| or |has| or rewrite for active voice. --> allocated to a user, they cannot be re-assigned to another user.

If running a supervised workshop you therefore need to<!-- |must| is preferred. --> ensure that you set the capacity higher than the expected<!-- Consider replacing with |in most cases| to sound more confident. --> number in case you have extra users you didn't expect which you need to<!-- Consider replacing with just |To|. --><!-- |must| is preferred. --> accomodate. You can use the setting for the reserved number of instances so that although a higher capacity is set, workshop sessions are only created as required, rather than all being created up front.

For supervised workshops when the training is over you would<!-- Re-phrase for present tense if possible. --> delete the whole training environment and all workshop sessions would<!-- Re-phrase for present tense if possible. --> then be deleted.

If you need to<!-- Consider replacing with just |To|. --><!-- |must| is preferred. --> host a training portal over an extended period and you don't know when users will<!-- Avoid |will|: present tense is preferred. --> want to do a workshop, you can setup workshop sessions to expire after a set time. When expired the workshop session will<!-- Avoid |will|: present tense is preferred. --> be deleted, and a new workshop session can be<!-- Consider switching to active voice. --> created in its place.

The maximum capacity is therefore the maximum at any one point in time, with the number being able to<!-- |can| is preferred. --> grow<!-- |extend| or |expand| is preferred when referring to disk size. --> and shrink over time. In this way, over an extended time you could<!-- |can| or |might| whenever possible is preferred. When providing examples, use simple present tense verbs instead of postulating what someone or something could or would do. --> handle many more sessions that what the maximum capacity is set to. The maximum capacity is in this case used to ensure you don't try and allocate more workshop sessions than you have resources to handle at any one time.

Setting a maximum time allowed for a workshop session can be<!-- Consider switching to active voice. --> done using the ``expires`` setting.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  workshops:
  - name: lab-markdown-sample
    capacity: 8
    reserved: 1
    expires: 60m
```

The value needs to<!-- |must| is preferred. --> be an integer, followed by a suffix of 's', 'm' or 'h', corresponding to seconds, minutes or hours<!-- Insert the Oxford comma if it is missing here. -->.

The time period is calculated from when the workshop session is allocated to a user. When the time period is up, the workshop session will<!-- Avoid |will|: present tense is preferred. --> be automatically<!-- Avoid if not definitely useful to mention. --> deleted.

When an expiration period is specified, when a user finishes a workshop, or restarts the workshop, it will<!-- Avoid |will|: present tense is preferred. --> also be deleted.

To cope with users who grab a workshop session, but then leave and don't actually<!-- Delete unless referring to a situation that is actual instead of virtual. Most uses are extraneous. --> use it, you can also set a time period for when a workshop session with no activity is deemed as being orphaned and so deleted. This is done using the ``orphaned`` setting.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  workshops:
  - name: lab-markdown-sample
    capacity: 8
    reserved: 1
    expires: 60m
    orphaned: 5m
```

For supervised workshops where the whole event only lasts a certain amount of time, you should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> avoid this setting so that a users session is not deleted when they take breaks and their computer goes to sleep.

The ``expires`` and ``orphaned`` settings can also be set against ``portal`` instead, if you want to<!-- Maybe replace with just |to|. --> have them apply to all workshops.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Updates to workshop environments

The list of workshops for an existing training portal can be<!-- Consider switching to active voice. --> changed by<!-- Active voice is preferred. --> modifying the training portal definition applied to the Kubernetes cluster.

If a workshop is removed from the list of workshops, the workshop environment will<!-- Avoid |will|: present tense is preferred. --> be marked as stopping, and will<!-- Avoid |will|: present tense is preferred. --> be deleted when all active workshop sessions have completed.

If a workshop is added to the list of workshops, a new workshop environment for it will<!-- Avoid |will|: present tense is preferred. --> be created.

Changes to settings such as the maximum number of sessions for the training portal, or capacity settings for individual workshops will<!-- Avoid |will|: present tense is preferred. --> be applied to the existing workshop environments.

By default a workshop environment will<!-- Avoid |will|: present tense is preferred. --> be left unchanged if the corresponding workshop definition is changed. In the default configuration you would<!-- Re-phrase for present tense if possible. --> therefore need to<!-- |must| is preferred. --> explicitly delete the workshop from the list of workshops managed by the training portal, and then add it back again, if the workshop definition had changed.

If you would<!-- Re-phrase for present tense if possible. --> prefer for workshop environments to automatically<!-- Avoid if not definitely useful to mention. --> be replaced when the workshop definition changes, you can enable it by setting the ``portal.updates.workshop`` setting.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  portal:
    sessions:
      maximum: 8
    updates:
      workshop: true
  workshops:
  - name: lab-markdown-sample
    reserved: 1
    expires: 60m
    orphaned: 5m
```

When using this option you should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> use the ``portal.sessions.maximum`` setting to cap the number of workshop sessions that can be<!-- Consider switching to active voice. --> run for the training portal as a whole. This is because when replacing the workshop environment the old workshop environment will<!-- Avoid |will|: present tense is preferred. --> be retained so long as there is still an active workshop session being used. If the cap isn't set, then the new workshop environment will<!-- Avoid |will|: present tense is preferred. --> be still able to<!-- |can| is preferred. --> grow<!-- |extend| or |expand| is preferred when referring to disk size. --> to its specific capacity and will<!-- Avoid |will|: present tense is preferred. --> not be limited based on how many workshop sessions are running against old instances of the workshop environment.

Overall it is recommended<!-- Specify the party that recommends (VMware, Cloud Foundry, etc). --> that the option to update workshop environments when workshop definitions change only be used in development environments where working on workshop content, at least until you are quite familiar with the mechanism for how the training portal replaces existing workshop environments, and the resource implications of when you have old and new instances of a workshop environment running at the same time.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Overriding the ingress domain

In order to be able to<!-- |can| is preferred. --> access a workshop instance using a public URL, you will<!-- Avoid |will|: present tense is preferred. --> need to<!-- |must| is preferred. --> specify<!-- Use as a general term to instruct users when they have several selections to make. Do not use in procedures when a more specific term, such as enter, select, or click, is appropriate. --> an ingress domain. If an ingress domain isn't specified, the default ingress domain that the Learning Center operator has been<!-- Consider changing to |is| or |has| or rewrite for active voice. --> configured with will<!-- Avoid |will|: present tense is preferred. --> be used.

When setting a custom domain, DNS must have been<!-- Consider replacing with |were| or shifting to present tense. --> configured with a wildcard domain to forward all requests for sub domains of the custom domain, to the ingress router of the Kubernetes cluster.

To provide the ingress domain, you can set the ``portal.ingress.domain`` field.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  portal:
    ingress:
      domain: training.eduk8s.io
  workshops:
  - name: lab-markdown-sample
    capacity: 3
    reserved: 1
```

If overriding the domain, by default, the workshop session will<!-- Avoid |will|: present tense is preferred. --> be exposed using a HTTP connection. If you require a secure HTTPS connection, you will<!-- Avoid |will|: present tense is preferred. --> need to<!-- |must| is preferred. --> have access to a wildcard SSL certificate for the domain. A secret of type ``tls`` should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> be created for the certificate in the ``educates`` namespace or the namespace where the Learning Center operator is deployed. The name of that secret should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> then be set in the ``portal.ingress.secret`` field.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  portal:
    ingress:
      domain: training.eduk8s.io
      secret: training.eduk8s.io-tls
  workshops:
  - name: lab-markdown-sample
    capacity: 3
    reserved: 1
```

If HTTPS connections are being terminated using an external load balancer and not by specificying a secret for ingresses managed by the Kubernetes ingress controller, with traffic then routed into the Kubernetes cluster as HTTP connections, you can override the ingress protocol without specifying an ingress secret by setting the ``portal.ingress.protocol`` field.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  portal:
    ingress:
      domain: training.eduk8s.io
      protocol: https
  workshops:
  - name: lab-markdown-sample
    capacity: 3
    reserved: 1
```

If you need to<!-- Consider replacing with just |To|. --><!-- |must| is preferred. --> override or set the ingress class, which dictates which ingress router is used when more than one option is available, you can add ``portal.ingress.class``.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  portal:
    ingress:
      domain: training.eduk8s.io
      secret: training.eduk8s.io-tls
      class: nginx
  workshops:
  - name: lab-markdown-sample
    capacity: 3
    reserved: 1
```

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Overriding the portal hostname<!-- |host name| is preferred. -->

The default hostname<!-- |host name| is preferred. --> given to the training portal will<!-- Avoid |will|: present tense is preferred. --> be the name of the resource with ``-ui`` suffix, followed by the domain specified by the resource, or the default inherited from the configuration of the Learning Center operator.

If you want to<!-- Maybe replace with just |To|. --> override the generated hostname<!-- |host name| is preferred. -->, you can set ``portal.ingress.hostname<!-- |host name| is preferred. -->``.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  portal:
    ingress:
      hostname: labs
      domain: training.eduk8s.io
      secret: training.eduk8s.io-tls
  workshops:
  - name: lab-markdown-sample
    capacity: 3
    reserved: 1
```

This will<!-- Avoid |will|: present tense is preferred. --> result in the hostname<!-- |host name| is preferred. --> being ``labs.training.eduk8s.io``, rather than the default generated name for this example of ``lab-markdown-sample-ui.training.eduk8s.io``.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Setting extra environment variables

If you want to<!-- Maybe replace with just |To|. --> override any environment variables for workshop instances created for a specific work, you can provide the environment variables in the ``env<!-- |environment| is preferred -->`` field of that workshop.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  workshops:
  - name: lab-markdown-sample
    capacity: 3
    reserved: 1
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

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Overriding portal credentials

When a training portal is deployed, the username<!-- |user name| is preferred. --> for the admin and robot accounts will<!-- Avoid |will|: present tense is preferred. --> use the defaults of ``eduk8s`` and ``robot@eduk8s``. The passwords for each account will<!-- Avoid |will|: present tense is preferred. --> be randomly set.

For the robot account, the OAuth application client details used with the REST API will<!-- Avoid |will|: present tense is preferred. --> also be randomly generated.

You can see what the credentials and client details are by running ``kubectl describe`` against the training portal resource. This will yield output which includes:

```
Status:
  eduk8s:
    Clients:
      Robot:
        Id:      ACZpcaLIT3qr725YWmXu8et9REl4HBg1
        Secret:  t5IfXbGZQThAKR43apoc9usOFVDv2BLE
    Credentials:
      Admin:
        Password:  0kGmMlYw46BZT2vCntyrRuFf1gQq5ohi
        Username:  eduk8s
      Robot:
        Password:  QrnY67ME9yGasNhq2OTbgWA4RzipUvo5
        Username:  robot@eduk8s
``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->

If you wish<!-- |want| is preferred. --> to override any of these values in order to<!-- |to| is preferred. --> be able to<!-- |can| is preferred. --> set them to a pre-determined<!-- |determine| has two meanings. Consider if the univocal |discover| or |verify| would be better. --> value, you can add ``credentials`` and ``clients`` sections to the training portal specification.

To overload the credentials for the admin and robot accounts use:

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  portal:
    credentials:
      admin:
        username: admin-user
        password: top-secret
      robot:
        username: robot-user
        password: top-secret
  workshops:
  - name: lab-markdown-sample
    capacity: 3
    reserved: 1
```

To override the application client details for OAuth access by the robot account use:

``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  portal:
    clients:
      robot:
        id: application-id
        secret: top-secret
  workshops:
  - name: lab-markdown-sample
    capacity: 3
    reserved: 1
```

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Controlling registration type

By default the training portal web interface will present a registration page for users to create an account, before they can select a workshop to do. If you only want to allow the administrator to login, you can disable the registration page. This would be done if using the REST API to create and allocate workshop sessions from a separate application.

``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  portal:
    registration:
      type: one-step
      enabled: false
  workshops:
  - name: lab-markdown-sample
    capacity: 3
    reserved: 1
```

If rather than requiring users to register, you want to allow anonymous access, you can switch the registration type to anonymous.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  portal:
    registration:
      type: anonymous
  workshops:
  - name: lab-markdown-sample
    capacity: 3
    reserved: 1
```

In anonymous mode, when users visit the home page for the training portal an account will<!-- Avoid |will|: present tense is preferred. --> be automatically<!-- Avoid if not definitely useful to mention. --> created and they will<!-- Avoid |will|: present tense is preferred. --> be logged in.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Specifying an event access code

Where deploying the training portal with anonymous access, or open registration, anyone would<!-- Re-phrase for present tense if possible. --> be able to<!-- |can| is preferred. --> access workshops who knows the URL. If you want to<!-- Maybe replace with just |To|. --> at least prevent access to those who know a common event access code or password, you can set ``portal.password``.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  portal:
    password: workshops-2020-07-01
  workshops:
  - name: lab-markdown-sample
    capacity: 3
    reserved: 1
```

When the training portal URL is accessed, users will<!-- Avoid |will|: present tense is preferred. --> be asked to enter the event access code before they are redirected to the list of workshops (when anonymous access is enabled), or to the login page.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Making list of workshops public

By default the index page providing the catalog of available workshop images is only available once a user has logged in, be that through<!-- Do not use as a synonym for |finished| or |done|. Do not use when you mean |by using|. --> a registered account or as an anonymous user.

If you want to<!-- Maybe replace with just |To|. --> make the catalog of available workshops public, so they can be<!-- Consider switching to active voice. --> viewed before logging in, you can set the ``portal.catalog.visibility`` property.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  portal:
    catalog:
      visibility: public
  workshops:
  - name: lab-markdown-sample
    capacity: 3
    reserved: 1
```

By default the catalog has visibility set to ``private``. Use ``public`` to expose it.

Note<!-- Format the note properly. --> that<!-- Notes must be in Note boxes and start with |Note: |. --> this will<!-- Avoid |will|: present tense is preferred. --> also make it possible to access the list of available workshops from the catalog, via<!-- |through|, |using| and |by means of| are preferred. --> the REST API, without authenticating against the REST API.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Using an external list of workshops

If you are using the training portal with registration disabled<!-- |deactivated| is preferred. --> and are using the REST API from a separate web site to control creation of sessions, you can specify<!-- Use as a general term to instruct users when they have several selections to make. Do not use in procedures when a more specific term, such as enter, select, or click, is appropriate. --> an alternate URL for providing the list of workshops.

This helps in the situation where for a session created by the REST API, cookies were deleted, or a session URL was shared with a different user, meaning the value for the ``index_url`` supplied with the REST API request is lost.

The property to set the URL for the external site is ``portal.index``.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  portal:
    index: https://www.example.com/
    registration:
      type: one-step
      enabled: false
  workshops:
  - name: lab-markdown-sample
    capacity: 3
    reserved: 1
```

If the property is supplied, passing the ``index_url`` when creating a workshop session using the REST API is optional, and the value of this property will<!-- Avoid |will|: present tense is preferred. --> be used. You may<!-- |can| usually works better. Use |might| to convey possibility. --> still want to supply ``index_url`` when using the REST API however if you want a user to be redirected back to a sub category for workshops on the site providing the list of workshops. The URL provided here in the training portal definition would<!-- Re-phrase for present tense if possible. --> then act only as a fallback when the redirect URL becomes unavailable, and would<!-- Re-phrase for present tense if possible. --> direct back to the top level page for the external list of workshops.

Note<!-- Format the note properly. --> that<!-- Notes must be in Note boxes and start with |Note: |. --> if a user has logged into the training portal as the admin user, they will<!-- Avoid |will|: present tense is preferred. --> not be redirected to the external site and will<!-- Avoid |will|: present tense is preferred. --> still see the training portals own list of workshops.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Overriding portal title and logo

The web interface for the training portal will<!-- Avoid |will|: present tense is preferred. --> display a generic Learning Center logo by default, along with<!-- Do not use as a conjunction. Whenever possible, use |with| or |and| instead. --> a page title of "Workshops". If you want to<!-- Maybe replace with just |To|. --> override these, you can set ``portal.title`` and ``portal.logo``.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  portal:
    title: Workshops
    logo: data:image/png;base64,....
  workshops:
  - name: lab-markdown-sample
    capacity: 3
    reserved: 1
```

The ``logo`` field should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> be a graphical image provided in embedded data URI format which displays the branding you desire<!-- |want| is preferred. -->. The image is displayed with a fixed height of "40px". The field can also be a URL for an image stored on a remote web server.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Allowing the portal in an iframe

By default if you try and display the web interface for the training portal in an iframe of another web site, it will<!-- Avoid |will|: present tense is preferred. --> be prohibited due to content security policies applying to the training portal web site.

If you want to<!-- Maybe replace with just |To|. --> enable the ability to<!-- |can| is shorter. Avoid nounification of verbs where possible. --> iframe the full training portal web interface, or even a specific workshop session created using the REST API, you need to<!-- Consider replacing with just |To|. --><!-- |must| is preferred. --> provide the hostname<!-- |host name| is preferred. --> of the site which will<!-- Avoid |will|: present tense is preferred. --> embed it. This can be<!-- Consider switching to active voice. --> done using the ``portal.theme.frame.ancestors`` property.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  portal:
    theme:
      frame:
        ancestors:
        - https://www.example.com
  workshops:
  - name: lab-markdown-sample
    capacity: 3
    reserved: 1
```

The property is a list of hosts, not a single value. If needing to use a URL for the training portal in an iframe of a page, which is in turn embedded in another iframe of a page on a different site again, the hostnames of all sites need to be listed.

Note that the sites which embed the iframes must be secure and use HTTPS, they cannot use plain HTTP. This is because browser policies prohibit promoting of cookies to an insecure site when embedding using an iframe. If cookies aren't able to be stored, a user would not be able to authenticate against the workshop session.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Collecting analytics on workshops

To collect analytics data on usage of workshops, you can supply a webhook URL. When this is supplied, events will be posted to the webhook URL for events such as workshops being started, pages of a workshop being viewed, expiration of a workshop, completion of a workshop, or termination of a workshop.

``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  analytics:
    webhook:
      url: https://metrics.eduk8s.io/?client=name&token=password
  workshops:
  - name: lab-markdown-sample
    capacity: 3
    reserved: 1
```

At present there is no metrics collection service compatible with the portal webhook reporting mechanism, so you will<!-- Avoid |will|: present tense is preferred. --> need to<!-- |must| is preferred. --> create a custom service or integrate it with any existing web front end for the portal REST API service.

If the collection service needs to<!-- |must| is preferred. --> be provided with a client ID or access token, that must be able to<!-- |can| is preferred. --> be accepted using query string parameters which would<!-- Re-phrase for present tense if possible. --> be set in the webhook URL.

The details of the event are subsequently included as HTTP POST data using the ``application/json<!-- |JSON| is preferred. -->`` content type.

```
{
  "portal": {
    "name": "lab-markdown-sample",
    "uid": "91dfa283-fb60-403b-8e50-fb30943ae87d",
    "generation": 3,
    "url": "https://lab-markdown-sample-ui.training.eduk8s.io"
  },
  "event": {
    "name": "Session/Started",
    "timestamp": "2021-03-18T02:50:40.861392+00:00",
    "user": "c66db34e-3158-442b-91b7-25391042f037",
    "session": "lab-markdown-sample-w01-s001",
    "environment": "lab-markdown-sample-w01",
    "workshop": "lab-markdown-sample",
    "data": {}
  }
}
```

Where an event has associated data, it is included in<!-- Consider deleting |included|. --> the ``data`` dictionary.

```
{
  "portal": {
    "name": "lab-markdown-sample",
    "uid": "91dfa283-fb60-403b-8e50-fb30943ae87d",
    "generation": 3,
    "url": "https://lab-markdown-sample-ui.training.eduk8s.io"
  },
  "event": {
    "name": "Workshop/View",
    "timestamp": "2021-03-18T02:50:44.590918+00:00",
    "user": "c66db34e-3158-442b-91b7-25391042f037",
    "session": "lab-markdown-sample-w01-s001",
    "environment": "lab-markdown-sample-w01",
    "workshop": "lab-markdown-sample",
    "data": {
      "current": "workshop-overview",
      "next": "setup-environment",
      "step": 1,
      "total": 4
    }
  }
}
```

The ``user`` field will be the same portal user identity that is returned by the REST API when creating workshop sessions.

Note that the event stream only produces events for things as they happen. If you need a snapshot of all current workshop sessions, you should use the REST API to request the catalog of available workshop environments, enabling the inclusion of current workshop sessions.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Tracking using Google Analytics

If you want to record analytics data on usage of workshops using Google Analytics, you can enable tracking by supplying a tracking ID for Google Analytics.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  analytics:
    google:
      trackingId: UA-XXXXXXX-1
  workshops:
  - name: lab-markdown-sample
    capacity: 3
    reserved: 1
``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->

Custom dimensions are used in Google Analytics to record details about the workshop a user is doing, and through which training portal and cluster it was accessed. You can therefore use the same Google Analytics tracking ID for multiple training portal instances running on different Kubernetes clusters if desired.

To support use of custom dimensions in Google Analytics you must configure the Google Analytics property with the following custom dimensions. They must be added in the order shown as Google Analytics doesn't allow you to specify the index position for a custom dimension and will allocate them for you. You can't already have custom dimensions defined for the property, as the new custom dimensions must start at index of 1.

```
| Custom Dimension Name | Index |
| <!-- Insert |_n/a_| within purposefully empty cells. -->-----------------------|-------|
| <!-- Insert |_n/a_| within purposefully empty cells. --> workshop_name         | 1     |
| <!-- Insert |_n/a_| within purposefully empty cells. --> session_namespace     | 2     |
| <!-- Insert |_n/a_| within purposefully empty cells. --> workshop_namespace    | 3     |
| <!-- Insert |_n/a_| within purposefully empty cells. --> training_portal       | 4     |
| <!-- Insert |_n/a_| within purposefully empty cells. --> ingress_domain        | 5     |
| <!-- Insert |_n/a_| within purposefully empty cells. --> ingress_protocol      | 6     |
```

In addition to custom dimensions against page accesses, events are also generated. These include:

* W<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->orkshop/Start
* W<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->orkshop/Finish
* W<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->orkshop/Expired

If a Google Analytics tracking ID is provided with the ``TrainingPortal`` resource definition, it will<!-- Avoid |will|: present tense is preferred. --> take precedence over one set by the ``SystemProfile`` resource definition.

Note<!-- Format the note properly. --> that<!-- Notes must be in Note boxes and start with |Note: |. --> Google Analytics is not a reliable way to collect data. This is because individuals or corporate firewalls can block the reporting of Google Analytics data. For more precise statistics, you should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> use the webhook URL for collecting analytics with a custom data collection platform.
