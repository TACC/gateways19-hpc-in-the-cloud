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

Each system is of a specific type and owned by a specific user who has special privileges for the system. The system
definition also includes the user that is used to access the system, referred to as *effectiveUserId*. This access user
can be a specific user (such as a service account) or dynamically specified as ``${apiUserId}`` in which case the
username is extracted from the identity associated with the request to the service.

At a high level a system represents the following information:

* **id** - A short descriptive name for the system that is unique within the tenant.
* **description** - An optional more verbose description for the system.
* **systemType** - Type of system. LINUX, S3, IRODS or GLOBUS
* **owner** - A specific user set at system creation. By default this is ``${apiUserId}``, the user making the request to
              create the system.
* **host** Host name or IP address.
* **effectiveUserId** - The username to use when accessing the system. A specific user (such as a service account) or the dynamic user ``${apiUserId}``
* **bucketName** For an S3 system this is the name of the bucket.
* **rootDir** - Effective root directory. Directory to be used when listing files or moving files to and from the system.
* **canExec** - Flag indicating if the system can be used to execute jobs.
* **job execution attributes** - Various attributes related to job execution such as *jobRuntimes*, *jobWorkingDir*, etc.

Note that a system may be created as storage-only resource (*canExec=false*) or as a system that can be used for both
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

where **<userid>** is replaced with your username. Note that although it is possible we have not provided any login
credentials in the system definition. For security reasons, it is recommended that login credentials be updated
using a separate API call as discussed below.

Using PySDK:

``` python
 import json
 from tapipy.tapis import Tapis
 t = Tapis(base_url='https://tacc.tapis.io', username='<userid>', password='************')
 with open('system_s3.json', 'r') as openfile:
     my_s3_system = json.load(openfile)
 t.systems.createSystem(**my_s3_system)
```

Using CURL::

   $ curl -X POST -H "content-type: application/json" -H "X-Tapis-Token: $JWT" https://tacc.tapis.io/v3/systems -d @system_s3.json

### Viewing Systems

To retrieve details for a specific system, such as the one above:

Using PySDK:

.. code-block:: python

 t.systems.getSystem(systemId='tacc-bucket-sample-<userid>')

Using CURL::

 $ curl -H "X-Tapis-Token: $JWT" https://tacc.tapis.io/v3/systems/tacc-bucket-sample-<userid>

The response should look similar to the following::

 {
    "result": {
        "tenant": "dev",
        "id": "tacc-bucket-sample-<userid>",
        "description": "My Bucket",
        "systemType": "S3",
        "owner": "<userid>",
        "host": "tapis-sample-test-<userid>.s3.us-east-1.amazonaws.com",
        "enabled": true,
        "effectiveUserId": "<userid>",
        "defaultAuthnMethod": "ACCESS_KEY",
        "authnCredential": null,
        "bucketName": "tapis-tacc-bucket-<userid>",
        "rootDir": "/",
        "port": 9000,
        "useProxy": false,
        "proxyHost": "",
        "proxyPort": -1,
        "dtnSystemId": null,
        "dtnMountPoint": null,
        "dtnMountSourcePath": null,
        "isDtn": false,
        "canExec": false,
        "jobRuntimes": [],
        "jobWorkingDir": null,
        "jobEnvVariables": [],
        "jobMaxJobs": 2147483647,
        "jobMaxJobsPerUser": 2147483647,
        "jobIsBatch": false,
        "batchScheduler": null,
        "batchLogicalQueues": [],
        "batchDefaultLogicalQueue": null,
        "jobCapabilities": [],
        "tags": [],
        "notes": {},
        "uuid": "f83606bf-7a1a-4ff0-9953-dd732cc07ac0",
        "deleted": false,
        "created": "2021-04-26T18:45:40.771Z",
        "updated": "2021-04-26T18:45:40.771Z"
    },
    "status": "success",
    "message": "TAPIS_FOUND System found: tacc-bucket-sample-<userid>",
    "version": "0.0.1",
    "metadata": null
 }

