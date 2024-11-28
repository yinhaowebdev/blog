title: 踩坑-create-react-app 支持decorators的方法
author: Billy Yin
tags: []
categories: []
date: 2019-07-23 22:14:38
---
```bash
cnpm i @babel/core --save
cnpm i @babel/plugin-proposal-class-properties --save
cnpm i @babel/plugin-proposal-decorators --save
cnpm i @babel/preset-env --save
```

创建 .babelrc
```
{
    "presets": [
        "@babel/preset-env"
    ],
    "plugins": [
        [
            "@babel/plugin-proposal-decorators",
            {
                "legacy": true
            }
        ],
        [
            "@babel/plugin-proposal-class-properties",
            {
                "loose": true
            }
        ]
    ]
}
```

创建config-override.js

```
const path = require('path')
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
const { override, addDecoratorsLegacy } = require('customize-cra')

function resolve(dir) {
    return path.join(__dirname, dir)
}

const customize = () => (config, env) => {
    config.resolve.alias['@'] = resolve('src')
    if (env === 'production') {
        config.externals = {
            'react': 'React',
            'react-dom': 'ReactDOM'
        }
        // config.plugins.push(new BundleAnalyzerPlugin())
    }

    return config
};


module.exports = override(addDecoratorsLegacy(), customize())
```
安装依赖

```
cnpm i customize-cra --save-dev
cnpm i react-app-rewired --save-dev
cnpm i webpack-bundle-analyzer --save-dev
```


修改package.json
```
...
"scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test",
    "eject": "react-app-rewired eject"
  },
...

```
