# CPSC5910 HW2 - Azure RBAC - Colin Aylawrd

## Pre-Requisites

This walkthrough assumes that you have an Azure Subscription that is enabled for pay-as-you-go.  I recommend that you sign up for a student account and then update the subscription to pay-as-you-go. This ensures that you will not be charged for any resources that you create as Microsoft gives students several free tiers and $100/year of credit for anything else.

## Overview of the Walkthrough

In this walkthrough we will create two resource groups, one for development and one for production. We will then create three service principles, one for each of the roles that we will be using in this walkthrough. We will then grant the service principles access to the resource groups. Finally we will use the service principles to exercise the permissions that we have granted them.

As a first step, search and replace all instances of ```<YOUR INITIALS>``` in the code base and put in your intitials, this should help avoid name collisions.

## Login To Azure CLI

```az login```

## Create Resource Groups

```az group create --name HW2Dev --location eastus```

```az group create --name HW2Prod --location eastus```

## Create The Service Principles

For simplicity we will be creating Service Principles for each of the types of identities that we want to demonstrate.  Service Principles can be made with several forms of auth, to keep things easy we are using password auth.  Please don't ever do this on a real implementation.

Run:
```az ad sp create-for-rbac --name cpsc5910-<YOUR INITIALS>-dev```

Expected Output:

```json
{
  "appId": <DEV SP ID>,
  "displayName": "cpsc5910-<YOUR INITIALS>-dev",
  "password": <DEV SP Password>,
  "tenant": "bc10e052-b01c-4849-9967-ee7ec74fc9d8"
}
```

Run:
```az ad sp create-for-rbac --name cpsc5910-<YOUR INITIALS>-sre```

Expected Output:

```json
{
  "appId": <SRE SP ID>,
  "displayName": "cpsc5910-<YOUR INITIALS>-sre",
  "password": <SRE SP Password>,
  "tenant": "bc10e052-b01c-4849-9967-ee7ec74fc9d8"
}
```

Run:
```az ad sp create-for-rbac --name cpsc5910-<YOUR INITIALS>-secops```

Expected Output:

```json
{
  "appId": <SECOPS SP ID>,
  "displayName": "cpsc5910-<YOUR INITIALS>-secops",
  "password": <SECOPS SP Password>,
  "tenant": "bc10e052-b01c-4849-9967-ee7ec74fc9d8"
}
```

:emphasis: Save the output from all three of these commands into a file called .passwords (which is ignored already via .gitignore).  You will need to refer to the SP ID's and Passwords in subsequent steps.

## Grant The Service Principles Access To The Resource Groups

```az role assignment create --assignee <DEV SP ID> --role owner --resource-group HW2Dev```

```az role assignment create --assignee <SRE SP ID> --role owner --resource-group HW2Prod```

```az role assignment create --assignee <SRE SP ID> --role reader --resource-group HW2Dev```

```az role assignment create --assignee <SECOPS SP ID> --role reader --resource-group HW2Dev```

```az role assignment create --assignee <SECOPS SP ID> --role reader --resource-group HW2Prod```

## Have An Application To Play With

To make the demonstration of permissions more interesting let's deploy a real application that we are all familiar with... Hello world! To do this we will use a ARM deployment that creates the whole application in a single deployment based off of this walkthrough of manual configuration.

<https://learn.microsoft.com/en-us/azure/container-instances/container-instances-quickstart-portal>

I've included the ARM template as deploy_dev.json and deploy_prod.json respectively.  You can see if your deployments worked or not fully by looking at the ACI instance to find the FQDN and test it out!

## Logout Of Your Own Account

```az logout```

## Login As Dev and Exercise Permissions

```az login --service-principal -u <DEV SP ID> -p <DEV SP Password> --tenant bc10e052-b01c-4849-9967-ee7ec74fc9d8```

| Expected Result    | Scenario |
|--------------------|----------|
| :heavy_check_mark: | ```az deployment group create --resource-group HW2Dev --template-file deploy_dev.json```|
| :x:                | ```az deployment group create --resource-group HW2Prod --template-file deploy_prod.json```|
| :heavy_check_mark: | ```az group show --name HW2Dev```|
| :x:                | ```az group show --name HW2Prod```|

## Login As SRE and Exercise Permissions

```az login --service-principal -u <SRE SP ID> -p <SRE SP Password> --tenant bc10e052-b01c-4849-9967-ee7ec74fc9d8```

| Expected Result    | Scenario |
|--------------------|----------|
| :x:                | ```az deployment group create --resource-group HW2Dev --template-file dev.bicep```|
| :heavy_check_mark: | ```az deployment group create --resource-group HW2Prod --template-file prod.bicep```|
| :heavy_check_mark: | ```az group show --name HW2Dev```|
| :heavy_check_mark: | ```az group show --name HW2Prod```|

## Login As SecOps and Exercise Permissions

```az login --service-principal -u <SECOPS SP ID> -p <SECOPS SP Password> --tenant bc10e052-b01c-4849-9967-ee7ec74fc9d8```

| Expected Result    | Scenario |
|--------------------|----------|
| :x:                | ```az deployment group create --resource-group HW2Dev --template-file dev.bicep```|
| :x:                | ```az deployment group create --resource-group HW2Prod --template-file prod.bicep```|
| :heavy_check_mark: | ```az group show --name HW2Dev```|
| :heavy_check_mark: | ```az group show --name HW2Prod```|

## Clean Up Resources

```az logout```
```az login```
```az group delete --resource-group HW2Dev```
```az group delete --resourec-group HW2Prod```
```az ad sp delete --id <DEV SP ID>```
```az ad sp delete --id <SRE SP ID>```
```az ad sp delete --id <SECOPS SP ID>```