Note that authnCredential is *null*. Only specific Tapis services are authorized to retrieve credentials.




### Tapis Storage Systems

Storage systems tell Tapis where data resides.  You can store files for running compute jobs, archive results, share files with collaborators, and maintain copies of your Tapis apps on storage systems.  Tapis supports many of the communication protocols and  permissions models that go along with them, so you can work privately, collaborate with individuals, or provide an open community resource.  It's up to you.  Here is an example of a simple data storage system template accessed via SFTP for the TACC Corral cloud storage system:
```json
{
  "id": "UPDATEUSERNAME.tacc.corral.storage",
  "name": "Storage system for TACC cloud storage on corral",
  "status": "UP",
  "type": "STORAGE",
  "description": "Storage system for TACC cloud storage on corral",
  "site": "www.tacc.utexas.edu",
  "public": false,
  "default": true,
  "storage": {
    "host": "cloud.corral.tacc.utexas.edu",
    "port": 22,
    "protocol": "SFTP",
    "rootDir": "/",
    "homeDir": "/home/UPDATEUSERNAME/",
    "auth": {
      "username": "UPDATEUSERNAME",
      "password": "UPDATEPASSWORD",
      "type": "PASSWORD"
    }
  }
}
```

* **id** -This needs to be a unqiue identifier amongst all systems in Tapis - so using your username helps ensure this.
* **name** - This can be whatever you like, but should be descriptive for you.
* **status** - This is used when querying systems and can give other users an idea if the system is UP or DOWN- only sytems that are UP can be accessed.
* **type** - A system can be STORAGE or EXECUTION.
* **site** - A url typically with information about the system.
* **host** -  This is the ip or domain of the server we need to connect to
* **port** -  This is the port we need to use when connecting, this is usally tied to the proctocal (SFTP is usually port 22)
* **protocol** - This is the communication protocol most systems use SFTP but others are supported.
* **rootDir** - This is the lowest directory any Tapis user accessing this system can navigate.
* **homeDir** - This is the directory that a Tapis user will access by default.
* **auth** - The Authenication type to use when accessing the system - in this tutorial we are using a PASSWORD Auth but SSH-KEYS is usually recommended.
* **public** - Is this a shared resource available to all users - only Administrators can set this to TRUE.
* **default** - TRUE or FALSE if this is the default system for Tapis to use when not explicitly passed a system.

