# Information

Welcome to Introduction to Azure API management.
Here you will find labs and related information.

## Session 1

### Weather Demo

Overview and demo of calling the Open Weather API.

- [v1](Session1/WeatherDemo/API/ARMTemplates/extracted-v1.json) Just a basic call
- [v2](Session1/WeatherDemo/API/ARMTemplates/extracted-v2.json) Updated following common usage guidelines.

### Routing Demo

Using Products and Policies to route incoming calls to either "Europe" or "America" based on the subscriber's product.
Here is the [api code](Session1/RoutingDemo/API/ARMTemplates/Ordermanagement-v1.json). In order to use it you need to setup backends to call. You can use the two Logic Apps provided. One for [America](Session1/RoutingDemo/Backends/logicapp-ERP-America.json) and one for [Europe](Session1/RoutingDemo/Backends/logicapp-ERP-Europe.json).

## Labs

### Lab 1

In this lab you will call the SL realtime APIs to get information about public transport.
It show how to manupulate the call using policies and protect the backend API-key.

You can find the lab [here](Session2/Lab1/instructions.md).

### Lab 2

In this lab you will configure a product and assing a subscription to that product. This is to show how API management handles authentication out of the box.

It also shows how products are related to APIs in APIm.

You can find the lab [here](Session2/Lab2/instructions.md).

### Lab 3

This lab shows you how you can protect your APIs in a very granular way using Oauth based on Azure AD and claims.

You can find the lab [here](Session2/Lab3/instructions.md).