# Anonymous access

The REST API with client authentication provides a means to have the portal create and manage workshop sessions on your behalf but have user authentication handled by a separate system.

If you do not need to have users be authenticated, but still want to provide your own front end from which users select a workshop, such as when integrating workshops into an existing web property, you can enable anonymous mode and redirect users direct to a URL for workshop session creation.

Note that using this is only recommended for temporary deployments and not for a permanent web site providing access to workshops.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Enabling anonymous access

To enable full anonymous access to the training portal, you need to set the registration type to anonymous.

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
  ...
```

Note that users will still be able to visit the training portal directly and view the catalog of available workshops. You therefore shouldn't link to the main page of the training portal. Instead you need to link from your custom index page, to the individual links for creating each workshop.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Triggering workshop creation

To trigger creation and allocation of a workshop to a user, you need to direct users browsers to a URL specific to the workshop. The form of this URL should be:

``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->
<training-portal-url>/workshops/environment/<name>/create/?index_url=<index>
```

The value ``<name>`` should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> be replaced with the name of the workshop environment corresponding to the workshop which needs to<!-- |must| is preferred. --> be created.

The value ``<index>`` should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> be replaced with the URL for your custom index page where you list the workshops available. When the workshop is completed by<!-- Active voice is preferred. --> a user, they will<!-- Avoid |will|: present tense is preferred. --> be redirected back to this index page. They will<!-- Avoid |will|: present tense is preferred. --> also be redirected back to this index page when an error occurs.

When a user is redirected back to the index page, a query string parameter will<!-- Avoid |will|: present tense is preferred. --> be supplied to notify of the reason the user is being returned. This can be<!-- Consider switching to active voice. --> used to display a banner or other indication as to why they were returned.

The name of the query string parameter is ``notification`` and the possible values are:

* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`session-deleted`` - Used when the workshop session was completed or restarted.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`workshop-invalid`` - Used when the name of the workshop environment supplied when attempting to create the workshop was invalid.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`session-unavailable`` - Used when capacity has been<!-- Consider changing to |is| or |has| or rewrite for active voice. --> reached and a workshop session cannot be created.
* `<!-- Use |-| for bullet points. DocWorks is unreliable with |+| and |*|. -->`session-invalid`` - Used when an attempt is made to access a session which doesn't exist. This can occur when the workshop dashboard is refreshed sometime after the workshop session had expired and been deleted.
