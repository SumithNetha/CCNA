# Skill 26 — Network Automation and SDN

# Lesson 03: Working with RESTful APIs

> **Core idea:** A RESTful API lets software communicate with a system using the **HTTP request/response model**. In network automation, REST APIs allow scripts, controllers, and management platforms to **read, create, modify, and delete network resources programmatically**.

This lesson is important because it connects everything you've learned so far:

```text
Network Automation
       ↓
Software needs to communicate with network infrastructure
       ↓
API
       ↓
REST
       ↓
HTTP requests + responses
       ↓
CRUD operations
       ↓
Automated network management
```

---

# 1. What Is an API?

Before understanding REST, understand **API**.

**API = Application Programming Interface**

An API is an interface that allows one software system to communicate with another software system in a defined way.

Think of an API as a **doorway**.

```text
┌──────────────────┐
│ Application      │
│ / Script         │
└────────┬─────────┘
         │
         │ API
         ↓
┌──────────────────┐
│ Network System   │
│ / Controller     │
│ / Device         │
└──────────────────┘
```

The application doesn't necessarily need to understand the internal implementation of the system.

It just needs to know:

* Where to send the request
* What format to use
* What information to provide
* What response to expect

---

# 2. What Is REST?

**REST = Representational State Transfer**

REST is an **architectural style** for designing network-accessible services/APIs.

REST commonly uses:

* HTTP
* URLs/URIs
* HTTP methods
* HTTP status codes
* Representations of resources, commonly JSON

The lesson's main idea is that REST uses the same basic **request → response** model that the Web uses.

```text
Client
  │
  │ HTTP Request
  ↓
Server / API
  │
  │ HTTP Response
  ↓
Client
```

---

# 3. REST Is Based on the Web

When you visit a website:

```text
Browser
   │
   │ HTTP request
   ↓
Web Server
   │
   │ HTTP response
   ↓
Browser
```

For example, conceptually:

```text
Browser:
"Give me this webpage."

Server:
"Here it is."
```

REST APIs use essentially the same communication model.

Instead of requesting a webpage:

```text
Browser → Web Server
```

you might have:

```text
Python Script → Network Controller
```

The communication still follows the HTTP request/response pattern.

---

# 4. REST and Network Automation

This is where REST becomes particularly useful to you as a network engineer.

Traditional approach:

```text
Engineer
   ↓
SSH
   ↓
Cisco device
   ↓
CLI commands
```

REST automation:

```text
Automation Software
        ↓
    REST API
        ↓
Network Controller
        ↓
Network Devices
```

Instead of an administrator manually clicking through a GUI or typing commands, software can send API requests.

This allows you to:

* Retrieve information
* Create resources
* Modify configuration
* Delete resources
* Automate repetitive operations
* Schedule operations
* Integrate different systems

---

# 5. REST Is Request and Response

A REST interaction has two major components:

### Request

The client asks the server to perform an operation.

```text
Client
  │
  │ HTTP Request
  ↓
Server
```

### Response

The server tells the client what happened.

```text
Server
  │
  │ HTTP Response
  ↓
Client
```

So:

```text
REQUEST
   ↓
PROCESSING
   ↓
RESPONSE
```

This is the fundamental REST communication model.

---

# 6. Client and Server

You'll frequently see these two terms.

## Client

The **client** initiates the request.

Examples:

* Web browser
* Python script
* Postman
* Automation platform
* Network management application

## Server

The **server** receives and processes the request.

It could be:

* Web server
* Network controller
* Cloud service
* Network device/API server

Example:

```text
Python Script
    │
    │ REST API request
    ↓
Cisco Controller
    │
    │ response
    ↓
Python Script
```

---

# 7. Resources

REST APIs generally work with **resources**.

A resource is something that the API allows you to interact with.

For networking, resources might represent:

* Devices
* Interfaces
* VLANs
* Users
* Routes
* Wireless networks
* Configuration objects
* Interface statistics

For example:

```text
Network Controller
       │
       ├── Devices
       ├── Interfaces
       ├── VLANs
       ├── Users
       └── Policies
```

An API provides ways to interact with these resources.

---

# 8. URLs / Endpoints

An API generally exposes specific **endpoints** through which resources can be accessed.

For example, conceptually:

