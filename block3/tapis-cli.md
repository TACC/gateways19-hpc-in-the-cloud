
Tutorial: Getting Started with Tapis CLI
===============================================
The following instructions will guide you through setting up Tapis CLI.  As an aside, everything we do today can also be accomplished from a command line interface or by directly calling API endpoints.  

The Tapis CLI commands all respond with help for -h and return back information on the parameters that can be passed.  

Need help?  Ask your questions using the [TACC Cloud Slack Channel](https://bit.ly/join-tapis)

Initial Requirements
===============================================

Before getting started, you need to have the following:
* A TACC Account - today you have a test account
* SSH access to the Stampede 2 compute cluster and an allocation.
* Familiarity with [editing text files](https://www.nano-editor.org/dist/v2.7/nano.html) and [working at the command line](http://www.gnu.org/software/bash/manual/bashref.html#Introduction)

Any questions?  Join the [TACC CLOUD SLACK CHANNEL](https://bit.ly/2XHYJEk) and ask away.


Command Line Access
===================

We won't install it in this workshop (since it is already installed on the VM), but everything we do today can also be done from the standard shell using the Tapis CLI tools.  

Open a terminal in your Jupyter instance and that is what we will use to run the CLI commands.

Jump to the [Authentication](#authentication) section for the workshop.

Installing the Tapis CLI Tools (Skip for Workshop- this is at home )
------------------------------

Tapis has a downloadable set of command line tools that make it easier to work with the API from the shell. Using these scripts is generally easier than hand-crafting cURL commands, but if you prefer that route, consult the [Tapis API Documentation](https://tacc-cloud.readthedocs.io/en/latest/). We include these scripts in the training virtual machines and supplement them with additional support scripts, example files, and documents.

During the course, we will use the Jetstream Cloud virtual machines, but if you have a shell on your personal computer, you can install these tools on your own later.

To access the CLI for this tutorial you can open a Terminal  in Jupyter which give you access to the shell in the Jetstream VM, OR *ssh* into the system from your own terminal:

```ssh ubunut@jetstreamVM_ip_address```

Install the CLI tools (Skip for Workshop- this is at home )
----------------------------------

The CLI tools and instructions for installation can be found in the [CLI repository](https://github.com/TACC-Cloud/agave-cli)


Authentication
----------------

Tapis has robust Authentication/Authorization pathways - we could easliy spend an hour or more discussing them, but will keep our focus simple for this tutorial.

The Tapis API uses OAuth 2 for managing authentication and authorization. OAuth 2 is an open standard for access delegation, commonly used as a way for Internet users to grant websites or applications access to their information on other websites but without giving them the passwords.

Just understand that instead of passing a username and password every time we want to make an authenticated/authorized request to the Tapis APIs we will be using an Access Token that has a defined expiration - this keeps our credentials safe and ensures that if someone were to obtain the token it could not be used forever.

Run the following in the CLI
```
>auth-check
Please run /agave-cli/bin/tenants-init to initialize your client before attempting to interact with the APIs.
```
We will see that we have to initialize some things before we can use Tapis.

Initialize the CLI
------------------

The first time you install the CLI tools on a computer, you need to initialize it.
You can initialize the TACC tenant by runnning:

```
>auth-session-init
Client does not exist. Creating one...
Name was not specified. Using *really-first-orca*
Tapis Username:
ID                   NAME                                     URL
3dem                 3dem Tenant                              https://api.3dem.org/
agave.prod           Agave Public Tenant                      https://public.agaveapi.co/
araport.org          Araport                                  https://api.araport.org/
bridge               Bridge                                   https://api.bridge.tacc.cloud/
designsafe           DesignSafe                               https://agave.designsafe-ci.org/
iplantc.org          CyVerse Science APIs                     https://agave.iplantc.org/
irec                 iReceptor                                https://irec.tenants.prod.tacc.cloud/
portals              Portals Tenant                           https://portals-api.tacc.utexas.edu/
sd2e                 SD2E Tenant                              https://api.sd2e.org/
sgci                 Science Gateways Community Institute     https://sgci.tacc.cloud/
tacc.prod            TACC                                     https://api.tacc.utexas.edu/
vdjserver.org        VDJ Server                               https://vdj-agave-api.tacc.utexas.edu/

Please specify the ID for the tenant you wish to interact with: tacc.prod
Tapis Password:
Client *really-first-orca* created
Creating an access token for really-first-orca...
b9346eea62918ebebb2c0eb9d6265
```
Select the 'tacc.prod' tenant and then use the username and password provided for this tutorial.

The 'auth-session-init' command creates a Tapis client and then request an API token and will then place the TACC tenant,client and API token information into a cache in ~/.agave/current. This is the file that the CLI tools will look for when making API calls so that you don't have to enter those parameters for every call.


Creating a Client 
----------------
The Tapis API uses OAuth 2 for managing authentication and authorization. Before you work with Tapis, you must create an OAuth client application and record the API keys that are returned. This is a one-time action per machine that you use the CLI on and the 'auth-session-init' can take care of this.  In the event you need to create your own client you can pass additional parameters to the 'auth-session-init' command.  For instance if we want to make a new client.

```
>auth-session-init -N myclient
Loading myclient for train351 from tacc.prod
Unable to load client from session cache
Client does not exist. Creating one...
Tapis Password for train351:
Client *myclient* created
Creating an access token for myclient...
b2219eaceca1195469a0adcf1e6d6177
```

*Note:* The -N flag allows you to specify a human-readable name for your client. 

You will need access to the ```consumerKey``` and ```consumerSecret``` values when setting up on other hosts to use that client. "auth-check -v" will provide the current client key and secret in addition to your api token and refresh token.
```
>auth-check -v
{
  "access_token": "b2219eaceca1195469a0adcf1e6d6177",
  "apikey": "DWc_In83HUo342qbLHxyUAJWnS4a",
  "apisecret": "gmh7nmEEaEgQDMYpkQ6zR6Awjsca",
  "baseurl": "https://api.tacc.utexas.edu",
  "created_at": "1568816585",
  "devurl": "",
  "expires_at": "Wed Sep 18 18:23:05 UTC 2019",
  "expires_in": 14400,
  "refresh_token": "11c09d65f7697d0c73d7248b5824d58",
  "tenantid": "tacc.prod",
  "username": "train351"
}
```

So, please take a moment and record *client_name*, *consumerKey*, and *consumerSecret* somewhere safe. If you lose these values, you can create a new instance of the client by deleting the old client (clients-delete CLIENT_NAME) and creating it again (or create a new client with a different name).

 OAuth 2 API authentication token
----------------

Tokens are a form of short-lived, temporary authenticiation and authorization used in place of your username and password. To interact with Tapis, you will need to acquire one. Each Tapis token, typically, expires after 4 hours, but can easily be refreshed.

On a host where you have configured a Tapis OAuth2 client already, the CLI command to get a new token is:

```
> ubuntu@abaco:~$ auth-session-init
Loading None for train351 from tacc.prod
Creating an access token for really-first-orca...
Tapis Password:
cesseea62918ebebb2c0eb9d6iec
```
You will then be prompted to enter your *Tapis Password*. **Type your user password**.  You should receive an affirmation of success in your terminal that resembles the one above.

## Refreshing your token

This tutorial won't take very long, but if you are interrupted and come back later, you might find your token has expired. You can always refresh a token as follows:

```> auth-session-init```

This topic is covered in great detail at the Tapis [Authorization Guide](https://tacc-cloud.readthedocs.io/projects/agave/en/latest/agave/guides/authorization/introduction.html)

NOTE that most CLI commands will attempt to do a token refresh on your behalf if the access token is expired.

## Command Help

Note that all the CLI commands take the '-h' flag to display a short description and the accept parameters for the command.

