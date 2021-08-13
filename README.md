<!-- Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
SPDX-License-Identifier: Apache-2.0 -->

# Amazon API Gateway CORS Configurator

The CORS Configurator helps you properly configure CORS for Amazon API Gateway and HTTP APIs. The CORS configurator assumes you are using AWS SAM to build an API Gateway endpoint with a proxy integration to an AWS Lambda function.

Fill in the information on the left side and the CORS Configurator will generate a snippet for a AWS SAM. You can insert this snippet as a global setting to affect all APIs in the template or at the individual API rtesource level.

The CORS Configurator also generates an example proxy response that is appropriate to the API type you are using.

A working version of this tool can be found at [https://serverlessland.com/tools/cors](https://serverlessland.com/tools/cors)

## Run locally
You can also run the code locally

### Requirements
* Node
* VueJS 3+

### Run locally
From the root of the project run the following

```
npm install
```
Then
```
npm run dev
```

The local version will be available at https://localhost:3000