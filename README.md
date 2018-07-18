# teenyBlog

## 简介

---

TeenyBlog 2.0 是我自己摸索总结出来的一套个人静态博客的解决方案。

这套解决方案的目标是：

1. __写博友好__：支持 Markdown / 富文本，支持预览，支持复制传图和提供图床，支持 UML 图 / 思维导图
2. __读博友好__：访问速度快，稳定，界面效果良好，有 TOC 导航
3. __维护友好：__免维护，一键发布，极高可用率，极低的价格

* 演示： https://luan.ma
* 项目地址：[https://github.com/xenv/teenyBlog](https://github.com/xenv/teenyBlog)

## 方案简介

---


<div id="uepesa" data-type="puml" data-display="block" data-align="left" data-src="https://cdn.yuque.com/__puml/3a9061ee9ef48d6e93aaaabf849ced22.svg" data-width="503" data-height="390" data-text="%40startuml%0A%0Aautonumber%0A%0A%E5%8D%9A%E4%B8%BB%20-%3E%20%E8%AF%AD%E9%9B%80%3A%20%20%E5%86%99%E5%8D%9A%E5%AE%A2%20%0Aactivate%20%E8%AF%AD%E9%9B%80%0Adeactivate%20%E8%AF%AD%E9%9B%80%0A%0A%E5%8D%9A%E4%B8%BB%20-%3E%20%E4%BA%91%E5%87%BD%E6%95%B0%3A%20%E8%A7%A6%E5%8F%91%E5%90%8C%E6%AD%A5%0Aactivate%20%E4%BA%91%E5%87%BD%E6%95%B0%0A%E4%BA%91%E5%87%BD%E6%95%B0%20-%3E%20%E8%AF%AD%E9%9B%80%3A%20%E6%8B%89%E5%8F%96%E6%95%B0%E6%8D%AE%0Aactivate%20%E8%AF%AD%E9%9B%80%0A%E8%AF%AD%E9%9B%80%20-%3E%20%E4%BA%91%E5%87%BD%E6%95%B0%3A%20%E8%BF%94%E5%9B%9E%E6%95%B0%E6%8D%AE%0Adeactivate%20%E8%AF%AD%E9%9B%80%0A%E4%BA%91%E5%87%BD%E6%95%B0%20-%3E%20OSS%3A%20%E5%AD%98%E6%94%BE%E6%95%B0%E6%8D%AE%0Aactivate%20OSS%0Adeactivate%20OSS%0Adeactivate%20%E4%BA%91%E5%87%BD%E6%95%B0%0A%0ACDN%20-%3E%20OSS%3A%20%E7%BC%93%E5%AD%98%E6%95%B0%E6%8D%AE%0Aactivate%20OSS%0Adeactivate%20OSS%0A%0A%E7%94%A8%E6%88%B7%20-%3E%20CDN%3A%20%E5%B0%B1%E8%BF%91%E8%AE%BF%E9%97%AE%0A%0A%0A%40enduml"><img src="https://cdn.yuque.com/__puml/3a9061ee9ef48d6e93aaaabf849ced22.svg" width="503"/></div>


1. 博主在[语雀](https://yuque.com)写博客，语雀支持 Markdown / 富文本，直接预览，复制传图，图床，UML 图 / 思维导图 等功能，提供 TOC，由蚂蚁金服运营。
2. 博主触发同步，云函数从语雀API拉取数据，并且永久静态存放在OSS中（OSS支持未备案域名）
3. 用户访问时，经过CDN访问OSS，提供高速的访问体验  （CDN可选）


## 方案部署详解

---

1. 写博客：在 [https://yuque.com/](https://yuque.com/) 注册账号并新建知识库，新建文章后，在知识库首页编辑目录，可以自由控制展示规则。
2. 选择一个 __OSS__ 方案，本站使用阿里云的 OSS ，选择海外机房不用备案可以直接绑定域名，如果已备案则绑定到CDN上
3. 选择一个 __云函数__ 方案，本站使用阿里云的函数计算，支持 HTTP访问 触发，也可以由其他事件触发。
4. 选择一个 __CDN __方案，本站主站不能备案，所以主页没有使用 CDN，但是静态资源和博文数据可以存放在 可以备案的 CDN 域名上面。本站的静态资源使用 阿里云 CDN
5. 上传博客静态页面：[https://github.com/xenv/teenyBlog/blob/master/index.html](https://github.com/xenv/teenyBlog/blob/master/index.html) 到 OSS，并且设置为首页
    1. 修改 title 和 首页数据的 json 等信息
    2. 静态页面仅供演示，所以__仅有首页，内页跳转到语雀__，有能力的同学完全可以根据 [语雀API](https://yuque.com/yuque/developer)  开发内容页等页面
    3. __能显示博文数据的核心在于：从语雀拉取了博文数据的 json，并且放在OSS中，我们的静态页面就可以从 JSON获取到数据，然后使用 VUE 渲染出来。__
6. 新建云函数，选择 HTTP 访问触发，我提供一下环境为 Python 3.6 的同步代码：
     ```python
        import json
        import oss2
        import logging
        import requests

        # config
        yuque_auth_token = 'YXDQoZMhHxNw1zJ+bIRZCL5KVOOlj1OD'  # 语雀后台取得，可选
        yuque_path = "page/luan.ma"  # 语雀的用户号/仓库号

        aliyun_auth_access_key = 'LTAITMKYkxY'  # https://ram.console.aliyun.com/ 新建账号并授权 OSS 权限
        aliyun_auth_access_secret = 'TbiZpQRPuDGE68IT'

        aliyun_oss_endpoint = 'oss-cn-shenzhen-internal.aliyuncs.com'  # OSS 区域：https://help.aliyun.com/document_detail/31837.html
        aliyun_oss_bucket = 'nazhumi-static'  # OSS Bucket 名
        aliyun_oss_path = "luanma/list.json"  # OSS 博文数据路径


        # The main function
        def handler(environ, start_response):
            logger = logging.getLogger()
            logger.info('start sync work')

            endpoint = aliyun_oss_endpoint
            auth = oss2.Auth(aliyun_auth_access_key, aliyun_auth_access_secret)
            bucket = oss2.Bucket(auth, endpoint, aliyun_oss_bucket)

            url = 'https://yuque.com/api/v2/repos/' + yuque_path + '/toc'

            try:
                main_html = get_data(url)
                main_json = json.loads(main_html)

                for item in main_json["data"]:
                    if len(item["slug"]) > 1 and "://" not in item["slug"]:
                        item_html = get_data("https://yuque.com/api/v2/repos/" + yuque_path + "/docs/" + item["slug"])
                        item_date = json.loads(item_html)["data"]["updated_at"]
                        item["date"] = item_date.split("T")[0]

                bucket.put_object(aliyun_oss_path, json.dumps(main_json))
                response_body = {
                    'status': 'success'
                }
            except Exception as e:
                response_body = {
                    'status': 'fail'
                }

            status = '200 OK'
            response_headers = [('Content-type', 'text/json')]
            start_response(status, response_headers)
            return [json.dumps(response_body).encode()]


        # get url of content
        def get_data(url):
            headers = {
                'User-agent': 'aliFunction/0.01',
                'Content-Type': 'application/x-www-form-urlencoded',
                'X-Auth-Token': yuque_auth_token
            }
            html = requests.get(url, headers=headers).content

            return html

        ```
7. 开启CDN，绑定域名到CDN。不开启CDN则绑定域名到OSS。
8. 更新博文后，访问云函数提供的地址，即可同步语雀数据到 OSS。如果是语雀企业用户，则可以配置 WebHook。


