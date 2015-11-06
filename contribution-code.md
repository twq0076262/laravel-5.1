# 贡献代码
## 缺陷报告
为了鼓励促进更加有效积极的合作，Laravel 强烈鼓励使用 GitHub 的 pull requests，而不是仅仅报告缺陷，“缺陷报告”也可以通过一个包含失败测试的 pull request 的方式提交。

然而，如果你以文件的方式提交缺陷报告，你的问题应该包含一个标题和对该问题的明确说明，还要包含尽可能多的相关信息以及论证该问题的代码样板，缺陷报告的目的是为了让你自己和其他人更方便的重现缺陷并对其进行修复。

记住，缺陷报告被创建是为了其他人遇到同样问题的时候能够和你一起合作解决它，不要寄期望于缺陷会自动解决抑或有人跳出来修复它，创建缺陷报告是为了帮你你自己和别人走上修复问题之路。

Laravel 源码通过 Github 进行管理，每一个 Laravel 项目都有其对应的代码库：
- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Cashier](https://github.com/laravel/cashier)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Website](https://github.com/laravel/laravel.com)
- [Laravel Art](https://github.com/laravel/art)
## 核心开发讨论
你可以在 LaraChat 的 Slack 小组的 #internals 频道讨论关于 Laravel 的 bugs、新特性、以及如何实现已有特性等。Taylor Otwell，Laravel 的维护者，通常在工作日的上午8点到下午5点（西六区或美国芝加哥时间）在线，其它时间也可能偶尔在线。
## 哪个分支？
所有的 bug 修复应该被提交到最新的稳定分支，永远不要把 bug 修复提交到 master 分支，除非它们能够修复下个发行版本中的特性。

当前版本中完全向后兼容的次要特性也可以提交到最新的稳定分支。

重要的新特性总是要被提交到 master 分支，包括下个发行版本。

如果你不确定是重要特性还是次要特性，请在 #laravel-dev IRC 频道问一下 Taylor Otwell
## 安全漏洞
如果你在 Laravel 中发现安全漏洞，请发送邮件到 taylor@laravel.com，所有的安全漏洞将会被及时解决。
## 编码风格
Laravel 遵循 PSR-2 编码标准和 PSR-4 自动载入标准。