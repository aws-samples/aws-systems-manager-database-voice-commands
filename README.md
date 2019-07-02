## Monitor and manage your databases through voice-commands with Amazon Alexa and AWS Systems Manager

The code represents an Alexa skill, containing the structure of the skill in a JSON file and the code of the related Lambda function in Python.

## License Summary

This sample code is made available under a modified MIT license. See the LICENSE file.

## INSTALLATION GUIDE

### Pre-requirements

Choose the region where you want to create the infrastructure and verify if Lambda can be triggered by an Alexa Skill (this feature is not available everywhere yet).
The demo has been tested in Virginia (us-east-1) but you can change the region if needed.

1. Create an EC2 key pair in the region you want to deploy the infrastructure. The key pair will be associated to the database client


1.	Create an S3 bucket in the region you want to deploy the infrastructure. Inside the bucket you have to define the following hierarchy and upload the files in attached:

    **Your-S3-bucket-name**
    * Directory **binaries**
      * oracle-instantclient11.2-basic-11.2.0.4.0-1.x86_64.rpm (*)
      * oracle-instantclient11.2-sqlplus-11.2.0.4.0-1.x86_64.rpm (*)

    * Directory **scripts**
      * bash_profile.txt
      * lambda_function.zip
      * mysql_statusdb.sql
      * oracle_checkparameter.sql
      * oracle_statusdb.sql
      * oracle_tablespaceusage.sql
      * oracle_show_top_session_by_cpu.sql
    * Directory **templates**
      * alexards_compute.template
      * alexards_global.template
      * alexards_network.template
      * alexards_security.template
      * alexards_storage.template

    (*) The Oracle client binaries are required in order to work with the Oracle "sample" database created in this demo. You have to download the two binaries separately and stored them into the "binaries" directory in the S3 bucket. The minimum version required is the 11.2.0.4


1.	Create an SNS Topic. The SNS Topic will be used in order to receive the information requested and the logs via e-mail


1.	Create an e-mail SNS Subscriber for the SNS Topic created in the previous step


1.	Create an IAM role that will be associated with the Lambda function. Here the policies needed:
    * AmazonRDSFullAccess
    * AmazonEC2FullAccess
    * AWSLambdaFullAccess
    * AmazonSSMFullAccess


1.	You have to create an IAM role that will be associated with the RDS "sample" database created in this demo. Here the policies needed:
    * AmazonRDSEnhancedMonitoringRole

1.	Before submit the *.template files, you need to modify them according with the bucket you have created and the region you have chosen:
    * Substitute the string "region_to_replace" and "name_to_replace" with the name of the region you have chosen and the name of your bucket in the alexards_global.template and in the alexards_compute.template
      * [ex.] "TemplateURL": "https://s3.[region_to_replace].amazonaws.com/[name_to_replace]/templates/alexards_network.template"
      * [ex.] "TemplateURL": https://s3.eu-west-2.amazonaws.com/alexards-london-bck-2019/templates/alexards_network.template

    * Substitute the names of the Oracle binaries if you are using a different version in the alexards_compute.template:
      * [ex.] "aws s3 cp s3://[name_to_replace]/binaries/oracle-instantclient11.2-basic-11.2.0.4.0-1.x86_64.rpm .","\n",
      * [ex.] "aws s3 cp s3://alexards-london-bck-2019/binaries/oracle-instantclient12.2-basic-12.2.0.1.0-1.x86_64.rpm .","\n",

    * Substitute the name of the resource "mySSMParamStoreBastion" in the alexards_storage.template with the one that will contain the name of the region you have chosen:
      * [ex.] "Name" : "/rds/configuration/bastion/virginia"
      * [ex.] "Value" : "RDSBastionVirginia"

      * [ex.] "Name" : "/rds/configuration/bastion/virgina"
      * [ex.] "Value" : "RDSBastionVirginia"

    * [only if you are not working in us-east-1] Change the AMI Id in the alexards_compute.template using the latest Amazon Linux 2 AMI available in the region you have chosen:
      * [ex.] Virginia: ami-035be7bafff33b6b6
      * [ex.] London: ami-0664a710233d7c148

    * [only if you are not working in us-east-1] Change the names of the Availability zone in the alexards_network.template


## Deploy the infrastructure with AWS CloudFormation

Choose the region where you want to deploy your infrastructure and then submit the alexards_global.template to CloudFormation.
The creation will take a while because the RDS master instance and the related secondary instance.


## Create the Alexa skill

Create now the Alexa skill:

1.	Sign-in on https://developer.amazon.com/alexa

1.	Click on "Your Alexa Consoles" and then on "Skills"

1.	Click on "Create Skill":
    * Skill name: rdsdemo
    * Default language: English (US)
    * Model: Custom
    * Template: Start from scratch

1.	Click on "JSON Editor":
    1. Copy and Paste the content of the file alexa_skill_v2
    1. Click on "Save Model"

1.	Click on "Endpoint"
    1.	Copy to clipboard the Skill ID
    1.	In another window, open your Lambda function in the Lambda dashboard and add the "Alexa Skills Kit" trigger to the function. Remember to save the change.
    1.	Go back to the Alexa portal:
        * Service Endpoint Type: AWS Lambda ARN
        * Default Region: [the ARN of your Lambda function] 
    1.	Click on "Save Endpoints"
    1.	Click on "Build Model"


## Post-deploy steps

After the successful creation of the infrastructure by CloudFormation, you have to do the following actions:
1.	Change the source of the inbound rule of the alexardsOracleSecGrp security group attached to the RDS instance:
    * 0.0.0.0/0 --> [security group ID of the alexardsBastionSecGrp security group]

1.	Add a tag "Name" to the EC2 instance. The name should the same specified in the "mySSMParamStoreBastion" in the alexards_storage.template:
    * Name: RDSBastionVirginia

1.	Add the following tags to the RDS instance. Please maintain the exact case:
    * application: training
    * environment: production
    * instance: sample

1.	Publish the Alert log of the RDS instance on CloudwatchLogs:
    * Alert log


## Test the skill

Go to the on https://developer.amazon.com/alexa and use the "Test" section to test all the interactions without use the Alexa device.

In order to start the skill:
  * rds demo


Here the list of all the utterances you could try:
  * list instances in virginia

  * list applications in virginia

  * list instances of application training in virginia

  * number of instances in virginia

  * status for sample instance in virginia

  * free storage space for sample instance in viriginia now
  * database connections for sample instance in viriginia 1 hour ago
  * cpu utilization for sample instance in viriginia 5 minutes ago

  * connect to sample instance in virginia

  * latest restorable time for sample instance in virginia
  * parameter group for sample instance in virginia

  * tablespace usage for sample instance in virginia

  * list events for sample instance in virginia

  * parameter open cursors for sample instance in virginia

  * check alert log of sample instance in virginia

  * reboot sample instance in virginia

  * failover sample instance in virginia
