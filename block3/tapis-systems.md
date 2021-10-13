# Tapis Systems

Once you are authorized to make calls to the various services, one of first things you may want to do is view storage
and execution resources available to you or create your own. In Tapis a storage or execution resource is referred
to as a **system**.

## Overview
A Tapis system represents a server or collection of servers exposed through a single host name or IP address.
Each system is associated with a specific tenant. A system can be used for the following purposes:

* Running a job, including:

  * Staging files to a system in preparation for running a job.
  * Executing a job on a system.
  * Archiving files and data on a remote system after job execution.

* Storing and retrieving files and data.

Each system is of a specific type (such as LINUX or S3) and owned by a specific user who has special privileges for
the system. The system definition also includes the user that is used to access the system, referred to as
*effectiveUserId*. This access user can be a specific user (such as a service account) or dynamically specified as
``${apiUserId}``. For the case of ``${apiUserId}``, the username is extracted from the identity associated with the
request to the service.

At a high level a system represents the following information:

* **id** - A short descriptive name for the system that is unique within the tenant.
* **description** - An optional more verbose description for the system.
* **systemType** - Type of system. LINUX, S3, IRODS or GLOBUS
* **owner** - A specific user set at system creation. By default, this is ``${apiUserId}``, the user making the request to
              create the system.
* **host** Host name or IP address.
* **effectiveUserId** - The username to use when accessing the system. A specific user (such as a service account) or the dynamic user ``${apiUserId}``
* **bucketName** For an S3 system this is the name of the bucket.
* **rootDir** - Effective root directory. Directory to be used when listing files or moving files to and from the system.
* **canExec** - Flag indicating if the system can be used to execute jobs.
* **job execution attributes** - Various attributes related to job execution such as *jobRuntimes*, *jobWorkingDir*, etc.

Note that a system may be created as a storage-only resource (*canExec=false*) or as a system that can be used for both
execution and storage (*canExec=true*).

For more information about the Systems service please see [Tapis Systems Service documentation](https://tapis.readthedocs.io/en/latest/technical/systems.html).

## Getting Started

Here we review how to create a system and how to retrieve system details. In the examples below we assume you are using
the TACC tenant with a base URL of ``tacc.tapis.io`` and that you have authenticated using PySDK or obtained an
authorization token and stored it in the environment variable JWT.

### Creating a System

Create a local file named ``exec_system.json`` with json similar to the following::
``` json
{
  "id": "tapisv3-exec-<userid>",
  "description": "Tapis v3 execution system",
  "systemType": "LINUX",
  "host": "129.114.17.113",
  "effectiveUserId": "${apiUserId}",
  "defaultAuthnMethod": "PASSWORD",
  "rootDir": "/home/<userid>",
  "port": 22,
  "canExec": true,
  "jobRuntimes": [ { "runtimeType": "DOCKER" }, { "runtimeType": "SINGULARITY" } ],
  "jobWorkingDir": "workdir",
  "jobIsBatch": true,
  "batchScheduler": "SLURM",
  "batchLogicalQueues": [
    {
      "name": "tapisNormal",
      "hpcQueueName": "debug",
      "maxJobs": 50,
      "maxJobsPerUser": 10,
      "minNodeCount": 1,
      "maxNodeCount": 16,
      "minCoresPerNode": 1,
      "maxCoresPerNode": 68,
      "minMemoryMB": 1,
      "maxMemoryMB": 16384,
      "minMinutes": 1,
      "maxMinutes": 60
    }
  ],
  "batchDefaultLogicalQueue": "tapisNormal",
}
```

where ``<userid>`` is replaced with your username. Note that although it is possible, we have not provided any login
credentials in the system definition. For security reasons, it is recommended that login credentials be updated
using a separate API call as discussed below.

#### Using PySDK to register the system:
``` python
 import json
 from tapipy.tapis import Tapis
 t = Tapis(base_url='https://tacc.tapis.io', username='<userid>', password='************')
 with open('exec_system.json', 'r') as openfile:
     exec_system = json.load(openfile)
 t.systems.createSystem(**exec_system)
```

#### Using CURL to register the system:
```
   $ curl -X POST -H "content-type: application/json" -H "X-Tapis-Token: $JWT" https://tacc.tapis.io/v3/systems -d @exec_system.json
```

### Registering Credentials for a System
Now that you have registered a system you will need to register credentials so you can use Tapis to access the host.
Various authentication methods can be used to access a system, such as PASSWORD and PKI_KEYS. Here we will cover
registering a password.

#### Using PySDK to register the credential:
``` python
 t.systems.createUserCredential(systemId='tapisv3-exec-<userid>', userName='<userid>', password='<password>'))
```

#### Using CURL to register the credential:

Create a local file named ``cred_tmp.json`` with json similar to the following::
``` json
{
  "password": "<password>"
}
```

where ``<password>`` is replaced with your password for the host.

Run the CURL command:
```
   $ curl -X POST -H "content-type: application/json" -H "X-Tapis-Token: $JWT" https://tacc.tapis.io/v3/systems/credential/tapisv3-exec-<userid>/user/<userid> -d @cred_tmp.json
```

where ``<userid>`` is replaced with your username.


### Viewing Systems

To retrieve details for a specific system, such as the one above:

#### Using PySDK:
``` python
 t.systems.getSystem(systemId='tapisv3-exec-<userid>')
```

#### Using CURL:
```
 $ curl -H "X-Tapis-Token: $JWT" https://tacc.tapis.io/v3/systems/tapisv3-exec-<userid>
```

## Next Steps
Now that we have covered creating and viewing a system, we are now ready to create an application that can be run on
the system (or any other system).

 [Next-> Applications](../block4/apps.md)
