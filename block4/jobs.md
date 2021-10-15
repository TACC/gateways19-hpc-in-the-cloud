# Tapis Jobs

### Tapis Jobs service
Tapis Job service aims at launching applications directly on hosts or as job submitted to schedulers (currently only Slurm).
The Tapis v3 Jobs service is specialized to run containerized applications on any host that supports container runtimes.
Currently, Docker and Singularity containers are supported. The Jobs service uses the Systems, Apps, Files and Security Kernel services to process jobs.


### Life cycle of Jobs
When a job request is recieved as the payload of an POST call, the following steps are taken:

* Request authorization - The tenant, owner, and user values from the request and Tapis JWT are used to authorize access to the application, execution system and, if specified, archive system.
* Request validation - Request values are checked for missing, conflicting or improper values; all paths are assigned; required paths are created on the execution system; and macro substitution is performed to finalize all job parameters.
* Job creation - A Tapis job object is written to the database.
* Job queuing - The Tapis job is queue on an internal queue serviced by one or more Job Worker processes.
* Response - The initial Job object is sent back to the caller in the response. This ends the synchronous portion of job submission.

After these synchronous steps job processing proceeds asynchronously. Each job is assigned a worker thread and job proceeds until it completes successfully, fails or gets blocked.


### Job Status
PENDING - Job processing beginning
PROCESSING_INPUTS - Identifying input files for staging
STAGING_INPUTS - Transferring job input data to execution system
STAGING_JOB - Staging runtime assets to execution system
SUBMITTING_JOB - Submitting job to execution system
QUEUED - Job queued to execution system queue
RUNNING - Job running on execution system
ARCHIVING - Transferring job output to archive system
BLOCKED - Job blocked
PAUSED - Job processing suspended
FINISHED - Job completed successfully
CANCELLED - Job execution intentionally stopped
FAILED - Job failed


Simple job submission example:
```
{
    "name": "myJob",
    "appId": "myApp",
    "appVersion": "1.0"

}
```
* **appId**	- The Tapis application to execute.  This must be a valid application that the user has permission to run.
* **name**	-  The user selected name for the job.
* **appVersion** - The version of the application to execute.
* execSystemId** - Tapis execution system ID. It can be inherited from the app
* **parameterSet**	-	Runtime parameters organized by category
 <br/>
**appId**, **name** and **appVersion** are required parameters.

Please refer to all the job submission parameters here [Job Submission Parameters](https://tapis.readthedocs.io/en/latest/technical/jobs.html#the-job-submission-request)


### Exercise: Submitting a Job
Once you have at least one app registered, you can start running jobs.   <br/>

Lets run our very first Image Classifier Tapis Job ! <br/>

### Step 1: Submit job

Run the job submission command in your notebook

```
client.jobs.submitJob(name='img-classifier-job',description='image classifier',appId='img-classifier-scblack',appVersion= '0.0.2')

```
Everytime you submit a job, a unique job id is created. You will use this job id with tapipy to get the Job Status, output listing and much more.


### Jobs List
Now, when you do a jobs-list you can see your job id


```
client.jobs.list
```

### Jobs Status
Job status allows you to see the current state of the job.

```

```

Job enters into different states throughout the execution. Details about different job states are given here [JOB STATES]()


### Jobs Output


```
jobs-output-list -L <jobId>
```

With this command, you can see the current files in the output folder. <br/>
When archive is true, all the new files will get copied to archive directory on your archive system. When it is false, all the output files can be found on the execution system's scratch directory

To retrieve the output **predictions.txt** file we will use the `jobs-output-get` command below:

```
jobs-output-get -r <jobId>
```

For example, if using the `train510` account with job uuid `8c7a91ac-7da5-44ad-a6dd-39f010e87e54-007`, the command would be:

```
jobs-output-get -r 8c7a91ac-7da5-44ad-a6dd-39f010e87e54-007
```

You should see a "jobs-jobId" folder created in your present working directoty, which contains the predictions.txt file along with .err and .log files.


### Jobs Notifications
You can monitor progress of your job by setting by email or webhook notifications
Add this to your job definition and try to submit the job again

```
"notifications":[
    {
      "url":"UPDATEEMAILADDRESS",
      "event":"*",
      "persistent":true
    }
    ]
```

You should see email notifications pop up in your inbox as the job changes state.

## What's next?

If you made it this far, you have successfully created a new app within a container and have deployed that tool on an HPC system, and now you can run that tool through the cloud from anywhere!  That is quite a lot in one workshop.

At this point, it would be a good idea to connect with other developers that are publishing apps and running workflows through Tapis by joining the Tapis API Slack channel: [tacc-cloud.slack.com](https://bit.ly/2XHYJEk)

[BACK](https://tacc.github.io/pearc19-hpc-in-the-cloud/)


