# AWS App Mesh - Step by Step

## What is AWS App Mesh?

**AWS App Mesh**, which I'll simply refer to as App Mesh from here on in, is made of a few different components. I'm going to try to explain the relationship between the different components for before we get started.

Firstly, we have the **Mesh**. This is the logical boundary for network traffic that moves between the services within that boundary.

**Virtual Nodes**. This is a logical 'thing' that represents of a versioned instances of a given service within the mesh. As an example, if I have a service called **ServiceA**, there would be a corespding virtual node. The virtual node is a construct that includes metadata about **ServiceA**, including service discovery details on how to access that service, a list of backend services that the ServiceA communicates with and listener information that describes the ports over which requests to the **ServiceA** can be made.

**Virtual Router** and **Routes**. This is where the magic happens in terms of routing requests for a given service name to the appropriate virtual node. The Virtual Router and Routes work in conjunction to do fancy things like weighted distribution of requests to different version of running services.

This is a high-level overview of the different 'bits' that make up an App Mesh deployment. As we go through this post, I'll demonstrate how each of these components fits together to simplify the management of complex micro-service application architectures.

## What exactly are we going to build?

Great question! To demonstrate the value of a Service Mesh, and how specifically we setup and configure App Mesh, I am going to deploy a 2 microserice application.

The 2 microservices are 1) the **Requester** and 2) the **distributor**. I will then deploy a new version of the **distributor** and we'll see how App Mesh allows us to do do weighted distribution of traffic between V1 and V2 of the service.

## Environment Setup

Before we get started, we're going to need a few pre-requisites. 

First we need to deploy a new ECS cluster, then create some task definitions and then some services.

## Deploy a Cloud9 Environment

For the purpose of this blog post I'm going to be using Cloud9 as my working environment. Why? Well, it's awesome.
The more objective view however is that becuase I can directly connect the instance running Cloud9 in to my VPC, I can easily test access to the service I expose within my mesh without needing to expose them publically using a load balancer. Cloud9 also comes with the AWS CLI pre-installed and I can easily assign permissions for interacting with the AWS APIs I neeed using IAM Roles.

You can read more about Cloud9 [here](https://aws.amazon.com/cloud9).

## Create a Service Mesh

<pre>
aws appmesh create-mesh --mesh-name gophermesh --region us-east-1
</pre>

## Create a Virtual Node for the Gopher Requester

<pre>
aws appmesh create-virtual-node \
--cli-input-json file://./VirtualNodes/virtual-node-requestor.json \
--region us-east-1
</pre>

<pre>
aws appmesh list-virtual-nodes \
--mesh-name gophermesh \
--region us-east-1
</pre>

## Create a Virtual Node for the Gopher Distributor V1

<pre>
aws appmesh create-virtual-node \
--cli-input-json file://./VirtualNodes/virtual-node-distributor-v1.json \
--region us-east-1
</pre>

## Create a Virtual Node for the Gopher Distributor V2

<pre>
aws appmesh create-virtual-node \
--cli-input-json file://./VirtualNodes/virtual-node-distributor-v2.json \
--region us-east-1
</pre>

## Create a Virtual Router for the Gopher Requester

<pre>
aws appmesh create-virtual-router \
--cli-input-json file://./VirtualRouter/virtual-router-requester.json \
--region us-east-1
</pre>

Check that our virtual router created correctly

<pre>
aws appmesh list-virtual-routers \
--mesh-name gophermesh --region us-east-1
</pre>

## Registering a Task Defintiion

<pre>
aws ecs register-task-definition \
--cli-input-json file://./TaskDefinitions/gopher-requester-task \
--region ${REGION} --query 'taskDefinition.taskDefinitionArn'
</pre>