<p align="center">
  <img alt="NestJS-Pino logo" src="./logo.png"/>
</p>

<h1 align="center">NestJS-Pino</h1>

<p align="center">
  <a href="https://www.npmjs.com/package/nestjs-pino">
    <img alt="npm" src="https://img.shields.io/npm/v/nestjs-pino" />
  </a>
  <a href="https://travis-ci.org/iamolegga/nestjs-pino">
    <img alt="Travis (.org)" src="https://img.shields.io/travis/iamolegga/nestjs-pino" />
  </a>
  <a href="https://coveralls.io/github/iamolegga/nestjs-pino?branch=master">
    <img alt="Coverage Status" src="https://coveralls.io/repos/github/iamolegga/nestjs-pino/badge.svg?branch=master" />
  </a>
  <a href="https://snyk.io/test/github/iamolegga/nestjs-pino">
    <img alt="Snyk Vulnerabilities for npm package" src="https://img.shields.io/snyk/vulnerabilities/npm/nestjs-pino" />
  </a>
  <img alt="David" src="https://img.shields.io/david/iamolegga/nestjs-pino">
  <img alt="Dependabot" src="https://badgen.net/dependabot/iamolegga/nestjs-pino/?icon=dependabot">
  <img alt="Supported platforms: Express & Fastify" src="https://img.shields.io/badge/platforms-Express%20%26%20Fastify-green" />
</p>

<p align="center">✨✨✨ Platform agnostic logger for NestJS based on Pino with <b>REQUEST CONTEXT IN EVERY LOG</b> ✨✨✨</p>

## Example

Import module with `LoggerModule.forRoot(...)` or `LoggerModule.forRootAsync(...)`:

```ts
import { LoggerModule } from 'nestjs-pino';

@Module({
  imports: [LoggerModule.forRoot()],
  controllers: [AppController],
  providers: [MyService]
})
class MyModule {}
```

In controller let's use `Logger` - class with the same API as [built-in NestJS logger](https://docs.nestjs.com/techniques/logger):

```ts
import { Logger } from 'nestjs-pino';

@Controller()
export class AppController {
  constructor(
    private readonly myService: MyService,
    private readonly logger: Logger
  ) {}

  @Get()
  getHello(): string {

    // pass message
    this.logger.log("getHello()");

    // also we can pass context
    this.logger.log("getHello()", AppController.name);

    return `Hello ${this.myService.getWorld()}`;
  }
}
```

Let's compare it to another one logger - `PinoLogger`, it has same _logging_ API as `pino` instance.

For example in service it will be used instead of previous one:

```ts
import { PinoLogger } from 'nestjs-pino';

@Injectable()
export class MyService {
  constructor(private readonly logger: PinoLogger) {}

  getWorld(...params: any[]) {
    this.logger.info({ context: MyService.name }, "getWorld(%o)", params);
    return "World!";
  }
}
```

Also context can be set just once in `constructor` instead of every call:

```ts
import { PinoLogger } from 'nestjs-pino';

@Injectable()
export class MyService {
  constructor(private readonly logger: PinoLogger) {
    logger.setContext(MyService.name);
  }

  getWorld(...params: any[]) {
    this.logger.info("getWorld(%o)", params);
    return "World!";
  }
}
```

Also context can be set at injection via decorator `@InjectPinoLogger(...)`:

```ts
import { PinoLogger, InjectPinoLogger } from 'nestjs-pino';

@Injectable()
export class MyService {
  constructor(
    @InjectPinoLogger(MyService.name) private readonly logger: PinoLogger) {
  }

  getWorld(...params: any[]) {
    this.logger.info("getWorld(%o)", params);
    return "World!";
  }
}
```

