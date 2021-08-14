<!-- Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
SPDX-License-Identifier: Apache-2.0 -->
<template>
  <section v-if="data.AllowOrigins && data.AllowMethods">
    <article class="message">
      <div class="message-header">
        <p>SAM configuration for REST API</p>
      </div>
      <div class="message-body content">
        <p>This SAM confguration creates an <strong>OPTIONS</strong> method on the REST API resource. The method integration will be of type, <strong>mock</strong> and will handle <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#preflighted_requests" target="_blank">preflight</a> requests to the API.
          The following examples demonstrate adding CORS configuration to the <a href="https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification-template-anatomy-globals.html" target="_blank">Globals</a> section to affect all REST APIs in the template, including endpoints built as an eventsource for a Lambda function. 
          Or to the <a href="https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-resource-api.html" target="_blank">REST API resource</a> to affect that API only.
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
  Api:
    <span class="highlight">Cors:
      AllowOrigin: "'{{data.AllowOrigins[0]}}'"
      AllowHeaders: "'{{data.AllowHeaders.toString()}}'"
      AllowMethods: "'{{data.AllowMethods.toString()}}'"</span>

Resources:
  MyFunction:
...
</pre>         
        </div>
      </div>
      <div class="column is-half">
        <h4 class="is-size-4">Add to REST API resource</h4>
        <div class="notification code">
<pre>
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: CORS via resource

Resources:
  MyRestApi:
    Type: AWS::Serverless::Api
    Properties:
      StageName: Prod
      <span class="highlight">Cors:
        AllowOrigin: "'{{data.AllowOrigins[0]}}'"
        AllowHeaders: "'{{data.AllowHeaders.toString()}}'"
        AllowMethods: "'{{data.AllowMethods.toString()}}'"</span>
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
        <p>REST API example response</p>
      </div>
      <div class="message-body content">
        <p>
          Below is an example of a response from a proxied Lambda function response. The <strong>Allow-Access-Control-Origin</strong> header is required and returned directly to the client without any modification from REST API.
        </p>
      </div>
    </article>
    <div class="code notification">
<pre>
{
  <span class="highlight">statusCode: 200,</span>
  body: {message: "Hello world"},
  headers: {
    <span class="highlight">access-control-allow-origin: "{{data.AllowOrigins[0]}}"</span>
  }
}
</pre>
    </div>
    <div class="message is-info">
      <div class="message-header">
        Did you know?
      </div>
      <div class="message-body content">
        <p>
          When using SAM, CDK or other frameworks to create a serverless backend with API Gateway REST API and AWS Lambda functions, the integration is automatically created as a
          <a href="https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html" target="_blank">proxy</a>. A proxy integration
          on REST API means:
          <ul>
            <li>REST API creates a MOCK integration to process preflight <strong>OPTIONS</strong> calls.</li>
            <li>The preflight MOCK integration can only allow a single domain (https://mydomain.com) or all domains (*). If you would like to allow more than a single domain but not all domains, the convert the OPTIONS endpoint to use a Lambda function for dynamic responses.</li>
            <li>REST API does not modify the <strong>request</strong> or <strong>response</strong> in any way.</li>
            <li>The Lambda function must return the <strong>statusCode</strong></li>
            <li>The Lambda function must return the <strong>CORS related headers</strong></li>
          </ul>
        </p>
        <p>For more information see the <a href="https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-cors.html" target="_blank">CORS documentation for REST APIs</a>.</p>
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
  .code.notification{
    border: .15rem solid gray;
    padding:2rem !important;
    background-color:black;
    pre{
      background-color: transparent !important;
      color:gray;
      padding:0rem;
      .highlight{
        color:yellow;
      }
    }
  }
</style>