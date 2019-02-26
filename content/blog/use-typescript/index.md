---
title: 使用TypeScript开发React应用
date: '2019-02-26'
---

9102年，该使用TypeScript了😊。

## 如何给项目脚手架增加TypeScript支持

对象：基于webpack的脚手架

我在三个地方，看到了**六**种配置方法。

### [TypeScript官方文档](http://www.typescriptlang.org/docs/handbook/react-&-webpack.html)

1. 使用[awesome-typescript-loder](https://github.com/s-panferov/awesome-typescript-loader)解析ts文件

    ```bash
    yarn add -D typescript awesome-typescript-loader
    ```

    ```js
    // webpack.config.js
    {
      module: {
        rules: [
          {
            test: /\.tsx?$/,
            loader: 'awesome-typescript-loader'
          }
        ]
      },
      resolve: {
        extensions: [ '.tsx', '.ts', '.js' ]
      }
    }
    ```

2. 配置tsconfig.json
3. 安装React库对应的声明文件

    ```bash
    yarn add @types/react @types/react-dom
    ```

现在可以使用TypeScript写React了。觉得构建速度有点慢？设置`useCache`启用缓存，再结合[hard-source-webpack-plugin](https://github.com/mzgoddard/hard-source-webpack-plugin)。

### [webpack官方文档](https://webpack.js.org/guides/typescript)

1. 使用[ts-loader](https://github.com/TypeStrong/ts-loader)解析ts文件

    ```bash
    yarn add -D typescript ts-loader
    ```

    ```js
    // webpack.config.js
    {
      module: {
        rules: [
          {
            test: /\.tsx?$/,
            loader: 'ts-loader'
          }
        ]
      },
      resolve: {
        extensions: [ '.tsx', '.ts', '.js' ]
      }
    }
    ```

2. 配置tsconfig.json
3. 安装React库对应的声明文件

好了，看上去和第一种方法的区别就是使用的loader不同。ts-loader还提供了其他使用方式：

* 配合[happypack](https://github.com/amireh/happypack)或者[thread-loader](https://webpack.js.org/loaders/thread-loader/)提高构建速度

* ts-loader设置`transpileOnly: true`，关掉类型检查，这样可以提高开发阶段的编译速度。当然，没有类型检查，干嘛还用TypeScript。所以使用[fork-ts-checker-webpack-plugin](https://github.com/Realytics/fork-ts-checker-webpack-plugin)进行类型检查，它会在另外一个进程运行，有更快的构建速度。

* 和babel结合使用。ts-loader先处理ts文件，再交给babel-loder。

### [babel文档](https://babeljs.io/docs/en/babel-preset-typescript)

1. 使用`@babel/preset-typescript`

    ```bash
    yarn add -D @babel/preset-typescript
    ```

    ```js
    // babel.config.js
    {
      presets: ['@babel/preset-typescript']
    }
    ```

2. 配置tsconfig.json
3. 安装React库对应的声明文件

Wow,so easy.

有那么多配置方案，用哪一个🤔

## 讨论讨论

通过对TypeScript和webpack文档提供的配置方案进行比较，两者主要不同点在于各自使用的loader。如果能比较使用哪一个更好，再和babel的方案比较，我就可以得出一个让我满意的方案。

### `awesome-typescript-loader` VS `ts-loader`

awesome-typescript-loader(atl)是这样描述与ts-loader的不同的：alt与babel有较好的集成，可以利用babel的缓存有效的减少构建时间。alt也可以利用另一个进程进行类型检查，使得webpack能够更早的结束编译。但有人测试，在某些情况，alt没有ts-loader快。[issue#497](https://github.com/s-panferov/awesome-typescript-loader/issues/497)

ts-loader: webpack文档提供的用例，并且说了为什么
>We use ts-loader in this guide as it makes enabling additional webpack features, such as importing other web assets, a bit easier.

ts-loader与webpack有很好的配合。可以结合`fork-ts-checker-webpack-plugin`使用，提高构建速度。可以和babel-loader结合使用。alt提供的性能优化ts-loader也能提供，还比alt快。

这个回合的结果：我决定选择ts-loader。

### `ts-loader` VS `babel`

babel的配置是最简单的，配置一下`presets`就好，其它的与前面方案一致。`@babel/preset-typescript`包含了一个插件[@babel/plugin-transform-typescript](https://babeljs.io/docs/en/babel-plugin-transform-typescript)。

该插件的说明：
>Does not type-check its input. For that, you will need to install and set up TypeScript.

这个插件不会进行类型检查。那它做了什么？它在处理ts文件时，其实是直接移除掉了哪些类型声明。所以它的处理速度很快。缺点是不支持几个语法：

* `namespace`。在单独的文件中export好了。
* `const enums`。这个就麻烦了，没办法支持ts的这个语法。
* `export =`和`import = require()`。这种写法还是不要用了。

可以接受。但类型检查怎么办🤔。

babel和TypeScript功能上有重叠了，它们都可以将高版本js语法编译成低版本语法，比如es6->es5。是否我们可以不用babel了？至少进行web应用开发时不行，因为要用polyfill，特别是babel@7带来的新特性（比如`useBuiltIns`）；如果用了ant-design，为了能够按需加载，需要使用`babel-import-plugin`。

babel要用，那`ts-loader`加上`babel-loader`行不行，可以。那么为了提高性能，我们会关掉`ts-loader`的类型检查，加入`fork-ts-checker-webpack-plugin`。

既然这样，就不要这么麻烦了。使用babel + fs-ts-checker-webpack-plugin，前者解析ts文件，后者进行类型检查。实际上，`create-react-app`就是这样做的。

## 我的配置

1. babel设置

    ```bash
    yarn add -D @babel/preset-typescript
    ```

    ```js
    // babel.config.js
    {
      presets: ['@babel/preset-typescript']
    }
    ```

    ```js
    // webpack.config.js
    {
      resolve: {
        extensions: ['.js', '.jsx', '.ts', '.tsx']
      },
      module: {
        rules: [
          {
            test: /\.(js|jsx|ts|tsx)$/,
            exclude: /node_modules/,
            use: {
              loader: 'babel-loader',
              options: {
                cacheDirectory: true
              }
            },
          }
        ]
      }
    }
    ```

2. 类型检查器设置

    ```bash
    yarn add -D typescript fork-ts-checker-webpack-plugin
    yarn add @types/react @types/react-dom
    ```

    ```js
    // webpack.config.json
    {
      plugins: [
        // 参数说明可以到插件git仓库查看
        new ForkTsCheckerWebpackPlugin({
          compilerOptions: {},
          watch: path.resolve(__dirname, 'src'),
          async: false,
          reportFiles: ['src/**/*.{ts,tsx}'],
          checkSyntacticErrors: true,
          silent: true
        })
      ]
    }
    ```

3. [tsconfig.json](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)配置

    ```js
    {
      "compilerOptions": {
        "target": "esnext",
        "lib": [
          "dom",
          "dom.iterable",
          "esnext"
        ],
        "allowJs": true,
        "skipLibCheck": true,
        "esModuleInterop": true,
        "allowSyntheticDefaultImports": true,
        "strict": true,
        "forceConsistentCasingInFileNames": true,
        "module": "esnext",
        "moduleResolution": "node",
        "resolveJsonModule": true,
        "isolatedModules": true,
        "noEmit": true,
        "jsx": "preserve"
      },
      "include": [
        "src"
      ]
    }
    ```

4. 为了导入非ts文件，比如图片或者svg，还需要增加声明文件

    ```js
    // 在TypeScript能够解析到的目录添加一个*.d.ts文件
    // 比如types.d.ts，现在可以导入png资源了，其他类型资源类似
    declare module '*.png' {
      const src: string;
      export default src;
    }
    ```

好了，TypeScript已支持。

## 总结

为了在webpack中使用TypeScript，我比较了多种配置方式，从alt到ts-loader，再到babel。最终选择了babel+一个类型检查器的方案。现在可以用TypeScript写React了，不过少了lint，下篇再说。