```text
https://controller.example/api/devices
```

Here:

```text
https://
     ↓
Protocol

controller.example
     ↓
Server

/api/devices
     ↓
API endpoint/resource
```

The exact endpoint depends on the API implementation.

### Important term: Endpoint

An **endpoint** is a specific URL/API location through which a particular API resource or operation can be accessed.

---

# 9. CRUD — The Four Big API Operations

The lesson introduces **CRUD**.

CRUD stands for:

> **C — Create**
> **R — Read**
> **U — Update**
> **D — Delete**

These represent the fundamental types of operations you commonly perform against resources.

---

# 10. Create

**Create** means creating a new resource.

Networking example:

> Create a new VLAN.

Conceptually:

```text
API Client
    │
    │ CREATE VLAN
    ↓
Controller
    │
    ↓
New VLAN
```

---

# 11. Read

**Read** means retrieving information.

Networking example:

> Read interface statistics.

```text
API Client
    │
    │ READ interface statistics
    ↓
Controller
    │
    ↓
Interface information
```

The response might contain information such as:

```text
Interface
Status
Speed
Packets
Errors
Utilization
```

---

# 12. Update

**Update** means modifying an existing resource.

For example:

> Change a configuration value.

```text
Existing configuration
        ↓
       API
        ↓
Updated configuration
```

Examples could include:

* Updating a password
* Changing an interface description
* Modifying a policy
* Updating a configuration object

---

# 13. Delete

**Delete** means removing a resource.

For example:

> Delete an interface description or configuration object.

```text
Existing Resource
       ↓
    DELETE
       ↓
Resource removed
```

---

# 14. CRUD + Networking

Here's the important mapping:

| CRUD       | Meaning              | Networking Example            |
| ---------- | -------------------- | ----------------------------- |
| **Create** | Create resource      | Create VLAN                   |
| **Read**   | Retrieve information | Read interface statistics     |
| **Update** | Modify resource      | Change configuration          |
| **Delete** | Remove resource      | Delete resource/configuration |

### Memorize:

> **CRUD = Create, Read, Update, Delete**

---

# 15. CRUD and HTTP Methods

This is an important extension of the lesson.

REST commonly maps CRUD operations to HTTP methods.

| CRUD       | Common HTTP Method |
| ---------- | ------------------ |
| **Create** | `POST`             |
| **Read**   | `GET`              |
| **Update** | `PUT` / `PATCH`    |
| **Delete** | `DELETE`           |

So:

```text
CREATE → POST
READ   → GET
UPDATE → PUT / PATCH
DELETE → DELETE
```

### Important distinction

**CRUD describes the operation.**

**HTTP methods describe how the HTTP request expresses that operation.**

For example:

```text
CRUD:
Read

HTTP:
GET
```

---

# 16. GET

`GET` is generally used to **retrieve information**.

Example:

```text
GET /api/devices
```

Conceptually:

> "Give me the devices."

The server might respond with:

```text
200 OK
```

and data describing the devices.

---

# 17. POST

`POST` is commonly used to **create a new resource** or submit data for processing.

Example:

```text
POST /api/vlans
```

with information describing the VLAN.

Conceptually:

> "Create this VLAN."

---

# 18. PUT

`PUT` is commonly used to **replace or update a resource**.

Conceptually:

```text
PUT /api/interfaces/1
```

> "Update this interface resource with this representation."

---

# 19. PATCH

`PATCH` is commonly used for a **partial update**.

For example, instead of replacing an entire interface configuration:

```text
PATCH /api/interfaces/1
```

you might change only one property.

Conceptually:

```text
Interface
 ├── description
 ├── speed
 ├── duplex
 └── status

PATCH
 ↓

Change only:
description
```

---

# 20. DELETE

`DELETE` removes a resource.

Conceptually:

```text
DELETE /api/vlans/20
```

> "Delete VLAN 20."

---

# 21. The HTTP Request

A REST request can contain several important components.

Conceptually:

```text
HTTP METHOD
     ↓
GET /api/devices
     ↓
Headers
     ↓
Authentication
     ↓
Optional request body
```

For example:

```text
GET /api/devices
Host: controller.example
Authorization: Bearer <token>
Accept: application/json
```

Don't worry about memorizing the exact syntax yet. The important idea is that the client sends a structured HTTP request to an API endpoint.

