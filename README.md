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

```
