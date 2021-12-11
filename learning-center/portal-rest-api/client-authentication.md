# Client authentication

The training portal web interface is a quick way of providing access to a set of workshops when running a supervised training workshop. For integrating access to workshops into an existing web site, or for creating a custom web interface for accessing workshops hosted across one or more training portals, you can use can use the portal REST API.

The REST API will<!-- Avoid |will|: present tense is preferred. --> give you access to the list of workshops hosted by a training portal instance and allow you to request and access workshop sessions. This bypasses the training portal's own user registration and login so you can implement whatever access controls you need. This could<!-- |can| or |might| whenever possible is preferred. When providing examples, use simple present tense verbs instead of postulating what someone or something could or would do. --> include anonymous access, or access integrated into an organization's single sign-on system.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Querying the credentials

To provide access to the REST API a robot account is automatically<!-- Avoid if not definitely useful to mention. --> provisioned. The login credentials and details of the OAuth client endpoint used for authentication, can be<!-- Consider switching to active voice. --> obtained by<!-- Active voice is preferred. --> querying the resource definition for the training portal after it has been<!-- Consider changing to |is| or |has| or rewrite for active voice. --> created and the deployment completed. If using ``kubectl describe``, you would use:

```
kubectl describe trainingportal.training.eduk8s.io/<training-portal-name>
```

In the status section of the output you will see:

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
        Username:  educates
      Robot:
        Password:  QrnY67ME9yGasNhq2OTbgWA4RzipUvo5
        Username:  robot@educates
``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->

The admin login credentials is what you would<!-- Re-phrase for present tense if possible. --> use if logging into the training portal web interface to access admin pages.

The robot login credentials is what would<!-- Re-phrase for present tense if possible. --> be used if wish<!-- |want| is preferred. --> to access the REST API.

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Requesting an access token

Before you can make requests against the REST API to query details on<!-- |details about| is preferred. --> workshops or request a workshop session, you need to<!-- Consider replacing with just |To|. --><!-- |must| is preferred. --> login via<!-- |through|, |using| and |by means of| are preferred. --> the REST API to get an access token.

This would<!-- Re-phrase for present tense if possible. --> be done from any front end web application or provisioning system, but the step is equivalent to making a REST API call using ``curl`` of:

```
curl -v -X POST -H \
"Content-Type: application/x-www-form-urlencoded" \
-d "grant_type=password&username=robot@educates&password=<robot-password>" \
-u "<robot-client-id>:<robot-client-secret>" \
<training-portal-url>/oauth2/token/
``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->

The URL sub path is ``/oauth2/token/``.

Upon success, the output will be a JSON response consisting of:

```
{
    "access_token": "tg31ied56fOo4axuhuZLHj5JpUYCEL",
    "expires_in": 36000,
    "token_type": "Bearer",
    "scope": "user:info",
    "refresh_token": "1ryXhXbNA9RsTRuCE8fDAyZToJmp30"
}
``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->

##  <!-- Add an anchor ID unless you have reason not to: |<a id="NAME"></a>| -->Refreshing the access token

The access token which is provided will expire and will need to be refreshed before it expires if being used by a long running application.

To refresh the access token you would use the equivalent of:

```
curl -v -X POST -H \
"Content-Type: application/x-www-form-urlencoded" \
-d "grant_type=refresh_token&refresh_token=<refresh-token>& \client_id=<robot-client-id>&client_secret=<robot-client-secret>" \
https://lab-markdown-sample-ui.test/oauth2/token/
``` <!-- Define any non-obvious placeholders present in the code snippet in the style of |Where PLACEHOLDER is...| -->

As with requesting the initial access token, the URL sub path is ``/oauth2/token/``.

The JSON response will<!-- Avoid |will|: present tense is preferred. --> be of the same format as if a new token had been<!-- Consider replacing with |was| or shifting to present tense. --> requested.
