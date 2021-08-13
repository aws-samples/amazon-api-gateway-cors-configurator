<!-- Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
SPDX-License-Identifier: Apache-2.0 -->
<template>
  <section v-if="data.AllowOrigins && data.AllowMethods">
    <article class="message">
      <div class="message-header">
        <p>SAM configuration for HTTP API</p>
      </div>
      <div class="message-body content">
        <p>This SAM confguration configures HTTP API to handle <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#preflighted_requests" target="_blank">preflight</a> requests to the API.
          The following examples demonstrate adding CORS configuration to the <a href="https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification-template-anatomy-globals.html" target="_blank">Globals</a> section to affect all HTTP APIs in the template, including endpoints built as an eventsource for a Lambda function. 
          Or to the <a href="https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-resource-httpapi.html" target="_blank">HTTP API resource</a> to affect that API only.
        </p>
      </div>
    </article>
    <div class="columns">
      <div class="column is-half">
        <h4 class="is-size-4">Add to Globals section</h4>
        <div class="notification code">
<pre>
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: CORS via Globals

Globals:
  HttpApi:
    <span class="highlight">CorsConfiguration:
      AllowedOrigin: <span v-if="data.AllowOrigins[0] == '*'">*</span><span v-else v-for="origin in data.AllowOrigins" :key="origin"><br/>        - {{origin}}</span>
      AllowHeaders: <span v-for="header in data.AllowHeaders" :key="header"><br/>        - {{header}}</span>
      AllowMethods: <span v-if="data.AllowMethods[0] == '*'">*</span><span v-else v-for="method in data.AllowMethods" :key="method"><br/>        - {{method}}</span></span>

Resources:
  MyFunction:
...
</pre>         
        </div>
      </div>
      <div class="column is-half">
        <h4 class="is-size-4">Add to HTTP API resource</h4>
        <div class="notification code">
<pre>
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: CORS via resource

Resources:
  MyHttpApi:
    Type: AWS::Serverless::HttpApi
    Properties:
      <span class="highlight">CorsConfiguration:
        AllowedOrigin: <span v-if="data.AllowOrigins[0] == '*'">*</span><span v-else v-for="origin in data.AllowOrigins" :key="origin"><br/>          - {{origin}}</span>
        AllowHeaders: <span v-for="header in data.AllowHeaders" :key="header"><br/>          - {{header}}</span>
        AllowMethods: <span v-if="data.AllowMethods[0] == '*'">*</span><span v-else v-for="method in data.AllowMethods" :key="method"><br/>          - {{method}}</span></span>

...
</pre>
        </div>
      </div>
    </div>
    <article v-if="warnings.length > 0" class="message is-warning">
      <div class="message-header">
        Caution
      </div>
      <div class="message-body content">
        <ul>
          <li v-for="warning in warnings" :key="warning">{{warning}}</li>
        </ul>
      </div>
    </article>
    <article class="message">
      <div class="message-header">
        <p>HTTP API example responses</p>
      </div>
      <div class="message-body content">
        <p>
          HTTP API handles two types of responses from the backend proxy based on the <a href="https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-lambda.html" target="_blank">payload format</a>. This affects both the payload that HTTP API sends to the Lambda function and the response it expects back from the Lambda function. By default, HTTP API uses payload format version 2.
        </p>
      </div>
    </article>
    <div class="columns">
      <div class="column">
        <h4 class="is-size-4">Default response example (v2)</h4>
        <div class="code notification">
          <pre>"hello world"</pre>
        </div>
      </div>
      <div class="column">
        <h4 class="is-size-4">Legacy response example (v1)</h4>
        <div class="code notification">
          <pre>
{
  statusCode: 200,
  body: {message: "Hello world"},
  headers: {
    <span class="highlight">allow-access-control-origin: "{{p1Origin}}"</span>
  }
}</pre>
    </div>
      </div>
    </div>
    <div class="message is-info">
      <div class="message-header">
        Did you know?
      </div>
      <div class="message-body content">
        <p>
          When using SAM, CDK or other frameworks to create a serverless backend with API Gateway HTTP API and AWS Lambda functions, the integration is automatically created as a
          <a href="https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-lambda.html" target="_blank">proxy</a>.
        </p>
        <p>
          HTTP API supports two <a href="https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-lambda.html" target="_blank">payloads</a> 
          that affect how the data is sent to the Lambda function and the expected response from the Lambda function. By default, HTTP API 
          uses <strong>payload v2</strong>, which means:
          <ul>
            <li>HTTP API dynamically handles preflight <strong>OPTIONS</strong> requests.</li>
            <li>HTTP API does not modify the <strong>request</strong> but does modify the <strong>response</strong>.
              <ul>
                <li>HTTP API adds the <strong>statusCode</strong>.</li>
                <li>HTTP API adds the <strong>CORS related headers</strong>.</li>
              </ul>
            </li>
          </ul>
        </p>
        <p>To configure HTTP API to emulate REST APIs by not modifying the <strong>response</strong>. Set the payload option to v1.</p>
        <p>For more information see the <a href="https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-cors.html" target="_blank">CORS documentation for HTTP APIs</a>.</p>
      </div>
    </div>
  </section>
</template>

<script setup>
import { defineProps, computed, ref } from 'vue'
import * as yaml from 'js-yaml'
import { copyText } from 'vue3-clipboard'

const props = defineProps({
  data: {}
})

const p1Origin = computed(() => {
  let firstEntry = props.data.AllowOrigins[0]
  return (firstEntry.length > 1 && firstEntry.indexOf('*') !== -1) ? "[the origin of the request]" : firstEntry;
})

const warnings = computed(() => {
  let warn = []
  if(props.data.AllowOrigins[0] == "*") warn.push("While using the '*' option is possible, it is not best practice for security.")
  if(!props.data.AllowMethods.includes("OPTIONS") && !props.data.AllowMethods.includes("*")) warn.push("The OPTIONS method is not selected. This can affect your preflight requests.")
  return warn
})

const doCopy = () => {
  copyText(yml, undefined, (error, event) => {
    if (error) {
      alert('Can not copy')
      console.log(error)
    } else {
      alert('Copied')
      console.log(event)
    }
  })
}
</script>

<style lang="scss">
</style>