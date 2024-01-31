## AIJOB_v2 Job Definition
The `AIJOB_v2.ctmai` file is a base64 encoded job definition exported out of the BMC Control-M Application Integrator tool. It is based upon the original `AIJOB.ctmai` that was published in this project. It has been enhanced to improve the supportability of deploying and managing Mainframe Modernization Blu Age jobs within Control-M.

### Pre-requisites
This job definition requires BMC Control-M version 9.0.20.200 or above and relies upon [jq cli](https://jqlang.github.io/jq/) to be installed on the Control-M agent where the job is executing. Also recommend modifying the M2 application Logback configuration to be something similar to this example [logback.xml](logback.xml) config file. This configuration will ensure that the application log data forwarded to CloudWatch has the job name on each message. For more information about M2 Blu Age application configuration, please see the [AWS Mainframe Modernization User Guide](https://docs.aws.amazon.com/m2/latest/userguide/ba-app-config.html).

### Connection Profile
- `access_key_id` - AWS IAM access key to authenticate to the M2 API
- `secret_access_key` - AWS IAM secret key to authenticate to the M2 API
- `region` - AWS region to be used when calling the M2 API

### Job Properties
- `CON_PRO` - Connection profile that is to be used with this job
- `m2_app_name`- M2 Blu Age application name that this job is deployed within
- `job_name` - Name of the Blu Age batch job JCL groovy script to be executed <br>**Note:** Don't include the `.jcl.groovy` extension on the job name
- `job_params` - A JSON object with the key/value pairs to be used as job parameters during the batch job execution API call `{"key1": "value1","key2": "value2"}`<br>**Note:** The default value for this parameter is an empty JSON object `{}`; if you don't need to pass parameters, an empty JSON object `{}` is needed otherwise the M2 API call will fail due to malformed JSON
- `polling_frequency` - How often in seconds that Control-M will check the status of the batch job execution <br>**Note:** Recommend a minimum of 10 seconds or more between requests to ensure the job execution logs get forwarded to CloudWatch before the Fetch CloudWatch Logs step

### Step Details
The job definition contains seven different execution steps. The following are the details for each step.
- **Pre-Execution - Retrieve App ID** - Uses the M2 API [ListApplications](https://docs.aws.amazon.com/m2/latest/APIReference/API_ListApplications.html) with the `m2_app_name` to retrieve the `APP_ID` and check to see if the application is running
- **Execution - Log Failure** - Step will only execute if the pre-execution step fails; will log the failure to the job output log
- **Execution - Fetch Start Time** - Will get the `START_DATE` in EPOCH format to be used when querying CloudWatch for job execution logs
- **Execution - Start Batch Job** - Step will only execute if previous steps succeeded; uses the M2 API [StartBatchJob](https://docs.aws.amazon.com/m2/latest/APIReference/API_StartBatchJob.html) to start the batch job using `job_name` and will poll the job status using the M2 API [GetBatchJobExecution](https://docs.aws.amazon.com/m2/latest/APIReference/API_GetBatchJobExecution.html); will also provide a manual abort operation that uses the M2 API [CancelBatchJobExecution](https://docs.aws.amazon.com/m2/latest/APIReference/API_CancelBatchJobExecution.html) to allow the user to cancel the job from the Control-M console
- **Execution - Fetch End Time** - Will get the `END_DATE` in EPOCH format to be used when querying CloudWatch for job execution logs
- **Execution - Fetch CloudWatch Logs** - Step will only execute if the batch job was successfully started; uses the CloudWatch API [GetLogEvents](https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/API_GetLogEvents.html) to gather the job execution output using `APP_ID`, `job_name`, `START_DATE`, `END_DATE`; output is written to temporary file to be used in next step
- **Execution - Send jobs to job output** - Step will only execute if the batch job was successfully started; will read the temporary file from previous step, pipe it into jq to pull out only the event messages, and send it to the job output
