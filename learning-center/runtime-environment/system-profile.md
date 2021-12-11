# System profile resource

The ``SystemProfile`` custom resource is used to<!-- Redundant? --> configure the Learning Center operator. The default system profile can be<!-- Consider switching to active voice. --> used to set defaults for ingress and image pull secrets, with specific deployments being able to<!-- |can| is preferred. --> select an alternate profile if required.

The raw custom resource definition <!-- Shorten to |CRD| after first use. -->for the ``SystemProfile`` custom resource can be<!-- Consider switching to active voice. --> viewed at:

- [https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/system-profile.yaml](https://github.com/eduk8s/eduk8s/blob/develop/resources/crds-v1/system-profile.yaml) <!-- Type |in GitHub| somewhere in the cross-reference sentence. -->

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Operator default system profile

The Learning Center operator will<!-- Avoid |will|: present tense is preferred. --> by default use an instance of the ``SystemProfile`` custom resource, if it exists, named ``default-system-profile``. You can override the name of the resource used by the Learning Center operator as the default, by setting the ``SYSTEM_PROFILE`` environment variable on the deployment for the Learning Center operator.

```
kubectl set env deployment/eduk8s-operator -e SYSTEM_PROFILE=default-system-profile -n eduk8s
``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->

Any changes to an instance of the ``SystemProfile`` custom will<!-- Avoid |will|: present tense is preferred. --> be automatically<!-- Avoid if not definitely useful to mention. --> detected and used by the Learning Center operator and there is no need to<!-- |must| is preferred. --> redeploy the operator when changes are made.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Defining configuration for ingress

The ``SystemProfile`` custom resource replaces the use of environment variables to configure details such as the ingress domain, secret and class<!-- Insert the Oxford comma if it is missing here. -->.

Instead of setting ``INGRESS_DOMAIN``, ``INGRESS_SECRET`` and ``INGRESS_CLASS`` environment variables, create an instance of the ``SystemProfile`` custom resource named ``default-system-profile``.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  ingress:
    domain: training.eduk8s.io
    secret: training.eduks8.io-tls
    class: nginx
```

If HTTPS connections are being terminated using an external load balancer and not by specificying a secret for ingresses managed by the Kubernetes ingress controller, with traffic then routed into the Kubernetes cluster as HTTP connections, you can override the ingress protocol without specifying an ingress secret.

``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  ingress:
    domain: training.eduk8s.io
    protocol: https
    class: nginx
```

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Defining image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. --> pull secrets

If needing to work with custom workshop images stored in a private image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. -->, the system profile can define a list of image pull secrets that should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> be added to the service accounts used to deploy and run the workshop images. The ``environment.secrets.pull`` property should be set to the list of secret names.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  environment:
    secrets:
      pull:
      - private-image-registry-pull
```

The secrets containing the image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. --> credentials must exist within the ``educates`` namespace or the namespace where the Learning Center operator is deployed. The secret resources must be of type ``kubernetes<!-- |Kubernetes| is preferred. -->.io/dockerconfigjson``.

Note<!-- Format the note properly. --> that<!-- Notes must be in Note boxes and start with |Note: |. --> this doesn't result in any secrets being added to the namespace created for each workshop session. The secrets are only added to the workshop namespace and are not visible to a user.

For container images used as part of Learning Center itself, such as the container image for the training portal web interface, and the builtin base workshop images, if you have copied these from the public image registries and stored them in a local private registry, instead of the above<!-- IX Standards forbids referring to text |above|. Use |earlier| or, better, just an anchor. --> setting you should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> use the ``registry`` section as follows.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  registry:
    secret: eduk8s-image-registry-pull
```

The ``registry.secret`` is the name of the secret containing the image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. --> credentials. This must be present in the ``educates`` namespace or the namespace where the Learning Center operator is deployed.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Defining storage class for volumes

Deployments of the training portal web interface and the workshop sessions make use of persistent volumes. By default the persistent volume claims will<!-- Avoid |will|: present tense is preferred. --> not specify<!-- Use as a general term to instruct users when they have several selections to make. Do not use in procedures when a more specific term, such as enter, select, or click, is appropriate. --> a storage class for the volume and instead rely on the Kubernetes cluster specifying a default storage class that works. If the Kubernetes cluster doesn't define a suitable default storage class, or you need to<!-- Consider replacing with just |To|. --><!-- |must| is preferred. --> override it, you can set the ``storage.class`` property.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  storage:
    class: default
```

Note<!-- Format the note properly. --> that<!-- Notes must be in Note boxes and start with |Note: |. --> this only applies to persistent volume claims setup by the Learning Center operator. If the steps in a workshop which a user executes include making persistent volume claims, these will<!-- Avoid |will|: present tense is preferred. --> not be automatically<!-- Avoid if not definitely useful to mention. --> adjusted.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Defining storage group for volumes

Where persistent volumes are used by Learning Center for the training portal web interface and workshop environments, the application of pod<!-- |Pod| is capitalized per the K8s docs style. --> security policies by the cluster is relied on to ensure that the permissions of persistent volumes are set correctly such that they can be<!-- Consider switching to active voice. --> accessed by<!-- Active voice is preferred. --> containers mounting the persistent volume. For where the pod<!-- |Pod| is capitalized per the K8s docs style. --> security policy admission controller is not enabled, a fallback is instituted to enable access to volumes by enabling group access using the group ID of ``0``.

In situations where the only class of persistent storage available is NFS or similar, it may<!-- |can| usually works better. Use |might| to convey possibility. --> be necessary to override the group ID applied and set it to an alternate ID dictated by the file system storage provider. If this is required, you can set the ``storage.group`` property.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  storage:
    group: 1
```

Overriding the group ID to match the persistent storage relies on the group having write permission to the volume. If only the owner of the volume has permission this will<!-- Avoid |will|: present tense is preferred. --> not work.

In this case it is necessary to<!-- Consider deleting this or replacing it with the shorter |you must|. --> change the owner/group<!-- If this is a URL then you likely need to present it per xref rules: https://docs-wiki.cfapps.io/wiki/style/cross-ref-style.html --> and permissions of the persistent volume such that the owner matches the user ID a container runs as, or the group is set to a known ID which is added as a supplemental group for the container, and the persistent volume updated to be writable to this group. This needs to<!-- |must| is preferred. --> be done by an init container running in the pod<!-- |Pod| is capitalized per the K8s docs style. --> mounting the persistent volume.

To trigger this fixup of ownership and permissions, you can set the ``storage.user`` property.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  storage:
    user: 1
    group: 1
```

This will<!-- Avoid |will|: present tense is preferred. --> result in the init container being run as the root user, with the owner of the mount directory of the persistent volume being set to ``storage.user``, the group being set to ``storage.group``, and the directory being made group writable. The group will<!-- Avoid |will|: present tense is preferred. --> then be added as supplemental group to containers using the persistent volume so they can write to it, regardless of what user ID the container runs as. To that end, the value of ``storage.user`` doesn't matter, as long as<!-- |if| is preferred. --> it is set, but it may<!-- |can| usually works better. Use |might| to convey possibility. --> need<!-- Avoid anthropomorphizing: |require| might be better here. --> to be set to a specific user ID based on requirements of the storage provider.

Note<!-- Format the note properly. --> that<!-- Notes must be in Note boxes and start with |Note: |. --> both these variations on the settings only apply to the persistent volumes used by Learning Center itself. If a workshop asks users to create persistent volumes, those instructions or the resource definitions used may<!-- |can| usually works better. Use |might| to convey possibility. --> need to<!-- |must| is preferred. --> be modified in order to<!-- |to| is preferred. --> work where the storage class available requires access as a specific user or group ID. Further, the second method using the init container to fixup permissions will<!-- Avoid |will|: present tense is preferred. --> not work if pod<!-- |Pod| is capitalized per the K8s docs style. --> security policies are enforced, as the ability to<!-- |can| is shorter. Avoid nounification of verbs where possible. --> run a container as the root user would<!-- Re-phrase for present tense if possible. --> be blocked in that case due to the restricted PSP which is applied to workshop instances.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Restricting Network Access

Any processes run from the workshop container, and any applications deployed to the session namespaces associated with a workshop instance can contact any network IP addresses accessible from the cluster. If you need to<!-- Consider replacing with just |To|. --><!-- |must| is preferred. --> add restrictions on what IP addresses or IP<!-- Do not omit |address| from |IP address|. --> subnets can be<!-- Consider switching to active voice. --> accessed, you can set ``network.blockCIDRs``. This must be a CIDR block range corresponding to the subnet, or portion of a subnet you want to block. A Kubernetes ``NetworkPolicy`` will be used to enforce the restriction, so the Kubernetes cluster must use a network layer supporting network policies and the necessary Kubernetes controllers supporting network policies enabled when the cluster was installed.

If deploying to AWS, it is important to block access to the AWS endpoint for querying EC2 metadata as it can expose sensitive information that workshop users should not haves access to. The AWS endpoint in this case is a single IP address and the configuration would be:

```
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  network:
    blockCIDRs:
    - 169.254.169.254/32
``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Running docker<!-- |Docker| is preferred. --> daemon rootless

If ``docker<!-- |Docker| is preferred. -->`` is enabled for workshops, docker<!-- |Docker| is preferred. --> in docker<!-- |Docker| is preferred. --> is run using a side car container. Because of the current state of running docker<!-- |Docker| is preferred. --> in docker<!-- |Docker| is preferred. -->, and portability across Kubernetes environments, the ``docker<!-- |Docker| is preferred. -->`` daemon by default runs as ``root``. Because a privileged container is also being used, this represents a security risk and workshops requiring ``docker<!-- |Docker| is preferred. -->`` should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> only be run in disposable Kubernetes clusters, or for users who you trust.

The risks of running ``docker<!-- |Docker| is preferred. -->`` in the Kubernetes cluster can be<!-- Consider switching to active voice. --> partly mediated by running the ``docker<!-- |Docker| is preferred. -->`` daemon in rootless mode, however not all Kubernetes clusters may<!-- |can| usually works better. Use |might| to convey possibility. --> support this due to the Linux kernel configuration or other incompatibilities.

To enable rootless mode, you can set the ``dockerd.rootless`` property to ``true``.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  dockerd:
    rootless: true
```

Use of ``docker<!-- |Docker| is preferred. -->`` can be<!-- Consider switching to active voice. --> made even more secure by avoiding the use of a privileged container for the ``docker<!-- |Docker| is preferred. -->`` daemon. This requires specific configuration to be setup for nodes in the Kubernetes cluster. If such configuration has been<!-- Consider changing to |is| or |has| or rewrite for active voice. --> done, you can disable<!-- |deactivate| is preferred. --> the use of a privileged container by setting ``dockerd.privileged`` to ``false``.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  dockerd:
    rootless: true
    privileged: false
```

For further details on<!-- |details about| is preferred. --> the requirements for running rootless docker<!-- |Docker| is preferred. --> in docker<!-- |Docker| is preferred. -->, and using an non privileged container see:

- [https://docs.docker.com/engine/security/rootless/](https://docs.docker.com/engine/security/rootless/) <!-- The link name must be |Docker documentation|. -->

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Overriding network packet size

When support for building container images using ``docker<!-- |Docker| is preferred. -->`` is enabled for workshops, because of network layering that occurs when doing ``docker<!-- |Docker| is preferred. --> build`` or ``docker<!-- |Docker| is preferred. --> run``, it is necessary to<!-- Consider deleting this or replacing it with the shorter |you must|. --> adjust the network packet size (mtu) used for containers run from ``dockerd`` hosted inside of the workshop container.

The default mtu size for networks is 1500, but when containers are run in Kubernetes the size available to containers is often reduced. To deal with this possibility, the mtu size used when ``dockerd`` is run for a workshop is set as 1400 instead of 1500.

If you experience problems building or running images with the ``docker<!-- |Docker| is preferred. -->`` support, including errors or timeouts in pulling images, or when pulling software packages (PyPi, npm, etc) within a build,<!-- If a list, maybe reformat it as bullets. If this is a rambling sentence, break it up into smaller sentences. --> you may<!-- |can| usually works better. Use |might| to convey possibility. --> need<!-- Avoid anthropomorphizing: |require| might be better here. --> to override this value to an even lower value.

If this is required, you can set the ``dockerd.mtu`` property.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  dockerd:
    mtu: 1400
```

You can determine<!-- |determine| has two meanings. Consider if the univocal |discover| or |verify| would be better. --> what the size may<!-- |can| usually works better. Use |might| to convey possibility. --> need to<!-- |must| is preferred. --> be by accessing the ``docker<!-- |Docker| is preferred. -->`` container run with a workshop and run ``ifconfig eth0``. This will yield something similar to:

```
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:07
          inet addr:172.17.0.7  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1350  Metric:1
          RX packets:270018 errors:0 dropped:0 overruns:0 frame:0
          TX packets:283882 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:86363656 (82.3 MiB)  TX bytes:65183730 (62.1 MiB)
``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->

If the ``MTU`` size is less than 1400, then use the value given, or a smaller value, for the ``dockerd.mtu`` setting.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Image registry pull through<!-- Do not use as a synonym for |finished| or |done|. Do not use when you mean |by using|. --> cache

When running or building container images with ``docker<!-- |Docker| is preferred. -->``, if the container image is hosted on Docker Hub it will<!-- Avoid |will|: present tense is preferred. --> be pulled down direct from Docker Hub for each separate workshop session of that workshop.

Because the image is pulled from Docker Hub this will<!-- Avoid |will|: present tense is preferred. --> be slow for all users, especially for large images. With Docker Hub introducing limits on how many images can be<!-- Consider switching to active voice. --> pulled anonymously from an IP address within a set period, this also could<!-- |can| or |might| whenever possible is preferred. When providing examples, use simple present tense verbs instead of postulating what someone or something could or would do. --> result in the cap on image pulls being reached, preventing the workshop from being used until the period expires.

Docker Hub has a higher limit when pulling images as an authenticated user, but with the limit being applied to the user rather than by IP address. For authenticated users with a paid plan on Docker Hub, there is no limit.

To try and avoid the impact of the limit, the first thing you can do is enable an image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. --> mirror with image pull through<!-- Do not use as a synonym for |finished| or |done|. Do not use when you mean |by using|. -->. This is enabled globally and results in<!-- Consider replacing with |causes|. --> an instance of an image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. --> mirror being created in the workshop environment of workshops which enable ``docker<!-- |Docker| is preferred. -->`` support. This mirror will be used for all workshops sessions created against that workshop environment. When the first user attempts to pull an image, it will be pulled down from Docker Hub and cached in the mirror. Subsequent users will be served up from the image registry mirror, avoiding the need to pull the image from Docker Hub again. The subsequent users will also see a speed up in pulling the image because the mirror is deployed to the same cluster.

For enabling the use of an image registry mirror against Docker Hub, use:

```
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  dockerd:
    mirror:
      remote: https://registry-1.docker.io
```

For authenticated access to Docker Hub, create an access token under your Docker Hub account. Then<!-- Remove |Then| at the beginning of task steps. The sequence of events is already clear. --> set the ``username<!-- |user name| is preferred. -->`` and ``password``, using the access token as the ``password``. Do not use the password for the account itself. Using an access token makes it easier to revoke the token if necessary.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  dockerd:
    mirror:
      remote: https://registry-1.docker.io
      username: username
      password: access-token
```

Note<!-- Format the note properly. --> that<!-- Notes must be in Note boxes and start with |Note: |. --> an access token provides write access to Docker Hub. It is thus<!-- Re-write the sentence to drop |thus| or, if that is not possible, replace with |therefore|. --> also recommended you use a separate robot account in Docker Hub which isn't going to<!-- Maybe shorten to |to|. --> be used to host images, and also doesn't have write access to any other organizations. In other words, use it purely for reading images from Docker Hub.

If this is a free account, the higher limit on image pulls will<!-- Avoid |will|: present tense is preferred. --> then apply. If the account is paid, then there may<!-- |can| usually works better. Use |might| to convey possibility. --> be higher limits, or no limit all all.

Also note that<!-- Notes must be in Note boxes and start with |Note: |. --> the image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. --> mirror is only used when running or building images using the support for running ``docker<!-- |Docker| is preferred. -->``. The mirror does not come into play when creating deployments in Kubernetes which make use of images hosted on Docker Hub. Usage<!-- |Use| is preferred. --> of images from Docker Hub in deployments will<!-- Avoid |will|: present tense is preferred. --> still be subject to the limit for anonymous access, unless you were to supply image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. --> credentials for the deployment so an authenticated user were used.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Setting default access credentials

When deploying a training portal using the ``TrainingPortal`` custom resource, the credentials for accessing the portal will<!-- Avoid |will|: present tense is preferred. --> be unique for each instance. The details of the credentials can be<!-- Consider switching to active voice. --> found by viewing status information added to the custom resources using ``kubectl describe``.

If you want to<!-- Maybe replace with just |To|. --> override the credentials for the portals so the same set of credentials are used for each, they can be<!-- Consider switching to active voice. --> overridden by adding the desired<!-- Remove. --> values to the system profile.

To override the username<!-- |user name| is preferred. --> and password for the admin and robot accounts use ``portal.credentials``.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  portal:
    credentials:
      admin:
        username: eduk8s
        password: admin-password
      robot:
        username: robot@eduk8s
        password: robot-password
```

To override the client ID and secret used for OAuth access by the robot account, use ``portal.clients``.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  portal:
    clients:
      robot:
        id: robot-id
        secret: robot-secret
```

If the ``TrainingPortal`` has specified credentials or client information, they will<!-- Avoid |will|: present tense is preferred. --> still take precedence over the values specified in the system profile.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Overriding the workshop images

When a workshop does not define a workshop image to use, and instead downloads workshop content from GitHub or a web server, the ``base-environment`` workshop image is used. The workshop content is then added to the container, overlaid on this image.

The version of the ``base-environment`` workshop image used is what was the most up to date compatible version of the image available for that version of the Learning Center operator when it was released.

If necessary you can override what version of the ``base-environment`` workshop image is used by<!-- Active voice is preferred. --> defining a mapping under ``workshop.images``. For workshop images supplied as part of the Learning Center project, you can override the short names used to refer to<!-- If telling the reader to read something else, use |see|. --> them.

The short versions of the names which are recognised are:

* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`base-environment:*`` - A tagged version of the ``base-environment`` workshop image which has been<!-- Consider changing to |is| or |has| or rewrite for active voice. --> matched with the current version of the Learning Center operator.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`jdk8-environment:*`` - A tagged version of the ``jdk8-environment`` workshop image which has been<!-- Consider changing to |is| or |has| or rewrite for active voice. --> matched with the current version of the Learning Center operator.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`jdk11-environment:*`` - A tagged version of the ``jdk11-environment`` workshop image which has been<!-- Consider changing to |is| or |has| or rewrite for active voice. --> matched with the current version of the Learning Center operator.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`conda-environment:*`` - A tagged version of the ``conda-environment`` workshop image which has been<!-- Consider changing to |is| or |has| or rewrite for active voice. --> matched with the current version of the Learning Center operator.

If you wanted to override the version of the ``base-environment`` workshop image mapped to by the ``*`` tag, you would use:

```
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  workshop:
    images:
      "base-environment:*": "projects.registry.vmware.com/educates/base-environment:latest"
```

It is also possible to override where images are pulled from for any arbitrary image. This could be used where you want to cache the images for a workshop in a local image registry and avoid going outside of your network, or the cluster, to get them. This means you wouldn't need to override the workshop definitions for a specific workshop to change it.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  workshop:
    images:
      "quay.io/eduk8s-labs/lab-k8s-fundamentals:master": "registry.test/lab-k8s-fundamentals:master"
```

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Tracking using Google Analytics

If you want to record analytics data on usage of workshops using Google Analytics, you can enable tracking by supplying a tracking ID for Google Analytics.

``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  analytics:
    google:
      trackingId: UA-XXXXXXX-1
```

Custom dimensions are used in Google Analytics to record details about the workshop a user is doing, and through which training portal and cluster it was accessed. You can therefore use the same Google Analytics tracking ID with Learning Center running on multiple clusters.

To support use of custom dimensions in Google Analytics you must configure the Google Analytics property with the following custom dimensions. They must be added in the order shown as Google Analytics doesn't allow you to specify the index position for a custom dimension and will allocate them for you. You can't already have custom dimensions defined for the property, as the new custom dimensions must start at index of 1.

``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->
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

- W<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->orkshop/Start
- W<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->orkshop/Finish
- W<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->orkshop/Expired

Note<!-- Format the note properly. --> that<!-- Notes must be in Note boxes and start with |Note: |. --> Google Analytics is not a reliable way to collect data. This is because individuals or corporate firewalls can block the reporting of Google Analytics data. For more precise statistics, you should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> use the webhook URL for collecting analytics with a custom data collection platform. Configuration of a webhook URL for analytics can only be specified on the ``TrainingPortal`` definition and cannot be specified globally on the ``SystemProfile`` configuration.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Overriding styling of the workshop

If using the REST API to create/manage<!-- If this is a URL then you likely need to present it per xref rules: https://docs-wiki.cfapps.io/wiki/style/cross-ref-style.html --> workshop sessions and the workshop dashboard is then embedded into an iframe of a separate site, it is possible<!-- |might| might be better. --> to<!-- Active voice |you can| might be better. --> perform minor styling changes of the dashboard, workshop content and portal to match the separate site. To do this you can provide CSS styles under ``theme.dashboard.style``, ``theme.workshop.style`` and ``theme.portal.style``. For dynamic styling, or for adding hooks to report on progress through<!-- Do not use as a synonym for |finished| or |done|. Do not use when you mean |by using|. --> a workshop to a separate service, you can also supply Javascript<!-- |JavaScript| is preferred. --> as part of the theme under ``theme.dashboard.script``, ``theme.workshop.script`` and ``theme.portal.script``.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: SystemProfile
metadata:
  name: default-system-profile
spec:
  theme:
    dashboard:
      script: |
        console.log("Dashboard theme overrides.");
      style: |
        body {
          font-family: "Comic Sans MS", cursive, sans-serif;
        }
    workshop:
      script: |
        console.log("Workshop theme overrides.");
      style: |
        body {
          font-family: "Comic Sans MS", cursive, sans-serif;
        }
    portal:
      script: |
        console.log("Portal theme overrides.");
      style: |
        body {
          font-family: "Comic Sans MS", cursive, sans-serif;
        }
``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Additional custom system profiles

If the default system profile is specified, it will<!-- Avoid |will|: present tense is preferred. --> be used by<!-- Active voice is preferred. --> all deployments managed by the Learning Center operator unless the system profile to use has been<!-- Consider changing to |is| or |has| or rewrite for active voice. --> overridden for a specific deployment. The name of the system profile can be<!-- Consider switching to active voice. --> set for deployments by setting the ``system.profile`` property of ``TrainingPortal``, ``WorkshopEnvironment`` and ``WorkshopSession`` custom resources.

```
apiVersion: training.eduk8s.io/v1alpha1
kind: TrainingPortal
metadata:
  name: lab-markdown-sample
spec:
  system:
    profile: training-eduk8s-io-profile
  workshops:
  - name: lab-markdown-sample
    capacity: 1
```
