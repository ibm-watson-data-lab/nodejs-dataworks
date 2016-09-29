Introduction
------------
nodejs-dataworks is a module library for IBM Bluemix Data Connect APIs. At this time, only the data load APIs are covered by this library, but more Data Conect APIs will be covered over time including "Address Cleansing" and "Data Profiling" apis.  
As a reference, [The Simple Data Pipe example app](https://developer.ibm.com/clouddataservices/simple-data-pipe/) is using nodejs-dataworks to programmatically create activities for moving data from Cloudant to DashDB. (See the documentation [here](https://www.ng.bluemix.net/docs/api/content/api/dataworks/data-load/index.html).)

This post provides documentation on how to use our Data Connect client library for Node.js.

How to use the library
----------------------
```javascript
var dataworks = require("nodejs-dataworks").dataload;
var dwInstance = new dataworks();
...
```  

Let's take a deeper look at what's happening behind the `var dwInstance = new dataworks();` code.

By default the library will automatically try to find a service bound to the application using a regular expression that contains the word "dataworks" in its name. If you know the Bluemix Data Connect service you want to bind to, you can always pass it as an option object like so:

```javascript  
var dwInstance = new dataworks({dwServiceName: "myDataConnectServiceName"});
```

Once the Data Connect service is found in Bluemix's VCAP_SERVICES environment variable, the library will extract the credential information (url, userid, password) and use it in all subsequent API calls.
  
_Note:_

* _This work exactly the same when running locally._
* _If you use the `START_PROXY=true` env variable, the library will automatically start a proxy using the `/proxy` context root. (See [https://github.com/ibm-cds-labs/pipes/wiki/Instructions-for-setting-up-a-local-dev-environment-for-Simple-Data-Pipe](https://github.com/ibm-cds-labs/pipes/wiki/Instructions-for-setting-up-a-local-dev-environment-for-Simple-Data-Pipe) for more information on how to configure your local environment for using the proxy)._

####listActivities API####
All these APIs use the callback pattern, i.e., you pass a callback function that takes an error object and an array of activities as arguments.

```javascript
dwInstance.listActivities( function( err, activities ){
   if ( err ){
	return console.log( "Unable to get list of activities: %s", err );
   }
   console.log( "activities: %s", require('util').inspect( activities ) );
});
...
```  

####getActivity API####
Query an activity by its id:

```javascript
dwInstance.getActivity( activityId, function( err, activity ){
   if ( err ){
	return console.log( "Unable to get activity for id %s : %s", activityId, err );
   }
   console.log( "activity: %s", require('util').inspect( activity ) );
});
...
```  

####getActivityByName API####
Query an activity by its name.

_Note: return value can be null._

```javascript
dwInstance.getActivityByName( activityName, function( err, activity ){
   if ( err ){
	return console.log( "Unable to get activity for name %s : %s", activityName, err );
   }
   console.log( "activity: %s", require('util').inspect( activity ) );
});
...
```  

####deleteActivity API####
Delete an activity by its ID:

```javascript
dwInstance.deleteActivity( activityId, function( err ){
   if ( err ){
	return console.log( "Unable to delete activity for id %s : %s", activityId, err );
   }
   console.log( "activity successfully deleted" );
});
...
```  

####createActivity API####
Create a new activity by specifying the source and target connection.

_Note: this library uses a factory to create connections. At this time, only Cloudant and DashDB are supported._

```javascript
  var srcConnection = dwInstance.newConnection("cloudant");  //Create a source connection for cloudant
  srcConnection.setDbName( "sf_campaign__c" );   //Set the cloudant db name for the source connection
  srcConnection.addTable( {
     name: "sf_campaign__c".toUpperCase()
  });   //Specify the table
  var targetConnection = dwInstance.newConnection("dashDB"); //Create a target connection for dashDB
  targetConnection.setSourceConnection( srcConnection ); //Simply set the associated source connection
  dwInstance.createActivity({
	name: "test",
	desc: "Test instance",
	srcConnection: srcConnection,
	targetConnection: targetConnection
  }, function( err, activity ){
	if ( err ){
		return console.log("Unable to create activity: %s", err);
	}
	console.log("SuccessFully created a new activity: %s", util.inspect( activity, { showHidden: true, depth: null } ) );
  });
...
```  

####runActivity API####
Run an activity by its ID:

```javascript
dwInstance.runActivity( activityId, function( err, activityRun ){
   if ( err ){
	return console.log( "Unable to run activity for id %s : %s", activityId, err );
   }
   console.log( "activity successfully run: %s", require('util').inspect( activityRun ) );
});
...
```  

####monitorActivityRun API####
Monitor an activity run by its ID:

```javascript
  dwInstance.runActivity( activityId, function( err, activityRun ){
   if ( err ){
	return console.log( "Unable to run activity for id %s : %s", activityId, err );
   }
		
   var monitor = function(){
       dwInstance.monitorActivityRun( activityId, activityRun.id, function( err, activityRun ){
            if ( err ){
	        return console.log("Error retrieving activity run details %s", err );
	    }
	    if ( dwInstance.isFinished( activityRun.status ) ){   //Query whether the activity is finished
		 return console.log("ActivityRun complete");
	    }
            console.log( "Activity Running: %s", util.inspect( activityRun, { showHidden: true, depth: null } ));
	    setTimeout( monitor, 5000 ); //Poll again in 5s
	})
   };
   console.log("SuccessFully submitted a activity for running. Waiting for results...: %s", util.inspect( activityRun, { showHidden: true, depth: null } ) );
   setTimeout( monitor, 5000 );
});
...
```  

####listActivityRuns API####
List all activity runs for a specified activity:

```javascript
dwInstance.listActivityRuns( activityId, function( err, activityRuns ){
   if ( err ){
	return console.log( "Unable to get activity runs for activity id %s : %s", activityId, err );
   }
   console.log( "List of activity runs: %s", require('util').inspect( activityRuns ) );
});
...
```  

####deleteActivityRun API####
Delete an activity run by its ID:

```javascript
dwInstance.deleteActivityRun( activityId, activityRunId, function( err ){
   if ( err ){
	return console.log( "Unable to delete activity run for id %s and activity id %s : %s", activityRunId, activityId, err );
   }
   console.log( "Successfully deleted activity Run id %s", activityRunId );
});
...
```

