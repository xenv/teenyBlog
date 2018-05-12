# teenyBlog

teenyBlog可能是全球第二小的博客系统，是的，__它只有一个页面__，156行代码。
本项目依赖 [语雀](http://yuque.com) 提供后台管理和API。页面渲染采用 vue，获取数据使用 axios。

__演示：__[http://notes.cx](http://notes.cx)


---

## 安装方法
1. 在 [语雀](http://yuque.com) 创建知识库，并设置权限为公开，这里举例知识库的url最后为 /user/repo 
2. 修改index.html中的118行 `https://yuque.com/user/repo`  修改为自己的知识库地址
3. 修改index.html中 135 行 `axios.get()`中的地址改为 `https://yuque.com/api/v2/repos/user/repo/toc`  的自行代理地址。
很遗憾，语雀API并不提供跨域支持，所以您必须对该API进行代理后才能使用。也许日后会放开这个限制。
我这里给出一个PHP的代理源码：
```php
<?php
//缓存，不登陆每个小时有200次访问API的机会，所以必须要缓存
$mmc = memcache_init();
$res = $mmc->get("yuque_cache");
if(empty($res)){
  $res = getData();
    $mmc->set('yuque_cache', $res, time() + 60); //60s
}
header("Access-Control-Allow-Origin: *"); // 开放跨域，您也可以修改为您的指定域名
header("Content-type: application/json; charset=utf-8");
echo $res;


function getData(){
    $ch = curl_init(); 
    curl_setopt($ch,CURLOPT_URL,'https://yuque.com/api/v2/repos/page/blog/toc');
    curl_setopt($ch, CURLOPT_HEADER, 0);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
    $header=array(  
    "Content-Type: application/x-www-form-urlencoded",  
    "User-Agent: myProxy"
    );
    curl_setopt($ch,CURLOPT_HTTPHEADER,$header);
    $res = curl_exec($ch);
    return $res;
}
?>
```
4. 修改博客标题、github地址等个性化内容，直接上传到一个静态空间即可。


