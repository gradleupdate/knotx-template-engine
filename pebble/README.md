# Knot.x Template Engine Pebble
This section describes Pebble template engine strategy that is an alternative implementation of the
template engine used in Knot.x examples. 


## How does it work
Template Engine Pebble uses 
[Pebble Templates](https://pebbletemplates.io/) to compile and evaluate templates.
Please refer to its documentation for any details.
Additionally, Knot.x Template Engine Pebble has built-in Guava in-memory cache for compiled PEB
Templates. Key computation is performed using the `cacheKeyAlgorithm` defined in the configuration
(the default is `MD5` of the Fragment's `body`).

## How to configure
For all configuration fields and their defaults consult [io.knotx.te.pebble.options.PebbleEngineOptions](https://github.com/Knotx/knotx-template-engine/blob/master/pebble/docs/asciidoc/dataobjects.adoc)

### Custom delimiters
By default, the Pebble engine uses `{{` and `}}` symbols as print delimiters, `{%` and `%}` as computation delimiters, `{#` and `#}` as comment delimiters and `#{` and `}` as string interpolation delimiters.
Knot.x Template Engine Pebble allows to change them using configuration

E.g.:
In order to use different symbols as below
```html
<div class="col-md-4">
  <h2>Snippet1 - {: message :}</h2>
</div>
```
You can reconfigure an engine as follows in the pebble engine entry section:
```hocon
  {
    factory = pebble
    config = {
      cacheSize = 1000
      syntax = { 
         delimiterPrintOpen = "{:"     
         delimiterPrintClose = ":}"    
      }
    }
  }
```

## Known issues

### Dashed variables

When referencing variables with dash `-` character, the `.` operator cannot be used. 
Pebble provides an alternative solution: the `[]` operator. 
When printing a variable `dashed-var` under `dashed-var-parent` which is under `data` node, the following construction can be used:
`{{ data['dashed-var-parent']['dashed-var'] }}`.
 
This, however, does not help when a root variable has dashes (`{{ ['dashed-data'] }}` will not work).
To solve this issue, a special `wrappingRootNodeName` option is provided the in [PebbleEngineSyntaxOptions](https://github.com/Knotx/knotx-template-engine/blob/master/pebble/docs/asciidoc/dataobjects.adoc#PebbleEngineSyntaxOptions). If this option is not empty, the `payload` of the templated `Fragment` will be wrapped in a `JsonObject` with `wrappingRootNodeName` as key.
For example, if `wrappingRootNodeName = root`, the variable above is referenced by `{{ root['dashed-data'] }}`. As a result, all context variable references have to be preceded by this root variable.

This feature is not enabled by default.
