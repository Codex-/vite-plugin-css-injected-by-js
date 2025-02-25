# Vite Plugin: CssInjectedByJs

A Vite plugin that takes the CSS and adds it to the page through the JS. For those who want a single JS file.

The plugin can be configured to execute the CSS injection before or after your app code, and you can also provide a
custom id for the injected style element and a function to customize the injection code used.

## How does it work

Essentially what it does is take all the CSS generated by the build process and add it via JavaScript. The CSS file is
therefore not generated and the declaration in the generated HTML file is also removed. You can also configure when the
CSS injection will be executed (before or after your app code).

## Installation

```
npm i vite-plugin-css-injected-by-js --save
```

## Usage

```ts
import cssInjectedByJsPlugin from 'vite-plugin-css-injected-by-js'

export default {
    plugins: [
        cssInjectedByJsPlugin(),
    ]
}
```

### Configurations

When you add the plugin, you can provide a configuration object. For now, you can configure only when the injection of
CSS is done at execution time ```topExecutionPriority```.

#### topExecutionPriority

The default behavior adds the injection of CSS before your bundle code. If you provide ```topExecutionPriority``` equal
to: ```false```  the code of injection will be added after the bundle code. This is an example:

```ts
import cssInjectedByJsPlugin from 'vite-plugin-css-injected-by-js'

export default {
    plugins: [
        cssInjectedByJsPlugin({topExecutionPriority: false}),
    ]
}
```

#### styleId

If you provide a `string` for `styleId` param the code of injection will set the `id` attribute of the `style` element
with the value of the parameter provided. This is an example:

```ts
import cssInjectedByJsPlugin from 'vite-plugin-css-injected-by-js'

export default {
    plugins: [
        cssInjectedByJsPlugin({styleId: "foo"}),
    ]
}
```

The output injected into the DOM will look like this example:

```html

<head>
    <style id="foo">/* Generated CSS rules */</style>
</head>
```

#### preRenderCSSCode

You can use the preRenderCSSCode parameter to make specific changes to your CSS before it is printed in the output JS
file. This parameter takes the CSS code extracted from the build process and allows you to return the modified CSS code
to be used within the injection code.

This way, you can customize the CSS code without having to write additional code that runs during the execution of your
application.

This is an example:

```ts
import cssInjectedByJsPlugin from 'vite-plugin-css-injected-by-js'

export default {
    plugins: [
        cssInjectedByJsPlugin({preRenderCSSCode: (cssCode) => cssCode}), // The return will be used as the CSS that will be injected during execution.
    ]
}
```

#### injectCode

You can provide also a function for `injectCode` param to customize the injection code used. The `injectCode` callback
must return a `string` (with valid JS code) and it's called with two arguments:

- cssCode (the `string` that contains all the css code that need to be injected via JavaScript)
- options (a simple object that currently contains only the `styleId` param)

This is an example:

```ts
import cssInjectedByJsPlugin from 'vite-plugin-css-injected-by-js'

export default {
    plugins: [
        cssInjectedByJsPlugin({
            injectCode: (cssCode, options) => {
                return `try{if(typeof document != 'undefined'){var elementStyle = document.createElement('style');elementStyle.appendChild(document.createTextNode(${cssCode}));document.head.appendChild(elementStyle);}}catch(e){console.error('vite-plugin-css-injected-by-js', e);}`
            }
        }),
    ]
}
```

#### injectCodeFunction

If you prefer to specify the injectCode as a plain function you can use the `injectCodeFunction` param.

The `injectCodeFunction` function is a void function that will be called at runtime application with two arguments:

- cssCode (the `string` that contains all the css code that need to be injected via JavaScript)
- options (a simple object that currently contains only the `styleId` param)

This is an example:

```ts
import cssInjectedByJsPlugin from 'vite-plugin-css-injected-by-js'

export default {
    plugins: [
        cssInjectedByJsPlugin({
            injectCodeFunction: function injectCodeCustomRunTimeFunction(cssCode, options) {
                try {
                    if (typeof document != 'undefined') {
                        var elementStyle = document.createElement('style');
                        elementStyle.appendChild(document.createTextNode(${cssCode}));
                        document.head.appendChild(elementStyle);
                    }
                } catch (e) {
                    console.error('vite-plugin-css-injected-by-js', e);
                }
            }
        }),
    ]
}
```

#### jsAssetsFilterFunction

The jsAssetsFilterFunction parameter allows you to specify which JavaScript file(s) the CSS injection code should be
added to. This is useful when using a Vite configuration that exports multiple entry points in the building process. The
function takes in an OutputChunk object and should return true for the file(s) you wish to use as the host of the CSS
injection. If multiple files are specified, the CSS injection code will be added to all files returned as true.

Here is an example of how to use the jsAssetsFilterFunction:

```javascript
import cssInjectedByJsPlugin from 'vite-plugin-css-injected-by-js'

export default {
    plugins: [
        cssInjectedByJsPlugin({
            jsAssetsFilterFunction: function customJsAssetsfilterFunction(outputChunk) {
                return outputChunk.fileName == 'index.js';
            }
        }),
    ]
}
```

In this example, the CSS injection code will only be added to the index.js file. If you wish to add the code to multiple
files, you can specify them in the function:

```javascript
import cssInjectedByJsPlugin from 'vite-plugin-css-injected-by-js'

export default {
    plugins: [
        cssInjectedByJsPlugin({
            jsAssetsFilterFunction: function customJsAssetsfilterFunction(outputChunk) {
                return outputChunk.fileName == 'index.js' || outputChunk.fileName == 'subdir/main.js';
            }
        }),
    ]
}
```

This code will add the injection code to both index.js and main.js files.
**Be aware that if you specified multiple files that the CSS can be doubled.**

#### useStrictCSP

The `useStrictCSP` configuration option adds a nonce to style tags based
on `<meta property="csp-nonce" content={{ nonce }} />`. See the following [link](https://cssinjs.org/csp/?v=v10.9.2) for
more information.

This is an example:

```ts
import cssInjectedByJsPlugin from 'vite-plugin-css-injected-by-js'

export default {
    plugins: [
        cssInjectedByJsPlugin({useStrictCSP: true}),
    ]
}
```

The tag `<meta property="csp-nonce" content={{ nonce }} />` (nonce should be replaced with the value) must be present in
your html page. The `content` value of that tag will be provided to the `nonce` property of the `style` element that
will be injected by our default injection code.

## Contributing

When you make changes to plugin locally, you may want to build the js from the typescript file of the plugin. Here the
guidelines:

### Install

```
npm install
```

### Build plugin

```
npm run build
```

See CONTRIBUTING.md for more information.
