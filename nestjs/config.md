## Config <sup>07.11.2020</sup>
> Конфигурация через окружение (`.env`) и валидация параметров, через `joi`

### Содержание
* [Используемые пакеты](#packages)
* [Установка](#install)
* [Конфигурация](#configuration)
* [Настройка](#settings)
* [Использование](#usage)
* [Использование (main.ts)](#usage-main)

### Используемые пакеты <a name="packages"></a>
* [@nestjs/config](https://www.npmjs.com/package/@nestjs/config)
* [joi](https://www.npmjs.com/package/joi)

### Установка <a name="install"></a>
```bash
npm i @nestjs/config joi
```

### Конфигурация <a name="configuration"></a>
Разместим наши настройки в корне приложения, в файле `.env`. Пусть там будут настройки порта и названия приложения:
```ini
PORT=3000
APP_NAME='My App'
```

### Настройка <a name="settings"></a>
В главном модуле (`app.module.ts`), импортируем и регистрируем конфиг (`ConfigModule`) и используя библиотеку `joi` валидируем параметры. Весь код главного модуля может выглядеть примерно так:
```ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import * as Joi from 'joi';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [
    /**
     * Config
     */
    ConfigModule.forRoot({
      validationSchema: Joi.object({
        PORT: Joi.number().required(),
        APP_NAME: Joi.string().required(),
      }),
      expandVariables: true,
      isGlobal: true,
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

### Использование <a name="usage"></a>
Через `ConfigService` мы можем получить значение любого из параметров, например в контроллере это может выглядеть так:
```ts
import { Controller, Get } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Controller()
export class AppController {
  constructor(
    private readonly configService: ConfigService
  ) {}

  @Get()
  myApp(): string {
    const appName = this.configService.get('APP_NAME');

    return `My app name: ${appName}`;
  }
}
```

### Использование в main.ts <a name="usage-main"></a>
Мы можем захотеть пробросить, например, порт в `main.ts`, все это мы можем сделать так:
```ts
import { NestFactory } from '@nestjs/core';
import { ConfigService } from '@nestjs/config';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  const configService = app.get(ConfigService);
  await app.listen(configService.get('PORT'));
}

bootstrap();
```
