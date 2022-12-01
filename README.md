# loader plugin 说明

- `cross-env` 跨平台设置和使用环境变量的脚本

  ```typescript
  "build": "cross-env NODE_ENV=production webpack --config ./config/webpack.prod.js"
  ```

- `webpack-dev-server` 用于启动 webpack 服务
  ```typescript
  "start": "cross-env NODE_ENV=development webpack-dev-server --config ./config/webpack.dev.js"
  ```
- `html-webpack-plugin` 用于将 `JavaScript` 文件打包在一起，打包后的文件用于在浏览器中使用

  ```typescript
  // webpack.common.js
  const HtmlWebpackPlugin = require("html-webpack-plugin")
  plugins: [
    new HtmlWebpackPlugin({
      template: path.join(__dirname, "../tempate.html"),
      filename: "index.html",
    }),
  ],
  ```

- `@babel/core` 是 `Babel` 的核心库，所有的核心 `Api` 都在这个库里，这些 `Api` 供 `babel-loader` 调用
- `babel-loader` 是 `@babel/core` 在做 `es6` 的语法转换和弥补缺失的功能，但是在使用 `webpack` 打包 `js` `时，webpack` 并不知道应该怎么去调用这些规则去编译 `js`。这时就需要 `babel-loader` 了，它作为一个中间桥梁，通过调用 `babel/core` 中的 `api` 来告诉 `webpack` 要如何处理 `js`Z
- `@babel/preset-react` 预设了一些 `Babel` 插件，主要负责编译 `React` 语法
- `@babel/preset-typescript` 预设了一些 `Babel` 插件，主要负责编译 `Typescript` 语法

  ```typescript
  // webpack.common.js
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)/,
        loader: "babel-loader",
      },
    ],
  },

  // .babelrc
  {
    "presets": [
      "@babel/preset-react",
      "@babel/preset-typescript"
    ]
  }
  ```

- `css-loader` 是 `js` 中导入了 `css` ，那么就需要使用 `css-loader` 来识别这个模块，通过特定的语法规则进行转换内容最后导出
- `style-loader` 是通过一个 `js` 脚本创建一个 `style` 标签，里面包含一些样式。`style-loader` 是不能单独使用的，应为它并不负责解析 css 之前的依赖关系，每个 `loader` 的功能都是单一的，各自拆分独立

  ```typescript
  // webpack.common.js
  module: {
    rules: [
      {
        // other
      },
      {
        test: /\.(css)$/,
        use: ["style-loader", "css-loader"],
      },
    ],
  },
  ```

- `less-loader` 编译 `less` 文件
- `sass-loader` 编译 `sass` 文件

  ```typescript
  // webpack.common.js
  module: {
    rules: [
      {
        // other
      },
      {
        test: /\.(less)$/,
        use: ["style-loader", "css-loader", "less-loader"],
      },
      {
        test: /\.(scss)$/,
        use: ["style-loader", "css-loader", "sass-loader"],
      },
    ],
  },
  ```

- `postcss-loader` 需要在 `css-loader` 之前，`less-loader`或者 `scss-loader` 之前之后使用，所以修改 `webpack.common.js` 如下修改，可支持 `less` 和 `sass` 预编译文件支持自动添加浏览器前缀

  ```typescript
  // webpack.common.js
  module: {
    rules: [
      {
        // other
      },
      {
        test: /\.(css)$/,
        use: [
          "style-loader",
          {
            loader: "css-loader",
            options: {
              importLoaders: 1,
            },
          },
          {
            loader: "postcss-loader",
          },
        ],
      },
      {
        test: /\.(less)$/,
        use: [
          "style-loader",
          {
            loader: "css-loader",
            options: {
              importLoaders: 2,
            },
          },
          {
            loader: "postcss-loader",
          },

          "less-loader",
        ],
      },
      {
        test: /\.(scss)$/,
        use: [
          "style-loader",
          {
            loader: "css-loader",
            options: {
              importLoaders: 2,
            },
          },
          {
            loader: "postcss-loader",
          },
          "sass-loader",
        ],
      },
    ],
  },

  // postcss.config.js
  module.exports = {
    plugins: [require("autoprefixer")],
  }

  // package.json
  "browserslist": [
    ">0.2%",
    "not dead",
    "ie >= 9",
    "not op_mini all"
  ]
  ```

