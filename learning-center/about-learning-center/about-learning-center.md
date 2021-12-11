# Learning Center overview

The Learning Center is designed to<!-- Do not use. Tell the reader what the product or information does. --> provide a platform for hosting workshops. Although Learning Center requires
Kubernetes to run, and is being used to teach users about Kubernetes, it could<!-- |can| or |might| whenever possible is preferred. When providing examples, use simple present tense verbs instead of postulating what someone or something could or would do. --> also be used to host training for other
purposes as well. It may<!-- |can| usually works better. Use |might| to convey possibility. --> for example <!-- Consider adding a comma after |for example|. -->be used to help train users in web based applications, use of databases, or
programming languages, where the user has no interest or need for Kubernetes.


##   <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Overall goals

The use case<!-- Just |use| is probably better here; avoid the nounification of verbs. --> scenarios which Learning Center has been<!-- Consider changing to |is| or |has| or rewrite for active voice. --> designed to<!-- Do not use. Tell the reader what the product or information does. --> support are as follows.

- S<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->upervised workshops. This could<!-- |can| or |might| whenever possible is preferred. When providing examples, use simple present tense verbs instead of postulating what someone or something could or would do. --> be a workshop run at a conference, at a customer site, or purely online.
  The workshop has a set time period and you know the maximum number of users to expect. Once<!-- Only use |once| when you mean |one time|, not when you mean |after|. --> the training has
  completed, the Kubernetes cluster created for the workshop would<!-- Re-phrase for present tense if possible. --> be destroyed.

- T<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->emporary learning portal. This is where you need to<!-- Consider replacing with just |To|. --><!-- |must| is preferred. --> provide access to a small set of workshops of short duration for
  use as hands on demos at a conference vendor booth. Users would<!-- Re-phrase for present tense if possible. --> select which topic they want to learn about and do
  that workshop. The workshop instance would<!-- Re-phrase for present tense if possible. --> be created on demand. When they have finished the workshop, that workshop
  instance would<!-- Re-phrase for present tense if possible. --> be destroyed to free up resources. Once<!-- Only use |once| when you mean |one time|, not when you mean |after|. --> the conference had finished, the Kubernetes cluster would<!-- Re-phrase for present tense if possible. --> be
  destroyed.

- P<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->ermanent learning portal. Similar to the temporary learning portal, but would<!-- Re-phrase for present tense if possible. --> be run on an extended basis as a public
  web site where anyone could<!-- |can| or |might| whenever possible is preferred. When providing examples, use simple present tense verbs instead of postulating what someone or something could or would do. --> come and learn at any time.

- P<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->ersonal training or demos. This is where anyone wants to run a workshop on their own Kubernetes cluster to learn that
  topic, or where a product demo was packaged up as a workshop and they want to use it to demonstrate the product to a
  customer. The workshop environment can be<!-- Consider switching to active voice. --> destroyed when complete, but there is no need for the cluster to be destroyed.

When running workshops, where ever possible a shared Kubernetes cluster would<!-- Re-phrase for present tense if possible. --> be used so as to reduce the amount of set
up required. This works for developer focused workshops as it is usually not necessary to provide elevated access to the
Kubernetes cluster, and role based access controls (RBAC) can be<!-- Consider switching to active voice. --> used to prevent users from interfering with each other.
Quotas can also be set so that users are restricted to how much resources they can use.

In the case of needing to run workshops which deal with cluster operations, for which users need cluster admin access,
a separate cluster would<!-- Re-phrase for present tense if possible. --> be created for each user. Learning Center doesn't deal with provisioning clusters, only with
deploying a workshop environment in a cluster once it<!-- Only use |once| when you mean |one time|, not when you mean |after|. --> exists.

In catering for the scenarios listed above<!-- IX Standards forbids referring to text |above|. Use |earlier| or, better, just an anchor. -->, the set of primary requirements related to creation of workshop content,
and what could<!-- |can| or |might| whenever possible is preferred. When providing examples, use simple present tense verbs instead of postulating what someone or something could or would do. --> be done at run time<!-- |runtime| is preferred. --><!-- |runtime| is preferred unless referring to the time it takes a program to run. --> were as follows.

- E<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->verything for a workshop needed to be able to<!-- |can| is preferred. --> be stored in a Git repository, with no dependency on using a special
  web application or service to create a workshop.

- U<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->se of GitHub as a means to distribute workshop content. Alternatively, optional distribution of a workshop as a
  container image. The latter also being necessary if special tools need to<!-- |must| is preferred. --> be installed for use in a workshop.

- T<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->he<!-- |they| is preferred. --> instructions for a user to follow to do the workshop would<!-- Re-phrase for present tense if possible. --> be provided as Markdown or AsciiDoc files. <!-- Close all items in the list with periods or close no items in the list with periods. No mixing. -->

- I<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->nstructions can be<!-- Consider switching to active voice. --> annotated as executable commands so that when clicked on in the workshop dashboard they can be<!-- Consider switching to active voice. -->
  automatically<!-- Avoid if not definitely useful to mention. --> executed for the user in the appropriate terminal to avoid mistakes when commands are entered manually.

