# Tapis Jobs

### Tapis Jobs service
Tapis Job service aims at launching applications directly on hosts or as job submitted to schedulers (currently only Slurm).
The Tapis v3 Jobs service is specialized to run containerized applications on any host that supports container runtimes.
Currently, Docker and Singularity containers are supported. The Jobs service uses the Systems, Apps, Files and Security Kernel services to process jobs.


### Life cycle of Jobs
When a job request is recieved as the payload of an POST call, the following steps are taken:

* **Request authorization** - The tenant, owner, and user values from the request and Tapis JWT are used to authorize access to the application, execution system and, if specified, archive system.
* **Request validation** - Request values are checked for missing, conflicting or improper values; all paths are assigned; required paths are created on the execution system; and macro substitution is performed to finalize all job parameters.
* **Job creation** - A Tapis job object is written to the database.
* **Job queuing** - The Tapis job is queue on an internal queue serviced by one or more Job Worker processes.
* **Response** - The initial Job object is sent back to the caller in the response. This ends the synchronous portion of job submission.

After these synchronous steps job processing proceeds asynchronously. Each job is assigned a worker thread and job proceeds until it completes successfully, fails or gets blocked.


### Job Status
**PENDING** - Job processing beginning <br/>
**PROCESSING_INPUTS** - Identifying input files for staging <br/>
**STAGING_INPUTS** - Transferring job input data to execution system <br/>
**STAGING_JOB** - Staging runtime assets to execution system <br/>
**SUBMITTING_JOB** - Submitting job to execution system <br/>
**QUEUED** - Job queued to execution system queue <br/>
**RUNNING** - Job running on execution system <br/>
**ARCHIVING** - Transferring job output to archive system <br/>
**BLOCKED** - Job blocked <br/>
**PAUSED** - Job processing suspended <br/>
**FINISHED** - Job completed successfully <br/>
**CANCELLED** - Job execution intentionally stopped <br/>
**FAILED** - Job failed <br/>


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
* **execSystemId** - Tapis execution system ID. It can be inherited from the app
* **parameterSet**	-	Runtime parameters organized by category
 <br/>
**appId**, **name** and **appVersion** are required parameters.

