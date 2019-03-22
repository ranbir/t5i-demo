# T5i-Demo 

Experimenting with an IoT Data Pipeline on Google Cloud Platform (GCP) using: 

  1. Google Compute Engine
  2. Google Cloud Pub/Sub
  3. Google Cloud IoT Core
  4. Google Cloud Dataflow
  5. Google BigQuery
  6. Google Cloud Storage
  
Aim:

To Create a registry of external sensors/devices inside Cloud IOT that connect via MQTT and then do analytics using BigQuery. The flow is Sensor via MQTT --> Connects to MQTT hub at GCP Cloud IOT --> Creates Pub/Sub Entry that sends entries to CloudDataflow --> Dataflow logs to a time series BigQuery database --> From BigQuery you analyze that info using Cloud DataLab or Cloud DataStudio.


  
Steps 

(1) Start Cloud Shell
		- gcloud auth list
			output - Credentialed accounts: - <myaccount>@<mydomain>.com (active)
		- gcloud config list project
			output - [core] project = <PROJECT_ID>
			
(2) Create a Pub/Sub Topic
		- In the GCP Console, go to Navigation menu> Pub/Sub> Topics
		- Click Create Topic. 
			- The Create a topic dialog shows you a partial URL path, consisting of projects followed by your project name and a trailing slash, then topics/ . 
			- Confirm that the project name is the one you noted above.	
		- Give this string as your topic name: iotlab
		- Then click Create.
		- In the resulting list of topics, you will see a new topic whose partial URL ends in __iotlab__
		- Click the three-dot icon at the right edge of its row to open its context menu. Choose Permissions
		- In the Permissions dialogue, add this member: cloud-iot@system.gserviceaccount.com
		- Grant the new member the Pub/Sub > Pub/Sub Publisher role. Click Add.
		
(3) Create a BigQuery dataset
		- In the GCP Console, go to Navigation menu> BigQuery.
		- Click on the blue arrow next to the name of your project and select Create new dataset
		- Give the new dataset the name iotlab and click OK.
		- When the dataset is created, to the right of iotlab, click the "Add table" icon. The Create Table dialog opens.
		- In the Source Data section, click Create empty table.
		- In the Destination Table section's Table name field, enter sensordata.
		- In the Schema section, enter timestamp for the field name. Set the field's Type to TIMESTAMP.
		- Click the Add Field button.
		- In the newly created line, enter device for the field name. Set the field's Type to STRING.
		- Click the Add Field button.
		- In the newly created line, enter temperature for the field name. Set the field's Type to FLOAT.
		- Leave the other defaults unmodified. Click Create Table.

(4) Create a Cloud Storage Bucket
		- In the GCP Console, go to Navigation menu > Storage.
		- Click CREATE BUCKET.
		- For Name, paste in your GCP project ID.
		- For Default storage class, click Multi-regional if it is not already selected.
		- For Location, choose the selection closest to you.
		- Click Create.

(5) Set up a Cloud Dataflow Pipeline
		- In the GCP Console, go to Navigation menu > Dataflow.
		- In the top menu bar, click CREATE JOB FROM TEMPLATE.
		- In the job-creation dialog, for Job name, enter iotlab.
		- For Cloud Dataflow template, choose PubSub to BigQuery. 
			- When you choose this template, the form updates to review new fields below.
				- For Cloud Dataflow Regional Endpoint, choose the region closest to you.
		- For Cloud Pub/Sub input topic, enter projects/ followed by your GCP project ID then add /topics/iotlab . 
			- The resulting string will look like this: projects/t5i-demo/topics/iotlab
		- For BigQuery output table, enter your GCP project ID followed by :iotlab.sensordata. 
			- The resulting string will look like this: t5i-demo:iotlab.sensordata

		- For Temporary location, enter gs:// followed by your GCP project ID and then /tmp/. 
			- The resulting string will look like this: gs://t5i-demo/tmp/

		- Click Optional parameters.
			- For Max workers, enter 2.
			- For Machine type, enter n1-standard-1.

		- Click Run Job. A new streaming job is started. You can now see a visual representation of the data pipeline.


(6) Prepare Your Compute Engine VM
		- In your SSH session on the iot-device-simulator VM instance, enter this command to remove the default Google Cloud Platform SDK installation
			- sudo apt-get remove google-cloud-sdk -y
		- Now install the latest version of the Google Cloud Platform SDK and accept all defaults:
			- curl https://sdk.cloud.google.com | bash
			
		- End your ssh session on the iot-device-simulator VM instance:
			- exit
			
		- Start another SSH session on the iot-device-simulator VM instance.
		- Initialize the gcloud SDK.
			- gcloud init
				- If you get the error message "Command not found," you might have forgotten to exit your previous SSH session and start a new one.
				- If you are asked whether to authenticate with an @developer.gserviceaccount.com account or to log in with a new account, choose to log in with a new account.
				- If you are asked "Are you sure you want to authenticate with your personal account? Do you want to continue (Y/n)?" enter Y.
				
		- Click on the URL shown to open a new browser window that displays a verification code.

		- Use your browser to copy the verification code.

		- Paste the verification code in response to the "Enter verification code:" prompt and press Enter.

		- In response to "Pick cloud project to use," pick the GCP project t5i-demo.

		- Enter this command to make sure that the components of the SDK are up to date:
			- gcloud components update
		- Enter this command to install the beta components:
			- gcloud components install beta
		- Enter this command to update the system's information about Debian Linux package repositories:
			- sudo apt-get update
		- Enter this command to make sure that various required software packages are installed:
			- sudo apt-get install python-pip openssl git -y
		- Use pip to add needed Python components:
			- sudo pip install pyjwt paho-mqtt cryptography
		- Enter this command to add data to analyze during this lab:
			- https://github.com/ranbir/t5i-demo.git	
			
(7) Create a Registry for IoT Devices			
			
			
			
