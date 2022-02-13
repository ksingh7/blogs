
# Tired of logging regularly to your K8s/OpenShift environments ? You gotta read this my friend

A little hack to leverage Service Accounts to save sometime

![Photo by [Micah Williams](https://unsplash.com/@mr_williams_photography?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](w)*Photo by [Micah Williams](https://unsplash.com/@mr_williams_photography?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

If you are like me, you must be having multiple kubernetes environments that you use for your dev/test/prod & hacking purpose. And you need to manage login to each of them separately. So read on !

## Why do i need to Login from the CLI ? Again and Again ?

**TL;DR**

Because user tokens gets expire after a day :(

**What happens in the background**

When you login from the command line as a user, a new session is created. A token corresponding to the session will be cached within your local account. When you run the oc / kubectlcommand line tool, that token will be supplied with each request made to OpenShift / K8s.

In a typical OpenShift environment these session tokens will expire after one day. The expiry time will differ if the administrator of the OpenShift environment has overridden the default value used. But generally its 24 hours. Once the session token expires, to use the oc command line tool, you will need to login again, and again, and again.

Because the login session will expire without notice, you can avoid using the session token for a normal user account and instead use a service account. The service account has an associated access token which you can use and which will not expire.

## Service Accounts

When logging in to a Kubernetes cluster from an automated environment, it is recommended to use a functional Service Account rather than personal credentials

The advantages SA are:

* Service Account tokens do not expire, resolving the problem of constantly needing to log in.

* Developers working on workflow automation do not have to worry about using or securing personal credentials.

* Service Account tokens can be trivially revoked without disrupting a specific user.

## Configuring SA to be used for Regular Logins
> **Caveat :** What you are going to read next is not a recommended practice, Its a hack to avoid logins. Don’t try this with your production clusters.

**Step-1** :: Login to oc or kubectl CLI via the regular way of providing credentials or user token. From OpenShift Console > Top Right User > Copy Login Command

![](https://cdn-images-1.medium.com/max/2500/1*JBdRtZXb-Uac4aLl-E_z4A.png)

**Step-2** :: Create Service Account and grab its secret and encode the token from the secret

From bash or sh shell or script file, run the following commands
> **Note :** Make sure you are in the correct OpenShift project or K8s Namespace before you run these commands. Service Accounts are specific to a project. Depending on your use-case you might need to repeat these steps for other projects
```
    # **Create a new service account for the CLI access
    **export SA=$(oc project -q)-cli-sa
    oc create sa $SA

    # **Find the name of the secret in which the Service Account's apiserver token is stored**

    export SECRETS=$(oc get sa $SA -o jsonpath='{.secrets[*].name}{"\n"}') ;

    # **Select the one with "token" in the name - the other is for the container registry
    **export SECRET_NAME=$(printf "%s\n" $SECRETS | grep "token") ;

    # **Get the encoded token from the secret
    **export ENCODED_TOKEN=$(oc get secret $SECRET_NAME -o jsonpath='{.data.token}{"\n"}') ;

    # **Finally decode the token
    **export OPENSHIFT_TOKEN=$(echo $ENCODED_TOKEN | base64 -d) ;
```
Print your SA Token. The Service Account Token will be a JWT. You can use [this tool](https://jwt.io/) to test your token is valid. JWTs always start with eyJhb.
```
    echo $OPENSHIFT_TOKEN
```
![](https://cdn-images-1.medium.com/max/3826/1*KbbezlMbC037Lfq88auxvw.png)

![](https://cdn-images-1.medium.com/max/2000/1*mos2S-cFoU__jjiu0BQEmg.png)

**Step-3** :: Next you need give your Service Account permissions to make changes to your cluster. By default, it has no permissions. You will need permission to edit rolebindings in your namespace in order to edit the Service Account’s permissions.

If you receive permissions errors when running oc policy commands, ask your cluster administrator for help.

Use the following command to allow your Service Account to have user-level edit permissions:
```
    oc policy add-role-to-user edit -z $SA ;
```
With the necessary permissions configured, you can now use the Service Account in your workflows or for CLI access.

**Step-4** :: Login to CLI using Service Account token (that never expires)
```
    export OPENSHIFT_SERVER_URL=$(oc whoami --show-server) ;

    oc login --server=$OPENSHIFT_SERVER_URL --token=$OPENSHIFT_TOKEN

    oc whoami
    oc project -q
```
![](https://cdn-images-1.medium.com/max/2184/1*LRbQ67LJu3LBfOmk2bgy9Q.png)
> **Summary** : So now you no longer need to login again to your OpenShift environment under training project. Remember, SA is a namespace/project scoped resource. You would still need to login for other project. However you can follow the same procedure for other projects. And as soon as you login with service account tokens, your local kubeconfig file will get updated with non-expiring tokens, and you would not ask to login again for all the projects for which you have done this setup.

**Tl ; dr Notes**
```
    bash
    # **Create a new service account for the CLI access
    **export SA=$(oc project -q)-cli-sa
    oc create sa $SA

    # **Find the name of the secret in which the Service Account's apiserver token is stored
    **export SECRETS=$(oc get sa $SA -o jsonpath='{.secrets[*].name}{"\n"}') ;

    # **Select the one with "token" in the name - the other is for the container registry
    **export SECRET_NAME=$(printf "%s\n" $SECRETS | grep "token") ;

    # **Get the encoded token from the secret
    **export ENCODED_TOKEN=$(oc get secret $SECRET_NAME -o jsonpath='{.data.token}{"\n"}') ;

    # **Finally decode the token
    **export OPENSHIFT_TOKEN=$(echo $ENCODED_TOKEN | base64 -d) ;

    oc policy add-role-to-user edit -z $SA ;

    export OPENSHIFT_SERVER_URL=$(oc whoami --show-server) ;
    oc login --server=$OPENSHIFT_SERVER_URL --token=$OPENSHIFT_TOKEN
    oc whoami
```