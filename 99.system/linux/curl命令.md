curl发送post请求：

``` shell
curl -i -X POST -H "'Content-type':'application/json'" -d '{"ATime":"'$atime'","BTime":"'$btime'"}' $url
```

curl发送get请求：

```shell
curl http://localhost:2375/info
```



