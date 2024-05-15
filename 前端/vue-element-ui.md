# 解决vue启动时出现Missing script: “serve“的错误

这时就可以去package.json去查看[vue](https://so.csdn.net/so/search?q=vue)项目启动方式，发现它是以dev方式启动这时就可以去package.json去查看[vue](https://so.csdn.net/so/search?q=vue)项目启动方式，发现它是以dev方式启动

```json
"scripts": {
    "dev": "vue-cli-service serve",
    "build:prod": "vue-cli-service build",
    "build:stage": "vue-cli-service build --mode staging",
    "preview": "node build/index.js --preview",
    "svgo": "svgo -f src/icons/svg --config=src/icons/svgo.yml",
    "lint": "eslint --ext .js,.vue src",
    "test:unit": "jest --clearCache && vue-cli-service test:unit",
    "test:ci": "npm run lint && npm run test:unit"
  }
```

`"dev": "vue-cli-service serve",`修改`"serve": "vue-cli-service serve",`