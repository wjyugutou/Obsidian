```ts

@Module({
  imports: [
    LoggerModule,
    CoreModule,
  ],
  providers: [
    {
      // provider 的名称
      provide: 'SUFFIX',
      useValue: 'suffix',
    },
    {
      // provider 的名称
      provide: LoggerService,
      useClass: LoggerService,
    },
    {
      // provider 的名称
      provide: 'StringToken',
      useValue: new UseValueService('value-prefix'),
    },
    {
      // provider 的名称
      provide: 'FectoryToken',
      inject: ['factory-prefix1', 'SUFFIX'],
      useFactory: (prefix1: string, prefix2: string) => new UseFectoryService(prefix1, prefix2),
    },
  ],
  controllers: [AppController],

})
Class AppModule {}

```

## DynamicModule

```ts


```