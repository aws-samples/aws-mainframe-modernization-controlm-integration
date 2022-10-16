# aws-mainframe-modernization-controlm-integration

This module provides an integration between AWS mainframe modernization service and Control-M workflow orchestrator to schedule and run batch jobs, embrace automation for mainframe batch running in AWS mainframe modernization service.

The repo has sample jobtype, jobflow and connection profile templates that has to be imported in control-m  product.

AWS Services that are used in this solutions are:

- [AWS Mainframe Modernization Service](https://aws.amazon.com/mainframe-modernization/)

## Components details
Be sure to:

[assets/AIJOB.ctmai](assets/AIJOB.ctmai) - This is a job defintion file. The job type deployment is usually a one-time activity. 

[assets/AWS-jobs.json](assets/AWS-jobs.json) - This job file is a business process that you schedule or run in your enterprise environment.  

[assets/cp-JOB.json ](assets/cp-JOB.json) - This file is Connection profiles specific for AWS connectivity and is used to define access methods and security credentials for a specific application. They can be referenced by multiple jobs. To do this, you must deploy the connection profile definition before running the relevant jobs.   

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

