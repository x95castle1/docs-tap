# Workshop catalog

A single training portal can hosted one or more workshops. The REST API endpoints for the workshops catalog provide a means to list the available workshops and get information on<!-- |information about| is preferred. --> them.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Listing available workshops

The URL sub path for accessing the list of available workshop environments is ``/workshops/catalog/environments/``. When making the request, the access token must be supplied in the HTTP ``Authorization`` header with type set as ``Bearer``:

```
curl -v -H "Authorization: Bearer <access-token>" \
<training-portal-url>/workshops/catalog/environments/
```

The JSON response will be of the form:

``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->
{
  "portal": {
    "name": "educates-tutorials",
    "uid": "9b82a7b1-97db-4333-962c-97be6b5d7ee0",
    "generation": 451,
    "url": "<training_portal_url>",
    "sessions": {
      "maximum": 10,
      "registered": 0,
      "anonymous": 0,
      "allocated": 0
    }
  },
  "environments": [
    {
      "name": "educates-tutorials-w01",
      "state": "RUNNING",
      "workshop": {
        "name": "lab-et-self-guided-tour",
        "id": "15e5f1a569496f335049bb00c370ee20",
        "title": "Workshop Building Tutorial",
        "description": "A guided tour of how to build a workshop for your team's learning center.",
        "vendor": "",
        "authors": [],
        "difficulty": "",
        "duration": "",
        "tags": [],
        "logo": "",
        "url": "<workshop_repository_url"
      },
      "duration": 1800,
      "capacity": 10,
      "reserved": 0,
      "allocated": 0,
      "available": 0
    }
  ]
}
```

For each workshop listed under ``environments``, where a field listed under ``workshop`` has the same name as it appears in the ``Workshop`` custom resource, it has the same meaning. The ``id`` field is an additional field which tries to uniquely identify a workshop based on the name of the workshop image, the Git repository for the workshop, or the web site hosting the workshop instructions. The value of the ``id`` field doesn't rely on the name of the ``Workshop`` resource and should<!-- In most cases, replace with |must|. If using |should| is unavoidable, it must be paired with information on the exceptions that |should| implies exist. --> be the same if same workshop details are used but the name of the ``Workshop`` resource is different.

The ``duration`` field provides the time in seconds after which the workshop environment will<!-- Avoid |will|: present tense is preferred. --> be expired. The value can be<!-- Consider switching to active voice. --> ``null`` if there is no expiration time for the workshop.

The ``capacity`` field is the maximum number of workshop sessions that can be<!-- Consider switching to active voice. --> created for the workshop.

The ``reserved`` field indicates how many instances of the workshop will<!-- Avoid |will|: present tense is preferred. --> be reserved as hot spares. These will<!-- Avoid |will|: present tense is preferred. --> be used to service requests for a workshop session. If no reserved instances are available and capacity has not been reached, a new workshop session will<!-- Avoid |will|: present tense is preferred. --> be created on demand.

The ``allocated`` field indicates how many workshop sessions are active and currently allocated to a user.

The ``available`` field indicates how many workshop sessions are available for immediate allocation. This will<!-- Avoid |will|: present tense is preferred. --> never be more than the number of reserved instances.

Under ``portal.sessions``, the ``allocated`` field indicates the total number of allocated sessions across all workshops hosted by the portal.

Where ``maximum``, ``registered`` and ``anonymous``<!-- Insert the Oxford comma if it is missing here. --> are non zero<!-- |nonzero| is preferred. -->, they indicate caps on number of workshops that can be<!-- Consider switching to active voice. --> run.

The ``maximum`` indicates a maximum on the total number of workshop sessions that can be<!-- Consider switching to active voice. --> run by the portal across all workshops. Even where a specific workshop may<!-- |can| usually works better. Use |might| to convey possibility. --> not have reached capacity, if ``allocated`` for the whole portal has reached ``maximum``, then no more workshop sessions will<!-- Avoid |will|: present tense is preferred. --> be able to<!-- |can| is preferred. --> be created.

The value of ``registered`` when non zero<!-- |nonzero| is preferred. --> indicates a cap on the number of workshop sessions a single registered portal user can have running at the one time.

The value of ``anonymous`` when non zero<!-- |nonzero| is preferred. --> indicates a cap on the number of workshop sessions an anonymous user can have running at the one time. Anonymous users are users created as a result of the REST API being used, or if anonymous access is enabled when accessing the portal via<!-- |through|, |using| and |by means of| are preferred. --> the web interface.

By default, only workshop environments which are currently marked with a ``state`` of ``RUNNING`` are returned. That is, those workshop environments which are taking new workshop session requests. If you also want to see the workshop environments which are currently in the process of being shutdown<!-- The noun is |shutdown|. The action is |shut down|. -->, you need to<!-- Consider replacing with just |To|. --><!-- |must| is preferred. --> provide the ``state`` query string parameter to the REST API call and indicate which states workshop environments should be returned for.

```
curl -v -H "Authorization: Bearer <access-token>" \
https://lab-markdown-sample-ui.test/workshops/catalog/environments/?state=RUNNING&state=STOPPING
``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->

The ``state`` query string parameter can be<!-- Consider switching to active voice. --> included more than once to be able to<!-- |can| is preferred. --> see workshop environments in both ``RUNNING`` and ``STOPPING`` states.

Note<!-- Format the note properly. --> that<!-- Notes must be in Note boxes and start with |Note: |. --> if anonymous access to the list of workshop environments is enabled and you are not authenticated when using the REST API endpoint, only workshop environments in a running state will<!-- Avoid |will|: present tense is preferred. --> be returned.
