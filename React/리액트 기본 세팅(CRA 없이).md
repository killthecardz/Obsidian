
```sh
npm i
	react
	react-dom
	typescript
	@types/react
	@types/react-dom

	-D eslint (문법오류 캐치)
	-D prettier
	-D eslint-plugin-prettier
	-D eslint-config-prettier
	-D webpack
	-D @babel/core
	-D @babel-loader
	-D @babel/preset-env
	-D @babel/preset-react

	-D @types/webpack
	-D @types/node
	-D @babel/preset-typescript

	style-loader
	css-loader
	cross-env
	ts-node

	webpack-dev-server -D (핫 리로딩, cors에러 해결 용도)
	webpack-cli
	-D @types/webpack-dev-server
	@pmmmwh/react-refresh-webpack-plugin
	react-refresh
	@loadable/component (컴포넌트 분할 로딩해서 로딩속도 빠르게 한다)
	@types/loadable__component
	
	
```

## .prettierrc 파일 설정

```JS
{

  "printWidth": 120,

  "tabWidth": 2,

  "singleQuote": true,

  "trailingComma": "all",

  "semi": true

}
```

## .eslintrc 파일 설정

```js
{

  "extends": ["plugin:prettier/recommended", "react-app"]

}
```

## tsconfig.json 파일 설정

```js
{

  "compilerOptions": {

    "esModuleInterop": true, // import * as React from 'react' 여기서 * as 생략가능

    "sourceMap": true, // 에러시 원래 파일 위치 찾아준다

    "lib": ["ES2020", "DOM"],

    "jsx": "react",

    "module": "esnext", // 모듈 최신문법인  import export 사용가능

    "moduleResolution": "Node",

    "target": "es5",

    "strict": true, // 코드를 엄격하게 검사한다

    "resolveJsonModule": true,

    // baseUrl 이랑 paths 는 import export 시의 경로 설정

    "baseUrl": ".",

    "paths": {

      "@hooks/*": ["hooks/*"],

      "@components/*": ["components/*"],

      "@layouts/*": ["layouts/*"],

      "@pages/*": ["pages/*"],

      "@utils/*": ["utils/*"],

      "@typings/*": ["typings/*"]

    }

  },

  "ts-node": {

    "compilerOptions": {

      "module": "commonjs",

      "moduleResolution": "Node",

      "target": "es5",

      "esModuleInterop": true

    }

  }

}
```


## 바벨 설정 (webpack.config.ts)

```js
import path from 'path';

import ReactRefreshWebpackPlugin from '@pmmmwh/react-refresh-webpack-plugin';

import webpack, { Configuration as WebpackConfiguration } from "webpack";

import { Configuration as WebpackDevServerConfiguration } from "webpack-dev-server";

import { BundleAnalyzerPlugin } from 'webpack-bundle-analyzer';

  

interface Configuration extends WebpackConfiguration {

  devServer?: WebpackDevServerConfiguration;

}

  

import ForkTsCheckerWebpackPlugin from 'fork-ts-checker-webpack-plugin';

  

const isDevelopment = process.env.NODE_ENV !== 'production';

  

const config: Configuration = {

  name: 'sleact',

  mode: isDevelopment ? 'development' : 'production',

  devtool: !isDevelopment ? 'hidden-source-map' : 'eval',

  resolve: {

    extensions: ['.js', '.jsx', '.ts', '.tsx', '.json'],

    alias: {

      '@hooks': path.resolve(__dirname, 'hooks'),

      '@components': path.resolve(__dirname, 'components'),

      '@layouts': path.resolve(__dirname, 'layouts'),

      '@pages': path.resolve(__dirname, 'pages'),

      '@utils': path.resolve(__dirname, 'utils'),

      '@typings': path.resolve(__dirname, 'typings'),

    },

  },

  entry: {

    app: './client',

  },

  module: {

    rules: [

      {

        test: /\.tsx?$/,

        loader: 'babel-loader',

        options: {

          presets: [

            [

              '@babel/preset-env',

              {

                targets: { browsers: ['last 2 chrome versions'] },

                debug: isDevelopment,

              },

            ],

            '@babel/preset-react',

            '@babel/preset-typescript',

          ],

          env: {

            development: {

              plugins: [require.resolve('react-refresh/babel')],

            },

          },

        },

        exclude: path.join(__dirname, 'node_modules'),

      },

      {

        test: /\.css?$/,

        use: ['style-loader', 'css-loader'],

      },

    ],

  },

  plugins: [

    new ForkTsCheckerWebpackPlugin({

      async: false,

      // eslint: {

      //   files: "./src/**/*",

      // },

    }),

    new webpack.EnvironmentPlugin({ NODE_ENV: isDevelopment ? 'development' : 'production' }),

  ],

  output: {

    path: path.join(__dirname, 'dist'),

    filename: '[name].js',

    publicPath: '/dist/',

  },

  devServer: {

    historyApiFallback: true, // react router

    port: 3090,

    devMiddleware: { publicPath: '/dist/' },

    static: { directory: path.resolve(__dirname) },

    proxy: {

      '/api/': {

        target: 'http://localhost:3095',

        changeOrigin: true,

      },

    },

  },

};

  

if (isDevelopment && config.plugins) {

  config.plugins.push(new webpack.HotModuleReplacementPlugin());

  config.plugins.push(new ReactRefreshWebpackPlugin());

  config.plugins.push(new BundleAnalyzerPlugin({ analyzerMode: 'server', openAnalyzer: true }));

}

if (!isDevelopment && config.plugins) {

  config.plugins.push(new webpack.LoaderOptionsPlugin({ minimize: true }));

  config.plugins.push(new BundleAnalyzerPlugin({ analyzerMode: 'static' }));

}

  

export default config;
```