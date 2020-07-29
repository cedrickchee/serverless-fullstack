# Serverless Full Stack App on AWS

An example project that shows how to setup and build a frontend for Serverless and a Node.js REST API service allowing you to create, get, authenticate our user, and validate user JWT tokens. We will deploy this to AWS Lambda. DynamoDB is our data store.

What is our project:
- Multiple microservices
- A simple frontend (work-in-progress)
- User management
- CRM type app with multiple related entities (work-in-progress)
    - Contacts entity
    - Companies entity
- Each entity is its own modular service with asynchronous communication

## Setup

First, install the Serverless Framework.

```sh
$ npm install -g serverless

> protobufjs@6.10.1 postinstall /home/cedric/.nvm/versions/node/v12.16.0/lib/node_modules/serverless/node_modules/protobufjs
> node scripts/postinstall


> serverless@1.76.1 postinstall /home/cedric/.nvm/versions/node/v12.16.0/lib/node_modules/serverless
> node ./scripts/postinstall.js


   ┌───────────────────────────────────────────────────┐
   │                                                         │
   │   Serverless Framework successfully installed!          │
   │                                                         │
   │   To start your first project run 'serverless'.         │
   │                                                         │
   └───────────────────────────────────────────────────┘

+ serverless@1.76.1
added 752 packages from 501 contributors in 33.039s
```

Next, sign up for a Serverless account.

Then, head over to Serverless Dashboard and look for your deployment "profiles".
Go to your "dev" profile and add an additional "parameters" to that:

```text
JWT_SECRET: xxxxxxxxxxxxx
```

## Build serverless endpoint

Create a User service:
- User management service
- We would use an external authentication provider (which is recommended) - Cognito or Auth0 - we're going with Auth0
- We will create a custom authoriser
- Manage user state (user CRUD)
- Login request
- Token generation

### Create User service

This project was created with this command.

```sh
$ serverless create --template-path ./serverless-template --path user-service
Serverless: Generating boilerplate...
```

Create a Lambda function for authenticating our user:

```sh
$ serverless create function -f createUser --handler src/functions/createUser.createUser --path src/tests
Serverless: Generating function...
Serverless: Created function file "/home/cedric/m/dev/scratch/serverless/user-service/src/functions/createUser.js"
Serverless: serverless-mocha-plugin: created src/tests/createUser.js
```

Create a Lambda function for validating user JWTs:

```sh
$ serverless create function -f validate --handler src/functions/validate.validate --path src/tests
Serverless: Generating function...
Serverless: Created function file "/home/cedric/m/dev/scratch/serverless/user-service/src/functions/validate.js"
Serverless: serverless-mocha-plugin: created src/tests/validate.js
```

### Deploy

Deploy via Serverless Dashboard profile.

```sh
$ sls login
Serverless: Logging you in via your default browser...
... trucated ...
Serverless: You sucessfully logged in to Serverless.
Serverless: Please run 'serverless' to configure your service
```

Deploy the endpoint:

```sh
$ serverless deploy
```

The expected result should be similar to:

```sh
$ sls deploy
Serverless: Packaging service...
Serverless: Excluding development dependencies...
Serverless: Installing dependencies for custom CloudFormation resources...
Serverless: Safeguards Processing...
Serverless: Safeguards Results:

   Summary --------------------------------------------------

   passed - allowed-regions
   passed - allowed-runtimes
   passed - allowed-stages
   passed - framework-version
   passed - no-secret-env-vars
   passed - no-unsafe-wildcard-iam-permissions
   warned - require-cfn-role

   Details --------------------------------------------------

   1) Warned - no cfnRole set
      details: http://slss.io/sg-require-cfn-role
      Require the cfnRole option, which specifies a particular role for CloudFormation to assume while deploying.


Serverless: Safeguards Summary: 6 passed, 1 warnings, 0 errors
Serverless: Creating Stack...
Serverless: Checking Stack create progress...
........
Serverless: Stack create finished...
Serverless: Uploading CloudFormation file to S3...
Serverless: Uploading artifacts...
Serverless: Uploading service user-service.zip file to S3 (138.58 KB)...
Serverless: Uploading custom CloudFormation resources...
Serverless: Validating template...
Serverless: Updating Stack...
Serverless: Checking Stack update progress...
..................................................................
Serverless: Stack update finished...
Service Information
service: user-service
stage: dev
region: us-east-1
stack: user-service-dev
resources: 33
api keys:
  None
endpoints:
  POST - https://XXXXXXXXXX.execute-api.us-east-1.amazonaws.com/dev/v1/user/login
  POST - https://XXXXXXXXXX.execute-api.us-east-1.amazonaws.com/dev/v1/user
functions:
  loginUser: user-service-dev-loginUser
  createUser: user-service-dev-createUser
layers:
  None
Serverless: Publishing service to the Serverless Dashboard...
Serverless: Successfully published your service to the Serverless Dashboard: https://dashboard.serverless.com/tenants/{username}/applications/{application name}/services/user-service/stage/dev/region/us-east-1
```

## Usage

Send an HTTP request directly to the endpoint using a tool like curl.

- Register a new user:

```sh
$ curl --request POST --url https://XXXXXXX.execute-api.us-east-1.amazonaws.com/dev/v1/user --data '{"username":"cedric","password":"superpass321"}' -H 'Content-Type: application/json' -i
HTTP/1.1 201 Created
Content-Type: application/json
Content-Length: 0
Connection: keep-alive
Date: Wed, 29 Jul 2020 04:31:52 GMT
x-amzn-RequestId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: Authorization
x-amz-apigw-id: XXXXXXXXXXXXXXX=
X-Amzn-Trace-Id: Root=1-xxxxxx-xxxxxxxxxxxxxxxx;Sampled=0
Access-Control-Allow-Credentials: true
X-Cache: Miss from cloudfront
Via: 1.1 xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.cloudfront.net (CloudFront)
X-Amz-Cf-Pop: FRA2-C1
X-Amz-Cf-Id: xxxxxxxxxxxxxxxxxxxxxxxxxx-xxxxxxxxxxxxxxxxx==
```

- Authenticate a user and and return a JWT:

```sh
$ curl --request POST --url https://XXXXXXX.execute-api.us-east-1.amazonaws.com/dev/v1/user/login --data '{"username":"cedric","password":"superpass321"}' -H 'Content-Type: application/json'

HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 141
Connection: keep-alive
Date: Wed, 29 Jul 2020 13:23:44 GMT
x-amzn-RequestId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx
x-amz-apigw-id: XXXXXXXXXXXXXXX=
X-Amzn-Trace-Id: Root=1-xxxxxx-xxxxxxxxxxxxxxxx;Sampled=0
X-Cache: Miss from cloudfront
Via: 1.1 xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.cloudfront.net (CloudFront)
X-Amz-Cf-Pop: AMS54-C1
X-Amz-Cf-Id: xxxxxxxxxxxxxxxxxxxxxxxxxx-xxxxxxxxxxxxxxxxx==

{"token":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"}
```

That's it. We're done.

You can remove your service:

```sh
$ serverless remove

Serverless: Getting all objects in S3 bucket...
Serverless: Removing objects in S3 bucket...
Serverless: Removing Stack...
Serverless: Checking Stack removal progress...
.............................
Serverless: Stack removal finished...
Serverless: Publishing service to the Serverless Dashboard...
Serverless: Successfully published your service to the Serverless Dashboard: https://dashboard.serverless.com/tenants/{username}/applications/{app name}/services/{service name}/stage/dev/region/{AWS region}
```
