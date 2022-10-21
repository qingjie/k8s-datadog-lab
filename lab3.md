lab3
We have created a trial Datadog account for you that will be used throughout the workshop. To retrieve your credentials execute the following command:
```
creds
```
You can use those credentials to log in to the Datadog app. Open Datadog in a browser window (or click on the Datadog tab) and test those credentials. You will find that your Datadog account is still quite empty. This is because we haven't deployed the Datadog Agent yet.
You will be deploying Datadog in your cluster. For that you need to retrieve the API key for your Datadog organization. To make things easier we have already injected the Datadog API key of your newly created Datadog account in an environment variable. Check that it has a value by executing the following command:
```
echo $DD_API_KEY
```
