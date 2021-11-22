---
title: 'Azure Serverless (FAAS) Integration with Apache APISIX'
date: '2021-11-18'
lastmod: '2021-11-22'
tags: ['azure', 'microsoft-azure', 'serverless', 'open-source', 'Apache', 'APISIX', 'Lua']
draft: false
layout: PostSimple
images: ['/static/images/azure/azure.png']
summary: 'Incorporating Azure Serverless Function as Dynamic Upstream in Apache APISIX.'
---

<div className="flex flex-wrap justify-center    -mx-2 overflow-hidden xl:-mx-2">
  <div className="my-1 px-2 overflow-hidden xl:my-1 xl:px-2 xl:w-1/2">
    <Image alt="apisix" src="/static/images/apisix.png" width={100} height={100} />
  </div>
  <div className="my-1 px-2 overflow-hidden xl:my-1 xl:px-2 xl:w-1/2 ml-6">
    <Image alt="azure" src="/static/images/azure/azure.png" width={100} height={100} />
  </div>
</div>

In recent years, everything is moving into the cloud. With the emergence of microservice architecture, we have a proliferation of services - some of them are highly used while some are very infrequent leading rise to resource under-utilization and the birth of serverless.

`Apache APISIX` provides support for serverless frameworks for popular cloud vendors (more coming on the way). Instead of hardcoding the function URL into the application, APISIX suggests defining a route with the serverless plugin enabled. It gives the developers the flexibility to hot update the function URI along with completely changing the faas vendor to a different cloud provider with zero hassle. Also, this approach mitigates authorization and authentication concerns from application logic as APISIX has very strong authentication support that could be used to identify and authorize client consumers to access the particular route with the faas. This article talks about the recent addition of a new plugin `azure-functions`, and gives detailed instructions on how to integrate [Azure Functions](https://azure.microsoft.com/en-us/services/functions), which is a widely used serverless solution, into the Apache APISIX serverless suite.

> TLDR; if you are only interested in integrating Azure Faas with Apache APISIX head over to [how-to-use](#how-to-use)

## Contents Summary

- [Apache APISIX](#introducing-apache-apisix)
- [Microsoft Azure Cloud](#introducing-microsoft-azure-cloud)
- [Serverless - What is it?](#demystifying-serverless---what-is-it)
- [azure-functions Plugin](#about-azure-functions-plugin)
  - [Steps to Run Azure FAAS on Azure Cloud](#steps-to-run-azure-faas-on-azure-cloud)
  - [How to Use](#how-to-use)
  - [Custom Configuration](#custom-configuration)
    - [Plugin Schema](#plugin-schema)
    - [Metadata Schema](#metadata-schema)
- [Related pull request](#related-pull-request)

## Introducing Apache APISIX

[Apache APISIX](https://apisix.apache.org/) is a dynamic, real-time, high-performance API gateway that provides load balancing, dynamic upstream, canary release, fine-grained routing, rate limiting, service degradation, circuit breaking, authentication, observability, and hundreds of other features. In addition, the gateway supports dynamic plugin changes along with hot-loading. Apache APISIX can be used to proxy traditional NORTH-SOUTH traffic, as well as EAST-WEST traffic between services, or as a k8s ingress controller.

## Introducing Microsoft Azure Cloud

[Azure](https://azure.microsoft.com/en-in/), a public and private _cloud computing_ platform operated by Microsoft and launched in 2010, provides solutions for software as a service (SaaS), platform as a service (PaaS) and infrastructure as a service (IaaS) and supports many different programming languages, tools, and frameworks, including both Microsoft-specific and third-party software and systems. As of Q2-2021, Azure, along with Microsoft's software-as-a-service effort and its footprint in enterprises, make the company a strong No. 2 to AWS. Many enterprise customers have moved their infrastructure to azure or use azure in some ways in this multi-cloud era. [Azure Functions] is one of the most used serverless offerings from Azure cloud and the discussion of this article revolves around it.

## Demystifying Serverless - What is it?

`Serverless` is a cloud computing execution model where the cloud vendor runs the server by dynamically managing the allocation and deallocation of hardware resources so that the end-users can just focus on the main business logic aka the actual code that is going to be executed. Here, the logic runs as a function and there are two important traits of these functions - they are **stateless**, i.e. function instances are created and destroyed on demand so there should not be any in-memory states and they are **event-driven** that means the piece of code aka functions get executed via a triggering signal. These triggers could be HTTP requests, file upload to blob storage or the addition of new rows on cloud-SQL. So the benefits are:

- Easy deployment: The serverless framework needs no manual provisioning of the VM or compute instances as everything has been abstracted into functions and the cloud provider manages the infrastructure.
- Highly Scalable and Cost-Effective: The function application is automatically scaled up or down depending on the network traffic or load. So this is a cost-effective solution that leads to a shift from VM to serverless for the services which are less frequently invoked. But there is no such thing as free lunch. If the application gets bombarded by multiple clients, deploying a compute instance VM might be a cheaper alternative.

## About azure-functions Plugin

The `azure-functions` plugin lets the users define an upstream to the azure `HTTP Trigger` serverless function for a gateway URI. If enabled, this plugin terminates the ongoing request to that particular URI and initiates a new request to the azure faas (the new upstream) on behalf of the client with the suitable authorization details set by the users, request headers, request body, params ( all these three components are passed from the original request ) and returns the response body, status code and the headers back to the original client that has invoked the request to the APISIX agent.  
The plugin supports authorization to azure faas service via API keys and azure active directory.

### Steps to Run Azure FAAS on Azure Cloud

The primary goal of the plugin is to proxy the gateway route specified in the route configuration to the azure functions URI. This section gives you a hands-on how to configure and create a serverless HTTP Trigger on the azure cloud.

1. First sign up/in to Microsoft Azure and sets up a trial plan. Azure Functions are forever free up to 1 million invocations. To know more about how the pricing, visit [here](https://azure.microsoft.com/en-us/services/functions/#pricing).

2. Visit the [Azure Portal](https://portal.azure.com/#home) (FYI, azure services can be accessed via the web portal, CLI & VSCode. for user-friendliness we are using the web).  
   a. First, create a resource group to logically partition your faas that's you are going to create - fig-a.  
   b. Create a function app with the URL of your choice (I am going to pick `test-apisix`) - fig-b.

<div className="flex flex-wrap justify-center    -mx-2 overflow-hidden xl:-mx-2 py-0">
 <div className="my-1 px-2 overflow-hidden xl:my-1 xl:px-2 xl:w-1/2">
   <img quality={100} alt="gsoc"  src="/static/images/azure/resource-gp.png" width={500} height={250 } />
   <h6 className="text-center py-0 text-xs  italic">fig a </h6>
 </div>
 <div className="my-1 px-2 overflow-hidden xl:my-1 xl:px-2 xl:w-1/2 ml-6">
   <img alt="chromium" src="/static/images/azure/func.png" width={500} height={250} />
   <h6 className="text-center py-0 text-xs  italic">fig b</h6>
 </div>
</div>

3. Install the Azure Functions [extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) into VSCode editor. Upon installation, authenticate via extension and install the azure function core tool for local development with:

```shell
npm install -g azure-functions-core-tools@3 --unsafe-perm true
```

4. Deploy the following snippet to the same function app that we just created via the `Azure Functions` extension panel in VSCode:

```js
module.exports = async function (context, req) {
  context.log('HTTP trigger invoked on Test-APISIX.')

  const name = req.query.name || (req.body && req.body.name)
  const responseMessage = name
    ? 'Hello, ' + name
    : 'This HTTP triggered function executed successfully. Pass a name in the query string or in the request body to generate a personalized response.'

  context.res = {
    // status: 200, /* Defaults to 200 */
    body: responseMessage,
  }
}
```

\- this snippet takes the name from query parameters (if present, else from the request body) and greets the user.

### How to Use

The following is an example of how to **enable** the azure-functions plugin for a specific route. We are assuming your HTTP Trigger is deployed and ready to be served.

```shell
# enable plugin for a specific route
curl http://127.0.0.1:9080/apisix/admin/routes/1 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "plugins": {
        "azure-functions": {
            "function_uri": "http://test-apisix.azurewebsites.net/api/HttpTrigger",
            "authorization": {
                "apikey": "<Generated API key to access the Azure-Function>"
            }
        }
    },
    "uri": "/azure"
}'
```

Now any requests (HTTP/1.1, HTTPS, HTTP2) to URI `/azure` on the APISIX gateway will trigger an HTTP invocation to the aforesaid function URI and response body along with the response headers and response code will be proxied back to the client. For example ( here azure cloud function just take the `name` query param and returns `Hello $name` ) :

```shell
$ curl -i -XGET http://localhost:9080/azure\?name=Bisakh
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Request-Context: appId=cid-v1:38aae829-293b-43c2-82c6-fa94aec0a071
Date: Wed, 19 Nov 2021 18:46:55 GMT
Server: APISIX/2.10.2

Hello, Bisakh
```

Considering, APISIX is also running with `enable_http2: true` on APISIX [config-default.yaml](https://github.com/apache/apisix/blob/master/conf/config-default.yaml#L26) for port 9081 (say), any `HTTP/2` communication between client and APISIX agent will be proxied to the azure faas similar to HTTP/1.1 and responses will be proxied back to the client with proper headers. For example:

```shell
$ curl -i -XGET --http2 --http2-prior-knowledge http://localhost:9081/azure\?name=Bisakh
HTTP/2 200
content-type: text/plain; charset=utf-8
request-context: appId=cid-v1:38aae829-293b-43c2-82c6-fa94aec0a071
Date: Wed, 19 Nov 2021 18:46:56 GMT
server: APISIX/2.10.2

Hello, Bisakh
```

Now, to **disable** the plugin simply remove the corresponding JSON configuration in the plugin configuration to disable the `azure-functions` plugin and add the suitable upstream configuration.
APISIX plugins are hot-reloaded, therefore is no need to restart APISIX.

```shell
$ curl http://127.0.0.1:9080/apisix/admin/routes/1 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "uri": "/azure",
    "plugins": {},
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "127.0.0.1:1980": 1
        }
    }
}'
```

### Custom Configuration

In a minimal configuration while creating a new route with the azure-functions plugin enabled, `function_uri` is the mandatory attribute of the plugin config that points to the function URL. There is a lot of additional options that can be tweaked with plugin schema:

#### Plugin Schema

| Name                   | Type    | Requirement | Default | Valid      | Description                                                                                                                         |
| ---------------------- | ------- | ----------- | ------- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------- | --- |
| function_uri           | string  | required    |         |            | The azure function endpoint which triggers the serverless function code (eg. http://test-apisix.azurewebsites.net/api/HttpTrigger). |
| authorization          | object  | optional    |         |            | Authorization credentials to access the cloud function.                                                                             |
| authorization.apikey   | string  | optional    |         |            | Field inside _authorization_. The generate API Key to authorize requests to that endpoint.                                          |     |
| authorization.clientid | string  | optional    |         |            | Field inside _authorization_. The Client ID ( azure active directory ) to authorize requests to that endpoint.                      |     |
| timeout                | integer | optional    | 3000    | [100,...]  | Proxy request timeout in milliseconds.                                                                                              |
| ssl_verify             | boolean | optional    | true    | true/false | If enabled performs SSL verification of the server.                                                                                 |
| keepalive              | boolean | optional    | true    | true/false | To reuse the same proxy connection in near future. Set to false to disable keepalives and immediately close the connection.         |
| keepalive_pool         | integer | optional    | 5       | [1,...]    | The maximum number of connections in the pool.                                                                                      |
| keepalive_timeout      | integer | optional    | 60000   | [1000,...] | The maximal idle timeout (ms).                                                                                                      |

This gives a whole lot of flexibility to tightly bind the behaviour of the azure faas - from configuring the timeout to the keepalive pool and validating the SSL certificate of the serverless faas. To be honest, this actually means a lot when it comes to serverless as the services are event-driven and resources are being allocated by the cloud provider on the fly.

Similarly, there are a few attributes that can be tweaked by using the metadata.

#### Metadata Schema

| Name            | Type   | Requirement | Default | Valid | Description                                                                        |
| --------------- | ------ | ----------- | ------- | ----- | ---------------------------------------------------------------------------------- |
| master_apikey   | string | optional    | ""      |       | The API KEY secret that could be used to access the azure function URI.            |
| master_clientid | string | optional    | ""      |       | The Client ID (active directory) that could be used the authorize the function uri |

Metadata for `azure-functions` plugin provides the functionality for authorization fallback. It defines `master_apikey` and `master_clientid` (azure active directory client id) where users (optionally) can define the master API key or Client ID for mission-critical application deployment. So if there are no authorization details found inside the plugin attribute the authorization details present in the metadata kicks in.

The relative priority ordering is as follows:

- First, the plugin looks for `x-functions-key` or `x-functions-clientid` keys inside the request header to the APISIX agent.
- If they are not found, the azure-functions plugin checks for the authorization details inside plugin attributes. If present, it adds the respective header to the request sent to the Azure cloud function.
- If no authorization details are found inside plugin attributes, APISIX fetches the metadata config for this plugin and uses the master keys.

To add a new Master APIKEY, make a request to _/apisix/admin/plugin_metadata_ endpoint with the updated metadata as follows:

```shell
$ curl http://127.0.0.1:9080/apisix/admin/plugin_metadata/azure-functions \
-H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "master_apikey" : "<Your azure master access key>"
}'
```

<hr />

## Related pull request

✅ [PR#5479](https://github.com/apache/apisix/pull/5479) feat(plugin): azure serverless functions
