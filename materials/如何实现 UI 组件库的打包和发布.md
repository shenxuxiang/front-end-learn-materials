# 前端必须的 UI 组件库打包和发布

最新闲来无事，把近半年写的一些 UI 组件优化了一下。然后就来了兴趣，又写了一个使用说明文档，还研究了一下 gulp 和 rollup 的打包方式，最终实现了可按需加载的 UI 组件库。库已经丢到 npm 上了（qm-vnit），顺手的朋友可以帮忙点个❤️，使用文档可以点这里 [document](http://47.102.203.153/qm-vnit)。


好了，其他的废话就不说了，开始介绍 UI 组件库打包。以我的项目（qm-vnit）为例，我需要创建一个项目，然后将我开发的库丢到项目中，库的代码全部存放在 `src/lib` 这个目录下面。然后我们需要将库分别打包成 es 模块、commonjs 模块，es、commonjs 模块分别存放在项目根目录的 es、lib 目录中，并保持和原来一样的目录结构。同时还需要包含 css 样式以及 ts 声明文件。


## 1、前期开发流程
关于前端 UI 组件库的开发，我们需要在开发前进行规划和设计，比说样式问题，按需引入的问题，以及库中一些公共资源共享的问题，这些我们都需要考虑。这里我不对如何开发一个公共组件进行介绍。

### 一、关于样式
> 对于样式来说，我们关心的重点并不是使用 less、还是 scss，此时我们应该考虑是是否使用 Css Module。
>
> 因为对于预编译器来说，不论是 less、还是 scss 我们都可以将其转换成 css，并引入到文件当中。
>
> 由于我是将 js、css 是分开进行构建，对于样式使用 Css Module 后，无法将类名注入到 js 文件中去，所以不能使用 Css Module，
>
> 这里我啰嗦一下，如果使用 rollup 将 js、css 一同打包的话，所有的样式内容都将被注入到一个文件中去，就无法实现样式文件的按需加载了
>
> 所以我将 css 使用 gulp 进行打包，并保持与原有的 css 文件相同的目录结构。
>

### 二、关于按需引入
> 对于按需引入的问题，我们不需要考虑太多，因为 webpack 和 vite 都支持 ESM Treehaking。只要我们的代码写的没有问题即可。
>
> 同时我们需要提供一个统一的入口文件，关于入口文件我们需要在库的 package.json 文件中添加 main、和 module、以及 types 字段。
- main   - 表示 commonjs 文件入口
- module - 表示 es 文件入口
- types  - 表示 typescript 声明文件入口
>
> 如果你的项目不区分 commonjs、es，使用的是 umd 格式，那么只需要添加一个 main 字段就行了。
>
> webpack、vite 在开发阶段编译打包时会从 module 字段指定的入口加载文件，如果没有 module 字段，转而就会使用 main 字段。
>
> 生成打包时，直接从 main 字段指定的入口加载文件。



### 三、按需引入第三方库
> 就拿我自己的组件库来说，我自己的组件是依赖 antd、@ant-design/icons 这两个组件库的。
>
> 在打包构建时需要对这些库进行按需引入，这对于打包生成 es 模块是没有问题的，但对于 commonjs 模式来说，就存在问题了。
>
```js
import { Button, Spin, Select } from 'antd';
import { UploadOutlined, PlusOutlined } '@ant-design/icons';
```
> 对于上面的两行代码，如果打包模式为 commonjs 时，生成的代码如下：
```js
const antd = require('antd');
const antDesign = require('@ant-design/icons');
```
> 对于这种情况，当别人使用了你的库后进行生产构建时就会将 `antd`、`@ant-design/icons` 所有的内容都打包到 bundle 中。
> 此时我们需要借助 `babel-plugin-import` 插件，并将其添加至 `babel` 配置文件中即可解决 antd 的按需引入问题。
> 但对于 `@ant-design/icons` 来说，该插件就无能为力了，但是这也难不倒我们，后面我们可以通过自定义 rollup、或 gulp 插件来解决这个问题。


### 四、静态文件资源要求
> 对于一些静态的的资源文件，我推荐将这些资源存放在一个公共目录下，不建议放在每个组件内部。这些文件应该尽可能的被压缩。
> 我们在构建库时一般使用 rollup 构建 js 部分以及被引入的静态资源文件，
> 使用 gulp 对 css 部分以及被引入的静态资源文件进行打包构建。
> 所有的这些静态文件内容都将打包成 base64。

**以上便是我对 UI 组件库前期开发的建议，从第二部分开始就是打包部分了**

## 2、构建 es 模块
> 我们假定现在你的组件库已经开发完成了，剩下的工作就是如何打包了。现在我们开始介绍如何进行库的打包构建
> 首先，我们将库打包成 ES 模块，这需要借助 rollup 帮我们完成。

### install dependencies
```bash
  yarn add --dev rollup @rollup/plugin-typescript @rollup/plugin-babel @rollup/plugin-image @rollup/plugin-alias @rollup/plugin-commonjs @rollup/plugin-node-resolve
```
### code 
```js
  // rollup.js
  import path from 'path';
  import fs from 'fs';
  import { rollup } from 'rollup';
  import typescript from '@rollup/plugin-typescript';
  import babel from '@rollup/plugin-babel';
  import image from '@rollup/plugin-image';
  import alias from '@rollup/plugin-alias';
  import commonjs from '@rollup/plugin-commonjs';
  import nodeResolve from '@rollup/plugin-node-resolve';

  const inputOptions = {
      // 入口文件
      input: path.resolve('src/lib/index.ts'),
      plugins: [
          // rollup 无法直接对 node_modules 中的第三方库进行加载，必须依赖 @rollup/plugin-node-resolve 插件
          nodeResolve(),
          // 如果依赖项是 commonjs 模块，必须要依赖 @rollup/plugin-commonjs 插件才能完成加载
          commonjs(),
          // 指定路径别名，与 webpack 或者 vite 中一样即可
          alias({ entries: { '@': path.resolve('src') } }),
          // 处理 ts
          typescript(),
          // 对于打包库来说，babelHelpers 必须为 runtime，
          babel({
              babelHelpers: 'runtime',
              // 默认情况下，不包含 .ts 和 .tsx，所以我们需要自己设置
              extensions: [ '.ts', '.tsx', '.js', '.jsx', '.cjs', '.mjs' ],
              exclude: /[\\/]node_modules[\\/]/,
          }),
          // 将 ts、js、tsx、jsx 文件中引入的 @ant-design/icons 进行拆分
          // divisionAntDesignIcons(),
          // 将文件中的所有引入 '.less' 全部转换成 '.css'
          // replaceLessToCss(),
          // 将所有的被 js 引入的图片资源全部打包成 base64
          image(),
      ],
      // 对于所有的外部资源，全部采用绝对路径
      makeAbsoluteExternalsRelative: false,
      // 这里不打包 node_modules 中任何的第三方库，已经样式文件，样式文件我们会使用 gulp 进行打包
      external: [ /[\\/]node_modules[\\/]/, /\.less/, /\.css/, /\.scss/ ],
  };

  const outputOptions = {
      dir: path.resolve('es'),
      format: 'es',
      // 保持原有的目录结构
      preserveModules: true,
      // 原来库的代码是存在 src/lib 目录下的。如果要保持原有的目录结构输出到 es 目录下，就需要将 src/lib 这个路径去掉。
      // 否则打包生成的文件路径就包含 /lib，例如 src/lib/a.tsx => es/lib/a.js
      // 这与我们的预期不一致，我们希望的结果应该是 es/a.js。
      preserveModulesRoot: 'src/lib',
  };

  async function build() {
      let bundle = null;
      let exitCode = 0;
      try {
          // 开始构建
          bundle = await rollup(inputOptions);
          // 写入到本地文件系统
          await bundle.write(outputOptions);

      } catch (error) {
          exitCode = 1;
          const msg = error.stack.replace(/^\b/gm, '   ');
          console.log('\n');
          console.log(msg);
          console.log('\n');
      }
      if (bundle) {
          // rollup 构建完成后，需要手动关闭
          await bundle.close();
          console.log('build success');
      }

      process.exit(exitCode);
  }
  build();
```
> 执行 node ./rollup.js 命令，
> 然后你会发现在项目根目录下会生成一个 es 的目录，并与 src/lib 目录中的结构是一样的。
> 到这里，es 模块的打包还没有完成，因为我们还需要将 `import './index.less'` 这样的内容替换成 css，
> 以及对 `import { ... } from '@ant-design/icons'` 内容进行拆分。
> 此时我们要通过自定义 rollup 插件帮我们来完成这两个任务。

### rollup-plugin-replace-less-to-css
```js
  // 将 ts、js、tsx、jsx 文件中引入的 less 转成 css
  function replaceLessToCss() {
      return {
          name: 'rollup-plugin-replace-less-to-css',
          transform (code, id) {
              // id 表示即将进行编译的资源的绝对路径
              if (!code || !/\.(jsx?|tsx?)$/.test(id)) return;
      
              return code.replace(/\.less/g, '.css');
          }
      }
  }
```

### rollup-plugin-division-ant-design-icons
```js 
  // 将 ts、js、tsx、jsx 文件中引入的 @ant-design/icons 进行拆分
  function divisionAntDesignIcons() {
      return {
          name: 'rollup-plugin-division-ant-design-icons',
          transform (code, id) {
              if (!code || !/\.(jsx?|tsx?)$/.test(id)) return null;
          
              // 正则匹配 import { ... } from '@ant-design/icons'
              if (/import {\s*(.+)\s*} from (["'])@ant-design\/icons\2/.test(code)) {
                  const values = RegExp.$1;
                  const regexp = /([^,\s]+)/g;
                  const result = [];
                  // regexp 是一个全局匹配模式，需要匹配多次才能将所有的内容找出
                  while (regexp.exec(values)) {
                      result.push(RegExp.$1);
                  }
                  let text = '';
                  result.forEach(item => {
                      text += `import ${item} from '@ant-design/icons/${item}';\n`;
                  });
                  // 替换掉原来的内容。如果返回 null 则表示不对内容进行修改
                  return code.replace(/import .* from (['"])@ant-design\/icons\1;\n/, text);
              }
          
              return null;
          }
      }
  }
```   
**将以上两个插件添加到 rollupOptions.plugins 中去，至此一个完整的 es 模块就打包成功了**


## 3、构建 commonjs  模块
> 前面我们已经完成了 es 模块的构建，现在我们开始构建 commonjs 模块的构建。
> commonjs 模块的构建我们不再使用 rollup 了，而是使用 gulp 来帮我们完成。
> 在项目根目录下创建 gulpfile.js

### install dependencies
```bash
  yarn add --dev gulp gulp-cli gulp-babel gulp-base64 gulp-less gulp-postcss
```

### 第一步通过 gulp 调用 rollup 完成 es 模块的构建
```js  
  async function buildEs(cb) {
      await buildRollup();
      cb();
  }
```   
    
### 创建一个 gulp task 完成 commonjs 模块构建
```js
  function buildCjs() {
      return gulp.src([ './es/**/*.js' ])
          .pipe(babel({ configFile: './babel.config.lib.cjs' }))
          .pipe(gulp.dest('./lib'));
  }
```  

### 最后通过 gulp.series() 组合任务
```js  
  // 每次构建开始时都将 lib、es 目录删除，并重新构建。
  function clean(cb) {
      fs.rmSync(path.resolve('es'), { force: true, recursive: true });
      fs.rmSync(path.resolve('lib'), { force: true, recursive: true });
      cb();
  }
  
  export default gulp.series(clean, buildEs, buildCjs);
```

> 然后执行 npx gulp。
> 最后就得到我们想要的结果了。
**注意，这里我们为什么要基于打包后 es 模块来打包成 commonjs 模块呢？？？** 
> 这是因为，我们在打包 es 模块时，我们对 引入的 less 和 @ant-design/icons 进行了处理的，
> 如果基于 src/lib 进行打包，同样的事情有需要在处理一遍。所以我们就直接使用 es 模块了。




## 4、样式构建
> 到目前为止，我们已经将 ts 部分全部转换成了 commonjs、和 es 模块。接下来我们将对样式部分进行构建。
> 我们继续在 gulpfile.js 文件中添加任务
```js     
  function buildStyle() {
      // 先读取 src/lib 目录下的所有 less 文件，然后对 less 文件进行编译，编译后会转成 css 文件保存在内存中。
      return gulp.src('./src/lib/**/*.less')
          .pipe(less())
          // 紧接着继续读取 src/lib 目录下的所有 css 文件
          .pipe(gulp.src('./src/lib/**/*.css'))
          // 然后将以上步骤中所有的 css 文件都交由 postcss 进行处理
          // 如果项目根目录下已经存在 postcss.config.cjs 文件，这里可以不用配置 options
          .pipe(postcss({ plugins: [ postcssPresetEnv ] }))
          // 最后是将 css 文件中所有依赖 的静态资源全部打包成 base64 内嵌在 css 文件中
          .pipe(base64())
          // 写入到 es 目录下
          .pipe(gulp.dest('es'))
          // 写入到 lib 目录下
          .pipe(gulp.dest('lib'));
  }

  export default gulp.series(clean, buildEs, buildCjs, buildStyle);  
```   
> 此时，执行 npx gulp 。
> 最后就得到我们想要的结果了。




## 5、生成 TS 声明文件
> 这是打包的最后一步了，我们只需要将生成 ts 声明文件输出到 lib、es 目录结构中就可以了。
> 这听上去比较简单，但我在打包过程中发现，不论是使用 tsc 命令生成声明文件，还是借助 rollup/gulp 工具，都无法满足我们需求，
> 因为以上两种方法，在打包后的产物中，会有一个 lib 这个前置目录存在。这与我们的期望有所不同。
> 最终，我的解决办法时先使用 tsc 生成 .d.ts 文件，然后借助 gulp 进行处理。
```js
  const buildTsc = gulp.series(
      function() {
          // 这里使用的 NodeJS 的 child_process 帮我们完成 tsc 命令执行。
          return child_process.exec('npx tsc -p tsconfig.lib.json');
      },
      function() {
          return gulp.src('./dts/**/*.d.ts')
              .pipe(through(
                  { objectMode: true },
                  function(chunk, encoding, callback) {
                      const newBase = path.join(chunk.base, 'lib');
                      // gulp.dest() 在输出时将每个资源的 chunk.base 路径全部截取掉。
                      // 默认情况下，base 就是 “/Users/shenxuxiang/Documents/work/vnit/dts”，这取决于 gulp.src() 的参数
                      // 这里我们给原本包含 lib 目录的资源修改为 base = base + '/lib'，这样在输出时就会将 base + '/lib' 部分的路径全部截取掉。
                      if (chunk.path.startsWith(newBase)) chunk.base = newBase;
  
                      callback(null, chunk);
                  }
              ))
              // 最后输出到本地的 lib、es 目录
              .pipe(gulp.dest('es'))
              .pipe(gulp.dest('lib'));
      },
      function (cb) {
          // 最后删除 dts 文件目录
          fs.rmSync(path.resolve('dts'), { force: true, recursive: true });
          cb();
      }
  )
  
  
  export default gulp.series(clean, buildEs, buildCjs, buildStyle, buildTsc);
```

> 注意，我是单独指定一个 `tsconfig.lib.json` 文件来生成 .d.ts 文件，
>
> 这是因为默认的 `tsconfig.json` 是不会输出内容的，而我们指定的 `tsconfig.lib.json` 只需要输出声明文件即可；
>
> `tsconfig.lib.json` 和 `tsconfig.json` 不同之处看下面
```json
  {
    "declaration": true,
    "emitDeclarationonly": true,
    "declarationDir": "dts",
    // 去掉noEmit
    // "noEmit": true
  }
```
> 好了，到这里我们所有的打包都已经完成了，剩下的就是如何发布 npm 包了。


## 6、npm publish
> 在发布之前我们应该做一下几件事情
- 1、添加 readme
- 2、修改 version、name 字段 
- 3、修改 main、module、types 等字段  
- 4、修改 files 字段
- 5、将一些公共依赖项从 dependencies中移动到 peerDependencies 中
- 6、添加 keywords 和 description
- 7、添加 contributes 和 respository
```json
    {
        "name": "qm-vnit",
        "version": "0.0.1-beta1",
        "main": "./lib/index.js",
        "module": "./es/index.js",
        "types": "./lib/index.d.ts",
        "files": [ "lib", "es" ],
        "keywords": [ "Antd", "React antd", "React UI", "UI" ],
        "description": "React antd ui library",
        "contributes": [ "https://github.com/shenxuxiang/" ],
        "respository": "https://github.com/shenxuxiang/qm-vnit"
    }
```  
> 如果以上你都已经准备好了，就可以发布了。

## 总结
关于库的打包方式，我研究了挺长一段时间的，发现使用 webpack、vite 都没办法做到按需引入的功能，并且生成的 commonjs 也不能正常使用，必须打包成 umd 格式才可以。当然这可能与我的能力有问题，试了好久都没有成功，所以才使用了 rollup、和 gulp 来完成。
还有一点就是所有的 node_module 中的包都被定义为外部依赖了，所有的这些都没有被打包到 bundle 中，rollup 将这些代码都替换成了 `@babel/runtime-corejs3` 的引用，这些代码将在应用中被打包。