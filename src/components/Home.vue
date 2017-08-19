<template>
  <div class="home">
    <div class="row vertical-buffer-lg">
      <div class="col-3">
        <div class="row">
          <div class="col">
            <b-list-group flush>
              <div v-for="group in objectToSortedArray(resourceGroups, 'name', 'name')">
                <b-list-group-item class="list-group-item-action" v-b-toggle="group.name" role="button">
                  <h6>{{group.name}}</h6>
                </b-list-group-item>
                <b-list-group-item>
                  <b-collapse v-bind:id="group.name">
                    <b-list-group flush>
                      <b-list-group-item class="list-group-item-action" v-on:click.native="addResourceToStack(resource)" role="button" v-for="resource in sortArray(group.resources, 'Name')" :key="resource.Name">
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
                  <div v-if="stack.Resources.length > 0" class="row">
                    <div class="col">
                      <div class="row" v-for="(resource, index) in stack.Resources">
                        <div class="col">
                          <b-card>
                            <div class="row" slot="header">
                              <div class="col">
                                {{resource.Name}}
                              </div>
                              <div class="col text-right">
                                <b-dropdown size="sm" right>
                                  <b-dropdown-item v-bind:href="resource.Documentation" target="_blank">Documentation</b-dropdown-item>
                                  <b-dropdown-item v-on:click="stack.Resources.splice(index, 1)">Remove</b-dropdown-item>
                                </b-dropdown>
                              </div>
                            </div>
                            <div class="row">
                              <div class="col">
                                <b-form-fieldset v-bind:state="(resource.LogicalId === '') ? 'danger': null">
                                  <b-input-group>
                                    <b-input-group-addon class="col-3" left>Logical ID</b-input-group-addon>
                                    <b-form-input v-bind:state="(resource.LogicalId === '') ? 'danger': null" v-model="resource.LogicalId"></b-form-input>
                                    <b-input-group-button right>
                                      <b-button v-b-toggle="'logicalId'">...</b-button>
                                    </b-input-group-button>
                                  </b-input-group>
                                </b-form-fieldset>
                                <b-collapse v-bind:id="'logicalId'">
                                  <b-card no-block>
                                    <div class="row">
                                      <div class="col text-center">
                                        This is <b>required</b> and must be <b>unique</b>.
                                      </div>
                                    </div>
                                  </b-card>
                                </b-collapse>
                              </div>
                            </div>
                            <div class="row" v-for="(property, propertyName) in resource.Properties">
                              <div class="col">
                                <b-form-fieldset v-bind:state="propertyState(property)">
                                  <b-input-group>
                                    <b-input-group-addon class="col-3" left>{{propertyName}}</b-input-group-addon>
                                    <b-form-input v-bind:state="propertyState(property)" v-model="property.value"></b-form-input>
                                    <b-input-group-button right>
                                      <b-button v-b-toggle="propertyName">...</b-button>
                                    </b-input-group-button>
                                  </b-input-group>
                                </b-form-fieldset>
                                <b-collapse v-bind:id="propertyName">
                                  <b-list-group flush>
                                    <b-list-group-item class="list-group-item" v-show="showDetail(detailName)" v-for="(detail, detailName) in property">
                                      <div class="col-6 text-right">
                                        {{detailName}} :
                                      </div>
                                      <div class="col-6">
                                        {{detail}}
                                      </div>
                                    </b-list-group-item>
                                  </b-list-group>
                                </b-collapse>
                              </div>
                            </div>
                          </b-card>
                        </div>
                      </div>
                    </div>
                  </div>
                  <p v-else class="text-center">
                    There are no resources in the stack.
                    <br><br> Start by selecting resources from the sidebar or importing an existing Cloudformation template in the <b>Import Template</b> tab.
                  </p>
                </b-tab>
                <b-tab title="Import Template">
                  <div class="row">
                    <div class="col ">
                      <b-form-input textarea v-model="templateField" placeholder="Paste template here..." :rows="5"></b-form-input>
                    </div>
                  </div>
                  <div class="row">
                    <div class="col text-center ">
                      <b-btn v-on:click="importTemplate" variant="primary">Import</b-btn>
                    </div>
                  </div>
                </b-tab>
                <b-tab title="Export Template">
                  <div class="row">
                    <div class="col text-center ">
                      <b-btn v-on:click="exportTemplate" variant="primary">Export</b-btn>
                    </div>
                  </div>
                  <div class="row">
                    <div class="col ">
                      <b-form-input textarea v-model="exportField" placeholder="Exported template will go here..." :rows="5"></b-form-input>
                    </div>
                  </div>
                </b-tab>
              </b-tabs>
            </b-card>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Home',
  data () {
    return {
      resourceGroups: [],
      template: {
        Description: '',
        Parameters: {},
        Mappings: {},
        Conditions: {},
        Resources: {},
        Outputs: {}
      },
      stack: {
        Description: '',
        Parameters: [],
        Mappings: [],
        Conditions: [],
        Resources: [],
        Outputs: []
      },
      templateField: null,
      exportField: null,
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
      this.template = data
    },
    importTemplate: function () {
      try {
        this.validateTemplate()
      } catch (e) {
        return console.log(`Validation Error: ${e.message}`)
      }
      this.stack = {
        Description: null,
        Parameters: [],
        Mappings: [],
        Conditions: [],
        Resources: [],
        Outputs: []
      }
      for (let logicalId in this.template.Resources) {
        let resource = this.template.Resources[logicalId]
        let group = resource.Type.split('::')[1]
        let index = -1
        for (let i = 0; i < this.resourceGroups[group].resources.length; i++) {
          if (this.resourceGroups[group].resources[i].Type === resource.Type) {
            index = i
          }
        }
        let resourceTemplate = JSON.parse(JSON.stringify(this.resourceGroups[group].resources[index]))
        resourceTemplate.LogicalId = logicalId
        for (let propertyName in resourceTemplate.Properties) {
          let value = (propertyName in resource.Properties) ? JSON.stringify(resource.Properties[propertyName]) : ''
          resourceTemplate.Properties[propertyName].value = value
        }
        this.stack.Resources.push(resourceTemplate)
      }
    },
    addResourceToStack: function (data) {
      data = JSON.parse(JSON.stringify(data))
      data.LogicalId = ''
      for (let key in data.Properties) {
        data.Properties[key].displayDetails = false
        data.Properties[key].value = null
      }
      this.stack.Resources.push(data)
    },
    showDetail: function (key) {
      let blacklist = [
        'Documentation',
        'value',
        'displayDetails'
      ]
      return (blacklist.indexOf(key) === -1)
    },
    propertyState: function (property) {
      if (property.Required && (property.value === '' || property.value === null)) {
        return 'danger'
      } else {
        return
      }
    },
    exportTemplate: function () {
      for (let index in this.stack.Resources) {
        let resourceTemplate = JSON.parse(JSON.stringify(this.stack.Resources[index]))
        let resource = {
          Type: resourceTemplate.Type,
          Properties: {}
        }
        for (let property in resourceTemplate.Properties) {
          if (resourceTemplate.Properties[property].value) {
            try {
              resource.Properties[property] = JSON.parse(resourceTemplate.Properties[property].value)
            } catch (e) {
              resource.Properties[property] = resourceTemplate.Properties[property].value
            }
          }
        }
        this.template.Resources[resourceTemplate.LogicalId] = resource
      }
      this.exportField = JSON.stringify(this.template, null, 2)
    },
    objectToSortedArray: function (obj, keyName, sortKey) {
      let arr = Object.keys(obj).map(function (key) {
        obj[key][keyName] = key
        return obj[key]
      })
      arr.sort(function (a, b) {
        let aKey = a[sortKey].toUpperCase()
        let bKey = b[sortKey].toUpperCase()
        if (aKey < bKey) {
          return -1
        }
        if (aKey > bKey) {
          return 1
        }
        return 0
      })
      return arr
    },
    sortArray: function (arr, sortKey) {
      arr.sort(function (a, b) {
        let aKey = a[sortKey].toUpperCase()
        let bKey = b[sortKey].toUpperCase()
        if (aKey < bKey) {
          return -1
        }
        if (aKey > bKey) {
          return 1
        }
        return 0
      })
      return arr
    }
  }
}
</script>

/*<style>
.list-group-item {
    padding: 3px 10px
}
</style>*/
