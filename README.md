# Creating an IoT solution using BlueMix. 

## Scenario

You want to create a product which allows data centers to monitor the temperature and humidity of each rack. For this you would need sensors which measures temperature and humidity. You would need to read these sensors and perform some action on the data read. these actions could be
 
* creating alerts for sys ops team (email / sms alerts) 
* persist data for further analysis (data storage)
* take corrective actions if the measurements are beyond the prescribed threshhold limits. (counter measure tools)

An easy way to access this sensor data is to have APIs which would allow various differnet clients such a web app or mobile app or a desktop app to access the sensor data, and in a more governed way. We will see that ahead on how to use APIM to craete and manage such APIs.

APIM allows us to create and publish APIs where the API Developer can control 
  1. Authentication and authorization to access the APIs ( hence the senor data)
  1. Put rate limit to control how nany times a client can make request per day/hour/min/sec
  1. Provides a mechanism to gracefully update the APIs without causing any disruption to client.
  
## Creating IoT

Pre-req :  Following steps needs a BlueMix account. You can sign up for one here at [BlueMix](www.ibm.com/Bluemix).

Once you have a BlueMix account follow steps below.


1. Make sure you are in Dashboard view by clicking on Dashboard on top right menu.
1. Choose 'Create a new space' option on left had window and create a new space as 'APIM-IOT'
1. From top right menu choose 'catalog' and select 'Internet of things foundation starter' 
1. An IoT foundation starter wizard page will appear which promps for app name. Provide any name for your IoT app, For the example we will name it "DataCenterMonitor"
1. Leave everything else to default and click on 'create app' button on bottom right of screen.
1.  You will now see 'your application is staging" message with progress indicator. 


The proces above creates an IoT app with IoT services and Node-Red vsiual editor. We will see more about Node-Red in few minutes. 

Click on "App Overview" Button in the bottom of the page and it will bring you to a page where you see various components which builds your basic IoT starter kit,


* SDK For Node.js
* Cloudant NoSQL DB



To see your current IoT solution, click on your project link shown on top half of the page. The link appears like, {solution_name}.mybluemix.net. for our example it would be 'datacentermonitor.mybluemix.net'. This link will take you to page with the link to Node-Red flow editor. Click on this tile and it will show a ready-made flow that can process temperature readings from a simulated device. 

Double click each component to understand what the flow does. The basic flow passes the input read from IoT Service to a code snippet. The message is passed in as a JavaScript object called ```msg```. By convention it will have a ```msg.payload``` property containing the body of the message.

The flow then passes the payload to a rules engine which will check if the temperature is less than or equal to prescribed value and depending on the rule it will either continue to snippet where a happy message is printed or will trigger a snippet where an alert is printed. 
[IBM Intenet of things foundation](https://quickstart.internetofthings.ibmcloud.com) provides some simulated device which we can use for our IoT service.

For our example we will use a simulated sensor from the link below 

<pre>
http://quickstart.internetofthings.ibmcloud.com/iotsensor
</pre>

1. Note the device id on top left. Copy the device id. 
1. In our Node-Red flow, double click the 'IBM IOT App In' which is the beginoing of the flow and set the device id to the value copied in step 1.
2. Now on top left of the page click 'Deploy'. This will restart your node.
3. Now on right hand panel, select debug and you will see the json data from the sensor showing temperature, humidity and object temperature.
4. Change the values in simulator and you will see with every change the new data is printed in debug window. 
4. We want to persist this data read from sensor to our cloud ant database. And using Node-Red its just a matter of drag and drop.
5. From left hand window choose 'cloudant' from under 'storage' and drag it to our flow.
6. Now connect the right end of 'IBM IoT app In' to the left end of the 'cloudant' componet.
7. Double click on 'cloudant' and click on drop down box for 'service' and choose our cloud ant instance which in this case is 'DataCenterMonitor-cloudantNoSQLDB'.
8. Type in the database name as 'apimdb'
9. Select the 'insert' operation.
10. Check the box 'only store msg.payload' object.
11. Set name as 'Persist Sensor Data'. And click 'OK'
12. Click the 'Deploy' button on top left of the screen to restart the node app.
 
##### Check Cloud Ant Database to see if the IoT data is persisted.

1. Go back to BlueMix Dashboard, and select the 'DataCenterMonitor-cloudantNoSQLDB' tile.
1. On top left click on 'Launch' button to open the CloudAnt dashboard.
1. In the cloud ant dashboard click on the 'apimdb' database and you will see all the documents generated by sensor on left.
1. By default CloudAnt webtool only shows id and revision of each document. To see the details click on the 'Query Options' and enable the 'include docs'.

Our next aim is to create APIs to access this data. Which We will see in the following section of creating APIs.

## Creating APIs using IBM API Management

### Setting up IBM APIM Service in BlueMix.


* In BueMix dashboard view click the 'DataCenterMonitor' space. 
* In the overview of the 'DataCenterMonitor' space click the 'Add a Service or API' tile. 
* From the catalog choose 'API Management'. 
* Leave all the default values and choose 'create'


### Creating API in APIM and publish

#### First we will find the URL to pull query from cloudant. To do this we launch the cloudant dashboard


1. In BlueMix dashboard click the "CloudAnt" tile and select "Launch" to open up the cloudant dashboard.
1. In the cloudant dashboard select the 'apindb' database and on top right choose query options and select "include documents" and run query.
1. Select 'permissions' from left hand menu and give 'Everybody else' a 'Reader' access.
1. Click on the 'API URL' icon on the top right of cloudant dashboard. Click and save the URL 
1. Open the URL in a web browser and save the json response.
1. Click on the APIM tile in 'DataCenterMonitor' space. 
1. Select "Open API Manager"
1. In the API Manager click on API icon on left hand bar
1. In the API Window click on '+' icon to create new API and choose "compose"
1. Set the name as "WeatherAPI" and basepath as "weather".
1. In the API Editor unser "operations" create GET operation and set the path as "/current_temp".
1. Once the operation is created, click the edit button on right end of operation name.
1. In the oepration editor choose 'Overview' and under response set the json as 
   [
     {
        "condition": {
           "name": "6a67b6fa9e68",
           "temp": 18
        }
     }
   ]

1. Under 'Implementation' window add a HTTP GET operation. And set the name as "my cloud ant" and set the url to cloudant we saved in step 4.
1. Click 'Define' and save the response saved in step 5.
1. In 'Configure' set the value of "include_docs" to true.
1. Click save button on top right corner to save the API.
1. Now to extract just the temperature values from the response json, click the Response and map the values from cloudant api response to the response format we defined in step 13
1. Save the API.
1. Click on Plan icon on left hand bar and create a new Plan. name it as "Silver".
1. Add API to the 'Silver' plan. and save the plan with default values.
1. Click on API on left hand bar and choose the "WeatherAPI". 
1. Click on the edit button for GET operation and choose Test..
1. Click on invoke. You will see a response body with HTTP Response status code of 200 OK. The API is ready to deploy.

#### We will need to create an Environment to deploy the plan to.

1. On left hand bar choose the 'Environments' icon and create a new Environment
1. Set the name as 'Test environment' and path segment as 'test'. Click on Save to create new environment.
1. Go back to 'Silver' plan and click on the stage button. Choose 'Test environment'. The Plan now gets staged tp the Test Environment.
1. To allow the API to be accessed by BlueMix user click on 'Developer' icon on left and add the "BlueMix organization" using the BlueMix user email id.
1. Click the 'Management' icon on left bar and choose the 'Test environment' environment
1. For the 'Silver' plan choose 'Manage' (gear icon) under Actions and choose 'Publsh'.




