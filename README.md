<p align="center">
  <img src="./assets/logo.svg" alt="Logo">
</p>

# wechat-article-exporter

![GitHub stars]
![GitHub forks]
![GitHub License]
![Package Version]


> [!WARNING]
> **本项目已停止维护（2026-07-30）。**
>
> 项目所依赖的微信上游核心接口已被官方关闭，且大概率不会再开放。这是整个项目赖以工作的地基，无法通过修改代码绕过 —— 开箱即用的一键同步公众号文章已不复存在。完整说明见 [关于本项目停止维护的说明 #200](https://github.com/wechat-article/wechat-article-exporter/issues/200)。
>
> - 仓库不会删除，将转为**只读归档**。代码、文档与 issue 讨论均留在原处，任何人都可以继续阅读、fork、学习和二次开发。
> - **不再接受新的 issue 和 PR**，也不会再有新版本，请不要等待「明天可能修好」。
> - 在线网站的域名将于 **2026-10-30 到期**，届时将无法访问，请在此之前导出你需要保留的数据。
> - **你已经同步过的文章都还是你的**，它们存放在你自己的设备上，不受任何接口影响。


一款在线的 **微信公众号文章批量下载** 工具，支持导出阅读量与评论数据，无需搭建任何环境，~~可通过 [在线网站] 使用~~（域名 2026-10-30 到期，届时将无法访问），同时也支持 docker 私有化部署和 Cloudflare 部署。

支持下载各种文件格式，其中 HTML 格式可100%还原文章排版与样式。


## :books: 如何使用？

该工具的使用教程已移至 [文档站点](https://docs.mptext.top)。


## :dart: 特性

- [x] 搜索公众号，支持关键字搜索
- [x] 支持导出 html/json/excel/txt/md/docx 格式(html 格式打包了图片和样式文件，能够保证100%还原文章样式)
- [x] 缓存文章列表数据，减少接口请求次数
- [x] 支持文章过滤，包括作者、标题、发布时间、原创标识、所属合集等
- [x] 支持合集下载
- [x] 支持图片分享消息
- [x] 支持视频分享消息
- [x] 支持导出评论、评论回复、阅读量、转发量等数据 (需要抓包获取 credentials 信息，[查看操作步骤](https://docs.mptext.top/advanced/wxdown-service.html))
- [x] 支持 Docker 部署
- [x] 支持 Cloudflare 部署
- [x] ~~开放 API 接口~~（公开 API 已下线，不再对外提供服务）


## 公号三刀

同一作者的另一个项目 —— **[公号三刀](https://wechat.zoro.build)**。

受同样的接口关闭影响，它也无法再批量同步公众号历史文章。目前仅支持抓取非群发等少量文章，阅读量与评论数据的抓取仍然可用。

如果你的需求在这个范围内，可以去试试。



## :heart: 感谢

- 感谢 [Deno Deploy]、[Cloudflare Workers] 提供免费托管服务
- 感谢 [WeChat_Article] 项目提供原理思路


## :star: 支持

如果你觉得本项目帮助到了你，请给作者一个免费的 Star，感谢你的支持！


## :bulb: 原理

在公众号后台写文章时支持搜索其他公众号的文章功能，以此来实现抓取指定公众号所有文章的目的。


## :memo: 许可

MIT

## :red_circle: 声明

本程序承诺，不会利用您扫码登录的公众号进行任何形式的私有爬虫，也就是说不存在把你的账号作为公共账号为别人爬取文章的行为，也不存在类似账号池的东西。

您的公众号只会服务于您自己的抓取文章的目的。

通过本程序获取的公众号文章内容，版权归文章原作者所有，请合理使用。若发现侵权行为，请联系我们处理。


## :chart_with_upwards_trend: Star 历史

[![Star History Chart]][Star History Chart Link]



<!-- Definitions -->

[GitHub stars]: https://img.shields.io/github/stars/wechat-article/wechat-article-exporter?style=social&label=Star&style=plastic

[GitHub forks]: https://img.shields.io/github/forks/wechat-article/wechat-article-exporter?style=social&label=Fork&style=plastic

[GitHub License]: https://img.shields.io/github/license/wechat-article/wechat-article-exporter?label=License

[Package Version]: https://img.shields.io/github/package-json/v/wechat-article/wechat-article-exporter


[Deno Deploy]: https://deno.com/deploy

[Cloudflare Workers]: https://workers.cloudflare.com

[Wechat_Article]: https://github.com/1061700625/WeChat_Article

[Star History Chart]: https://api.star-history.com/svg?repos=wechat-article/wechat-article-exporter&type=Timeline

[Star History Chart Link]: https://star-history.com/#wechat-article/wechat-article-exporter&Timeline

[在线网站]: https://down.mptext.top
