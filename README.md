# CPSC5910 HW2 - Azure RBAC - Colin Aylawrd

## Login To Azure CLI

```az login```

## Create Resource Groups

```az group create --name HW2Dev --location eastus```

```az group create --name HW2Prod --location eastus```

## Create The Dev Service Principle

```az ad sp create-for-rbac --name cpsc5910-cpa-dev```

```json
{
  "appId": "fb3876a4-7201-4bf3-a9ff-5e4c53f99ae3",
  "displayName": "cpsc5910-cpa-dev",
  "password": <SP Password>,
  "tenant": "bc10e052-b01c-4849-9967-ee7ec74fc9d8"
}
```

## Create The SRE Service Principle

```az ad sp create-for-rbac --name cpsc5910-cpa-sre```

```json
{
  "appId": "5d004dd7-6784-46c8-a669-0b48a70c2764",
  "displayName": "cpsc5910-cpa-sre",
  "password": <SP Password>,
  "tenant": "bc10e052-b01c-4849-9967-ee7ec74fc9d8"
}
```

## Create The SecOps Service Principle

```az ad sp create-for-rbac --name cpsc5910-cpa-secops```

```json
{
  "appId": "5bf7c882-0048-4885-ba62-ddbc167bebc1",
  "displayName": "cpsc5910-cpa-secops",
  "password": <SP Password>,
  "tenant": "bc10e052-b01c-4849-9967-ee7ec74fc9d8"
}
```

## Grant The Service Principles Access To The Resource Groups

```az role assignment create --assignee fb3876a4-7201-4bf3-a9ff-5e4c53f99ae3 --role owner --resource-group HW2Dev```

```az role assignment create --assignee 5d004dd7-6784-46c8-a669-0b48a70c2764 --role owner --resource-group HW2Prod```

```az role assignment create --assignee 5d004dd7-6784-46c8-a669-0b48a70c2764 --role reader --resource-group HW2Dev```

```az role assignment create --assignee 5bf7c882-0048-4885-ba62-ddbc167bebc1 --role reader --resource-group HW2Dev```

```az role assignment create --assignee 5bf7c882-0048-4885-ba62-ddbc167bebc1 --role reader --resource-group HW2Prod```

## Have An Application To Play With

To make the demonstration of permissions more interesting let's deploy a real application that we are all familiar with... Hello world! To do this we will use a ARM deployment that creates the whole application in a single deployment based off of this walkthrough of manual configuration.

<https://learn.microsoft.com/en-us/azure/container-instances/container-instances-quickstart-portal>

I've included the ARM template as deploy_dev.json and deploy_prod.json respectively.

## Logout Of Your Own Account

```az logout```

## Login As Dev and Exercise Permissions

```az login --service-principal -u fb3876a4-7201-4bf3-a9ff-5e4c53f99ae3 -p <SP Password> --tenant bc10e052-b01c-4849-9967-ee7ec74fc9d8```

| Expected Result    | Scenario |
|--------------------|----------|
| :heavy_check_mark: | ```az deployment group create --resource-group HW2Dev --template-file deploy_dev.json```|
| :x:                | ```az deployment group create --resource-group HW2Prod --template-file deploy_prod.json```|
| :heavy_check_mark: | ```az group show --name HW2Dev```|
| :x:                | ```az group show --name HW2Prod```|

## Login As SRE and Exercise Permissions

```az login --service-principal -u 5d004dd7-6784-46c8-a669-0b48a70c2764 -p <SP Password> --tenant bc10e052-b01c-4849-9967-ee7ec74fc9d8```

| Expected Result    | Scenario |
|--------------------|----------|
| :x:                | ```az deployment group create --resource-group HW2Dev --template-file dev.bicep```|
| :heavy_check_mark: | ```az deployment group create --resource-group HW2Prod --template-file prod.bicep```|
| :heavy_check_mark: | ```az group show --name HW2Dev```|
| :heavy_check_mark: | ```az group show --name HW2Prod```|

## Login As SecOps and Exercise Permissions

```az login --service-principal -u 5bf7c882-0048-4885-ba62-ddbc167bebc1 -p <SP Password> --tenant bc10e052-b01c-4849-9967-ee7ec74fc9d8```

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
