# cloudvue

A 10% time project to create a smart user interface for creating and updating CloudFormation templates created with Vue.js

## How it works

Utilizing a gigantic JSON file AWS publishes documenting the specs for every property of every resource possible in CloudFormation, this app parses the data and helps you write CloudFormation. You can select any resource and add it to a "stack" and it will show you all properties available for that resource along with text fields to fill them in. It highlights required properties and will show you the data types and other helpful information about each property and resource. It also provides hyperlinks directly to the official AWS resource reference for quick lookups if you need more information. 

You can import existing CloudFormation templates and edit them, or create from scratch. Once you a done, you can export the "stack" to a new template that you can copy and paste. Since the JSON data is public, this app requires no backend and can be hosted statically in a S3 bucket.

## State of the Project

This is currently in a barely working state, and is very fragile. Also since this was a fast and loose single-day project, the whole app is made of a single "home" component. It will currently let you create stacks of resources from scratch and export the template.

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report

# run unit tests
npm run unit

# run all tests
npm test
```

For detailed explanation on how things work, checkout the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).
