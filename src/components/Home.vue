<template>
  <div class="home">
    <div class="row top-buffer">
      <div class="col-3">
        <div class="row">
          <div class="col">
            <b-list-group flush>
              <div v-for="(group, groupName, index) in resourceGroups">
                <b-list-group-item class="list-group-item-action" v-b-toggle="groupName" role="button">
                  <h6>{{groupName}}</h6>
                </b-list-group-item>
                <b-list-group-item>
                  <b-collapse v-bind:id="groupName">
                    <b-list-group flush>
                      <b-list-group-item class="list-group-item-action" v-on:click.native="addResourceToStack(resource)" role="button" v-for="resource in group.resources" :key="resource.Name">
                        {{resource.Name}}
                      </b-list-group-item>
                    </b-list-group>
                  </b-collapse>
                </b-list-group-item>
              </div>
            </b-list-group>
          </div>
        </div>
      </div>
      <div class="col-9">
        <div class="row">
          <div class="col">
            <b-card no-block>
              <b-tabs small card ref="tabs" v-model="tabIndex">
                <b-tab title="Stack">
                  <div v-if="stack.Resources.length > 0">
                    <b-card v-for="resource in stack.Resources">
                      <div class="row" slot="header">
                        <div class="col">
                          {{resource.Name}}
                        </div>
                        <div class="col text-right">
                          <a v-bind:href="resource.Documentation">Docs</a>
                        </div>
                      </div>
                      <b-input-group left="Logical ID" right="*">
                        <b-form-input v-model="resource.LogicalId"></b-form-input>
                      </b-input-group>
                      <!-- Loop Properties -->
                    </b-card>
                  </div>
                  <p v-else class="text-center">
                    There are no resources in the stack.
                    <br><br> Start by selecting resources from the sidebar or importing an existing Cloudformation template in the <b>Import Template</b> tab.
                  </p>
                </b-tab>
                <b-tab title="Import Template">
                  <b-form-input textarea v-model="templateField" placeholder="Paste template here..." ></b-form-input>
                  <div class="text-center">
                    <br>
                    <b-btn v-on:click="importTemplate" variant="primary">Submit</b-btn>
                  </div>
                </b-tab>
              </b-tabs>
            </b-card>
          </div>
        </div>
      </div>
    </div>
    <div class="top-buffer"></div>
  </div>
</template>

<script>
export default {
  name: 'Home',
  data () {
    return {
      resourceGroups: {},
      template: {
        Description: null,
        Parameters: {},
        Mappings: {},
        Conditions: {},
        Resources: {},
        Outputs: {}
      },
      stack: {
        Description: null,
        Parameters: [],
        Mappings: [],
        Conditions: [],
        Resources: [],
        Outputs: []
      },
      templateField: null,
      tabIndex: 0
    }
  },
  created: function () {
    this.importAWSSpecification()
  },
  methods: {
    importAWSSpecification: function () {
      let awsData = require('../data/CloudFormationResourceSpecification.json')
      for (let key in awsData.ResourceTypes) {
        let group = key.split('::')[1]
        let name = key.split('::')[2]
        let value = awsData.ResourceTypes[key]
        value.Type = key
        value.Name = name
        if (group in this.resourceGroups) {
          this.resourceGroups[group].resources.push(value)
        } else {
          this.resourceGroups[group] = {
            resources: [value],
            visable: false
          }
        }
      }
    },
    validateTemplate: function () {
      let data
      try {
        data = JSON.parse(this.templateField)
      } catch (e) {
        throw new Error('Template is not valid JSON')
      }
      for (let key in this.template) {
        if (key in data) {
          continue
        } else {
          throw new Error(`The required key "${key}" does not exist in the template given`)
        }
      }
      if (Object.keys(data).length !== 6) {
        throw new Error('Template contains invalid keys')
      }
    },
    importTemplate: function () {
      try {
        this.validateTemplate()
      } catch (e) {
        return console.log(`Validation Error: ${e.message}`)
      }
    },
    addResourceToStack: function (data) {
      data = JSON.parse(JSON.stringify(data))
      data.LogicalId = ''
      this.stack.Resources.push(data)
    }
  }
}
</script>

/*<style>
.list-group-item {
    padding: 3px 10px
}
</style>*/
