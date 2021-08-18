<!-- Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
SPDX-License-Identifier: Apache-2.0 -->
<template>

  <nav class="navbar is-info" role="navigation" aria-label="main navigation">
    <div class="navbar-brand">
      <div class="navbar-item">
        <h2 class="is-size-5">Amazon API Gateway CORS Configurator (preview)</h2>
      </div>
    </div>
    <div class="navbar-end">
        <a href="https://serverlessland.com" class="navbar-item">Return to Serverless Land</a>
      </div>
  </nav>

  <section class="container is-fluid">
    <div class="columns mt-3">
      <div class="column is-one-quarter">
        <div class="configurator p-5 has-background-light">
          <div class="level">
            <div class="level-item">
                <button class="button is-fullwidth is-small is-link" :class="{'is-light': !v1}" @click="apiVersion = 'v1'">REST API</button>
            </div>
            <div class="level-item">
                <button class="button is-fullwidth is-small is-link" :class="{'is-light': v1}" @click="apiVersion = 'v2'">HTTP API</button>
            </div>
          </div>
          <div class="field">
            <label class="label">Origin
              <span class="icon has-text-info">
                <i v-if="currentInfo == 'origin'" class="fas fa-minus-circle" @click="currentInfo = ''"></i>
                <i v-else class="fas fa-info-circle" @click="currentInfo = 'origin'"></i>
              </span>
            </label>
            <div class="control">
              <input class="input" type="text" placeholder="https://mydomain.com" v-model="origin">
            </div>
            <div v-if="currentInfo == 'origin'" class="notification is-info mt-3 content">
              <p>The <strong>origin</strong> is the domain name of the client calling the server. It must include the protocol (http or https) and the port if other than 80 or 443.</p>
              <div v-if="v1">
                <p>REST APIs support two options:</p>
                <ul>
                  <li>A single domain (<em>https://mydomain.com</em>)</li>
                  <li>All domains (*)</li>
                </ul>
              </div>
              <div v-else>
                <p>HTTP APIs support multiple options:</p>
                <ul>
                  <li>A single origin (<em>https://mydomain.com</em>)</li>
                  <li>Multiple origins seperated by a comma (<em>https://yourdomain.com, http://localhost:3000</em>)</li>
                  <li>All origins (*)</li>
                  <li>Any origin that starts with http (<em>http://*</em>)</li>
                  <li>Any origin that starts with https (<em>https://*</em>)</li>
                </ul>
              </div>
            </div>
          </div>
          <div class="field">
            <label class="label">Methods
              <span class="icon has-text-info">
                <i v-if="currentInfo == 'methods'" class="fas fa-minus-circle" @click="currentInfo = ''"></i>
                <i v-else class="fas fa-info-circle" @click="currentInfo = 'methods'"></i>
              </span>
            </label>
            <div class="control">
              <Multiselect v-model="methods.values" mode="tags" :options="methods.options" placeholder="Methods"/>
            </div>
            <div v-if="currentInfo == 'methods'" class="notification is-info mt-3 content">
              <p>List the methods that the backend server will support. See the dropdown for available choices. Be sure and select the <strong>OPTIONS</strong> method if your browser will make a <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#preflighted_requests" target="_blank">preflight request</a>.</p>
            </div>
          </div>
          <div class="field">
            <label class="label">Common headers</label>
            <label class="checkbox">
              <input type="checkbox" v-model="authorization"> Authorizer
            </label>
            <span class="icon has-text-info">
              <i v-if="currentInfo == 'authorizer'" class="fas fa-minus-circle" @click="currentInfo = ''"></i>
              <i v-else class="fas fa-info-circle" @click="currentInfo = 'authorizer'"></i>
            </span>
            <div class="control" v-if="authorization">
              <input class="input" type="text" v-model="tokenSource" placeholder="Token source">
            </div>
            <div v-if="currentInfo == 'authorizer'" class="notification is-info mt-3 content">
              <div v-if="v1">
                <p>REST API supports <strong>Cognito</strong>, <strong>IAM</strong>, and <strong>Lambda</strong> authorizers.</p>
                <p>If you plan to use a Cognito authorizer check this box and enter the name of the header that will contain the token.</p>
                <p>If you are using a Lambda authorizer you can also opt to pass the token through a header. If so, check this box and enter the name of the header that will contain the token.</p>
                <p>The default value of <em>Authorization</em> will be used if <em>token source</em> is left blank.</p>
              </div>
              <div v-else>
                <p>HTTP API supports <strong>JWT</strong>, <strong>IAM</strong>, and <strong>Lambda</strong> authorizers.</p>
                <p>If you plan to use a JWT authorizer check this box and enter the name of the header that will contain the token.</p>
                <p>If you are using a Lambda authorizer you can also opt to pass the token through a header. If so, check this box and enter the name of the header that will contain the token.</p>
                <p>The default value of <em>Authorization</em> will be used if <em>token source</em> is left blank.</p>
              </div>
            </div>
          </div>
          <div class="field" v-if="v1">
              <label class="checkbox">
                <input type="checkbox" v-model="apiKey"> API key
              </label>
              <span class="icon has-text-info">
                <i v-if="currentInfo == 'apiKey'" class="fas fa-minus-circle" @click="currentInfo = ''"></i>
                <i v-else class="fas fa-info-circle" @click="currentInfo = 'apiKey'"></i>
              </span>
            <div v-if="currentInfo == 'apiKey'" class="notification is-info mt-3 content">
              <p>Check this box if your endpoint requires an API Key.</p>
            </div>
          </div>
          <div class="field">
            <label class="label">Custom headers
              <span class="icon has-text-info">
                <i v-if="currentInfo == 'customHeader'" class="fas fa-minus-circle" @click="currentInfo = ''"></i>
                <i v-else class="fas fa-info-circle" @click="currentInfo = 'customHeader'"></i>
              </span>
            </label>
            <div class="control">
              <input class="input" type="text" placeholder="Custom headers" v-model="customHeaders">
            </div>
            <div v-if="currentInfo == 'customHeader'" class="notification is-info mt-3 content">
              <p>Enter any other headers you might need to pass to the API endpoint seperated by a comma.</p>
            </div>
          </div>
        </div>
      </div>
      <div v-if="cors.AllowOrigins && cors.AllowMethods" class="column is-three-quarters">
        <div v-if="v1">
          <rest :data="cors" :auth="authorization"></rest>
        </div>
        <div v-else>
          <http :data="cors" :auth="authorization"></http>
        </div>
      </div>
      <div v-else class="column">
        <div class="message is-info">
          <div class="message-header">
            Welcome to the CORS configurator for API Gateway.
          </div>
          <div class="message-body content">
            <p>
              The <strong>CORS configurator</strong> helps you configure CORS on API Gateway for <em>REST</em> or <em>HTTP</em> APIs. Fill in the information <span class="is-hidden-mobile">on the left</span><span class="is-hidden-tablet">above</span> and the configurator will generate the AWS SAM configuration as well as a response example. To learn more about CORS itself, read <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS" target="_blank">this article</a>.
            </p>
            <p>
              Click on the <span class="icon has-text-info"><i class="fas fa-info-circle"></i></span> icon to learn about a property. A configuration requires an <strong>ORIGIN</strong> and at least one <strong>METHOD</strong>.
            </p>
            <p>Want to know more? Read <a href="https://aws.amazon.com/blogs/compute/configuring-cors-on-amazon-api-gateway-apis/" target="_blank">the blog post.</a></p>
            <p>Found a problem or want to contribute? Go to the <a href="https://github.com/aws-samples/amazon-api-gateway-cors-configurator/issues" target="_blank">GitHub repo</a></p>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import Rest from './components/Rest.vue'
import Http from './components/Http.vue'
import CorsTester from './components/CorsTester.vue'
import Multiselect from '@vueform/multiselect'

export default{
  components: {
    Rest,
    Http,
    CorsTester,
    Multiselect,
  },
  data(){
    return {
      apiVersion: "v1",
      host: "https://jno01zjo3m.execute-api.us-west-2.amazonaws.com/Prod/",
      origin: "",
      methods: {
        options:["GET","PUT","POST","DELETE","PATCH","HEAD","OPTIONS"],
        values:[]
      },
      authorization: false,
      tokenSource: "",
      apiKey: false,
      customHeaders: "",
      currentTab: "configure",
      currentInfo: ""
    }
  },
  computed:{
    v1: function(){
      return this.apiVersion == "v1";
    },
    cors: function(){
      let corsStruct = {}

      // Set origins
      if(this.origin){
        let o = []
        this.origin.split(",").map(item => {
          if(item.startsWith("http://") || item.startsWith("https://") || item === "*"){
            o.push(item);
          }
          if(o.length > 0) {
            if(this.v1) corsStruct.AllowOrigins = [o[0]]
            else corsStruct.AllowOrigins = o
          }
        })
      }

      // Set methods
      if(this.methods.values.length > 0){
        corsStruct.AllowMethods = this.methods.values.length == 7 ? ["*"] : this.methods.values;
      }

      // Set headers
      corsStruct.AllowHeaders = [];
      corsStruct.AllowHeaders.push("Content-Type");
      if(this.authorization) corsStruct.AllowHeaders.push(this.tokenSource ? this.tokenSource : "Authorization");
      if(this.apiKey && this.v1) corsStruct.AllowHeaders.push("x-api-key");

      // Set custom headers
      if(this.customHeaders.length > 0){
        this.customHeaders.split(",").map(item => {
          corsStruct.AllowHeaders.push(item)
        })
      }

      // dedup headers
      corsStruct.AllowHeaders = corsStruct.AllowHeaders.filter((c, index) => {
        return corsStruct.AllowHeaders.indexOf(c) == index;
      })

      return corsStruct
    }
  }
}
</script>
<style src="@vueform/multiselect/themes/default.css"></style>

<style lang="scss">
  .configurator{
    border-radius: .25rem
  }
  .icon .fa-question-circle{
    cursor: pointer
  }
</style>