Please refer to all the job submission parameters here [Job Submission Parameters](https://tapis.readthedocs.io/en/latest/technical/jobs.html#the-job-submission-request)


### Exercise: Running Image Classifier app on VM
When the Image classifier app runs it will execute a Singularity run command. To launch a container, the Jobs service will SSH to the target host and issue a command using this template: <br/>

```
singularity run [singularity options.] <image id> [application arguments] > tapisjob.out 2>&1 &

```

Tapis jobs service will add environment variables such as application dir, execution directories, app version, etc. in the singularity options. <br/>
image id for this job is docker://tapis/img-classify:0.1 <br/>
Application arguments are as defined below. <br/>
Output of job is written to tapisjob.out file under the jobs working directory on the execution system. <br/>

### Application Arguments
With appArgs parameter you  can specify one or more command line arguments for the user application. <br/>
Arguments specified in the application definition are appended to those in the submission request. Metadata can be attached to any argument.<br/>

Image classifier app needs two arguments:
* --image-file
* url of an image to be classified.

We will provide these app arguments in the job definition. In the arg2 pass a url of the image you would like to classify.

```
pa = {
 "parameterSet": {
      "appArgs": [
          {"arg": "--image_file", "meta": { "name": "arg1", "required": True}},
          {"arg": "'https://s3.amazonaws.com/cdn-origin-etr.akc.org/wp-content/uploads/2017/11/12231410/Labrador-Retriever-On-White-01.jpg'",
           "meta": {"name": "arg2", "required": True}
          }
      ]
}
}
```

### Submit a job on VM

```
job_response_vm=client.jobs.submitJob(name='img-classifier-job-vm',description='image classifier',appId=app_id,execSystemId=system_id_vm,appVersion= '0.0.1',
  **pa)
print(job_output_vm.uuid)

```

Everytime a job is submitted, a unique job id (uuid) is generated. We will use this job id with tapipy to get the job status, and download the job output.

```
# Get job uuid from the job submission response
print("****************************************************")
job_uuid_vm=job_response_vm.uuid
print("Job UUID: " + job_uuid_vm)
print("****************************************************")

```

### Jobs List
Now, when you do a jobs-list now, you can see your jobUuid.

```
client.jobs.getJobList()

```

### Jobs Status
Job status allows you to see the current state of the job.

```
# Check the status of the job
print("****************************************************")
print(client.jobs.getJobStatus(jobUuid=job_uuid_vm))
print("****************************************************")

```
Job enters into different states throughout the execution. Details about different job states are given here [JOB STATES](https://tapis.readthedocs.io/en/latest/technical/jobs.html#job-status)


### Jobs Output
To download the output of job you need to give it jobUuid and output path. Output path is

```
# Download output of the job
print("Job Output file:")

print("****************************************************")
jobs_output_vm= client.jobs.getJobOutputDownload(jobUuid=job_uuid_vm,outputPath='tapisjob.out')
print(jobs_output_vm)
print("****************************************************")
```
We will soon show you how to analyze the results. Before that lets try to submit a job on HPC machine.

## Submit job on HPC
Tapis supports porting the app from a virtual machine to HPC. You can run the same app on Stampede2 today by chaning the exec-system name registered on HPC in the job submission request.output

```
# Run Image classifier app on the HPC Machine
# In the arg pass a url of the image you would like to classify
pa = {
 "parameterSet": {
      "appArgs": [
          {"arg": "--image_file"},
          {"arg": "'https://s3.amazonaws.com/cdn-origin-etr.akc.org/wp-content/uploads/2017/11/12231410/Labrador-Retriever-On-White-01.jpg'"}

      ],
      "schedulerOptions": [
        {"arg": "--tapis-profile tacc"}

      ]
}
}
# Submit a hpc job
job_response_hpc=client.jobs.submitJob(name='img-classifier-job-hpc',description='image classifier',appId=app_id,execSystemId=system_id_hpc,appVersion= '0.0.1',
  **pa)

```
##  Get the Job Status

```
# Check the status of the job
print("****************************************************")
job_uuid_hpc=job_response_hpc.uuid
print(client.jobs.getJobStatus(jobUuid=job_uuid_hpc))
print("****************************************************")


```

## Download job output

```

# Download output of the job
print("Job Output file:")

print("****************************************************")
jobs_output_hpc= client.jobs.getJobOutputDownload(jobUuid=job_uuid_hpc,outputPath='tapisjob.out')
print(jobs_output_hpc)
print("****************************************************")
```


### Analyzing Jobs Output
With the code below, you can extract the image classifier output, which returns 5 prediction scores

```
print ("==============Image Classifier Scores ============================")
s = jobs_output_vm.split(b'\n')
# If you want to analyze the results of hpc output uncomment the line below and comment line above
# s = jobs_output_hpc.split(b'\n')
s.reverse()
scores=[]
for i in range(1,6):
    scores.append(s[i])
    print (s[i])

```

You should see the result as below:

```
==============Image Classifier Scores ============================
b'Saint Bernard, St Bernard (score = 0.00067)'
b'bull mastiff (score = 0.00095)'
b'kuvasz (score = 0.00099)'
b'golden retriever (score = 0.00324)'
b'Labrador retriever (score = 0.97471)'
```

### Sharing Results by sharing Tapis System and Jobs output

Tapis allows sharing your systems and output files with collaborators in the same tenant. Only the system owner may grant or revoke permissions on a storage system. <br/>
For example, if you want to grant a user train301 READ permissions on your system. You can run following command:

```
client.systems.grantUserPerms(systemId=system_id_vm,userName='train301',permissions=['READ'])

```
More on System [permissions]( https://tapis.readthedocs.io/en/latest/technical/systems.html#permissions)


Next, we will share the output file with another user.output
```
client.files.grantPermissions(systemId=system_id_vm, path='tapisjob.out', username='train301', permission='READ')

```



## What's next?

If you made it this far, you have successfully created a new app within a container and have deployed that tool on an HPC system, and now you can run that tool through the cloud from anywhere!  That is quite a lot in one workshop.

At this point, it would be a good idea to connect with other developers that are publishing apps and running workflows through Tapis by joining the Tapis API Slack channel: [tacc-cloud.slack.com](https://bit.ly/2XHYJEk)

[BACK](https://tacc-cloud.github.io/gateways21-portable-computing-cloud-hpc/)


