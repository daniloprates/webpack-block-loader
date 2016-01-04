# block-loader

A webpack loader for doing generic block-processing. You provide the delimiters and processor, block-loader will find your data and pass it through your processor.

## Install
```
npm install block-loader
```

## Write your own block loader
```
var blockLoader = require('block-loader');
var options = {
  op: '...',
  ed: '...',
  preprocessors: [
    function(content) { ...; return content; },
    ...
  ],
  process: function(content) {
   ...
    return content;
  }
};
module.exports = blockLoader(options);
```

`op` and `ed` are delimiter strings for you data block, `preprocessors` is optional and takes an array of `function(content)`.

## Example:  "write normal code in `<pre>` elements"

Say you need to write real code in `<pre>` elements, and don't want your Webpack/React build to break on using things like `<`. Let's write a loader that'll fix those things for us:

```
var blockLoader = require("./block-loader");
var options = {
  op: "<pre>",
  ed: "</pre>",
  process: function fixPreBlocks(pre) {
    return pre
    .replace(/&/g,'&amp;')       // 1. use html entity equivalent,
    .replace(/</g,'&lt;')        // 2. use html entity equivalent,
    .replace(/>/g,'&gt;')        // 3. use html entity equivalent,
    .replace(/([{}])/g,"{'$1'}") // 4. JSX-safify curly braces,
    .replace(/\n/g,"{'\\n'}");   // 5. and preserve line endings, thanks.
  }
};
module.exports = blockLoader(options);
```

And done. Save this as something like `./lib/pre-loader.js` and then we can use this with webpack by adding it to `webpack.config.js`:
```
module.exports = {
  entry:  ...,
  output: ...,
  module: {
    loaders: [
      {
        test: /.jsx?$/,
        loaders: [
          'babel-loader',
          __dirname + '/lib/pre-loader'
        ]
      }
    ]
  },
};

```
Remember that webpack loaders run LIFO, so the ones that need to come earlier come later in the array of loaders.
