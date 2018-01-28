# Swagger to Typescript Codegen
This package generates a TypeScript class from a [swagger specification file](https://github.com/wordnik/swagger-spec). The code is generated using [mustache templates](https://github.com/mtennoe/swagger-js-codegen/tree/master/lib/templates) and is quality checked by [jshint](https://github.com/jshint/jshint/) and beautified by [js-beautify](https://github.com/beautify-web/js-beautify).

The typescript generator is based on [superagent](https://github.com/visionmedia/superagent) and can be used for both nodejs and the browser via browserify/webpack.

This fork was made to simplify some parts, add some more features, and tailor it more to specific use cases.

## Installation
```bash
npm install swagger-typescript-codegen
```

## Example
```javascript
var fs = require('fs');
var CodeGen = require('swagger-typescript-codegen').CodeGen;

var file = 'swagger/spec.json';
var swagger = JSON.parse(fs.readFileSync(file, 'UTF-8'));
var tsSourceCode = CodeGen.getTypescriptCode({ className: 'Test', swagger: swagger, imports: ['../../typings/tsd.d.ts'] });
console.log(tsSourceCode);
```

## Custom template
```javascript
var source = CodeGen.getCustomCode({
    moduleName: 'Test',
    className: 'Test',
    swagger: swaggerSpec,
    template: {
        class: fs.readFileSync('my-class.mustache', 'utf-8'),
        method: fs.readFileSync('my-method.mustache', 'utf-8'),
        type: fs.readFileSync('my-type.mustache', 'utf-8')
    }
});
```

## Options
In addition to the common options listed below, `getCustomCode()` *requires* a `template` field:

    template: { class: "...", method: "..." }

`getTypescriptCode()`, `getCustomCode()` each support the following options:

```yaml
  moduleName:
    type: string
    description: Your module name
  className:
    type: string
  lint:
    type: boolean
    description: whether or not to run jslint on the generated code
  esnext:
    type: boolean
    description: passed through to jslint
  beautify:
    type: boolean
    description: whether or not to beautify the generated code
  beautifyOptions:
    type: object
    description: Options to be passed to the beautify command. See js-beautify for all available options.
  mustache:
    type: object
    description: See the 'Custom Mustache Variables' section below
  imports:
    type: array
    description: Typescript definition files to be imported.
  swagger:
    type: object
    required: true
    description: swagger object
```

### Template Variables
The following data are passed to the [mustache templates](https://github.com/janl/mustache.js):

```yaml
isES6:
  type: boolean
description:
  type: string
  description: Provided by your options field: 'swagger.info.description'
isSecure:
  type: boolean
  description: false unless 'swagger.securityDefinitions' is defined
moduleName:
  type: string
  description: Your module name - provided by your options field
className:
  type: string
  description: Provided by your options field
domain:
  type: string
  description: If all options defined: swagger.schemes[0] + '://' + swagger.host + swagger.basePath
methods:
  type: array
  items:
    type: object
    properties:
      path:
        type: string
      pathFormatString:
        type: string
      className:
        type: string
        description: Provided by your options field
      methodName:
        type: string
        description: Generated from the HTTP method and path elements or 'x-swagger-js-method-name' field
      method:
        type: string
        description: 'GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'COPY', 'HEAD', 'OPTIONS', 'LINK', 'UNLIK', 'PURGE', 'LOCK', 'UNLOCK', 'PROPFIND'
        enum:
        - GET
        - POST
        - PUT
        - DELETE
        - PATCH
        - COPY
        - HEAD
        - OPTIONS
        - LINK
        - UNLIK
        - PURGE
        - LOCK
        - UNLOCK
        - PROPFIND
      isGET:
        type: string
        description: true if method === 'GET'
      summary:
        type: string
        description: Provided by the 'description' or 'summary' field in the schema
      externalDocs:
        type: object
        properties:
          url:
            type: string
            description: The URL for the target documentation. Value MUST be in the format of a URL.
            required: true
          description:
            type: string
            description: A short description of the target documentation. GitHub-Markdown syntax can be used for rich text representation.
      isSecure:
        type: boolean
        description: true if the 'security' is defined for the method in the schema
      parameters:
        type: array
        description: Includes all of the properties defined for the parameter in the schema plus:
        items:
          camelCaseName:
            type: string
          isSingleton:
            type: boolean
            description: true if there was only one 'enum' defined for the parameter
          singleton:
            type: string
            description: the one and only 'enum' defined for the parameter (if there is only one)
          isBodyParameter:
            type: boolean
          isPathParameter:
            type: boolean
          isQueryParameter:
            type: boolean
          isPatternType:
            type: boolean
            description: true if *in* is 'query', and 'pattern' is defined
          isHeaderParameter:
            type: boolean
          isFormParameter:
            type: boolean
      successfulResponseType:
        type: string
        description: The type of a successful response. Defaults to any for non-parsable types or Swagger 1.0 spec files
```

#### Custom Mustache Variables
You can also pass in your own variables for the mustache templates by adding a `mustache` object:

```javascript
var source = CodeGen.getCustomCode({
    ...
    mustache: {
      foo: 'bar',
      app_build_id: env.BUILD_ID,
      app_version: pkg.version
    }
});
```

## Swagger Extensions

### x-proxy-header
Some proxies and application servers inject HTTP headers into the requests.  Server-side code
may use these fields, but they are not required in the client API.

eg: https://cloud.google.com/appengine/docs/go/requests#Go_Request_headers

```yaml
  /locations:
    get:
      parameters:
      - name: X-AppEngine-Country
        in: header
        x-proxy-header: true
        type: string
        description: Provided by AppEngine eg - US, AU, GB
      - name: country
        in: query
        type: string
        description: |
          2 character country code.
          If not specified, will default to the country provided in the X-AppEngine-Country header
      ...
```
