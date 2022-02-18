# Hls.js使用在Vue3中

## 动态获取m3u8视频链接

- ```js
  axios.post("请求路径",{参数})
      .then((res)=>{
           const resData = res.data
           if(res.data.rtn === "0" &&                  res.data.url){
               data.value.url = resData.url
           }
             console.log(
                '当前第',
                props.id,
                '号视频链接',
                data.value.url,
                '获取到了'
             )
       })
  ```

- 