---

# 22. HTTP Response

The server responds.

Conceptually:

```text
HTTP/1.1 200 OK

{
   "device": "Router1",
   "status": "up"
}
```

The response contains:

* HTTP status code
* Headers
* Response body/data

The lesson puts particular emphasis on **status codes**.

---

# 23. HTTP Status Codes

HTTP status codes tell you broadly what happened to your request.

They are grouped into five major categories:

```text
1xx → Informational
2xx → Success
3xx → Redirection
4xx → Client error
5xx → Server error
```

The lesson specifically focuses on:

```text
200 → Success
300 → Redirection
400 → Client-side problem
500 → Server-side problem
```

For CCNA-level understanding, these categories are important.

---

# 24. 2xx — Success

A **2xx** response generally means the request succeeded.

Common examples:

### 200 OK

The request succeeded.

```text
Client
  │
  │ GET
  ↓
Server
  │
  │ 200 OK
  ↓
Data
```

### 201 Created

The request successfully created a resource.

For example:

```text
POST /api/vlans
       ↓
201 Created
```

---

# 25. 3xx — Redirection

A **3xx** response generally means the client needs to follow another direction or the requested resource has been moved/redirected.

Conceptually:

```text
Client
  ↓
Request
  ↓
Server
  ↓
3xx
  ↓
"Go somewhere else / take another step"
```

You don't need to memorize every 3xx status code for this lesson.

---

# 26. 4xx — Client Errors

A **4xx** response generally means there is something wrong with the request from the client side.

Examples:

### 400 Bad Request

The request isn't valid.

### 401 Unauthorized

Authentication is missing or invalid.

### 403 Forbidden

The server understood the request but refuses to allow the operation.

### 404 Not Found

The requested resource/endpoint wasn't found.

---

# 27. Why 404 Is Important

Suppose you send:

```text
GET /api/devicess
```

but the correct endpoint is:

```text
GET /api/devices
```

You could receive:

```text
404 Not Found
```

The important troubleshooting thought process is:

> **Check what I sent first.**

Possible problems:

* Wrong URL
* Wrong endpoint
* Wrong resource
* Typographical error
* Incorrect API path

This is why the lesson says not to immediately blame the server when you receive a 4xx error.

---

# 28. 5xx — Server Errors

A **5xx** response generally means the server encountered a problem processing the request.

Example:

```text
500 Internal Server Error
```

The request may have reached the server correctly, but the server failed while processing it.

Conceptually:

```text
Client
  │
  │ Valid request
  ↓
Server
  │
  │ Something failed
  ↓
500 Internal Server Error
```

### Troubleshooting mindset

```text
4xx → Investigate my request/client
5xx → Investigate server-side problem
```

This isn't absolute—real troubleshooting can be more nuanced—but it's an excellent first classification.

---

# 29. HTTP Status Code Cheat Sheet

|     Code | Meaning               | Think                           |
| -------: | --------------------- | ------------------------------- |
|  **200** | OK                    | Success                         |
|  **201** | Created               | Resource created                |
| **300s** | Redirection           | Go elsewhere / another step     |
|  **400** | Bad Request           | Request problem                 |
|  **401** | Unauthorized          | Authentication issue            |
|  **403** | Forbidden             | Not allowed                     |
|  **404** | Not Found             | Resource/endpoint doesn't exist |
|  **500** | Internal Server Error | Server problem                  |

### Memory trick

```text
2xx → "I did it."
3xx → "Go somewhere else."
4xx → "You sent something wrong."
5xx → "I broke."
```

---

# 30. Authentication and API Security

This is one of the most important real-world sections.

If an API allows you to:

* Create
* Update
* Delete
* Configure

network resources, you absolutely need authentication and authorization.

Otherwise:

```text
Internet
   ↓
API
   ↓
Anyone can modify network
```

That would obviously be dangerous.

---

# 31. API Tokens

The lesson introduces **tokens**.

A token is a credential that can be used to prove that a client has been authenticated/authorized to access an API.

Conceptually:

```text
Client
   │
   │ Username + Password
   ↓
API
   │
   │ Authentication successful
   ↓
Token
```

The client then uses the token for subsequent API requests.

```text
Client
   │
   │ API request + Token
   ↓
API Server
   │
   ↓
Response
```