- `url-loader` `file-loader` 对引入的图片后者字体做一些处理
  ```typescript
  module: {
    rules: [
      {
        // other
      },
      {
        test: /\.(bmp|gif|png|jpe?g)$/,
        use: [
          {
            loader: "url-loader",
            options: {
              limit: 10 * 1024,
              name: "[name].[contenthash:8].[ext]",
              outputPath: "assets/images",
            },
          },
        ],
      },
      {
        test: /\.(ttf|woff|woff2|eot|otf)$/,
        use: [
          {
            loader: "file-loader",
            options: {
              name: "[name].[contenthash:8].[ext]",
              outputPath: "assets/fonts",
            },
          },
        ],
      },
    ],
  },
  ```
- `clean-webpack-plugin` 打包前清理 dist 文件夹

  ```typescript
  // webpack.prod.js
  const { merge } = require("webpack-merge");
  const common = require("./webpack.common.js");
  const { CleanWebpackPlugin } = require("clean-webpack-plugin");

  module.exports = merge(common, {
    mode: "production",
    plugins: [new CleanWebpackPlugin()],
  });
  ```

- `mini-css-extract-plugin` `style` 样式打包成文件，而不是在 `html` 中内联

  ```typescript
  // webpack.common.js
  const MiniCssExtractPlugin = require("mini-css-extract-plugin");

  module.exports = {
    plugins: [
      // other
      new MiniCssExtractPlugin(),
    ],
    module: {
      rules: [
        {
          test: /\.(css)$/,
          use: [
            MiniCssExtractPlugin.loader,
            {
              loader: "css-loader",
              options: {
                importLoaders: 1,
              },
            },
            {
              loader: "postcss-loader",
            },
          ],
        },
        {
          test: /\.(less)$/,
          use: [
            MiniCssExtractPlugin.loader,
            {
              loader: "css-loader",
              options: {
                importLoaders: 2,
              },
            },
            {
              loader: "postcss-loader",
            },

            "less-loader",
          ],
        },
        {
          test: /\.(scss)$/,
          use: [
            MiniCssExtractPlugin.loader,
            {
              loader: "css-loader",
              options: {
                importLoaders: 2,
              },
            },
            {
              loader: "postcss-loader",
            },
            "sass-loader",
          ],
        },
      ],
    },
  };
  ```

- `css-minimizer-webpack-plugin` `css` 代码压缩

  ```typescript
  // webpack.prod.js
  const CssMinimizerWebpackPlugin = require("css-minimizer-webpack-plugin");
  module.exports = merge(common, {
    mode: "production",
    plugins: [new CssMinimizerWebpackPlugin()],
  });
  ```

- `webpackbar` 控制台显示打包进度

  ```typescript
  // webpack.common.js
  const WebpackBar = require("webpackbar");
  module.exports = {
    // other
    plugins: [
      // other
      new WebpackBar(),
    ],
  };
  ```

- `webpack-bundle-analyzer` 可视化打包文件大小占比

  ```typescript
  // webpack.prod.js
  const BundleAnalyzerPlugin =
    require("webpack-bundle-analyzer").BundleAnalyzerPlugin
  module.exports = merge(common, {
    mode: "production",
    plugins: [
      // other
      new BundleAnalyzerPlugin({
        analyzerMode: "disabled", // 不启动展示打包报告的http服务器
        generateStatsFile: true, // 是否生成stats.json文件
      }),
    ],
  })

  // package.json
  "analyz": "webpack-bundle-analyzer --port 8887 ./dist/stats.json"
  ```

- `Externals` 将一些代码从输出的 `bundle` 中排除

  ```typescript
  // webpack.prod.js
  module.exports = {
  // other
  externals: {
      react: "React",
      "react-dom": "ReactDOM",
    },
  }

  // index.html
  <body>
    <div id="root"></div>
  </body>
  <script src="https://cdn.staticfile.org/react/16.4.0/umd/react.development.js"></script>
  <script src="https://cdn.staticfile.org/react-dom/16.4.0/umd/react-dom.development.js"></script>
  ```

# 参考链接

> [从 0 搭建一个简易版的 webpack+react 工程](https://juejin.cn/post/7076438081225785375)
