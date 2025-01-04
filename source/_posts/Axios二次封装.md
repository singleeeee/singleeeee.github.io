---
title: Axios二次封装
tags: 开发经验
categories: 开发经历
---

# Axios二次封装

## Axios是什么？

### 定义

> **Axios** 是一个基于 *[promise](https://javascript.info/promise-basics)* 网络请求库，作用于[`node.js`](https://nodejs.org/) 和浏览器中。 它是 *[isomorphic](https://www.lullabot.com/articles/what-is-an-isomorphic-application)* 的(即同一套代码可以运行在浏览器和node.js中)。在服务端它使用原生 node.js `http` 模块, 而在客户端 (浏览端) 则使用 `XMLHttpRequests`。

### 特性

- 从浏览器创建 [XMLHttpRequests](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)
- 从 node.js 创建 [http](http://nodejs.org/api/http.html) 请求
- 支持 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) API
- 拦截请求和响应
- 转换请求和响应数据
- 取消请求
- 超时处理
- 查询参数序列化支持嵌套项处理
- 自动将请求体序列化为：
  - JSON (`application/json`)
  - Multipart / FormData (`multipart/form-data`)
  - URL encoded form (`application/x-www-form-urlencoded`)
- 将 HTML Form 转换成 JSON 进行请求
- 自动转换JSON数据
- 获取浏览器和 node.js 的请求进度，并提供额外的信息（速度、剩余时间）
- 为 node.js 设置带宽限制
- 兼容符合规范的 FormData 和 Blob（包括 node.js）
- 客户端支持防御[XSRF](http://en.wikipedia.org/wiki/Cross-site_request_forgery)

### 使用

详细用法见 [Axios中文网](https://www.axios-http.cn/docs/intro)

``` javascript
// 方法一 axios(url, [config])
axios.post('https://httpbin.org/post', {
    firstName: 'Fred',
    lastName: 'Flintstone',
  }, {
    headers: {
      'Content-Type': 'multipart/form-data'
    }
  }
)

// 方法二 axios(config)
axios({
  method: 'post',
  url: 'https://httpbin.org/post',
  data: {
    firstName: 'Fred',
    lastName: 'Flintstone'
  },
   responseType: 'stream'
});

.......
```



# 为什么要进行二次封装？

## 常见的一些问题

- **写法不够简洁，代码量冗余**

可以看到每次调用axios方法，我们都需要写一遍接口的地址，如：https://httpbin.org/post/api/user/.... 等等，以及config的一些配置项，比如声明data然后去写下data内的数据，包括但不限于data，还有params等等。

- **代码零碎，不方便维护**

此外如果我们后端修改了服务部署的服务器，这时我们的域名需要修改，我们是不是得找到每一个接口然后手动修改一遍？又或者后端修改了路由规则比如将 `/getData` 换成了 `/user/getData`。那我们是不是又得去找到所有接口，手动修改一遍。

- **繁琐的错误处理**

我们在使用鉴权接口时，每一个接口都需要携带 `token` 用于验证， 此外每次返回结果的处理中，我们都需要根据返回的状态码进行相应的操作，比如 401(身份认证失败)、403(权限不足) 等等，并进行相应的处理，天呐，难道我每次调用接口都要手写一遍这些重复的处理吗。

## 封装的优点

### 统一配置管理

- 可以统一配置请求的基础 URL、请求头、超时时间、认证信息等，避免在每个请求中重复设置。
- **示例：**配置请求头中的 Authorization Token，或者在全局设置超时限制。

``` javascript
const instance = axios.create({
  baseURL: 'https://example.com/api',
  timeout: 5000, // 超时配置的默认值是 `0`, 表示永不超时
  headers: {'X-Custom-Header': 'foobar'}, // 统一配置自定义请求头
  withCredentials: true,
});

// 或者
axios.defaults.baseURL = 'https://example.com/api';
....
// 这样我们在调用axios时就可以更便捷，如
axios.get('/users')
// 实际发送时会将baseURL拼接起来 https://example.com/api/users 
```

### 错误处理

- 二次封装可以**集中处理请求错误**，例如：网络错误、权限错误（如 401 未授权）、服务器错误（如 500 内部服务器错误）等，避免每次都手动处理相同的错误逻辑。

  可以设置全局的错误拦截器来显示友好的错误提示。

```javascript
axiosInstance.interceptors.response.use(
  response => response,
  error => {
    /* 
    	....
    	统一的错误处理,如错误上报等等
    */
    console.error('API 请求失败:', error);
    return Promise.reject(error);
  }
);
```

### 请求和响应的拦截

- 在请求发起前、响应返回后执行一些额外的操作，比如`请求头加密`、`数据转换`等。
- 例如，在请求中`自动添加认证 Token `或者在响应中自动解析数据。

```javascript
axiosInstance.interceptors.request.use(config => {
  config.headers['Authorization'] = `Bearer ${localStorage.getItem('token')}`;
  return config;
});

axiosInstance.interceptors.response.use(
  response => {
	const code = response.data?.code
    switch (code) {
      case 401:
            handler();
            break;
      case 403:
      		handler();
            break;
        break
      ....
    }
  	return response;
  },
);

```

### 支持请求取消

- 在复杂的请求场景下，例如用户频繁操作`触发多个请求`时，可以通过二次封装实现请求取消，避免不必要的网络请求。

```javascript
const controller = new AbortController();

axios.get('/foo/bar', {
   signal: controller.signal
}).then(function(response) {
   //...
});
// 取消请求
controller.abort()
```

### **更好的调试和日志管理**

- 可以在请求和响应时添加日志功能，便于调试和分析。
- 例如，记录每次请求的 URL、参数、响应数据等，便于排查问题。

```javascript
axiosInstance.interceptors.response.use( response => {
  const sendedContent = { params: {}, data: {} }
  try {
    sendedContent.params = response.config.params
    sendedContent.data = JSON.parse(response.config.data)
  } catch {
    sendedContent.data = response.config.data
  }

  console.log(response.config.url, sendedContent, response.data)
  return response. data;
});
```

### 减少代码重复和简化请求使用

我们可以将频繁使用的接口封装起来，只对外暴露一些变化的参数，如请求体的内容，还方便接口类型的统一处理，没必要每个接口都单独写一遍类型定义，如果要修改接口地址、请求方法等等也更加方便。

``` typescript
// 封装
/** 为试卷移除若干道题 */
export function removeQuesionApi(data: addQuestion) {
  return request.post<addQuestion>({ url: '/api/v1/pqlink/remove', data })
}

// 使用，请求参数和响应结构都会有相应的类型提示
  const data = {
    paperId: paperDetail.value.id,
    questionIds: [questionId],
  }
  const res = await removeQuesionApi(data)
```

## 二次封装axios

### 文件结构

- **axios**	
  - `config.ts ` 提供一些基本的默认配置等
  - `service.ts` 主要的封装代码
  - `index.ts` 对外暴露封装好的方法

### 基本配置

`config.ts`

``` typescript
import { AxiosResponse, InternalAxiosRequestConfig } from "axios"

function defaultRequestInterceptors(config: InternalAxiosRequestConfig) {
    // 处理请求体的类型转换
    // application/x-www-form-urlencoded
    ...
    // multipart/form-data
    ...
    // params拼接到url上
    ...
  	return config
}

function defaultResponseInterceptors(response: AxiosResponse) {
    // 错误上报或者日志打印
    const sendedContent: { params: any, data: any } = { params: {}, data: {} }
    try {
    sendedContent.params = response.config.params
    sendedContent.data = JSON.parse(response.config.data)
    } catch {
    sendedContent.data = response.config.data
    }

    console.log(response.config.url, sendedContent, response.data)
    return response.data
}
```

### 业务封装

`service.ts`

``` typescript
const BASE_PATH = 'https://example.com/api'
const REQUEST_TIMEOUT = 10000

// 存储需要发送的请求，以及对应的中止控制器
const abortControllerMap: Map<string, AbortController> = new Map()
// 创建axios实例和配置基本属性
const axiosInstance: AxiosInstance = axios.create({
  timeout: REQUEST_TIMEOUT,
  baseURL: BASE_PATH,
  headers: {
    'Content-Type': CONTENT_TYPE,
  },
})

axiosInstance.interceptors.request.use((req: InternalAxiosRequestConfig) => {
  const url = req.url || ''
  // 记录需要发送的请求
  const controller = new AbortController()
  req.signal = controller.signal
  abortControllerMap.set(
    url,
    controller,
  )
    
  // 自动携带token
  const token = localstorage.getItem('token')
  if(token) {
	config.headers['Authrization'] = token
  }
  return req
})

axiosInstance.interceptors.response.use(
  (res: AxiosResponse) => {
    const url = res.config.url || ''
    const code = res.data?.code
    switch (code) {
		// 进行对应的错误处理，路由重定向等等
        ...
    }

    // 删掉相应的待发送请求
    abortControllerMap.delete(url)
    return res
  },
  (error: AxiosError) => {
    if (error.response?.status === 500)
      error.message = '服务器内部错误'

    return Promise.reject(error)
  },
)

// 挂载默认的拦截器（基本不需要修改）
axiosInstance.interceptors.request.use(defaultRequestInterceptors)
axiosInstance.interceptors.response.use(defaultResponseInterceptors)

// 封装提供的方法
const service = {
  request: (config: RequestConfig) => {
    return new Promise((resolve) => {
      if (config.interceptors?.requestInterceptors)
        config = config.interceptors.requestInterceptors(config as any)

      axiosInstance
        .request(config)
        .then((res) => {
          resolve(res)
        })
        .catch((err: any) => {
          // 这里返回resolve，确保后续代码可以不适用try catch 捕获async/await的错误，错误根据后端返回的状态码或者单独				设置的success字段等等来确定
          resolve(err)
        })
    })
  },
  cancelRequest: (url: string | string[]) => {
    const urlList = Array.isArray(url) ? url : [url]
    for (const _url of urlList) {
      abortControllerMap.get(_url)?.abort()
      abortControllerMap.delete(_url)
    }
  },
  cancelAllRequest() {
    for (const [_, controller] of abortControllerMap)
      controller.abort()

    abortControllerMap.clear()
  },
}

export default service
```

### 封装请求方法

`index.ts`

``` typescript
import service from './service'
interface IResponse<T = any> {
	code: number,
    data: T extends any ? T : T & any,
    message: string,
    success: boolean
}

// 提供默认的请求方法
export default {
  get: <T = any>(option: AxiosConfig): Promise<IResponse<T>> => {
    return service.request({ method: 'get', ...option })
  },
  post: <T = any>(option: AxiosConfig): Promise<IResponse<T>> => {
    return service.request({ method: 'post', ...option }) as 
  },
  delete: <T = any>(option: AxiosConfig) => {
    return service.request({ method: 'delete', ...option })
  },
  put: <T = any>(option: AxiosConfig): Promise<IResponse<T>> => {
    return service.request({ method: 'put', ...option })
  },
  cancelRequest: (url: string | string[]) => {
    return service.cancelRequest(url)
  },
  cancelAllRequest: () => {
    return service.cancelAllRequest()
  },
}

```

### API统一管理

- api
  - users
    - types.ts
    - index.ts

``` typescript
import type { LWEmail, LWPassword, PasswordResponse, EmailResponse } from './types'
/** 密码登录 */
export function loginWithPasswordApi(data: LWPassword) {
  return request.post<PasswordResponse>({ url: '/api/v1/auth/login', data })
}

/** 邮箱登录 */
export function loginWithEmailApi(data: LWEmail) {
  return request.post<EmailResponse>({ url: '/api/v1/auth/login', data })
}

......
```

## 局限性

### 性能开销

- **请求拦截器和响应拦截器的开销**：每个请求都会经过拦截器进行处理，增加了额外的计算和处理时间，尤其是在请求量较大时，可能对性能产生一定影响。

- **过多的中间层**：如果封装过于复杂，增加了多个中间层（例如，多个拦截器、请求方法的层级等），可能会导致请求的执行速度变慢。

### 复杂场景

- **复杂的动态请求**：如果请求的结构非常复杂，需要动态改变 headers、参数或请求方式，二次封装可能会变得不够灵活。例如，某些接口需要特殊的配置（如不带 token 的请求），如果过度封装可能难以应对这种复杂场景。
- **特定错误处理的局限**：统一的错误处理机制虽然方便，但可能无法应对所有的错误类型。某些 API 的错误逻辑可能与其他 API 不同，需要特定处理，而封装的统一处理可能无法满足这种需求。

### **封装的过度复杂**

- **封装过度**：如果封装的层次过多，可能会引入额外的复杂度。在简单的 API 请求中，可能并不需要复杂的封装逻辑，这样会导致代码冗余，不必要的抽象反而增加了维护难度。

### 维护困难

### 5. **调试困难**

- **封装层增加调试难度**：当出现请求错误时，封装层的存在可能会让问题的追踪变得更加困难。尤其是在请求或响应拦截器中加了很多自定义逻辑时，可能会掩盖真正的错误原因，使得调试变得复杂。
- **错误信息不够清晰**：统一处理的错误可能会缺乏具体的信息，开发者可能无法快速定位是哪个环节出现了问题。

# 总结

**1. 为什么要封装：**

- **减少重复代码**：每次请求都需要手动写 URL 和配置项，增加代码量和维护难度。
- **集中管理配置**：若后端修改了接口地址或路径，必须逐个修改请求，增加维护负担。
- **简化错误处理**：每个请求都需重复处理如 401、403 错误，封装后可以统一处理。

**2. 二次封装的优势：**

- **统一配置**：集中管理请求的基础 URL、请求头、超时时间等，避免重复配置。
- **统一错误处理**：通过拦截器集中处理错误，提高代码的可维护性。
- **请求拦截和响应拦截**：可以在请求前自动加 Token，响应后统一处理返回数据。
- **请求取消**：支持取消不必要的请求，提升性能。
- **简化代码**：减少每个接口的重复定义，易于维护和扩展。

**3. 封装步骤：**

- **配置文件**：统一设置请求配置。
- **请求和响应拦截器**：处理请求和响应的数据格式及错误。
- **封装请求方法**：简化接口调用，减少重复代码。

封装后，可以轻松地在项目中调用请求方法，并自动处理请求头、错误、Token 等问题，同时也方便修改和维护接口路径和配置。

通过二次封装，Axios 的使用变得更加高效、简洁和易维护，为项目的扩展和维护提供了便利。