---

# 32. Token-Based Authentication

The general workflow is:

```text
STEP 1
Client authenticates
        ↓
STEP 2
Server validates credentials
        ↓
STEP 3
Server issues token
        ↓
STEP 4
Client sends token with requests
        ↓
STEP 5
Server validates token
        ↓
STEP 6
Server allows/denies operation
```

This is common in API-based systems.

---

# 33. Why Tokens Are Dangerous

A token can provide significant access.

If an attacker obtains a valid token, they may be able to perform actions as the token's authorized identity.

Think of it like a temporary access badge:

```text
TOKEN
  ↓
Proof of access
  ↓
API
```

Therefore:

> **Treat API tokens like passwords.**

Never casually share them.

Don't put real credentials/tokens into:

* Screenshots
* Public Git repositories
* Chat messages
* Documentation
* Source code committed to Git

unless appropriately protected/redacted.

---

# 34. Token Security

The lesson mentions several ways tokens can be constrained, including:

* Expiration
* IP restrictions
* Additional authentication requirements

The broader principle is:

> **Limit the lifetime and scope of credentials whenever possible.**

A short-lived token is generally safer than a credential that remains valid indefinitely.

---

# 35. JSON

REST APIs commonly exchange structured data.

One format you'll encounter heavily is:

**JSON = JavaScript Object Notation**

Example:

```json
{
  "hostname": "R1",
  "interface": "GigabitEthernet0/1",
  "status": "up"
}
```

This represents structured information.

You can think of it as:

```text
Property → Value
```

For example:

```text
hostname → R1
interface → GigabitEthernet0/1
status → up
```

Your Castle Rysen RFP specifically mentions exploring REST-based APIs and learning the characteristics of **JSON-encoded data**. 

---

# 36. Why JSON Is Useful

Humans can read it relatively easily, while software can parse it efficiently.

For example:

```json
{
  "vlan": 20,
  "name": "PATRON-DEVICES"
}
```

An automation script can read this data and use it to configure systems.

This is one reason APIs and automation work so well together.

---

# 37. Postman

The lesson recommends **Postman** as a practical tool for working with APIs.

Postman allows you to manually construct and test API requests.

Instead of immediately writing Python:

```text
Python
  ↓
API
```

you can first test:

```text
Postman
  ↓
API
```

This is extremely useful when learning.

---

# 38. What Can You Configure in Postman?

You can typically specify:

* HTTP method
* URL
* Headers
* Authentication
* Request body
* Parameters

Then send the request and inspect:

* Status code
* Response headers
* Response body
* Error messages

Conceptually:

```text
┌─────────────────────────┐
│ Postman                 │
│                         │
│ Method: GET             │
│ URL: /api/devices       │
│ Auth: Token             │
│ Headers: JSON           │
└───────────┬─────────────┘
            │
            ↓
        Send Request
            │
            ↓
          API
            │
            ↓
       HTTP Response
```

---

# 39. Why Postman Is Valuable for Network Engineers

Suppose your automation isn't working.

Instead of debugging:

```text
Python code
     +
Authentication
     +
HTTP
     +
JSON
     +
API endpoint
```

all at once, you can test the API manually.

```text
Postman
   ↓
Does endpoint work?
   ↓
Does authentication work?
   ↓
Does request work?
   ↓
Does response look correct?
```

Once you've confirmed the API works manually, you can automate the same request with Python or another automation tool.

---

# 40. A Practical API Workflow

Imagine you want to retrieve information from a network controller.

### Step 1 — Authenticate

```text
Client
  ↓
Credentials
  ↓
Controller
  ↓
Token
```

### Step 2 — Send API request

```text
GET /api/devices
Authorization: Bearer <token>
```

### Step 3 — Controller processes request

```text
Controller
   ↓
Find devices
   ↓
Prepare response
```

### Step 4 — Receive response

```text
200 OK
+
JSON data
```

### Step 5 — Automation software processes data

```text
JSON
 ↓
Python
 ↓
Decision / report / next action
```

---

# 41. Example: Reading Interface Information

Suppose an API exposes an interface resource.

Conceptually:

```text
GET /api/interfaces/1
```

The server might respond:

```json
{
  "interface": "GigabitEthernet0/1",
  "status": "up",
  "speed": "1000Mbps",
  "description": "Uplink"
}
```

Your Python script could then determine:

```text
status == "up"
```

or:

```text
speed == "1000Mbps"
```

This is much more powerful than manually checking every device.

---

# 42. Example: Creating a VLAN

Suppose an API allows VLAN creation.

Conceptually:

```text
POST /api/vlans
```

Request body:

```json
{
  "id": 20,
  "name": "PATRON-DEVICES"
}
```

Possible response:

```text
201 Created
```

The API has now allowed the automation system to create the resource.

---

# 43. Example: Updating a Resource

Suppose VLAN 20 already exists and you want to change its name.

Conceptually:

```text
PUT /api/vlans/20
```

or potentially:

```text
PATCH /api/vlans/20
```

with:

```json
{
  "name": "GUEST-DEVICES"
}
```

The exact behavior depends on the API's specification.

---

# 44. Example: Deleting a Resource

Conceptually:

```text
DELETE /api/vlans/20
```

The server processes the request and may return a success status.

This demonstrates CRUD:

```text
CREATE → POST
READ   → GET
UPDATE → PUT/PATCH
DELETE → DELETE
```

---

# 45. REST + SDN

Now connect this lesson to the previous two lessons.

### Lesson 01

You learned:

```text
Network Automation
        ↓
Automate repetitive tasks

SDN
        ↓
Centralized network control
```

### Lesson 02

You learned:

```text
SD-Access → Campus
SD-WAN    → WAN
ACI       → Data Center
```

### Lesson 03

Now:

```text
Controller
     ↑
     │ REST API
     │
Automation Software
```

The API becomes the communication mechanism between software and the network platform.

---

# 46. Complete Modern Automation Picture

```text
                    ENGINEER
                       │
                       ↓
              Automation System
             /        |        \
          Python    Ansible    Other
             \        |        /
              \       |       /
                    API
                     │
                  REST/HTTP
                     │
             ┌───────┴────────┐
             ↓                ↓
         Controller        Device API
             │
             ↓
       Network Devices
             │
             ↓
       Network Operation
```

This is the big picture you're building toward.

---

# 47. REST vs SSH

You've already learned SSH extensively in your CCNA work.

It's useful to compare them.

### SSH

```text
Engineer / Script
       ↓
      SSH
       ↓
Cisco CLI
       ↓
Commands
```

Example:

```text
configure terminal
interface g0/1
description Uplink
```

### REST API

```text
Application
     ↓
HTTP/REST API
     ↓
Structured request
     ↓
Network system
```

The application doesn't necessarily need to simulate a human typing CLI commands.

---

# 48. SSH Automation Is Still Automation

Don't conclude:

> "REST is automation and SSH isn't."

Both can be used for automation.

```text
Automation
    │
    ├── SSH
    │
    ├── REST API
    │
    ├── NETCONF
    │
    └── Other mechanisms
```

REST is particularly useful because it provides a structured programmatic interface over HTTP.

---

# 49. REST vs GUI

### GUI

```text
Human
 ↓
Click buttons
 ↓
Controller
```

### REST

```text
Software
 ↓
API request
 ↓
Controller
```

The GUI itself may actually use APIs behind the scenes.

That's a very important modern networking concept:

> **A GUI is often just a user-friendly layer over programmatic interfaces.**

---

# 50. Why REST Is So Powerful for Automation

Suppose you have:

```text
100 Network Devices
```

You want to collect:

* Hostnames
* Interface status
* CPU utilization
* Memory utilization

Manually:

```text
Device 1 → Check
Device 2 → Check
Device 3 → Check
...
Device 100 → Check
```

Automation:

```text
Script
  ↓
API
  ↓
Collect information
  ↓
JSON
  ↓
Process results
  ↓
Report
```

This is where REST becomes operationally valuable.

---

# 51. Troubleshooting REST APIs

When an API request fails, don't randomly change everything.

Use a structured approach.

## Step 1 — Check the endpoint

Is the URL correct?

```text
/api/devices
```

vs.

```text
/api/device
```

---

## Step 2 — Check HTTP method

Are you using:

```text
GET
POST
PUT
PATCH
DELETE
```

as required by the API?

---

## Step 3 — Check authentication

Is the token:

