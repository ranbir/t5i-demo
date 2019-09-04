# T5i-Demo 

<h1>Experimenting with an IoT Data Pipeline on Google Cloud Platform (GCP) </h1>

We will be using:
  1. Google Compute Engine
  2. Google Cloud Pub/Sub
  3. Google Cloud IoT Core
  4. Google Cloud Dataflow
  5. Google BigQuery
  6. Google Cloud Storage
  
<h2>Aim:</h2>

To Create a registry of external sensors/devices inside Cloud IOT that connect via MQTT and then do analytics using BigQuery. 

The flow is 

Sensor via MQTT --> Connects to MQTT hub at GCP Cloud IOT --> Creates Pub/Sub Entry that sends entries to CloudDataflow --> Dataflow logs to a time series BigQuery database --> From BigQuery you analyze that info using Cloud DataLab or Cloud DataStudio.

<h2>Architecture Diagram </h2>

![](/IOT-GCP-Arch.png)

<h2>Steps </h2>

<h3>(1) Start Cloud Shell </h3>
		- gcloud auth list
			output - Credentialed accounts: - <myaccount>@<mydomain>.com (active)
		- gcloud config list project
			output - [core] project = <PROJECT_ID>
			
<h3>(2) Create a Pub/Sub Topic</h3>
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
		
<h3>(3) Create a BigQuery dataset</h3>
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

<h3>(4) Create a Cloud Storage Bucket</h3>
		- In the GCP Console, go to Navigation menu > Storage.
		- Click CREATE BUCKET.
		- For Name, paste in your GCP project ID.
		- For Default storage class, click Multi-regional if it is not already selected.
		- For Location, choose the selection closest to you.
		- Click Create.

<h3>(5) Set up a Cloud Dataflow Pipeline</h3>
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


<h3>(6) Prepare Your Compute Engine VM</h3>
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
			

<h2>Sequence for Creating IoT devices and start pumping data </h2>


<h3>(7) Create a Registry of Devices in the VMInstance</h3>

export PROJECT_ID=t5i-demo
export MY_REGION=us-central1

$PROJECT_ID=t5i-demo
$MY_REGION=us-central1


Enter this command to create the device registry:


gcloud beta iot registries create iotlab-registry \
   --project=$PROJECT_ID \
   --region=$MY_REGION \
   --event-notification-config=topic=projects/t5i-demo/topics/iotlab

--------------------------------------------------------------------------------------------------------------------------------  
   
<h3>(8) Create a Cryptographic Pair </h3>

cd $HOME/multipaxos/t5i-demo

openssl req -x509 -newkey rsa:2048 -keyout rsa_private.pem \
    -nodes -out rsa_cert.pem -subj "/CN=unused"
    
or with lots of expiry time as below
    
openssl req -x509 -nodes -newkey rsa:2048 -keyout rsa_private.pem \
    -days 1000000 -out rsa_cert.pem -subj "/CN=unused"
    
This openssl command creates an RSA cryptographic keypair and writes it to a file called rsa_private.pem.

--------------------------------------------------------------------------------------------------------------------------------  

<h3>(9) Add Simulated Devices to the Registry</h3>

   
gcloud beta iot devices create building-8-sensor \
  --project=$PROJECT_ID \
  --region=$MY_REGION \
  --registry=iotlab-registry \
  --public-key path=rsa_cert.pem,type=rs256
  
  
gcloud beta iot devices create building-10-sensor \
  --project=$PROJECT_ID \
  --region=$MY_REGION \
  --registry=iotlab-registry \
  --public-key path=rsa_cert.pem,type=rs256

--------------------------------------------------------------------------------------------------------------------------------  
<h3>(10) Run Simulated Devices </h3>
 
Enter these commands to download the CA root certificates from pki.google.com
  
cd $HOME/t5i-demo/
wget https://pki.google.com/roots.pem


python cloudiot_mqtt_example_json.py \
   --project_id=$PROJECT_ID \
   --cloud_region=$MY_REGION \
   --registry_id=iotlab-registry \
   --device_id=building-8-sensor \
   --private_key_file=rsa_private.pem \
   --message_type=event \
   --algorithm=RS256 > building-9-log.txt 2>&1 &
   
   
python cloudiot_mqtt_example_json.py \
   --project_id=$PROJECT_ID \
   --cloud_region=$MY_REGION \
   --registry_id=iotlab-registry \
   --device_id=building-8-sensor \
   --private_key_file=rsa_private.pem \
   --message_type=event \
   --algorithm=RS256
   
   
Telemetry data will flow from the simulated devices through Cloud IoT Core to the Cloud Pub/Sub topic. 

In turn, Dataflow job will read messages from the Pub/Sub topic and write their contents to the BigQuery table.

--------------------------------------------------------------------------------------------------------------------------------  			
			
