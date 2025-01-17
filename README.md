# @kubevue/md-vue-loader

Webpack loader for converting Markdown files to alive Vue components.

## Features

- Live vue/html code blocks
- Cache markdown component and code block components
- Hot reload
- Built-in **syntax highlighter** with [highlightjs](https://highlightjs.org)
- Code block style modifier
- Configurable [markdown-it](https://github.com/markdown-it/markdown-it) parser

## Example

### Live Code Blocks

Support two kinds of code blocks to live:

1. html code

``` html
<u-button>Button</u-button>
<u-input></u-input>
```

2. vue code

``` vue
<template>
<div :class="$style.root">
    <u-button>Button</u-button>
    <u-input v-model="value"></u-input>
</div>
</template>
<script>
export default {
    data() {
        return {
            value: 'Hello world!',
        };
    },
};
</script>
<style module>
.root {
    width: 200px;
    background: #eee;
}
</style>
```

### Style Modifier

You can attach more styles enclosed in braces after code block lang. For example: `html {width: 30%}`.

``` html {width: 30%}
<u-button>Button</u-button>
<u-textarea></u-textarea>
```

## Install

``` bash
npm i -D @kubevue/md-vue-loader
```

## Usage
### Basic

Simply use `@kubevue/md-vue-loader` to load `.md` files and chain it with your `vue-loader`.

``` js
module.exports = {
    module: {
        rules: [{
            test: /\.md$/,
            loader: 'vue-loader!@kubevue/md-vue-loader',
        }],
    },
};
```

Note that to get code highlighting to work, you need to:

- Include one of the highlight.js css files into your project. For example: (https://highlightjs.org/static/demo/styles/atom-one-dark.css).
- Specify a lang in code block. Ref: [creating and highlighting code blocks)](https://help.github.com/articles/creating-and-highlighting-code-blocks/).

### With Options

``` js
module.exports = {
    module: {
        rules: [{
            test: /\.md$/,
            use: [
                'vue-loader',
                {
                    loader: '@kubevue/md-vue-loader',
                    options: {
                        // your preferred options
                    },
                },
            ],
        }],
    },
};
```

### Resource Query

Remember that you can override options in markdown files query.

``` js
const routes = [
    { path: 'article', component: import('./article.md?live=false') },
]
```

### Vue CLI 3

Just chain `@kubevue/md-vue-loader` with `vue-loader` in your `vue.config.js` file:

``` js
module.exports = {
    chainWebpack(config) {
        config.module.rule('md')
            .test(/\.md$/)
            .use('vue-loader')
            .loader('vue-loader')
            .end()
            .use('@kubevue/md-vue-loader')
            .loader('@kubevue/md-vue-loader')
            .end();
    },
};
```

## Options

### live

Enable/Disable live detecting and assembling vue/html code blocks.

- Type: `boolean`
- Default: `true`

### codeProcess

Process after fetching live components from code blocks

- Type: `Function`
- Default: `null`
- @param {string} live - code of live components
- @param {string} code - highlighted code of raw content
- @param {string} content - raw content
- @param {string} lang - code block lang
- @param {string} modifier - string enclosed in braces after lang. Used to modify style by defaults. Actually, You can do whatever you want, but take care about XSS.

For example:

``` javascript
codeProcess(live, code, content, lang, modifier) {
    // do anything
    return `<div${modifier ? ' style="' + modifier + '"' : ''}>${live}</div>` + '\n\n' + code;
}
```

For another example, suppose you have a complex container component called `<code-example>`, with some useful slots.

``` javascript
codeProcess(live, code, content, lang, modifier) {
    // do anything
    return `<code-example lang="${lang}">
    <div${modifier ? ' style="' + modifier + '"' : ''}>${live}</div>
    <div slot="code">${code}</div>
</code-example>\n\n`;
}
```

### wrapper

The wrapper of entire markdown content, can be HTML tag name or Vue component name.

- Type: `string`
- Default: `'section'`

### markdown

[markdown-it](https://github.com/markdown-it/markdown-it) options.

- Type: `Object`
- Default:
``` js
{
    html: true,
    langPrefix: 'lang-',
    highlight: (content, lang, modifier) => {
        content = content.trim();
        lang = lang.trim();

        let hlLang = lang;
        if (lang === 'vue')
            hlLang = 'html';

        let code = '';
        if (hlLang && hljs.getLanguage(hlLang)) {
            try {
                const result = hljs.highlight(hlLang, content).value;
                code = `<pre class="hljs ${markdown.options.langPrefix}${lang}"><code>${result}</code></pre>\n`;
            } catch (e) {}
        } else {
            const result = markdown.utils.escapeHtml(content);
            code = `<pre class="hljs"><code>${result}</code></pre>\n`;
        }

        const live = this.options.live ? this.liveComponent(lang, content) : '';
        return this.options.codeProcess.call(this, live, code, content, lang, modifier);
    },
};
```

### plugins

[markdown-it](https://github.com/markdown-it/markdown-it) plugins list.

- Type: `Array`
- Default: `[]`

For example:

``` javascript
plugins: [
    require('markdown-it-task-lists'),
],
```

### rules

[markdown-it](https://github.com/markdown-it/markdown-it) renderer rules.

- Type: `Object`
- Default: `{}`

For example:

``` javascript
rules: {
  'table_open': () => '<div class="table-responsive"><table class="table">',
  'table_close': () => '</table></div>'
}
```

### preprocess

Process before converting.

- Type: `Function`
- Default: `null`
- @param {string} source - Markdown source content

For example:

``` javascript
preprocess(source) {
  // do anything
  return source
}
```

### postprocess

Process after converting.

- Type: `Function`
- Default: `null`
- @param {string} result - Final converted result

For example:

``` javascript
postprocess(result) {
  // do anything
  return result
}
```

- Type: `Function`
- Default: `null`

## Developing

### test

``` shell
npm run test
open test/index.html
```

### test:options

``` shell
npm run test:options
open test/index.html
```

### test:plugins

``` shell
npm run test:plugins
open test/index.html
```

### test:dev

``` shell
npm run test:dev
```

## Changelog

See [Releases](https://github.com/saashqdev/md-vue-loader/releases)

## Reference

- [vue-markdown-loader](https://github.com/QingWei-Li/vue-markdown-loader)
- [vue-md-loader](https://github.com/wxsms/vue-md-loader)

## License

[MIT](LICENSE)