* Present?
* Valid?
* Expired?
* Authorized for the operation?

---

## Step 4 — Check headers

For example, the API may expect a particular content type or authentication header.

---

## Step 5 — Check request body

If you're creating/updating something, verify the JSON structure.

---

## Step 6 — Read the status code

```text
2xx → Request succeeded
3xx → Redirection
4xx → Check request/client
5xx → Investigate server
```

---

## Step 7 — Read the response body

The response may provide a useful error message.

---

# 52. Important REST Terminology

| Term         | Meaning                                                        |
| ------------ | -------------------------------------------------------------- |
| **API**      | Interface allowing software to communicate with another system |
| **REST**     | Architectural style commonly implemented using HTTP            |
| **HTTP**     | Protocol used for web/API communication                        |
| **Client**   | System making the request                                      |
| **Server**   | System processing the request                                  |
| **Resource** | Object/data that the API exposes                               |
| **Endpoint** | API URL used to access a resource/function                     |
| **Request**  | Message sent by client                                         |
| **Response** | Message returned by server                                     |
| **CRUD**     | Create, Read, Update, Delete                                   |
| **Token**    | Credential used to authenticate/authorize API requests         |
| **JSON**     | Common structured data format                                  |
| **Postman**  | Tool for testing APIs                                          |

---

# 53. Most Important HTTP Methods

```text
GET
 ↓
Read


POST
 ↓
Create


PUT
 ↓
Update/replace


PATCH
 ↓
Partial update


DELETE
 ↓
Delete
```

### Memorize this table

| HTTP     | CRUD   | Purpose                   |
| -------- | ------ | ------------------------- |
| `GET`    | Read   | Retrieve resource         |
| `POST`   | Create | Create/submit resource    |
| `PUT`    | Update | Replace/update resource   |
| `PATCH`  | Update | Partially modify resource |
| `DELETE` | Delete | Remove resource           |

---

# 54. Status Codes — Memorize These

```text
2xx → SUCCESS
3xx → REDIRECTION
4xx → CLIENT ERROR
5xx → SERVER ERROR
```

Especially:

```text
200 → OK
201 → Created
400 → Bad Request
401 → Unauthorized
403 → Forbidden
404 → Not Found
500 → Internal Server Error
```

### Mental model

> **4xx: Check what I sent.**
> **5xx: Check what's happening on the server.**

---

# 55. Security Mental Model

Think of API security like this:

```text
             API
              │
      ┌───────┴────────┐
      ↓                ↓
Authentication      Authorization
      │                │
"Who are you?"     "What can you do?"
```

A token can prove/represent authenticated access, but **authentication and authorization are separate concepts**.

For example:

```text
Valid token
     ↓
Identity recognized
     ↓
Does this identity have permission
to DELETE VLAN 20?
     ↓
YES → Allow
NO  → Deny
```

---

# 56. API Token vs Password

Think:

```text
Password
   ↓
Used to authenticate

Token
   ↓
Credential/access artifact used
for API requests
```

The lesson's practical advice is particularly important:

> Treat API tokens like passwords.

If someone obtains a token with significant privileges, they may be able to use the API with those privileges.

---

# 57. REST and JSON Together

One common API interaction looks like:

```text
CLIENT
   │
   │ HTTP request
   │
   │ GET /api/devices
   │ Authorization: token
   ↓
SERVER
   │
   │ HTTP response
   │
   │ 200 OK
   │ JSON
   ↓
CLIENT
```

For a configuration operation:

```text
CLIENT
   │
   │ POST
   │ JSON body
   │ Token
   ↓
SERVER
   │
   │ 201 Created
   ↓
CLIENT
```

This is the foundation of modern API-driven automation.

---

# 58. Castle Rysen Example

The Castle Rysen RFP specifically includes **REST-based APIs** and **JSON-encoded data** under its network automation and programmability phase. 

Imagine Castle Rysen eventually wants to automate deployment of a new District Shop.

Instead of manually configuring every component:

```text
Engineer
 ↓
CLI
 ↓
Switch
 ↓
Router
 ↓
AP
```

a future automation system could conceptually do:

```text
Automation Platform
       ↓
      REST
       ↓
Controller/API
       ↓
Network Devices
```

The configuration/data could be represented using JSON:

