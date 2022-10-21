On this challenge we will learn how to use Logs to debug application issues. You will need to answer some questions about the state of the ecommerce application using Datadog.

# Lab13

## Application Logs

The Datadog Node Agent is configured to collect every log for the containers placed on the same node. Open the file called datadog-etcd.yaml in the Editor tab (this is the configuration that we have currently deployed).

This is the section where you require the Datadog Node Agent to collect logs from all containers:

```
log:
  enabled: true
  logsConfigContainerCollectAll: true
```

These include the logs coming from the control plane containers, but also from our Ecommerce application. Navigating to Logs > Search to browse the application logs.

Search for logs that were captured during the past hour by selecting Past 1 Hour in the time picker at the top right of the page:

![](!lab13-img1.jpeg)

Select the services that are part of our application (advertisements, storefrontend, discounts, postgres):

![](!lab13-img2.jpeg)

You can also find the same log query clicking on this link: https://app.datadoghq.com/logs?query=service%3A(storefront%20OR%20discounts%20OR%20advertisements%20OR%20postgres)

![](!lab13-img3.jpeg)

Spend some time navigating the application logs and getting familiar with the different filters, columns, etc.

## Correlating Traces and Logs

One of the best ways to debug application issues is by correlating traces and logs.

When a user gets an error while using a distributed application, the error may be originating from any of the services that are part of the application, or even the communication between those services. Being able to identify the different services that are part of a request (and where a potential error is) is something distributed tracing can help us with.

Once you have identified a trace that we want to investigate further and the different services that were part of it, you may want to dig into the logs that were generated on the different services as part of that request. This is where correlating logs and traces comes handy.

See how you can connect those two things together in Datadog.

Navigate back to APM > Traces and select a complete end to end trace. You can get one of the full end-to-end traces by clicking on this link.

Search for traces that were captured during the past 15 minutes ago by selecting Past 15 Minutes in the time picker on the top right of that page:

![](!lab13-img4.jpeg)

Select one of the traces you see, and you will see the flame graph appearing on the right panel.

Once in the flame graph, you can see all the logs that the application generated on that request (and that request only) by clicking on the Logs tab at the bottom of the panel:

![](!lab13-img5.jpeg)

Learning how to debug application issues by correlating traces, logs and infrastructure metrics will be very handy when we have to debug a real issue that we will have in our application.
