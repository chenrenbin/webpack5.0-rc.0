### 源码命令行运行流程（CommonJs---lib/webpack.js）
```
  ./bin/webpack.js --> webpck-cli(/bin/cli.js) --> lib/index.js（lib/webpack.js）
```

### lib/webpack.js---	(options,callback) => {}
```
  验证options：validateSchema(webpackOptionsSchema, options);
  创建compiler: compiler = createCompiler(options) / compiler = createMultiCompiler(options);

  createCompiler(options)：
    获取标准的options（加工）--- options = getNormalizedWebpackOptions(rawOptions)
    设置默认context --- applyWebpackOptionsBaseDefaults(options)
    创建编译实例（内置hooks）--- compiler = new Compiler(options.context)
    挂载node文件操作模块 --- new NodeEnvironmentPlugin({ infrastructureLogging: options.infrastructureLogging }).apply(compiler)
    挂载配置插件plugin的内容 --- options.plugins：plugin.apply(compiler)/plugin.call(compiler, compiler)
    设置默认options --- applyWebpackOptionsDefaults(options)
    执行环境相关钩子 --- compiler.hooks.environment.call(); compiler.hooks.afterEnvironment.call()
    开启默认的所有插件 --- new WebpackOptionsApply().process(options, compiler)
    执行初始化钩子 --- compiler.hooks.initialize.call()
  
  callback ? [watch ? compiler.watch(watchOptions, callback) : compiler.run()] : null
```

### lib\Compiler.js --- 执行编译compiler.run(callback)
```
  this.hooks.beforeRun.callAsync
  this.hooks.run.callAsync

  this.hooks.beforeCompile.callAsync
  this.hooks.compile.call

	const compilation = this.newCompilation(params)---构建编译:
    this.hooks.thisCompilation.call(compilation, params)
    this.hooks.compilation.call(compilation, params)

  this.hooks.make.callAsync(compilation, function)
  this.hooks.finishMake.callAsync(compilation, function)

  compilation.finish
  compilation.seal

  this.hooks.afterCompile.callAsync(compilation, function)

  if (this.hooks.shouldEmit.call(compilation) === false) {
    this.hooks.done.callAsync
    this.hooks.afterDone.call
  } else {
    this.hooks.emit.callAsync

    if (compilation.hooks.needAdditionalPass.call()) {
      this.hooks.done.callAsync
      this.hooks.additionalPass.callAsync
      再次执行compile:  this.hooks.beforeCompile.callAsync...
    } else {
      	writeFile

        this.hooks.done.callAsync
        this.hooks.afterDone.call
    }
  }

  补充：报错时this.hooks.failed.call
```