```json
{
  "site": "District-Shop-01",
  "vlans": [
    {
      "id": 10,
      "name": "ADMIN"
    },
    {
      "id": 20,
      "name": "PATRON"
    }
  ]
}
```

**That example is illustrative**, not a configuration specified by the RFP itself. The RFP establishes the requirement to explore REST APIs and JSON; it does not prescribe this exact API structure. 

---

# 59. The Full Lesson in One Flow

```text
                    REST API
                        │
                        ↓
               HTTP Request/Response
                        │
          ┌─────────────┴─────────────┐
          ↓                           ↓
       REQUEST                     RESPONSE
          │                           │
    HTTP Method                 Status Code
    Endpoint                    Headers
    Headers                     Body
    Body
          │                           │
          ↓                           ↓
        CRUD                        2xx
          │                         3xx
    ┌─────┼─────┐                   4xx
    ↓     ↓     ↓                   5xx
 Create  Read  Update/Delete
    │      │       │
   POST    GET   PUT/PATCH/DELETE
                        │
                        ↓
                       JSON
                        │
                        ↓
                 Network Automation
```

---

# 60. How the Three August 27 Lessons Connect

You have now covered all three pieces of the puzzle.

### Lesson 01 — Network Automation & SDN

```text
WHY?
 ↓
Reduce repetitive work
 ↓
Centralize network control
```

### Lesson 02 — Cisco SDN Models & Platforms

```text
WHERE?
 ↓
Campus → SD-Access
WAN → SD-WAN
Data Center → ACI
```

### Lesson 03 — RESTful APIs

```text
HOW DOES SOFTWARE COMMUNICATE?
 ↓
API
 ↓
REST
 ↓
HTTP
 ↓
CRUD
 ↓
JSON
```

Put them together:

```text
                 NETWORK AUTOMATION
                         │
                         ↓
                        SDN
                         │
              ┌──────────┼──────────┐
              ↓          ↓          ↓
           Campus       WAN     Data Center
              ↓          ↓          ↓
         SD-Access     SD-WAN      ACI
              │          │          │
              └──────────┼──────────┘
                         ↓
                    Controllers
                         ↓
                       APIs
                         ↓
                      REST/HTTP
                         ↓
                  CRUD + JSON
                         ↓
              Automated Operations
```

---

# ⭐ 61. What You Should Memorize

### API

> **An API is a defined interface that allows software to communicate with another system.**

### REST

> **REST is an architectural style commonly implemented using HTTP's request/response model.**

### CRUD

> **Create, Read, Update, Delete.**

### HTTP mapping

```text
POST   → Create
GET    → Read
PUT    → Update/Replace
PATCH  → Partial Update
DELETE → Delete
```

### Status codes

```text
2xx → Success
3xx → Redirection
4xx → Client error
5xx → Server error
```

### Important codes

```text
200 → OK
201 → Created
400 → Bad Request
401 → Unauthorized
403 → Forbidden
404 → Not Found
500 → Internal Server Error
```

### Token

> **A credential/access artifact used with API requests; protect it like a password.**

### JSON

> **A common structured data format used to represent information exchanged through APIs.**

### Postman

> **A tool used to construct, send, test, and troubleshoot API requests and responses.**

---

# 🧠 62. The Most Important Mental Model

If you only remember one diagram from this lesson, use this:

```text
                 APPLICATION
                      │
                      │ API REQUEST
                      ↓
                 REST / HTTP
                      │
        ┌─────────────┴─────────────┐
        │                           │
     Endpoint                   Authentication
        │                           │
        └─────────────┬─────────────┘
                      ↓
                  CONTROLLER
                      │
                      ↓
               NETWORK DEVICES
                      │
                      ↑
                  RESPONSE
                      │
              ┌───────┴────────┐
              ↓                ↓
         Status Code          JSON
              │
      ┌───────┼───────┐
      ↓       ↓       ↓
     2xx     4xx     5xx
   Success  Client   Server
            error    error
```

And the operational model:

> **API is the doorway → REST is the architectural style → HTTP carries the request → CRUD describes the operation → JSON carries structured data → status codes tell you what happened → tokens control access.**

That is the core of **Working with RESTful APIs** and the bridge from your CCNA networking knowledge into actual **network programmability and automation**.
