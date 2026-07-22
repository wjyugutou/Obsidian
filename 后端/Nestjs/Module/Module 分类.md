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
整个 APP 应用 只有一个实例

## DynamicModule

动态模块的目的是生成实例时 可以给 `forRoot/forFeature` 传参

每一次独立的 `imports: [XxxModule.forXxx()]` 调用，默认都会创建一个全新的模块实例和一套全新的 Provider 实例。

```ts
@Module({
  providers: [
    {
      provide: 'PREFIX',
      useValue: 'config',
    },
  ],
  exports: [],
})
export class DynamicConfigModule {
  static forRoot(entities = [], options?: Obj): DynamicModule {
    const providers: Provider[] = [
      {
        provide: 'CONFIG',
        useValue: { apiKey: '123' },
      },
    ]

    return {
      module: DynamicConfigModule,
      providers,
      controllers: [],
      exports: providers.map(provider => typeof provider === 'function' ? provider : provider.provide),
    }
  }
}

```

| 维度        | `forRoot()`                | `forFeature()`    |
| :-------- | :------------------------- | :---------------- |
| 语义        | 根配置 / 全局初始化                | 功能级配置 / 局部定制      |
| 调用位置      | 仅在 `AppModule` (或根模块) 调用一次 | 在任意业务/功能模块中多次调用   |
| 典型用途      | DB连接、全局Config、Logger       | 多租户数据源、模块级策略、实体注册 |
| 是否 Global | 通常配合 `global: true`        | 通常不设置 global      |
| 实例数量      | 1 个（单例）                    | N 个（每次调用可能产生新实例）  |
| 参数特点      | 接收全局基础设施配置                 | 接收当前模块所需的特定配置     |
