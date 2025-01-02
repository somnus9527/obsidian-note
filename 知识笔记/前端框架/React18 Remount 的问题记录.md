---
tags:
  - 知识笔记/前端框架/React
---
#### 场景
>[!Info] 
>写了一个useAxios作为请求的hook，但是却发现控制台出现了CancelError,大概的代码如下

>[!Failure] 代码
>```typescript
>import { useState, useEffect, useRef } from 'react';
>import axios from 'axios';
>const useAxios = () => {
>const [data, setData] = useState(null);
>const [error, setError] = useState(null);
>const [loading, setLoading] = useState(false);
>const abortControllerRef = useRef(null);
>const cache = useRef({});
>const cacheTimeout = useRef(null);
>const generateCacheKey = (url, options) => {
>  return `${url}_${JSON.stringify(options)}`;
>};
>const fetchData = async (url, options = {}, cacheTime = 5000) => {
>  if (abortControllerRef.current) {
>    abortControllerRef.current.abort();
>  }
>  const cacheKey = generateCacheKey(url, options);
>  if (cache.current[cacheKey] && Date.now() - cache.current[cacheKey].timestamp < cacheTime) {
> 	setData(cache.current[cacheKey].data);
> 	 return;
>}
>const controller = new AbortController();
>  abortControllerRef.current = controller;
>  setLoading(true);
>  try {
>    const response = await axios(url, { ...options, signal: controller.signal });
>    setData(response.data);
>    cache.current[cacheKey] = { data: response.data, timestamp: Date.now() };
>    clearTimeout(cacheTimeout.current);
>    cacheTimeout.current = setTimeout(() => {
> 	 delete cache.current[cacheKey];
>    }, cacheTime);
>  } catch (error) {
>    if (!controller.signal.aborted) {
>      setError(error);
>    }
>  }
>  setLoading(false);
>};
>useEffect(() => {
>  return () => {
>    if (abortControllerRef.current) {
>      abortControllerRef.current.abort();
>      abortControllerRef.current = null;
>    }
>  };
>}, []);
>return { data, error, loading, fetchData };
>};
>export default useAxios;
>```

#### 分析
>这段代码，我增加了组件卸载时的副作用清理，而路由初次加载就出现了CancelError，说明副作用清理的那段代码被调用了。

#### 原因
[React官方对于这种现象的解释](https://react.dev/blog/2022/03/08/react-18-upgrade-guide#updates-to-strict-mode)
>看下来主要就是官方想在未来添加组件多次卸载&安装的能力，并能够保持上一次卸载时的状态，所以在 StrictMode + Development的时候，会进行一项测试，具体表现出来就是React会在组件第一次装载时，自动进行一次卸载和重新装载，并且在重新装载时还原第一次的状态。所以官方的解释时，不用管，因为production不会这样做，或者不使用StrictMode也不会这样。

