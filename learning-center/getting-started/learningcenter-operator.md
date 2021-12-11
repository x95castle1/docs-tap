# Learning Center Kubernetes operator <!-- For VMware style, write all headers in sentence case. -->

Before you can start deploying workshops, you must install a Kubernetes Operator for Learning Center. <br/>
The operator manages the setup of the environment for each workshop and deploys instances of a workshop for each person.
Basic instructions for [Installing the Learning Center Operator](https://github.com/pivotal/docs-tap/blob/main/install-components.md#install-learning-center) is located in GitHub.

**Below is additional information on installing and setting up the Learning Center Operator**

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Kubernetes Cluster requirements

The Learning Center Operator can be<!-- Consider switching to active voice. --> deployed to any Kubernetes cluster that supports custom
resource definitions and the concept of operators.

The cluster must have an ingress router configured. Only a basic deployment of the ingress controller
is usually required. You do not need to<!-- |must| is preferred. --> configure the ingress controller to handle cluster-wide edge
termination of secure HTTP connections. Learning Center creates Kubernetes Ingress resources and
supplies any secret for use with secure HTTP connections for each ingress.

For the ingress controller, VMware recommends the use of Contour over alternatives such as Nginx<!-- |NGINX| for the brand. |nginx| for files and code. -->.
An nginx<!-- |NGINX| for the brand. |nginx| for files and code. -->-based ingress controller is of inferior design (every time a new ingress is
created or deleted, the nginx<!-- |NGINX| for the brand. |nginx| for files and code. --> config is reloaded, resulting in websocket connections being terminated
after a period of time). Learning Center terminals are implemented to reconnect automatically<!-- Avoid if not definitely useful to mention. --> if the websocket connection is lost, but not all applications with specific workshops will<!-- Avoid |will|: present tense is preferred. -->
handle loss of websocket connections gracefully, and these applications might be impacted due to the use of an nginx<!-- |NGINX| for the brand. |nginx| for files and code. -->
ingress controller. This problem is not specific to Learning Center. Any application can be<!-- Consider switching to active voice. --> effected when
an nginx<!-- |NGINX| for the brand. |nginx| for files and code. --> ingress controller is used frequently and ingresses are created/delete<!-- If this is a URL then you likely need to present it per xref rules: https://docs-wiki.cfapps.io/wiki/style/cross-ref-style.html --> frequently.

If using a hosted Kubernetes solution from an IaaS provider such as Google, AWS, or Azure, you must
ensure that any HTTP request timeout specified on the inbound load balancer for the ingress controller is
increased so that existing websocket connections can be<!-- Consider switching to active voice. --> used. In some cases, Load balancers of hosted Kubernetes solutions have only a 30-second timeout. If possible, configure the timeout to one hour.

If deploying the web-based training portal, the cluster must have available persistent volumes of type ``ReadWriteOnce (RWO)``.
A default storage class must be defined so that persistent volume claims are not required to specify<!-- Use as a general term to instruct users when they have several selections to make. Do not use in procedures when a more specific term, such as enter, select, or click, is appropriate. --> a storage class.
For some Kubernetes distributions, including those from IBM, you might be required to configure Learning Center to discover users and groups that should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> be used for persistent volumes. If no default storage class is specified, or a specified storage class is required, you can configure Learning Center with the name of the storage class.

You must have cluster admin access to install the Learning Center operator.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Cluster Pod<!-- |Pod| is capitalized per the K8s docs style. --> security policies

The Learning Center operator will<!-- Avoid |will|: present tense is preferred. --> define Pod security policies to limit what users can do from workshops when
deploying workloads to the cluster. The default policy prohibits running of images as the ``root`` user or using
a privileged Pod<!-- |Pod| is capitalized per the K8s docs style. -->. Specified workshops may<!-- |can| usually works better. Use |might| to convey possibility. --> relax these restrictions and apply a policy which enables additional
privileges required by the workshop.

It is highly recommended that the Pod<!-- |Pod| is capitalized per the K8s docs style. --> security policy admission controller be enabled for the cluster to ensure
that the Pod<!-- |Pod| is capitalized per the K8s docs style. --> security policies are applied. If the admission controller is not enabled, users would<!-- Re-phrase for present tense if possible. --> be able to<!-- |can| is preferred. -->
deploy workloads which run as the ``root`` user in a container, or run privileged pods<!-- |Pods| is capitalized per the K8s docs style. -->.

If you are unable to enable the pod<!-- |Pod| is capitalized per the K8s docs style. --> security policy admission controller, you should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> only provide access to workshops
deployed using the Learning Center operator to users you trust.

Also note that<!-- Notes must be in Note boxes and start with |Note: |. --> where the pod<!-- |Pod| is capitalized per the K8s docs style. --> security policy admission controller is not enabled, any workshops which have users create
persistent volumes may<!-- |can| usually works better. Use |might| to convey possibility. --> not work, as the user the pod<!-- |Pod| is capitalized per the K8s docs style. --> runs as may<!-- |can| usually works better. Use |might| to convey possibility. --> not have access to the volume. The pod<!-- |Pod| is capitalized per the K8s docs style. --> security policy
when applied usually enables access by ensuring that the security context of a pod<!-- |Pod| is capitalized per the K8s docs style. --> is modified to give access.

Whether the absence of the pod<!-- |Pod| is capitalized per the K8s docs style. --> security policy admission controller will<!-- Avoid |will|: present tense is preferred. --> cause issues with access to persistent volumes
will<!-- Avoid |will|: present tense is preferred. --> depend on the cluster. Although minikube does not enable the pod<!-- |Pod| is capitalized per the K8s docs style. --> security policy admission controller, it will<!-- Avoid |will|: present tense is preferred. --> still
work as persistent volumes when mounted give write permissions to all users.

No matter whether pod<!-- |Pod| is capitalized per the K8s docs style. --> security policies are enabled or not, individual workshops should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> always be reviewed as to what
additional privileges they grant before allowing their use in a cluster.

##   <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Specifying the ingress domain

The operator when deploying instances of the workshop environments needs to<!-- |must| is preferred. --> be able to<!-- |can| is preferred. --> expose them
via<!-- |through|, |using| and |by means of| are preferred. --> an external URL for access. To define the domain name that can be<!-- Consider switching to active voice. --> used as a suffix to hostnames<!-- |host names| is preferred. --> for instances.

>  <!-- Proper note formatting is |>**Note:**| or |>**Caution:**| or |>**Important:**|. -->**Note:** For the custom domain you are using, DNS must have been<!-- Consider replacing with |were| or shifting to present tense. --> configured with a wildcard domain to forward all requests for sub domains of the custom domain, to the ingress router of the Kubernetes cluster.

It is recommended<!-- Specify the party that recommends (VMware, Cloud Foundry, etc). --> that you avoid using a ``.dev`` domain name as such domain names have a requirement<!-- Requirements are things needed to install and run the product. We list them on the product index page. Prerequisites are things needed for a given procedure beyond the product requirements. We list those on the procedure page. -->
to always use HTTPS and you cannot use HTTP. Although you can provide a certificate for secure
connections under the domain name for use by Learning Center, this doesn't extend to what a workshop
may<!-- |can| usually works better. Use |might| to convey possibility. --> do. By using a ``.dev`` domain name, if workshop instructions have you creating ingresses in Kubernetes
using HTTP only, they will<!-- Avoid |will|: present tense is preferred. --> not work.

>  <!-- Proper note formatting is |>**Note:**| or |>**Caution:**| or |>**Important:**|. -->**Note:** If you are running Kubernetes on your local machine using a system like<!-- Do not use when citing examples. Use |such as| instead. --> ``minikube`` and you don't have a custom domain name which maps to the IP<!-- Do not omit |address| from |IP address|. --> for the cluster, you can use a ``nip.io`` address. For example, if ``minikube ip`` returned ``192.168.64.1``, you could<!-- |can| or |might| whenever possible is preferred. When providing examples, use simple present tense verbs instead of postulating what someone or something could or would do. --> use the 192.16<!-- Version numbers are written as |v1.11.11| unless brand styling dictates otherwise. -->8.64.1.nip.io domain. Note that<!-- Notes must be in Note boxes and start with |Note: |. --> you cannot use an address of form ``127.0.0.1.nip.io``, or ``subdomain.localhost``. This will<!-- Avoid |will|: present tense is preferred. --> cause a failure as internal services when needing to connect to each other, would<!-- Re-phrase for present tense if possible. --> end up<!-- Quite colloquial. Try rephrasing the sentence to replace with simply |are|. --> connecting to themselves instead, since<!-- Do not use |since| where you can use |because|. --> the address would<!-- Re-phrase for present tense if possible. --> resolve to the host loopback address of ``127.0.0.1``.

###   <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Configuration yaml<!-- |YAML| is preferred. -->
Define the ``ingressDomain`` property on the configuration yaml passed to Tanzu CLI.
 <!-- Proper note formatting is |>**Note:**| or |>**Caution:**| or |>**Important:**|. -->**Note:** This property is required to install Learning Center using Tanzu CLI
```
ingressDomain: learningcenter.my-domain.com
```

###   <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Set the environment variable manually
Set the ``INGRESS_DOMAIN`` environment variable on the operator deployment. To set the ``INGRESS_DOMAIN``
environment variable, run:

```
kubectl set env deployment/eduk8s-operator -n educates INGRESS_DOMAIN=test
``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| --> <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->
Replace ``test`` with the domain name for your Kubernetes cluster.

or if using a ``nip.io`` address
```
kubectl set env deployment/eduk8s-operator -n educates INGRESS_DOMAIN=192.168.64.1.nip.io
``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| --> <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->

Note<!-- Format the note properly. --> <!-- Proper note formatting is |>**Note:**| or |>**Caution:**| or |>**Important:**|. --><!-- Format the note properly. --> that<!-- Notes must be in Note boxes and start with |Note: |. --> use of environment variables to configure the operator is a short cut to cater for the
simple<!-- Avoid suggesting an instruction is |simple| or |easy|. --> use case<!-- Just |use| is probably better here; avoid the nounification of verbs. -->. The recommended way is to use Tanzu CLI or for more complicated scenarios the
``SystemProfile`` custom resource can be<!-- Consider switching to active voice. --> used.


##   <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Enforcing secure connections

By default, the workshop portal and workshop sessions will<!-- Avoid |will|: present tense is preferred. --> be accessible over HTTP connections. If you
wish<!-- |want| is preferred. --> to use secure HTTPS connections, you must have access to a wildcard SSL certificate for the domain
under which you wish<!-- |want| is preferred. --> to host the workshops. You cannot use a self signed<!-- Check if a hyphen is missing. --> certificate.

Wildcard certificates can be<!-- Consider switching to active voice. --> created using `letsencrypt <https://letsencrypt.org/>`_. Once<!-- Only use |once| when you mean |one time|, not when you mean |after|. --> you have the
certificate, you can:

###   <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Configuration yaml<!-- |YAML| is preferred. -->

The easiest way to define the certificate is through<!-- Do not use as a synonym for |finished| or |done|. Do not use when you mean |by using|. --> the configuration passed to Tanzu CLI. So define
the ``certificate`` and ``privateKey`` properties under the ``ingressSecret`` property to specify the
certificate on the configuration yaml passed to Tanzu CLI

```
ingressSecret:
  certificate: MIIC2DCCAcCgAwIBAgIBATANBgkqh ...
  privateKey: MIIEpgIBAAKCAQEA7yn3bRHQ5FHMQ ...
``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| --> <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->

If you already has a TLS secret, you can copy it to the ``educates`` namespace or that one you defined,
and use the ``secretName`` property.
```
ingressSecret:
  secretName: workshops.example.com-tls
```

###   <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Create the TLS secret manually

If you want to<!-- Maybe replace with just |To|. --> add the certificate as a secret in the ``educates`` namespace or that one you defined, the
secret needs to<!-- |must| is preferred. --> be of type ``tls``. You can create it using the ``kubectl create secret tls`` command.

```
kubectl create secret tls -n educates workshops.example.com-tls --cert=workshops.example.com/fullchain.pem --key=workshops.example.com/privkey.pem
```

Having created the secret, if it is the secret corresponding to the default ingress domain you specified
above<!-- IX Standards forbids referring to text |above|. Use |earlier| or, better, just an anchor. -->, set the ``INGRESS_SECRET`` environment variable (if you don't want to use the configuration passed to Tanzu CLI)
on the operator deployment. This will ensure that it is applied automatically to any ingress created.

```
kubectl set env deployment/eduk8s-operator -n eduk8s INGRESS_SECRET=workshops.example.com-tls
``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| --> <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->

If the certificate isn't that of the default ingress domain, you can supply the domain name and name of the
secret when creating a workshop environment or training portal. In either case, secrets for the wildcard
certificates must be created in the ``educates`` namespace or that one you defined.

##   <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Specifying the ingress class

Any ingress routes created will<!-- Avoid |will|: present tense is preferred. --> use the default ingress class. If you have multiple ingress class
types available, and you need to<!-- Consider replacing with just |To|. --><!-- |must| is preferred. --> override which is used, you can:

###   <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Configuration yaml<!-- |YAML| is preferred. -->
Define the ``ingressClass`` property on the configuration yaml passed to Tanzu CLI
```
ingressClass: contour
```

###   <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Set the environment variable manually
Set the ``INGRESS_CLASS`` environment variable for the eduk8s operator.
```
kubectl set env deployment/eduk8s-operator -n educates INGRESS_CLASS=contour
``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| --> <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->

This only applies to the ingress created for the training portal and workshop sessions. It does not apply
to the any ingress created from a workshop as part of the workshop instructions.

This may<!-- |can| usually works better. Use |might| to convey possibility. --> be necessary where a specific ingress provider is not as reliable in maintaining websocket connections,
as explained above<!-- IX Standards forbids referring to text |above|. Use |earlier| or, better, just an anchor. --> is the case for the nginx ingress controller when there are frequent creation or deletions
of ingresses occurring in the cluster.

##   <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Trusting insecure<!-- |not secure| is preferred. --> registries

One of the options available for workshops is to automatically<!-- Avoid if not definitely useful to mention. --> deploy an image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. --> per workshop session.
When the Learning Center operator is configured to use a secure ingress with valid wildcard certificate, the
image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. --> will<!-- Avoid |will|: present tense is preferred. --> work out of the box.

If the Learning Center operator is not setup to use secure ingress, the image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. --> will<!-- Avoid |will|: present tense is preferred. --> be accessed over
HTTP and will<!-- Avoid |will|: present tense is preferred. --> be regarded as an insecure<!-- |not secure| is preferred. --> registry.

When using the optional support for building container images using ``docker<!-- |Docker| is preferred. -->``, the docker<!-- |Docker| is preferred. --> daemon deployed for
the workshop session will<!-- Avoid |will|: present tense is preferred. --> be configured in this case so it knows<!-- Avoid anthropomorphizing: |can| or |detects| might be better here. --> the image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. --> is insecure<!-- |not secure| is preferred. --> and pushing
images to the image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. --> will<!-- Avoid |will|: present tense is preferred. --> still work.

In this case of an insecure<!-- |not secure| is preferred. --> image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. -->, deployment of images from the image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. --> to the Kubernetes
cluster will<!-- Avoid |will|: present tense is preferred. --> not however work unless the Kubernetes cluster is configured to trust the insecure<!-- |not secure| is preferred. --> registry.

How you configure a Kubernetes cluster to trust an insecure<!-- |not secure| is preferred. --> image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. --> will<!-- Avoid |will|: present tense is preferred. --> differ based on how the Kubernetes
cluster is deployed and what container runtime it uses.

If you are using ``minikube`` with ``dockerd``, to ensure that the image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. --> is trusted, you will<!-- Avoid |will|: present tense is preferred. --> need to<!-- |must| is preferred. -->
set up the trust the very first time you create the minikube instance.

To do this, first determine<!-- |determine| has two meanings. Consider if the univocal |discover| or |verify| would be better. --> which IP<!-- Do not omit |address| from |IP address|. --> subnet minikube uses for the inbound ingress router of the cluster. If you
already have a minikube instance running, you can determine<!-- |determine| has two meanings. Consider if the univocal |discover| or |verify| would be better. --> this by running ``minikube ip``. If for example <!-- Consider adding a comma after |for example|. -->this
reported ``192.168.64.1``, the subnet used is ``129.168.64.0/24``.

With this information, when you create a fresh ``minikube`` instance you would<!-- Re-phrase for present tense if possible. --> supply the ``--insecure<!-- |not secure| is preferred. -->-registry``
option with the subnet.

```
minikube start --insecure-registry="129.168.64.0/24"
```

What this option will<!-- Avoid |will|: present tense is preferred. --> do is tell ``dockerd`` to regard any image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. --> as insecure<!-- |not secure| is preferred. -->, which is deployed in the
Kubernetes cluster, and which is accessed via<!-- |through|, |using| and |by means of| are preferred. --> a URL exposed via<!-- |through|, |using| and |by means of| are preferred. --> an ingress route of the cluster itself.

Note<!-- Format the note properly. --> that<!-- Notes must be in Note boxes and start with |Note: |. --> at this time there is no known way to configure ``containerd`` to treat image registries matching a
wildcard subdomain, or which reside in a subnet, as insecure<!-- |not secure| is preferred. -->. It is therefore not possible to run workshops which
need to<!-- |must| is preferred. --> deploy images from the per session image registry<!-- If generic, |container image registry| on first use. If VMware-provided, |Tanzu Network Registry| on first use. In both cases, |registry| thereafter except where risking ambiguity. --> when using ``containerd`` as the underlying Kubernetes
cluster container runtime. This is a limitation of ``containerd`` and there are no known plans for ``containerd``
to support this ability. This will<!-- Avoid |will|: present tense is preferred. --> limit your ability to use Kubernetes clusters deployed with a tool like<!-- Do not use when citing examples. Use |such as| instead. --> ``kind``,
which relies on using ``containerd``.
