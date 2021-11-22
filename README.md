# Disable or Enable and Azure Function using REST calls

It is sometimes desirable to programmatically enable or disable and Azure function trigger. 

One use case could be an event hub triggered function that needs to update a database. if the database goes offline for a period of time, then a "circuit breaker" implementation could *open* to disable the function from processing more events. Later, when the database is once again available, the circuit breaker can be *closed*.

The circuit breaker pattern is explained here https://docs.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker

There is a really good blog article on reliable messaging with event hubs and functions https://hackernoon.com/reliable-event-processing-in-azure-functions-37054dc2d0fc which details the use of the circuit breaker in a logic app - but does not provide more detail on its implementation.

## Disabling a function using CLI
The Azure documentation provides information on how to disable a function using the Azure CLI, https://docs.microsoft.com/en-us/azure/azure-functions/disable-function?tabs=azurecli#disable-a-function but if you need to do this from some conventional code, then an HTTP REST implementation may be more desirable.

## Disabling a function using a REST call
There are a number of stackOverflow answers that discuss this, but this looks the closest one to this use case https://stackoverflow.com/questions/65657699/can-you-enable-and-disable-an-azure-timer-trigger-over-http

There are two steps to this.

### Use a service principal to get a bearer token
```
POST https://login.microsoftonline.com/<<azure-ad-tenant-id>>/oauth2/token
Content-Type: application/x-www-form-urlencoded

client_id=<<client-id>>&
client_secret=<<secret-of-service=principal>>&
resource=https://management.azure.com/&
grant_type=client_credentials
```

#### Use calls to management.azure.com to get and set the status of the function
```
POST https://management.azure.com/subscriptions/<<subscription-id>>/resourceGroups/<<resource-group-name>>/providers/Microsoft.Web/sites/<function-app-name>>/config/appsettings/list?api-version=2020-12-01
Authorization: Bearer <<bearer-token-from-above>>
Content-Type: application/json
```

This returns a JSON payload with the settings of the web app that underpins the function
```
{
  "id": "/subscriptions/<<subscription-id>>/resourceGroups/<<resource-group-name>>/providers/Microsoft.Web/sites/<function-app-name>>/config/appsettings",
  "name": "appsettings",
  "type": "Microsoft.Web/sites/config",
  "location": "<<app-region-name",
  "properties": {
    "AzureWebJobs.<function-name>.Disabled": "False"
  }
}
```

In the above, the function is enabled as its Disabled status is *False*.

### Update the status of the web app
This is done by doing a PUT HTTP request with the payload above, but its status must be set to *True* to disable the function.

```
PUT https://management.azure.com/subscriptions/<<subscription-id>>/resourceGroups/<<resource-group-name>>/providers/Microsoft.Web/sites/<function-app-name>>/config/appsettings?api-version=2020-12-01
Authorization: Bearer <<bearer-token-from-above>>
Content-Type: application/json

{
  "id": "/subscriptions/<<subscription-id>>/resourceGroups/<<resource-group-name>>/providers/Microsoft.Web/sites/<function-app-name>>/config/appsettings",
  "name": "appsettings",
  "type": "Microsoft.Web/sites/config",
  "location": "West Europe",
  "properties": {
    "AzureWebJobs.<function-name>.Disabled": "True"
  }
}
```

If all is correct, this will return an HTTP 200 with the response that you set on the PUT request.

To enable, the function again, do another PUT with the "AzureWebJobs.<function-name>.Disabled": "False".

 ### REST Client code
 Visual Studio Code has a simple REST client extension. This can send basic HTTP requests and use the results from one requests to feed another. The rest client HTTP calls are in this repo under *enable-function.http*.
  
# Summary
  To programmatically enable or disable a function via HTTP REST requires a service principal, which can then be used to authenticate against the management API of Azure and then update the status of the web app that hosts the function using an HTTP PUT request.
