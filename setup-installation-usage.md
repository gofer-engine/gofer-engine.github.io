[🏠 Gofer Engine](./index.md) > Setup, Installation, and Usage

# Setup, Installation, and Usage

I find it helpful to newer developers to have a step-by-step guide to get them started. If you are experienced already with Node projects, you can skip to the [Installation](#installation) section.

## Prerequisites

It is recommended to use the latest version of Node.js. You can download it [here](https://nodejs.org/en/download/).

1. Create a new project directory to contain your Node application.
1. Open a terminal/command prompt in your project folder.
1. Create a new node project by running `npm init` and follow the prompts.

The following packages are recommended to have installed, but not required:

- [TypeScript](https://www.npmjs.com/package/typescript) - A superset of JavaScript that compiles to plain JavaScript.
  Install by running: `npm install -D typescript`

- [nodemon](https://www.npmjs.com/package/nodemon) - A utility that will monitor for any changes in your source and automatically restart your server.
  Install by running: `npm install -D nodemon`

- [ts-node](https://typestrong.org/ts-node/) - A utility that will allow you to run TypeScript files directly without having to compile them first.
  Install by running: `npm install -D ts-node`

If you are using TypeScript, which is optional but highly recommended, then you should create a `tsconfig.json` file in your project's root directory. You can use the following as a starting point:

```json
{
  "compilerOptions": {
    "target": "ES2015",
    "module": "commonjs",
    "moduleResolution": "node",
    "strict": true,
    "esModuleInterop": true,
    "outDir": "out",
    "sourceMap": true,
    "rootDir": "src"
  },
  "include": ["src/**/*"]
}
```

_Just a side note, but if you are not already using VS-Code IDE, we highly recommend it!_

## Installation

1. Open a terminal in your project folder
1. Run `npm install @gofer-engine/engine` to install Gofer Engine dependency.

## Usage

1. Create a new directory called `src` in your project's root directory.
1. Create a new file called `server.ts` in your project's `src` folder.
1. Add the following code to the `server.ts` file:

```ts
import gofer from '@gofer-engine/engine'

gofer
  .listen('tcp', 'localhost', 5500)
  .name('My First Channel')
  .ack()
  .store({ file: {} })
  .run()
```

Alternatively, you could pass a configuration object:

```ts
import gofer, { ChannelConfig } from '@gofer-engine/engine'

const channel: ChannelConfig = {
  name: 'My First Channel',
  source: {
    kind: 'tcp',
    tcp: {
      host: 'localhost',
      port: 5500,
    },
  },
  ingestion: [
    { kind: 'ack', ack: {} },
    { kind: 'store', file: {} },
  ],
  routes: [],
}

gofer.configs([channel])
```

The above examples add a single channel that listens on `localhost` port `5500` via `tcp` for HL7 v2.x messages. It will reply with an acknowledgment message and write the message to a file in the default `./local` directory.

See the [Developing Interface Channels in OOP style](./developing-interface-channels-in-oop.md) or the [Developing Interface Channels with Config Files]('./developing-interface-channels-with-configs.md) pages for more information on building and configuring channels.

### Running in Development

1. Add a script to your `package.json` file:

```json
"scripts": {
  "dev": "nodemon src/server.ts"
}
```

2. Run `npm run dev` to start the server.

_By using nodemon as you make changes to your project files the server will restart._

### Version Control with Git

One of the beauties of using Gofer Engine is that you can version control your channels, by simply checking your Node project into a git repository. This allows you to easily branch and merge changes to your channels to ease with development and testing of new interfaces and changes to existing interfaces.

With git comes other tools to help develop pipelines for CI/CD. Here are some tools to check out:

- [jest](https://jestjs.io/) - A delightful JavaScript (and TypeScript) Testing Framework with a focus on simplicity,
- [husky](https://typicode.github.io/husky/#/) - A utility to make modern native git hooks easy.

And if you need more resources for CI/CD then check out these [38 Best CI/CS Tools for 2023](https://www.lambdatest.com/blog/best-ci-cd-tools/).

If you are an on-premise-only environment, then might I recommend [Bonobo Git Server](https://bonobogitserver.com/) as an alternative to [GitHub](http://github.com/).

### Preparing for Production

Add a build and start script to your package.json file:

```json
"scripts": {
  "build": "npx tsc",
  "start": "node out/server.js"
}
```

### Deploying to Production

The workflow outlined below uses a git repository. You could deploy to production using alternative means.

1. On your production server run `git clone <your-project-repo>` to clone your repository.
2. Run `npm install && run build && npm start` to install dependencies, build, and start the server.

If going to production in a Windows environment, you could use this setup guide to run the node server as a Windows Service: [https://www.helpmegeek.com/run-nodejs-application-as-windows-service/](https://www.helpmegeek.com/run-nodejs-application-as-windows-service/)

If going to production in a Linux environment, you could use this setup guide to run the server: [https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-18-04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-18-04)

You could alternatively use docker with a docker-compose file to clone your repository, install dependencies, build the project, and start the server in a container. This configuration is out of the current scope, but if anyone needs help or wants to contribute, please reach out.