And also `Logger` can be set as app logger, as it is compatible with [built-in NestJS logger](https://docs.nestjs.com/techniques/logger):

```ts
import { Logger } from 'nestjs-pino';

const app = await NestFactory.create(AppModule, { logger: false });
app.useLogger(app.get(Logger));
```

Output:

```json
// Logs by app itself
{"level":30,"time":1570470154387,"pid":17383,"hostname":"my-host","context":"RoutesResolver","msg":"AppController {/}: true","v":1}
{"level":30,"time":1570470154391,"pid":17383,"hostname":"my-host","context":"RouterExplorer","msg":"Mapped {/, GET} route true","v":1}
{"level":30,"time":1570470154405,"pid":17383,"hostname":"my-host","context":"NestApplication","msg":"Nest application successfully started true","v":1}

// Logs by injected Logger and PinoLogger in Services/Controllers
// Every log has it's request data and unique `req.id` (per process)
{"level":30,"time":1570470161805,"pid":17383,"hostname":"my-host","req":{"id":1,"method":"GET","url":"/","headers":{...},"remoteAddress":"::1","remotePort":53957},"context":"AppController","msg":"getHello()","v":1}
{"level":30,"time":1570470161805,"pid":17383,"hostname":"my-host","req":{"id":1,"method":"GET","url":"/","headers":{...},"remoteAddress":"::1","remotePort":53957},"context":"MyService","msg":"getWorld([])","v":1}

// Automatic logs of every request/response
{"level":30,"time":1570470161819,"pid":17383,"hostname":"my-host","req":{"id":1,"method":"GET","url":"/","headers":{...},"remoteAddress":"::1","remotePort":53957},"res":{"statusCode":304,"headers":{...}},"responseTime":15,"msg":"request completed","v":1}
```

## Comparison with others

There are other Nestjs loggers. The key purposes of this module are:
  - to be compatible with built-in `LoggerService`
  - to log with JSON (thanks to `pino` - [super fast logger](https://github.com/pinojs/pino/blob/master/docs/benchmarks.md)) ([why JSON?](https://jahed.dev/2018/07/05/always-log-to-json/))
  - to log every request/response automatically (thanks to `pino-http`)
  - to bind request data to the logs automatically from any service on any application layer without passing request context
  - to have another one alternative `PinoLogger` for experienced `pino` users for more comfortable usage.

| Logger             | Nest App logger | Logger service | Autobind request data to logs |
| ------------------ | :-------------: | :------------: | :---------------------------: |
| nest-morgan        |        -        |       -        |               -               |
| nest-winston       |        +        |       +        |               -               |
| nestjs-pino-logger |        +        |       +        |               -               |
| __nestjs-pino__    |        +        |       +        |               +               |

## Install

```sh
npm i nestjs-pino
```

## Register module

### Default params

Just import `LoggerModule` to your module:

```ts
import { LoggerModule } from 'nestjs-pino';

@Module({
  imports: [LoggerModule.forRoot()],
  ...
})
class MyModule {}
```

### Synchronous configuration

`LoggerModule.forRoot` has the same API as [pino-http](https://github.com/pinojs/pino-http#pinohttpopts-stream):

```ts
import { LoggerModule } from 'nestjs-pino';

@Module({
  imports: [
    LoggerModule.forRoot(
      {
        name: 'add some name to every JSON line',
        level: process.env.NODE_ENV !== 'production' ? 'debug' : 'info',
        prettyPrint: process.env.NODE_ENV !== 'production',
        useLevelLabels: true,
        // and all the others...
      },
      someWritableStream
    )
  ],
  ...
})
class MyModule {}
```


### Asynchronous configuration

With `LoggerModule.forRootAsync` you can, for example, import your `ConfigModule` and inject `ConfigService` to use it in `useFactory` method.

`useFactory` should return either:
- `null`
- or `typeof arguments` of [pino-http](https://github.com/pinojs/pino-http#pinohttpopts-stream)
- or `Promise` of it

Here's an example:

```ts
import { LoggerModule } from 'nestjs-pino';

@Injectable()
class ConfigService {
  public readonly level = "debug";
}

@Module({
  providers: [ConfigService],
  exports: [ConfigService]
})
class ConfigModule {}

@Module({
  imports: [
    LoggerModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: async (config: ConfigService) => {
        await somePromise();
        return { level: config.level };
      }
    })
  ],
  ...
})
class TestModule {}
```

Or you can just pass `ConfigService` to `providers`, if you don't have any `ConfigModule`:

```ts
import { LoggerModule } from 'nestjs-pino';

@Injectable()
class ConfigService {
  public readonly level = "debug";
  public readonly stream = stream;
}

@Module({
  imports: [
    LoggerModule.forRootAsync({
      providers: [ConfigService],
      inject: [ConfigService],
      useFactory: (config: ConfigService) => {
        return [{ level: config.level }, config.stream];
      }
    })
  ],
  controllers: [TestController]
})
class TestModule {}
```

### Extreme mode
> In essence, `extreme` mode enables even faster performance by Pino.

Please, read [pino extreme mode docs](https://github.com/pinojs/pino/blob/master/docs/extreme.md#extreme-mode) first. There is a risk of some logs being lost, but you can [minimize it](https://github.com/pinojs/pino/blob/master/docs/extreme.md#log-loss-prevention).

If you know what you're doing, you can enable it like so:

```ts
import * as pino from 'pino';
import { LoggerModule } from 'nestjs-pino';

const dest = pino.extreme();
const logger = pino(dest);

@Module({
  imports: [LoggerModule.forRoot({ logger })],
  ...
})
class MyModule {}
```

## Usage as Logger service

As it said before, there are 2 logger classes:

- `Logger` - implements standard NestJS `LoggerService` interface. So if you are familiar with [built-in NestJS logger](https://docs.nestjs.com/techniques/logger), you are good to go.
- `PinoLogger` - implements standard `pino` _logging_ methods: [trace](https://github.com/pinojs/pino/blob/master/docs/api.md#loggertracemergingobject-message-interpolationvalues), [debug](https://github.com/pinojs/pino/blob/master/docs/api.md#loggerdebugmergingobject-message-interpolationvalues), [info](https://github.com/pinojs/pino/blob/master/docs/api.md#loggerinfomergingobject-message-interpolationvalues), [warn](https://github.com/pinojs/pino/blob/master/docs/api.md#loggerwarnmergingobject-message-interpolationvalues), [error](https://github.com/pinojs/pino/blob/master/docs/api.md#loggererrormergingobject-message-interpolationvalues), [fatal](https://github.com/pinojs/pino/blob/master/docs/api.md#loggerfatalmergingobject-message-interpolationvalues). So if you are familiar with it, you are also good to go.

### Logger

```ts
// my.service.ts
import { Logger } from 'nestjs-pino';

@Injectable()
export class MyService {
  constructor(private readonly logger: Logger) {}

  getWorld(...params: any[]) {
    this.logger.log("getWorld(%o)", MyService.name, params);
    return "World!";
  }
}
```

### PinoLogger

See [pino logging method parameters](https://github.com/pinojs/pino/blob/master/docs/api.md#logging-method-parameters) for more logging examples.

```ts
// my.service.ts
import { PinoLogger, InjectPinoLogger } from 'nestjs-pino';

@Injectable()
export class MyService {

  // regular injecting
  constructor(private readonly logger: PinoLogger) {}

  // regular injecting and set context
  constructor(private readonly logger: PinoLogger) {
    logger.setContext(MyService.name);
  }

  // inject and set context via `InjectPinoLogger`
  constructor(
    @InjectPinoLogger(MyService.name) private readonly logger: PinoLogger
  ) {}


  getWorld(...params: any[]) {
    this.logger.info("getWorld(%o)", params);
    return "World!";
  }
}
```

## Usage as NestJS app logger

According to [official docs](https://docs.nestjs.com/techniques/logger#dependency-injection), loggers with Dependency injection should be set via following construction:

```ts
import { Logger } from 'nestjs-pino';

const app = await NestFactory.create(MyModule, { logger: false });
app.useLogger(app.get(Logger));
```

## FAQ

__Q__: _How does it work?_

__A__: It uses [pino-http](https://github.com/pinojs/pino-http) under hood, so every request has it's own [child-logger](https://github.com/pinojs/pino/blob/master/docs/child-loggers.md), and with help of [async_hooks](https://nodejs.org/api/async_hooks.html) `Logger` and `PinoLogger` can get it while calling own methods. So your logs can be grouped by `req.id`. If you run several instances, unique key is pair: `pid` + `req.id`.

---

__Q__: _Why use [async_hooks](https://nodejs.org/api/async_hooks.html) instead of [REQUEST scope](https://docs.nestjs.com/fundamentals/injection-scopes#per-request-injection)?_

__A__: [REQUEST scope](https://docs.nestjs.com/fundamentals/injection-scopes#per-request-injection) can have [perfomance issues](https://docs.nestjs.com/fundamentals/injection-scopes#performance). TL;DR: it will have to create an instance of the class (that injects `Logger`) on each request, and that will slow down your responce times.

---

__Q__: _I'm using old nodejs version, will it work for me?_

__A__: Please read [this](https://github.com/jeff-lewis/cls-hooked#continuation-local-storage--hooked-).

---

__Q__: _What about pino built-in methods/levels?_

__A__: Pino built-in methods are not compatible with NestJS built-in `LoggerService` methods. So for now there is option which logger to use, here is methods mapping:

| `pino` method | `PinoLogger` method | `Logger` method |
| ------------- | ------------------- | --------------- |
| trace         | trace               | **verbose**     |
| debug         | debug               | debug           |
| info          | info                | **log**         |
| warn          | warn                | warn            |
| error         | error               | error           |
| fatal         | fatal               | -               |

---

__Q__: _Fastify already includes pino, and I want to configure it on `Adapter` level, and use this config for logger_

__A__: You can do it by providing `useExisting: true`. But there is one caveat:

Fastify creates logger with your config per every request. And this logger is used by `Logger`/`PinoLogger` services inside that context underhood.

But Nest Application has another contexts of execution, for example [lifecycle events](https://docs.nestjs.com/fundamentals/lifecycle-events), where you still may want to use logger. For that `Logger`/`PinoLogger` services use separate `pino` instance with config, that provided via `forRoot`/`forRootAsync` methods.

**So, when you want to configure pino via `FastifyAdapter`, there is no way to get back this config from it and pass to that _out of context_ logger.**

And if you not pass config via `forRoot`/`forRootAsync` _out of context_ logger will be instantiated with default params. So if you want to configure it anyway with the same options, then you have to provide the same config. And then If you are already provide that then you don't have to duplicate your code and provide pino config via fastify.

So these property (`useExisting: true`) is not recommended and useful only for cases when:

- this logger is not using for lifecycle events and application level logging in Nest apps based on fastify
- pino using with default params in Nest apps based on fastify

All the other cases are lead to either code duplication or unexpected behaviour.
