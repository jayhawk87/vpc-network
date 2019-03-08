---


copyright:
  years: 2018, 2019
lastupdated: "2019-03-08"

keywords: load balancer, public, listener, back-end, front-end, pool, round-robin, weighted, connections, methods, policies, APIs, access, ports

subcollection: vpc-network

---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:note: .note}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# (Beta) Using Load Balancers for VPC
{: #--beta-using-load-balancers-in-ibm-cloud-vpc}

The {{site.data.keyword.cloud}} **Load Balancer for VPC** service distributes traffic among multiple server instances within the same region for your VPC.

This service is now in Beta release version. Resources used during the Beta period will not be available after the close of Beta, and no support will be available. Please direct comments and questions to your IBM Cloud sales representative.
{:important}

## Public load balancer
{: #public-load-balancer}

Your load balancer service instance is assigned a publicly-accessible, fully qualified domain name (FQDN), which you must use for access to your applications that are hosted behind the IBM Cloud Load Balancer for VPC. This domain name may be registered with one or more public IP addresses.

Over time, these public IP addresses and the number of public IP addresses may change, due to maintenance and scaling activities. The back-end server instances (VSIs) hosting your application must run in the same region, and under the same VPC.

## Private Load Balancer
{: #private-load-balancer}

The private load balancer is accessible only to internal clients on your private subnets, within the same region and VPC. The private load balancer accepts traffic only from [RFC1918 ![External link icon](../icons/launch-glyph.svg "External link icon")](https://tools.ietf.org/html/rfc1918){: new_window} address spaces.

Similar to a public load balancer, your private load balancer service instance is assigned a fully qualified domain name (FQDN). However, this domain name is registered with one or more private IP addresses.

IBM Cloud operations may change the number and value of your assigned private IP addresses over time, based on maintenance and scaling activities. The back-end server instances (VSIs) hosting your application must run in the same region, and under the same VPC.

See the following table for a summary comparison of features:

| Feature | Public Load Balancer | Private Load Balancer |
|--------|-------|-------|
| Accessible on internet? |  Yes, with FQDN, Internet | No, internal clients only, on same region and VPC |
| Accepts all traffic? | Yes | No, RFC 1918 only |
| Number of private IPs | changes over time | changes over time |
| How is domain name registered? | public IP addresses | private IP addresses |

## Front-end listeners and back-end pools
You may define up to ten (10) front-end listeners (application ports) and map them to respective back-end pools on the back-end application servers. The fully qualified domain name (FQDN) assigned to your load balancer service instance and the front-end listener ports are exposed to the public internet. Incoming user requests are received on these ports.

The supported front-end listener protocols are:
* HTTP
* HTTPS
* TCP

The supported back-end pool protocols are:
* HTTP
* TCP

Incoming HTTPS traffic is terminated at the load balancer to allow for plain-text HTTP communication with the back-end server.

You may attach up to fifty (50) server instances to a back-end pool. Each attached server instance must have a port configured. The port may or may not be the same as front-end listener port.

The port range of 56500 to 56520 is reserved for management purposes; these ports cannot be used as front-end listener ports.
{: note}

## Load balancing methods
{: #load-balancing-methods}

The following three load balancing methods are available for distributing traffic among back-end application servers:

* **Round-robin:** Round-robin is the default load balancing method. With this method, the load balancer forwards
incoming client connections in round-robin fashion to the back-end servers. As a result, all back-end servers receive
roughly an equal number of client connections.

* **Weighted Round-robin:** With this method, the load balancer forwards incoming client connections to the back-end
servers in proportion to the weight assigned to these servers. Each server is assigned a default weight of 50,
which may be customized to any value between 0 and 100.

As an example, if three application servers A, B and C, have weights customized to 60, 60, and 30 respectively, then servers A and B receive an equal number of connections, while server C receives half that number of connections.

* **Least connections:** With this method, the server instance serving the least number of connections at a given time receives the next client connection.

**Additional characteristics of these methods:**

* Re-setting a server weight to '0' means no new connections are forwarded to that server, but any existing traffic will continue to flow. Using a weight of '0' can help bring down a server gracefully and remove it from service rotation.
* The server weight values are applicable only with the weighted round-robin method. They are ignored with round-robin and least connections load balancing methods.

## Horizontal scaling
The load balancer adjusts its capacity automatically according to the load. When this occurs, you may see a change in the number of IP addresses associated with the load balancer's DNS name.

## Health Checks
{: #health-checks}

Health check definitions are mandatory for back-end pools.

The load balancer conducts periodic health checks to monitor the health of the back-end ports, and it forwards client traffic to them accordingly. If a given back-end server port is found to be unhealthy, no new connections are forwarded to it. The load balancer continues to monitor health of unhealthy ports, and it resumes their use if they become healthy again, which means that they successfully pass two consecutive health check attempts.

The health checks for HTTP and TCP ports are conducted as follows:

* **HTTP:** An `HTTP GET` request against a pre-specified URL is sent to the back-end server port. The server port is marked healthy upon receiving a `200 OK` response. The default `GET` health path is "/" through the UI, and it can be customized.

* **TCP:** The Load Balancer attempts to open a TCP connection with the back-end server on a specified TCP port. The server port is marked healthy if the connection attempt is successful, and the connection is closed.

The default health check interval is 5 seconds, the default timeout against a health check request is 2 seconds, and the default number of retrial attempts is 2.
{: note}

## SSL offloading and required authorizations

For all incoming HTTPS connections, the load balancer service terminates the SSL connection and establishes a plain-text HTTP communication with the back-end server instance. With this technique, CPU-intensive SSL handshakes and encryption or decryption tasks are shifted away from the back-end server instances, thereby allowing them to use all their CPU cycles for processing application traffic.

An SSL certificate is required for the load balancer to perform SSL offloading tasks. You may manage the SSL certificates through [IBM Certificate Manager ![External link icon](../icons/launch-glyph.svg "External link icon")](/docs/services/certificate-manager?topic=certificate-manager-gettingstarted){: new_window}.

For a load balancer to access your SSL certificate, you must enable **service-to-service authorization**, which grants your load balancer service instance access to your certificate manager instance. You may manage such an authorization by following this documentation [Granting access between services ![External link icon](../icons/launch-glyph.svg "External link icon")](/docs/iam?topic=iam-serviceauth#create-auth){: new_window}. Make sure to choose **Infrastructure Service** as the source service, **Load Balancer for VPC** as the resource type, **Certificate Manager** as the target service, and assign the **Writer** service access role.

If the required authorization is removed, errors may occur for your load balancer.
{: note}

## Identity and access management (IAM)

You can configure access policies for a **Load Balancer for VPC** instance. To manage your user access policies, visit [Managing access to resources ![External link icon](../icons/launch-glyph.svg "External link icon")](docs/services/iam?topic=iam-iammanidaccser#resourceaccess){: new_window} for more information on Identity and access management.

### Configuring resource group access policies for users

To create a load balancer, you'll need access to a resource group. The user who is creating a load balancer must have proper access to the resource group provided, or to the default resource group, if none is provided.

1. Navigate to **Manage > Account > Users**. You'll see a list of the users with access to your IBM Cloud account.
2. Select the name of the user to whom you want to assign an access policy. If the user is not shown, click **Invite users** to add the user to your IBM Cloud account.
3. Select **Assign access**.
4. Select **Assign access within a resource group**.
5. From the **Resource group** drop-down list, select the desired resource group.
6. From the **Assign access to a resource group** drop-down list, select the desired access.
7. From the **Services** drop-down list, select the desired services.
9. Select **Assign** to assign the resource group access policy to the user.

### Configuring resource access policies for users

| Platform Access Role | Load Balancer Action |
|-------------|-----|
| Administrator | Create/View/Edit/Delete load balancer |
| Editor | Create/View/Edit/Delete load balancer |
| Viewer | View load balancer |

1. Navigate to **Manage > Account > Users**. You'll see list of the users with access to your IBM Cloud account.
2. Select the name of the user to whom you want to assign an access policy. If the user is not shown, click **Invite users** to add the user to your IBM Cloud account.
3. Select **Assign access**.
4. Select **Assign access to resources**.
5. From the **Services** drop-down list, select **Infrastructure Service**.
6. From the **Resource type** drop-down list, select **Load Balancer for VPC**.
7. From the **Load Balancer ID** drop-down list, select a Load Balancer instance ID, or use the default value, **All load balancers**.
8. Assign a platform access role to the user.
9. Select **Assign** to assign the access policy to the user.

## Activity Tracker Integration

Load balancer service is integrated with **IBM Cloud Activity Tracker** which records events, compliant with CADF standard, triggered by user-initiated activities that change the state of service in the cloud.

The following actions on the load balancer service instances will be recorded as auditing events:
* Create load balancer and its sub-resources (Activity Tracker action string is **POST**)
* Update load balancer and its sub-resources (Activity Tracker action string is **PATCH** or **PUT**)
* Delete load balancer and its sub-resources (Activity Tracker action string is **DELETE**)

All auditing events are recorded to IBM Cloud Activity Tracker in `us-south` region. It does not matter in which region the load balancer service is provisioned.
{:note}

To view events, a user must have an IAM policy that grants the **Reader** role on the Log Analysis service.
More information on granting access is available at [Granting permissions to see account events. ![External link icon](../icons/launch-glyph.svg "External link icon")](https://console.bluemix.net/docs/services/cloud-activity-tracker/how-to/grant_permissions.html#grant_permissions){: new_window}

Users can monitor account-level operations performed on the load balancer service.

1. Navigate to IBM cloud log analysis interface in `us-south`, https://logging.ng.bluemix.net
2. Select **Domain** as Account.
3. Select the **Account** name.
4. Operations specific to the Load balancer can be found by using the search keyword `load_balancers`.

Here is the sample activity tracker message for a **Create Listener** operation:
```bash
"message": {
   "id":         "09debf32b30a9c961abc79c1a8436974",
   "typeURI":    "http://schemas.dmtf.org/cloud/audit/1.0/event",
   "eventType":  "activity",
   "eventTime":  "2019-03-05 18:53:25.504276583   +0000 UTC m=+318902.860688179",
   "action":     "POST",
   "outcome":    "success",
   "initiator":{
      "typeURI": "service/security/account/user",
      "name":    <USER_ID>,
      "host":{
         "address": <CLIENT_IP>,
         "agent":   "IBM_One_Cloud_IS_UI/2.4.2"
      },
      "project_id": <ACCOUNT_ID>
   },
   "target":{
      "id":      "TBD_CRN_From_Request",
      "typeURI": "/v1/load_balancers/59aae82b-27a1-40d0-9c99-ac852c47dcb1/listeners",
      "host":{
         "address": <API_END_POINT>
      }
   },
   "observer":{
      "typeURI": "service/security/edge/activity-tracker",
      "name":    "ActivityTracker",
      "host":{
         "address":  "activity-tracker.ng.bluemix.net"
      }
   },
   "requestPath":  "/v1/load_balancers/59aae82b-27a1-40d0-9c99-ac852c47dcb1/listeners",
   "reason":{
      "reasonCode": 201,
      "reasonType": "https://www.iana.org/assignments/http-status-codes/http-status-codes.xml"
   },
   "severity":  "normal",
   "requestData":{
      "headers":{
         "RayID":  "4c2a2337cbf8202a"
      }
   }
},
"type": "ActivityTracker",
"event_uuid": "ccb5e8d8-a30c-4449-b01b-b38444baa9c9",
"requestPath_str": "/v1/load_balancers/59aae82b-27a1-40d0-9c99-ac852c47dcb1/listeners",
"action_str": "POST",
"target":{
   "id_str": "TBD_CRN_From_Request",
   "typeURI_str": "/v1/load_balancers/59aae82b-27a1-40d0-9c99-ac852c47dcb1/listeners",
   "host_hash":{
      "address_str": <API_END_POINT>
   }
 }
```

## APIs available

To make API calls, you must use some form of REST client. For example, you can use the `curl` command to retrieve all existing load balancers:

```
curl -X GET $rias_endpoint/v1/load_balancers?version=2019-01-01 -H "Authorization: $iam_token"
```
{: pre}

The following section gives details about APIs you can use for load balancers in your VPC environment. For the full spec, see the [RIAS API reference](https://{DomainName}/apidocs/rias#retrieves-all-load-balancers).

| Description | API |
|-------------|-----|
| Creates and provisions a load balancer | POST /load_balancers |
| Retrieves all load balancers | GET /load_balancers |
| Retrieves a load balancer | GET /load_balancers/{id} |
| Deletes a load balancer | DELETE /load_balancers/{id} |
| Updates a load balancer | PATCH /load_balancers/{id} |
| Creates a listener | POST /load_balancers/{id}/listeners |
| Retrieves all listeners of the load balancer | GET /load_balancers/{id}/listeners |
| Retrieves a listener | GET /load_balancers/{id}/listeners/{listener_id} |
| Deletes a listener | DELETE /load_balancers/{id}/listeners/{listener_id} |
| Updates a listener | PATCH /load_balancers/{id}/listeners/{listener_id} |
| Creates a pool | POST /load_balancers/{id}/pools |
| Retrieves all pools of the load balancer | GET /load_balancers/{id}/pools |
| Retrieves a pool | GET /load_balancers/{id}/pools/{pool_id} |
| Deletes a pool | DELETE /load_balancers/{id}/pools/{pool_id} |
| Updates a pool | PATCH /load_balancers/{id}/pools/{pool_id} |
| Creates a member | POST /load_balancers/{id}/pools/{pool_id}/members |
| Retrieves all members of the pool | GET /load_balancers/{id}/pools/{pool_id}/members |
| Retrieves a member |GET /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Deletes a member from the pool | DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Updates a member | PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Updates members of the pool | PUT /load_balancers/{id}/pools/{pool_id}/members |
| Retrieves statistics of a load balancer | GET /load_balancers/{id}/statistics |

## Load balancer example

In the following example, you'll use the API to create a load balancer in front of 2 VPC server instances (`192.168.100.5` and `192.168.100.6`) running a web application listening on port `80`. The load balancer has a front-end listener, which allows access to the web application securely by means of HTTPS. Then you can use the API to get details of the load balancer instance after it is created, and to delete the load balancer instance.

### Example steps

The example steps that follow skip the prerequisite steps of using the [IBM Cloud UI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console), [CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) or [RIAS API](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis) to provision a VPC, subnet(s), and instances.


The load balancer example steps also can be run using the [CLI](/docs/cli/reference/ibmcloud?topic=infrastructure-service-cli-vpc-reference#vpc-reference).
{: note}

#### Step 1. Create a load balancer with listener, pool, and attached server instances (members)

```bash
curl -H "Authorization: $iam_token" -X POST
$rias_endpoint/v1/load_balancers?version=2019-01-01 \
    -d '{
        "name": "example-balancer",
        "is_public": true,
        "listeners": [
            {
                "certificate_instance": {
                    "crn": "crn:v1:bluemix:public:cloudcerts:us-south:a/123456:b8877ea4-b8eg-467e-912a-da1eb7f031cg:certificate:43219c4c97d013fb2a95b21dddde1234"
                },
                "port": 443,
                "protocol": "https",
                "default_pool": {
                    "name": "example-pool"
                }
            }
        ],
        "pools": [
            {
                "algorithm": "round_robin",
                "health_monitor": {
                    "delay": 5,
                    "max_retries": 2,
                    "timeout": 2,
                    "type": "http",
                    "url_path": "/"
                },
                "name": "example-pool",
                "protocol": "http",
                "session_persistence": {
                    "cookie_name": "string",
                    "type": "source_ip"
                },
                "members": [
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.5"
                        },
                        "weight": 50
                    },
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.6"
                        },
                        "weight": 50
                    }
                ]
            }
        ],
        "subnets": [
            {
                "id": "7ec87131-1c7e-4990-b4f0-a26f2e61f98e"
            }
        ]
        }'
```
{: codeblock}

Sample output:
```
{
    "created_at": "2018-07-12T23:17:07.5985381Z",
    "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "hostname": "ac34687d.lb.appdomain.cloud",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "is_public": true,
    "listeners": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
        }
    ],
    "name": "example-balancer",
    "operating_status": "offline",
    "pools": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
            "name": "example-pool"
        }
    ],
    "provisioning_status": "create_pending",
    "resource_group": {
        "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
    },
    "subnets": [
        {
            "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "name": "example-subnet"
        }
    ]
}
```
{: screen}

Save the ID of the load balancer to use in the next steps, for example, in the variable `lbid`.

```
lbid=dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```

#### Step 2. Get a load balancer

```
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/load_balancers/$lbid?version=2019-01-01
```
{: pre}

Allow some time for provisioning. When the load balancer is ready, it will be set to `online` and `active` status, as you'll see in the following sample output:

Sample output:

```bash
{
  "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "name": "example-balancer",
  "created_at": "2018-07-13T22:22:24.489Z",
  "hostname": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud",
  "is_public": true,
  "listeners": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
    }
  ],
  "operating_status": "online",
  "pools": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
      "name": "example-pool"
    }
  ],
  "private_ips": [
    {
      "address": "192.168.10.5"
    },
    {
      "address": "192.168.10.6"
    }
  ],
  "provisioning_status": "active",
  "public_ips": [
    {
        "address": "169.11.111.115"
    },
    {
        "address": "169.11.111.116"
    }
  ],
  "resource_group": {
    "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
  },
  "subnets": [
    {
      "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "name": "example-subnet"
    }
  ]
}
```
{: screen}

#### Step 3. Delete a load balancer

```bash
curl -H "Authorization: $iam_token" -X DELETE $rias_endpoint/v1/load_balancers/$lbid?version=2019-01-01
```
{: pre}

## FAQs

This section contains answers to some frequently asked questions about the **Load Balancer for VPC** service.

### Can I use a different DNS name for my load balancer?

The auto-assigned DNS name for the load balancer is not customizable. However, you can add a CNAME (Canonical Name) record that points your preferred DNS name to the auto-assigned load balancer DNS name. For example, your load balancer ID is `dd754295-e9e0-4c9d-bf6c-58fbc59e5727`, the auto-assigned load balancer DNS name is `dd754295.lb.appdomain.cloud`. Your preferred DNS name is `www.myapp.com`. You may add a CNAME record (through the DNS provider that you use to manage `myapp.com`) pointing `www.myapp.com` to the load balancer DNS name `dd754295.lb.appdomain.cloud`.

### What's the maximum number of front-end listeners I can define with my load balancer?

10.

### What's the maximum number of server instances I can attach to my back-end pool?

50.

### Is the load balancer horizontally scalable?

Yes. The load balancer automatically adjusts its capacity based on the load. When horizontal scaling takes place, the number of IP addresses associated with the load balancer's DNS name will change.

### What should I do if I am using ACLs or security groups on the subnets that are used to deploy the load balancer?

You'll need to ensure that the proper ACL or security group rules are in place to allow incoming traffic for configured listener ports and management ports (ports from 56500 - 56520). Traffic between the load balancer and back-end instances also should be allowed.

### Why am I receiving an error message: `certificate instance not found`?

* The certificate instance CRN may be invalid.
* You may not have granted **service-to-service authorization**. See the **SSL offloading** section of this document.

### Why am I receiving a `401 Unauthorized Error` code?

Check the following access policies for your user:
* The access policy for the Load balancer resource type
* The access policy for the Resource group
* If `HTTPS` listeners are used, also check the service-to-service authorization for the Certificate Manager instance.

### Why is my load balancer in `maintenance_pending` state?

The load balancer will be in `maintenance_pending` state during various maintenance activities such as:
* Horizontal scaling activities
* Recovery activities
* Rolling upgrade to address vulnerabilities and apply security patches

### Why do I need to choose multiple subnets during provisioning?

**Load Balancer for VPC** is Multi-Zone-Region (MZR) ready. Load balancer appliances are deployed to the subnets you've selected. It is highly recommended to choose subnets from different zones, to provide you with higher availability and redundancy.

### Why is back-end member health under my pool `unknown`?

* The pool is not associated with any listeners
* Configuration changes may have been made to the pool or its associated listener

### Which TLS version is supported with SSL offload?

The **Load Balancer for VPC** supports TLS 1.2 with SSL termination.

The following list details the supported ciphers (listed in order of precedence):
* ECDHE-RSA-AES256-GCM-SHA384
* ECDHE-RSA-AES256-SHA384
* AES256-GCM-SHA384
* AES256-SHA256
* ECDHE-RSA-AES128-GCM-SHA256
* ECDHE-RSA-AES128-SHA256
* AES128-GCM-SHA256
* AES128-SHA256

### What are the default settings and allowed values for health check parameters?

The default settings and allowed values are listed below:
* Health check interval: Default is 5 seconds, range is 2 to 60 seconds.
* Health check response timeout: Default is 2 seconds, range is 1 to 59 seconds.
* Maximum retry attempts: Default is 2 retry attempts, and the range is 1 to 10 retries.

The health check response timeout value must always be less than the health check interval value.
{:note}

### Are the load balancer IP addresses fixed?

Load balancer IP addresses are not guaranteed to be fixed, due to the built-in elasticity of the service. During horizontal scaling, you will see changes in the available IPs associated with the FQDN of your load balancer.

Use FQDN, rather than cached IP addresses.
{:note}

### Does the load balancer support Layer 7 switching?

No.

### Why does HTTPS listener creation or update tell me that my certificate is invalid?

Check for these possibilities:

* The provided certificate CRN may be invalid.
* The certificate instance given in the Certificate Manager may not have an associated private key.