- T<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->ext can be<!-- Consider switching to active voice. --> annotated as copyable so that when clicked on in the workshop dashboard it would<!-- Re-phrase for present tense if possible. --> be copied into the
  browser paste buffer ready for pasting into the terminal or other web application.

- E<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->ach user is provided access to one or more namespaces in the Kubernetes cluster unique to their session. For
  Kubernetes based workshops, this is where applications would<!-- Re-phrase for present tense if possible. --> be deployed as part of the workshop.

- A<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->dditional Kubernetes resources specific to a workshop session can be<!-- Consider switching to active voice. --> pre-created when the session is started. This
  is to enable the deployment of applications for each user session.

- A<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->dditional Kubernetes resources common to all workshop sessions are able to<!-- |can| is preferred. --> be deployed when the workshop environment
  is first created. This is to enable deployment of shared applications used by all users.

- A<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->pplication of resource quotas on each workshop session to control how much resources users can consume.

- A<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->pplication of role based access control (RBAC) on each workshop session to control what users can do. <!-- Close all items in the list with periods or close no items in the list with periods. No mixing. -->

- A<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->bility to provide access to an editor (IDE) in the workshop dashboard in the web browser for users to use to edit
  files during the workshop.

- A<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->bility to provide access to a web based console for accessing the Kubernetes cluster. Use of the Kubernetes dashboard
  or Octant is supported.

 <!-- When introducing a bullet, end with a colon. -->- A<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->bility to integrate additional web based applications into the workshop dashboard specific to the topic of the workshop.

 <!-- When introducing a bullet, end with a colon. -->* A<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->bility for the workshop dashboard to display slides used by an instructor in support of the workshop.

##   <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Platform architectural overview

The Learning Center relies on a Kubernetes operator<!-- |Kubernetes Operator| is preferred. --> to perform the bulk of the work. The actions of the operator are
controlled through<!-- Do not use as a synonym for |finished| or |done|. Do not use when you mean |by using|. --> a set of custom resources specific to the Learning Center.

There are multiple ways of using the custom resources to deploy workshops. The primary way is to create a training
portal, which in turn then triggers the setup of one or more workshop environments, one for each distinct workshop.
When users access the training portal and select the workshop they wish<!-- |want| is preferred. --> to do, the training portal allocates to that
user a workshop session (creating one if necessary) against the appropriate workshop environment, and the user is
redirected to that workshop session instance.

![](images/architectural-overview.png) <!-- Alt text must describe the image in detail. --> <!-- Alt text must describe the image in detail. -->

Each workshop session can be<!-- Consider switching to active voice. --> associated with one or more Kubernetes namespaces specifically for use during that session.
Role based access control (RBAC) applied to the unique Kubernetes service account for that session, ensures that the
user can only access the namespaces and other resources that they are allowed to for that workshop.

In this scenario, the custom resource types that come into play are:

- `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`Workshop`` - Provides the definition of a workshop. Preloaded by an administrator<!-- |admin| is preferred. --> into the cluster, it defines
  where the workshop content is hosted, or the location of a container image which bundles the workshop content and any
  additional tools required for the workshop. The definition also lists additional resources that should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> be created
  which are to be shared between all workshop sessions, or for each session, along with<!-- Do not use as a conjunction. Whenever possible, use |with| or |and| instead. --> details of resources quotas and
  access roles required by the workshop.

- `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`TrainingPortal`` - Created by an administrator<!-- |admin| is preferred. --> in the cluster to trigger the deployment of a training portal. The
  training portal can provide access to one or more distinct workshops defined by a ``Workshop`` resource. The training
  portal provides a web based interface for registering for workshops and accessing them. It also provides a REST API
  for requesting access to workshops, allowing custom front ends to be created which integrate with separate identity
  providers and which provide an alternate means for browsing and accessing workshops.

- `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`WorkshopEnvironment`` - Used by the training portal to trigger the creation of a workshop environment for a
  workshop. This causes the operator to setup<!-- |set up| is the action. |setup| is a noun. --><!-- |set up| is the action. | <!-- Do not include closing punctuation for a sentence fragment in a cell, unless it is followed by another fragment or sentence. -->setup| is a noun. --> a namespace for the workshop into which shared resources can be<!-- Consider switching to active voice. --> deployed,
  and where the workshop sessions are run.

- `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`WorkshopSession`` - Used by the training portal to trigger the creation of a workshop session against a specific
  workshop environment. This causes the operator to setup<!-- |set up| is the action. |setup| is a noun. --><!-- |set up| is the action. | <!-- Do not include closing punctuation for a sentence fragment in a cell, unless it is followed by another fragment or sentence. -->setup| is a noun. --> any namespaces specific to the workshop session and pre-create
  additional resources required for a workshop session. Workshop sessions can be<!-- Consider switching to active voice. --> created up front in reserve, to be
  handed out when requested, or they can be<!-- Consider switching to active voice. --> created on demand.
