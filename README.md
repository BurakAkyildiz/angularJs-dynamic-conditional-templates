# Dynamic Template
[![](https://angularjs.org/img/AngularJS-large.png)](https://angularjs.org/)


## What is DynamicTemplate ?
Dynamic Template is a service to create and compile compex templates with if-elseIf-else condition tags. With this condition shaped templates you can create any element, clean html from unused elements, save performance and get a fast diggest cycle.

There is also a directive created with DynamicTemplate service for general usages. It gives any template to used element like ngInclude. Difference from ngInclude is condition based templateHtml parse.

 > **Note:**  Dynamic Templates conditions are NOT DIRECTIVE. They are tag for define conditions to parse templateHtml and compile it. If you need a directive to use conditions ngIf does the job.

## Demos
[Basic Functionality Demo](https://codepen.io/NoSur/pen/eEgEVw)

## What does DynamicTemplate ?

### Create & save templates with or without condtions
```
DynamicTemplate.createSaveDTTemplate(templateName, templateHtml )
```
*createSaveDTTemplate* will save the templateHtml to angularJs's **$templateCache** and DynamicTemplate's cash. So you can use the template with any way you want.
This is where your condition tree is indexed.


###  Create condition based Html with just element tags( if-elseIf-else )


```
//Your templateHtml can be like

<element dynamic-template-if="age == 25 && experiance > 5">
</element>

<element dynamic-template-else-if="age >= 25 || experiance > 3">
</element>

<element dynamic-template-else> 	
</element>
```
You can define conditions inside conditions. When conditions are checking, if the parent condition returns false the child conditions will be not checked. So it improves diggest cycle.

Condition variables are ***$scope*** based. So **you can check everything what ngIf can check**.

After compile, DynamicTemplate selects first true condition like an if-elseIf-else structure.

```
//Lets say you have 2 variable
------------------------------
$scope.age = 25
$scope.experiance = 5;
-----------------------------------------------------------
//Output

<element dynamic-template-else-if="age >= 25 || experiance > 3" >
</element>
```

> **Note :** The if-elseIf-else conditions never inseparable. Like any if-elseIf-else structure tagged elements must be **siblings !** and must be in correct order.
> Siblings are checking with `nextElementSibling` and `previousElementSibling` methods so it ignores text and comment nodes.
>The  `dynamic-template-else-if` and `dynamic-template-else` tags  that not has `dynamic-template-if` on previousElementSibling are ignored.
>The first dynamicTemplate condition tag will be considered only on one element.

### Download a template
```
//Returns http request promise
DynamicTemplate.downloadTemplate( templateName )

```


  You can request and download a templateHtml.  *downloadTemplate* function sends the templateName and response to *createSaveDTTemplate* on successfull requests.

 Also it is cashing same requests to same url at the same time so you can use it in ngRepeated elements.

Optionaly you can use this params too

```
DynamicTemplate.downloadTemplate( templateName, requestParams, successCallBack, errorCallBack )
```
**`templateName :`** Downloaded templateHtml will be saved with this name. Request url will be created using `DynamicTemplate.config.autoRequesturl + templateName ` if no requestParams declared.

*This params are optional*
**`requestParams :`** GET parameters to send `DyanicTemplate.config.autoRequestUrl`.  Must be an key value like object.

**`successCallBack :`** Successfull request hook. Gets result as parameter.

**`errorCallBack :`** Unsuccessfull request or undefined result hook. Gets error as parameter.

## Compile elements & directives with any template

You can define a template with 3 way;

 1- A dynamicTemplate can be created with `createSaveDTTemplate` function.

 2- A script tag
```
<script type="text/ng-template" id="/tpl.html">
  <element>
    <h1 dynamic-template-if="checkThis()"> A Title </h1>		
  </element>
</script>

```
3- **$templateCache** service
```
$templateCache.put(templateName, templateHtml);
```
  If you are using conditions element will be shaped based on your condition matches.

```
DynamicTemplate.compileElement(templateName, element, scope);
```
**`templateName :`** You can use any angular template's name. Template names on angularjs's *$templateCache* or created dynamicTemplate's names.
**`element :`** The element that dynamicTemplateHtml output will be given.
**`scope :`** The scope of element which will be related in and conditions will be checked.

> **Note :**  If no template is found named *templateName* on both cash, DynamicTemplate automaticly requests downloads and saves the template after that compiles element. This auto request can be prevented with **DynamicTemplate.config.autoRequest = false**

> If you want to send **request params on autoRequest** you can send requestParams replacing with *`templateName`*  param.  The *`requestParams`* must have a value with key named *`templateName`* . This value will considered as original *`templateName`*  params value and not will be sended as request parameter.

Optionaly you can use **compileElement** like
*This params are optional*

```
DynamicTemplate.compileElement(templateName, element, scope, watchGroupExpression, beforeCompileHook, afterCompileHook);
```
**`watchGroupExpression :`** It is expression for `$scope.$watchGroup` function. With this expression a watchGroup is started. When any of watchGroup expression returns different the element will be re compiled with same *`$scope`* and *`templateName`*.

**`beforeCompileHook :`** This hook function will have templateHtml as parameter.  Before compile you can edit templateHtml string. The element will be compiled using this templateHtml.

**`afterCompileHook :`** This hook function will have templateHtml and compile link function as parameter. It will be triggered after `$compile` service compiles templateHtml.




## DynamicTemplate cashes $compile link functions
 DynamicTemplate cashes $compile link functions based on condition matches. So you will have better performance on recompile. Cashed linkFunctions are using by all DynamicTemplate service instances.
 DynamicTemplate destroyes the child scopes when element is destroyed and removes watchGroup expression on destroy.

## To use DynamicTemplate
Inject the module and get the service.
```
    var app = angular.module('app', 'DynamicTemplate');

    app.controller('ctrl', function($scope, DynamicTemplate){
        ....
    })

    // or

    app.directive('someDirective', function( DynamicTemplate ){

        return {

        }
    })


// An Example

  app.directive('someDirective', function ( DynamicTemplate ) {

    // You can download templates here too.
    // DynamicTemplate.downloadTemplate('someTemplateName');

    return {
      controller:controllerFunc,
      compile: function compile(tElement, tAttrs) { // If you compile element on preLink you will get better performance because in here element has not compiled yet.

        return
        {
          pre: function preLink(scope, element, attr) {

    	   var beforeCompile = function (templateStr)
    	   {
    		 // It is an string operation to change templateHtml so the element will be created differently if you want.
    		 if (templateStr!= undefined)
    		   return templateStr.replace("<an-other-directive></an-other-directive>", "<any-directive></any-directive>");

    		 return templateStr;
    	   };



           DynamicTemplate.downloadTemplate('someTemplateName', {reqParam1:'someValue' }).then(function()
           {
             //After download DynamicTemplate already created and saved templateHtml to cash so you can compile element.

             DynamicTemplate.compileElement('someTemplateName', element, scope, ['$root.someRootValue'], beforeCompile);

             // Element is compiled. When `$root.someRootValue` changes element will be re compiled with same template and scope.
           });


          }
        }
    }
  }

    });

```


## To use dynamicTemplate Directive

```
    // Insert the directive as attribute to any element
    // dynamic-template directive automaticly compile element with given templateName. If value changes element will be recompiled.

    <any-element dynamic-template="'someTemplateName'">

    </any-element>


    // or

    // Value considered same as compileElement functions templateName parameter. So you can send autoRequest parameters too.

    <any-element dynamic-template="{templateName:'someTemplateName', requestParam1:'someValue', requestParam2:12345}">

    </any-element>


    // You can watch values and automaticly recompile element.

    <any-element
        dynamic-template="{templateName:'someTemplateName', requestParam1:(aVariable), requestParam2:12345}"
        dynamic-template-watch-group="['aVariable']">

        // When `$scope.aVariable` changes element will be recompiled.

    </any-element>
```

## DynamicTemplate Config

|  Parame Name     | Type    | Default | Explanation |
 ----------------- | ------- | ------- | ----------- |
| autoRequest      | boolean | true    | If no template found with templateName param request will be sended to download a templateHtml.|
| autoRequestUrl   | string  | "/"     | Request url to downloadTemplate method. DynamicTemplate uses downloadTemplate method for autoRequest.|


## Dependencies
    DynamicTemplate uses jquery's `element` and `html` function.

## Known issues
    Jquery's `html` function has a bug that cant create element from string has not starts with element tag.
