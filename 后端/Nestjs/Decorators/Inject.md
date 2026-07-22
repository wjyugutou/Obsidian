```ts
Class AppService {
	// provideKey 来自 Module.providers.provide === string 
	constructor(privite @Inject('provideKey') loggerService: LoggerService) {}
	
	
	constructor(privite @Inject() loggerService: LoggerService) {}
	test()  {
		this.
	}
}


```