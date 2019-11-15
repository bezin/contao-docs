---
title: "Front End Modules"
description: "Complex functionalities within your web pages."
aliases:
    - /documentation/front-end-modules/
    - /framework/content-modules/
---


Front end modules in Contao are used for more complex functionality, which are typically
used on more than one page or even in page layouts. They are used to generate dynamic 
content, like news lists, displaying the detailed content of news or navigation items.

Creating a front end module is very similar to creating [content elements][1].


## Definition

To create a new front end module, the following things must be defined and implemented:

* __Fragment Controller__<br>
  The actual implementation of the front end module is done via a class that extends
  from `AbstractFrontendModuleController` of the Contao core.

* __Service Tag__<br>
  To identify the controller as a Contao front end module, the service must be tagged
  with service tag `contao.frontend_module`.

  * __Type__<a id="type"></a><br>
    The *type* of a front end module is a specifig string which is used to identify
    the front end module's template and DCA palette. The `type` can be set in the 
    service tag. If ommitted the type will be automatically generated by converting the 
    class name of the controller from pascal case to snake case and removing a possible 
    `Controller` postfix.
  
  * __Category__<br>
    All front end modules are categorised within the type dropdown of the front 
    end module's palette. A `category` must be defined in the service tag for each 
    module.

* __Template__<br>
  The template name follows the naming convention mentioned beforehand. It prepends
  the *type* of the module with the prefix `mod_`.


## Example

Usually a front end module is based on a specific [palette][2] in the `tl_content`
DCA configuration.

```php
// contao/dca/tl_module.php
$GLOBALS['TL_DCA']['tl_module']['palettes']['my_frontend_module'] = 
    '{type_legend},type;{redirect_legend},jumpTo'
;
```

This very simple palette enables a back end user to select a redirect page using
the pre-existing field `jumpTo` via the create and edit view of this front end module.

The controller for this module could look like this:

```php
// src/Controller/FrontendModule/MyFrontendModuleController.php
namespace App\Controller\FrontendModule;

use Contao\CoreBundle\Controller\FrontendModule\AbstractFrontendModuleController;
use Contao\CoreBundle\Exception\RedirectResponseException;
use Contao\CoreBundle\ServiceAnnotation\FrontendModule;
use Contao\ModuleModel;
use Contao\PageModel;
use Contao\Template;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

/**
 * @FrontendModule(category="miscellaneous")
 */
class MyFrontendModuleController extends AbstractFrontendModuleController
{
    protected function getResponse(Template $template, ModuleModel $model, Request $request): ?Response
    {
        if ($request->isMethod('post')) {
            if (null !== ($redirectPage = PageModel::findByPk($model->jumpTo))) {
                throw new RedirectResponseException($redirectPage->getAbsoluteUrl());
            }
        }

        $template->action = $request->getUri();

        return $template->getResponse();
    }
}
```

In this example the service tag was implemented via annotations. The controller itself
processes the request and checks, if it was a POST request. In that case, the
redirect page is loaded via Contao's model functionality and a `RedirectResponseException`
is thrown to redirect to that page.

Using the naming convention for templates mentioned above, the final template name
for this front end module will be `mod_my_frontend_module`:

```html
<!-- templates/mod_my_frontend_module.html5 -->
<div class="my-frontend-module">   
  <form action="<?= $this->action ?>" method="POST"> 
    <input type="hidden" name="REQUEST_TOKEN" value="{{request_token}}">
    <button type="submit">Submit</button>
  </form>
</div>
```

A template instance of this template will automatically be generated and passed 
to the controller's main method. The controller returns the parsed template
as a response.


## Options

The `contao.frontend_module` tag can be configured further more. The following
options are available.

| Option   | Type      | Description                                                                                         |
| -------- | --------- | ----------------------------------------------------------------------------------------------------|
| name     | `string`  | Must be `contao.frontend_module`.                                                                   |
| category | `string`  | Defines in which option group this front end module will be placed in the module type selector.     |
| type     | `string`  | _Optional:_ The *type* mentioned in [Type]({{< ref "#type" >}}) can be customized.                  |
| renderer | `string`  | _Optional:_ The renderer can be changed to `esi`. Defaults to `inline`.                             |
| method   | `integer` | _Optional:_  Which method should be invoked on the controller.                                      |

A more complex example of a front end module could look like this.

```yaml
# config/services.yaml
services:
    App\Controller\FrontendModule\MyFrontendModuleController:
        tags:
            -
                name: contao.frontend_module
                category: texts
                method: getCustomResponse
                renderer: esi
                type: my_custom_type
```


## Translations

In order to have a nice label in the back end, we also need to add a translation
for our front end module - otherwise it will only be named *my_frontend_module*.
The translation needs to be set as follows:

```php
// contao/languages/en/modules.php
$GLOBALS['TL_LANG']['FMD']['my_frontend_module'] = [
    'My front end module', 
    'A front end module for testing purposes.',
];
```


## Read more

* [DCA Configuration reference][2]
* [Manipulate and create palettes][3]
* [Create and use templates][4]
* [Customize Caching][5]


[1]: /framework/content-elements/
[2]: ../../reference/dca/reference
[3]: ../../reference/dca/palettes
[4]: ../templates
[5]: ../caching