More details on the possible parameters for storage systems can be found in the [Tapis Storage System documentation](https://tacc-cloud.readthedocs.io/projects/agave/en/latest/agave/guides/systems/systems-storage.html).

### Hands-on (We have already provisioned this storage system for the workshop on your behalf. Skip for the workshop.)

As a hands on exercise, using the Tapis CLI, register a data storage system using PASSWORD authentication with the above template for the TACC Corral Cloud store. Don't forget to replace *UPDATEUSERNAME* and *UPDATE PASSWORD*.  Call the JSON file "cloud_corral.json"

Then the CLI command to use is:
```
systems-addupdate -F cloud_corral.json
```

The above command will submit the JSON file "cloud_corral.json" to Tapis and create a new system with the attributes specified in the JSON file.

You can now see the you new system by running the following Tapis CLI command:
```
systems-list
```

---
### Tapis Execution Systems

Execution systems in Tapis are very similar to storage systems.  They just have additional information for how to launch jobs.  In this example, we are using the Stampede2 HPC system, so we have to give scheduler and queue information.  This system description is longer than the storage definition due to logins, queues, scratch systems definitions.

```json
{
  "id": "UPDATEUSERNAME.stampede2.execution",
  "name": "Execution system for Stampede2",
  "status": "UP",
  "type": "EXECUTION",
  "description": "Execution system for Stampede2 ",
  "site": "www.tacc.utexas.edu",
  "executionType": "HPC",
  "scratchDir": "/home1/0003/UPDATEUSERNAME/scratch",
  "workDir": "/home1/0003/UPDATEUSERNAME/work",
  "login": {
    "host": "login1.stampede2.tacc.utexas.edu",
    "port": 22,
    "protocol": "SSH",
    "scratchDir": "/home1/0003/UPDATEUSERNAME/scratch",
    "workDir": "/home1/0003/UPDATEUSERNAME/work",
    "auth": {
      "username": "UPDATEUSERNAME",
      "password": "UPDATEPASSWORD",
      "type": "PASSWORD"
    }
  },
  "storage": {
    "host": "login1.stampede2.tacc.utexas.edu",
    "port": 22,
    "protocol": "SFTP",
    "rootDir": "/",
    "homeDir": "/home1/0003/UPDATEUSERNAME",
    "auth": {
     "username": "UPDATEUSERNAME",
      "password": "UPDATEPASSWORD",
      "type": "PASSWORD"
    }
  },
  "maxSystemJobs": 100,
  "maxSystemJobsPerUser": 10,
  "scheduler": "SLURM",
  "queues": [
    {
      "name": "normal",
      "maxJobs": 100,
      "maxUserJobs": 10,
      "maxNodes": 128,
      "maxMemoryPerNode": "2GB",
      "maxProcessorsPerNode": 128,
      "maxRequestedTime": "24:00:00",
      "customDirectives":"-A UPDATEPROJECT -r UPDATERESERVATION",
      "default": true
    }
  ],
  "environment": "",
  "startupScript": null
}
```

We covered what some of these keywords are in the storage systems section.  Below is some commentary on the new fields:

* **executionType** - Either HPC, Condor, or CLI.  Specifies how jobs should go into the system. HPC and Condor will leverage a batch scheduler. CLI will fork processes.
* **scheduler** - For HPC or CONDOR systems, Agave is "scheduler aware" and can use most popular schedulers to launch jobs on the system.  This field can be LSF, LOADLEVELER, PBS, SGE, CONDOR, FORK, COBALT, TORQUE, MOAB, SLURM, UNKNOWN. The type of batch scheduler available on the system.
* **environment** - List of key-value pairs that will be added to the Linux shell environment prior to execution of any command.
* **scratchDir** - Whenever Agave runs a job, it uses a temporary directory to cache any app assets or job data it needs to run the job.  This job directory will exist under the "scratchDir" that you set.  The path in this field will be resolved relative to the rootDir value in the storage config if it begins with a "/", and relative to the system homeDir otherwise.
* **workDir** - Path to use for a job working directory. This value will be used if no scratchDir is given. The path will be resolved relative to the rootDir value in the storage config if it begins with a "/", and relative to the system homeDir otherwise.
* **queue** - An array of batch queue definitions providing descriptive and quota information about the queues you want to expose on your system. If not specified, no other system queues will be available to jobs submitted using this system.
* **startupScript** - Path to a script that will be run prior to execution of any command on this system. The path will be a standard path on the remote system. A limited set of system macros are supported in this field. They are rootDir, homeDir, systemId, workDir, and homeDir. The standard set of runtime job attributes are also supported. Between the two set of macros, you should be able to construct distinct paths per job, user, and app. Any environment variables defined in the system description will be added after this script is sourced. If this script fails, output will be logged to the .agave.log file in your job directory. Job submission will still continue regardless of the exit code of the script.

Complete reference information is located here:
[https://tacc-cloud.readthedocs.io/projects/agave/en/latest/agave/guides/systems/introduction.html]

### Hands-on (We have already provisioned this execution system for the workshop on your behalf. Skip for the workshop.)

As a hands on exercise, register the Stampede2 HPC as a execution system using the Tapis-CLI using the above JSON template. - Don't forget to change *UPDATEUSERNAME* and *UPDATEPASSWORD* to your tutorial or TACC username and *UPDATEPROJECT* and *UPDATERESERVATION* for this workshops Stampede2 provided project and reservation (or your personal ones if doing this on your own).  

In your CLI you can now get a list of your systems using:
```
systems-list
```

If you want to view just the storage systems you can use -S. For execution systems use -E.
