## Winston <sup>25.10.2020</sup>
> Установка и настройка логгера Winston для NestJS

### Содержание
* [Используемые пакеты](#packages)
* [Установка](#install)
* [Настройка](#settings)
* [Использование](#usage)
* [Winston как логгер по умолчанию](#default)
* [Обработка исключений](#exceptions)

### Используемые пакеты <a name="packages"></a>
* [winston](https://www.npmjs.com/package/winston)
* [nest-winston](https://www.npmjs.com/package/nest-winston)
* [winston-daily-rotate-file](https://www.npmjs.com/package/winston-daily-rotate-file)

### Установка <a name="install"></a>
```bash
npm i winston nest-winston winston-daily-rotate-file
```

### Настройка <a name="settings"></a>
В главном модуле (`app.module.ts`), импортируем и регистрируем логгер (`WinstonModule`). После чего весь код модуля может выглядеть примерно так:
```ts
import { Module } from '@nestjs/common';
import { join } from 'path';
import * as winston from 'winston';
import { WinstonModule } from 'nest-winston';
import WinstonDailyRotateFile = require('winston-daily-rotate-file');
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [
    /**
     * Winston
     */
    WinstonModule.forRoot({
      format: winston.format.combine(
        winston.format.simple(),
        winston.format.timestamp(),
        winston.format.printf(info => {
          const date = info.timestamp.replace(/^.+T(.+)Z$/, '$1');
          const message = (info.trace || info.message || info).toString().replace(/ {4}/g, '  ');
          return `[${date}]\n[${info.level}]: ${message}\n`;
        })
      ),
      transports: [
        new WinstonDailyRotateFile({
          filename: join(__dirname, '..', 'logs', '%DATE%.log'),
          datePattern: 'YYYY-MM-DD',
          maxSize: '10m',
          level: 'debug',
        })
      ]
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

### Использование <a name="usage"></a>
Теперь экземпляр логгера, как сервис, можно внедрить в любом месте проекта. Например, контроллер мог бы выглядеть так:
```ts
import { Controller, Get, Inject } from '@nestjs/common';
import { WINSTON_MODULE_PROVIDER } from 'nest-winston';
import { Logger } from 'winston';

@Controller()
export class AppController {
  constructor(
    @Inject(WINSTON_MODULE_PROVIDER)
    private readonly logger: Logger
  ) {}

  @Get()
  getHello(): string {
    this.logger.debug('Hello World!');
    return 'Hello World!';
  }
}
```

### Winston как логгер по умолчанию <a name="default"></a>
Можно установить winston в качестве логгера по умолчанию для всег опроекта. Для этого, в главном файле приложения (`main.ts`), необходимо установить обрабочик для логера, после чего файл `main.ts` может выглядеть например так:
```ts
import { NestFactory } from '@nestjs/core';
import { WINSTON_MODULE_NEST_PROVIDER } from 'nest-winston';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useLogger(app.get(WINSTON_MODULE_NEST_PROVIDER));
  await app.listen(3000);
}

bootstrap();
```
### Обработка исключений <a name="exceptions"></a>
Также можно залогировать все, не обработанные, исключения. Для этого необходимо создать фильтр. Для примера, в папке `src` создадим файл `log-exception.filter.ts` со следующим содержанием:
```ts
import { Catch, HttpException, Inject } from '@nestjs/common';
import { BaseExceptionFilter } from '@nestjs/core';
import { WINSTON_MODULE_PROVIDER } from 'nest-winston';
import { Logger } from 'winston';

@Catch()
export class LogExceptionsFilter extends BaseExceptionFilter {
  constructor(
    @Inject(WINSTON_MODULE_PROVIDER)
    private readonly logger: Logger
  ) {
    super();
  }

  catch(exception: HttpException) {
    const message: any = exception.stack || exception;
    this.logger.error(message);
  }
}
```

В главном модуле (`app.module.ts`), зарегистрируем фильтр в качестве провайдера. Теперь главный модуль может выглядеть приблизительно так:
```ts
import { Module } from '@nestjs/common';
import { APP_FILTER } from '@nestjs/core';
import { join } from 'path';
import * as winston from 'winston';
import { WinstonModule } from 'nest-winston';
import WinstonDailyRotateFile = require('winston-daily-rotate-file');
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { LogExceptionsFilter } from './log-exception.filter';

@Module({
  imports: [
    /**
     * Winston
     */
    WinstonModule.forRoot({
      format: winston.format.combine(
        winston.format.simple(),
        winston.format.timestamp(),
        winston.format.printf(info => {
          const date = info.timestamp.replace(/^.+T(.+)Z$/, '$1');
          const message = (info.trace || info.message || info).toString().replace(/ {4}/g, '  ');
          return `[${date}]\n[${info.level}]: ${message}\n`;
        })
      ),
      transports: [
        new WinstonDailyRotateFile({
          filename: join(__dirname, '..', 'logs', '%DATE%.log'),
          datePattern: 'YYYY-MM-DD',
          maxSize: '10m',
          level: 'debug',
        })
      ]
    }),
  ],
  controllers: [AppController],
  providers: [
    {
      provide: APP_FILTER,
      useClass: LogExceptionsFilter,
    },
    AppService,
  ],
})
export class AppModule {}
```
