# ky [![explain]][source] [![translate-svg]][translate-list] [![size-img]][size]

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/ky-zh

「 小巧典雅的基于 Fetch API 浏览器的 HTTP 客户端 」

## 使用

```js
import ky from 'ky';

(async () => {
	const json = await ky.post('https://some-api.com', {json: {foo: true}}).json();

	console.log(json);
	//=> `{data: '🦄'}`
})();
```


---

## explain ✅

<!-- doc-templite START generated -->
<!-- time = '2018 9.11' -->
<!-- name = 'sindresorhus' -->
<!-- repo = 'ky' -->
<!-- commit = 'e40e3e74ff29374480adcf11e87027f6b9bed39e' -->
版本 | 与日期 | 最新更新 | 更多
---|---|---|---
[commit] | ⏰ 2018 9.11 | ![version] | [源码解释][source]

[commit]: https://github.com/sindresorhus/ky/tree/e40e3e74ff29374480adcf11e87027f6b9bed39e
[version]: https://img.shields.io/npm/v/ky.svg

<!-- doc-templite END generated -->

### 贡献

欢迎 👏 勘误/校对/更新贡献 😊 [具体贡献请看](https://github.com/chinanf-boy/chinese-translate-list#贡献)

## 生活

[help me live , live need money 💰](https://github.com/chinanf-boy/live-need-money)

---

### 目录

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


  - [isObject](#isobject)
  - [deepMerge](#deepmerge)
  - [请求网络的方法](#%E8%AF%B7%E6%B1%82%E7%BD%91%E7%BB%9C%E7%9A%84%E6%96%B9%E6%B3%95)
  - [响应请求的类型](#%E5%93%8D%E5%BA%94%E8%AF%B7%E6%B1%82%E7%9A%84%E7%B1%BB%E5%9E%8B)
  - [可以重试的方法](#%E5%8F%AF%E4%BB%A5%E9%87%8D%E8%AF%95%E7%9A%84%E6%96%B9%E6%B3%95)
  - [重试的状态码](#%E9%87%8D%E8%AF%95%E7%9A%84%E7%8A%B6%E6%80%81%E7%A0%81)
  - [扩展错误类型: HTTPError](#%E6%89%A9%E5%B1%95%E9%94%99%E8%AF%AF%E7%B1%BB%E5%9E%8B-httperror)
  - [扩展错误类型: TimeoutError](#%E6%89%A9%E5%B1%95%E9%94%99%E8%AF%AF%E7%B1%BB%E5%9E%8B-timeouterror)
  - [异步等待函数](#%E5%BC%82%E6%AD%A5%E7%AD%89%E5%BE%85%E5%87%BD%E6%95%B0)
  - [请求的抢答](#%E8%AF%B7%E6%B1%82%E7%9A%84%E6%8A%A2%E7%AD%94)
- [ky](#ky)
  - [ky.prototype.\_retry](#kyprototype%5C_retry)
  - [ky.prototype.\_fetch](#kyprototype%5C_fetch)
- [导出](#%E5%AF%BC%E5%87%BA)
  - [createInstance](#createinstance)
  - [扩展的错误](#%E6%89%A9%E5%B1%95%E7%9A%84%E9%94%99%E8%AF%AF)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

### isObject

确认值是否`Object`

```js
const isObject = value => value !== null && typeof value === 'object';
```

### deepMerge

深度合并对象和数组,以及嵌套

```js
const deepMerge = (...sources) => {
  let returnValue = {};

  for (const source of sources) {
    if (Array.isArray(source)) {
      if (!Array.isArray(returnValue)) {
        returnValue = [];
      }

      returnValue = [...returnValue, ...source];
    } else if (isObject(source)) {
      for (let [key, value] of Object.entries(source)) {
        if (isObject(value) && Reflect.has(returnValue, key)) {
          value = deepMerge(returnValue[key], value);
        }

        returnValue = {...returnValue, [key]: value};
      }
    }
  }

  return returnValue;
};
```

### 请求网络的方法

```js
const requestMethods = ['get', 'post', 'put', 'patch', 'head', 'delete'];
```

### 响应请求的类型

```js
const responseTypes = ['json', 'text', 'formData', 'arrayBuffer', 'blob'];
```

### 可以重试的方法

```js
const retryMethods = new Set([
  'get',
  'put',
  'head',
  'delete',
  'options',
  'trace'
]);
```

### 重试的状态码

```js
const retryStatusCodes = new Set([408, 413, 429, 500, 502, 503, 504]);
```

### 扩展错误类型: HTTPError

```js
class HTTPError extends Error {
  constructor(response) {
    super(response.statusText);
    this.name = 'HTTPError';
    this.response = response;
  }
}
```

### 扩展错误类型: TimeoutError

```js
class TimeoutError extends Error {
  constructor() {
    super('Request timed out');
    this.name = 'TimeoutError';
  }
}
```

### 异步等待函数

```js
const delay = ms => new Promise(resolve => setTimeout(resolve, ms));
```

### 请求的抢答

正常与超时请求的抢答

```js
const timeout = (promise, ms) =>
// race 的应用在于抢答, 不管是错误还是成功, 多个Promise中, 谁先回来, 就用哪个
  Promise.race([
    promise, // 正常
    (async () => { // 超时,返回错误
      await delay(ms);
      throw new TimeoutError();
    })()
  ]);
```

## ky

主 ky 类/或者是需`new`的初始化函数

```ts
class Ky {
	constructor(input, {timeout = 10000, hooks = {beforeRequest: []}, throwHttpErrors = true, json, ...otherOptions}) {
		this._input = input; // url
		this._retryCount = 0; // 请求次数

		this._options = {
			method: 'get',
      credentials: 'same-origin', // TODO：在所有浏览器中实现规范更改时，可以删除此项. 
            // Context: https://www.chromestatus.com/feature/4539473312350208
			retry: 3,
			...otherOptions
		};

		this._timeout = timeout;
		this._hooks = hooks;
		this._throwHttpErrors = throwHttpErrors;

		const headers = new window.Headers(this._options.headers || {}); // 请求头

		if (json) { // 发送 json 类型的 body
			headers.set('content-type', 'application/json');
			this._options.body = JSON.stringify(json);
		}

		this._options.headers = headers;

		this._response = this._fetch(); // 开始请求,但却不等待, 留给用户或者,重试时再await

		for (const type of responseTypes) {
			// 构建响应-类型化函数, 方便用户使用, 同时验明是否需要重新请求
			this._response[type] = this._retry(async () => {
				if (this._retryCount > 0) {
					this._response = this._fetch();
				}

				const response = await this._response; // 重试时await

				if (!response.ok) {
					throw new HTTPError(response);
				}

				return response.clone()[type](); // 使用 window.fetch 内置的类型化函数
			});
		}

		return this._response; //留给用户 await
	}
```

### ky.prototype.\_retry

> 对于class类中, 出现的除初始化函数以外, 视为新实例.`prototype`的共同属性

[可看`babeljs.cn/repl`的示例](https://www.babeljs.cn/repl#?babili=false&browsers=&build=&builtIns=false&code_lz=MYGwhgzhAEDWCeBvAUNaB6d1BZ2oSTlAAcoFRyg-cqC_ioA6myAkMAPYB2EALgE4CuwTtLAFAJb0ADmyYBKRNFRpoVJgAs-EAHQB9AcKZoAvNHUipaAL5SqmaIF94wKaKZcoBC3aAHkARgCsAppyUATNwDMBbgAKLLSCbixM8DwmKixurFHiktLQxmimWFY29s7unj7-9EEhYRFRMb7xwHI84saGQA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=true&fileSize=true&lineWrap=true&presets=es2015%2Creact%2Cstage-2&prettier=true&targets=&version=6.26.0&envVersion=)

```js
	_retry(fn) {
		if (!retryMethods.has(this._options.method.toLowerCase())) { // 没有这种类型,返回 原函数
			return fn;
		}

		const retry = async () => {
			try {
				return await fn(); // fn 主要做了4件事
				// 1. _retryCount > 0 , 需要重新运行请求函数 ？
				// 2. 等待 请求响应
				// 3. 响应回来了, 但却是错误响应, 定义的HTTPError错误抛出, 扔给 下面的catch
				// 4. 响应回来了, 正常, 运行响应内置类型化函数,并返回最终值
			} catch (error) {

				// 能引起重新请求的, 也就是值 this._retryCount 的 大于 0
				// 但规定了, 一些错误识别, 要注意的是, 除了响应返回的错误抛出, 还有的是超时 timeout 的错误抛出

				// 1. HTTPError 验
				const shouldRetryStatusCode = error instanceof HTTPError ? retryStatusCodes.has(error.response.status) : true;
				// 2. 不能是超时错误, 且小于重试次数, 且 要在 HTTPError 限制的错误内
				if (!(error instanceof TimeoutError) && shouldRetryStatusCode && this._retryCount < this._options.retry) {
					this._retryCount++;
					const BACKOFF_FACTOR = 0.3;
					const delaySeconds = BACKOFF_FACTOR * (2 ** (this._retryCount - 1)); // 逐级提高等待时间,加大成功请求的概率
					await delay(delaySeconds * 1000);
					return retry();  // 若可以重试, 自
				}

				if (this._throwHttpErrors) { // 错过了, 上面的reutrn, 默认是抛出, 最后一次的错误的
					throw error;
				}
			}
		};

		return retry;
	}
```

### ky.prototype.\_fetch

```js
	_fetch() {
		(async () => {
			for (const hook of this._hooks.beforeRequest) { // 请求之前,想做什么,生命周期
				// eslint-disable-next-line no-await-in-loop
				await hook(this._options);
			}
		})();

		return timeout(window.fetch(this._input, this._options), this._timeout);
	}
}
```

## 导出

### createInstance

每次被导入,都创建一个新的实例

```ts
const createInstance = (defaults = {}) => {
  const ky = (input, options) =>
    new Ky(input, deepMerge({}, defaults, options));

  for (const method of requestMethods) {
    // 构造-简易api{get,post,...}, 方便用户使用
    ky[method] = (input, options) =>
      new Ky(input, deepMerge({}, defaults, options, {method}));
  }
//   也可以看出,请求是一招之功 - 都是`new`, 相互之间不存在初始值的互通

  ky.extend = defaults => createInstance(defaults); // 可供用户重新扩展,和返回新实例

  return ky;
};

export default createInstance();
```

### 扩展的错误

```ts
export {HTTPError, TimeoutError};
```
