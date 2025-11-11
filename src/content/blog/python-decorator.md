---
title: python装饰器学习
description: '学习python过程中了解到python的装饰器'
publishDate: 2025-11-11 21:01:02
tags:
  - python
  - study
---

## 发现
我再给学校ACM实验室新生群写一个机器人, 几乎是从零开始写. 在这期间我加了这个机器人框架的群聊, 这个框架刚诞生没多久, 但是适配了 `napcat` 这个客户端, 启动非常便捷, 不用自己配置客户端, 所以我就开始了自己的开发之旅

我设置了两个命令, 添加订阅比赛, 移除订阅比赛, 这两个命令我都想只让群聊管理员使用, 但是经过在群里的询问, 我发现框架没有实现这个过滤器, 只实现了少部分的过滤器, 尽管我可以直接在命令处理的函数中判断消息来源是不是管理员, 但是这样还是不太方便, 每次都重复判断, 感觉代码很shit, 所以我就萌生了自己写这个过滤器的想法.

我刚学python就开始做机器人练手, 所以部分语法不是很清晰, 当时群里有个大佬给我发了他自己实现的代码, 用于检测bot是不是管理员的过滤器,我当时看到之后十分感谢这位大佬, 于是我拿去自己测试了一下

## 问题出现

``` bash
[17:40:48.978] ERROR 执行函数random_god_image时发生错误： Plugin.random_god_image() missing required positional argument: 'event'
```
我人直接啥了, 这是啥玩意, 意思大概是没找到event这个参数
但是我明明写了啊!

``` python
import random

@require_sender_admin
@command_registry.command("/随机图片"，description='')
async def random_image(self, event: BaseMessageEvent):
  random_id = random.randint(1, 5)
  await self.api.send_group_image(event.group_id,f'plugins/scpc/assets/image{random_id}.png')
```

```python
def require_sender_admin():
    """
    用于群聊命令的权限过滤装饰器：仅允许群管理员/群主使用被装饰的命令。
    """
    def decorator(func: Callable):
        @wraps(func)
        async def wrapper(self: NcatBotPlugin, event: BaseMessageEvent, *args, **kwargs):
            group_id = getattr(event, "group_id", None)
            user_id = getattr(event, "user_id", None)
            if group_id is None or user_id is None:
                return await func(self, event, *args, **kwargs)
            try:
                member_info = await self.api.get_group_member_info(
                    group_id=group_id,
                    user_id=user_id,
                )
                if member_info.role == "owner" or member_info.role == "admin":
                    return await func(self, event, *args, **kwargs)
                return await event.reply("您不是群管理员或群主，无法执行此命令。")
            except Exception as e:
                _logger.warning(f"Failed to get sender's group role: {e}")
                await event.reply("无法获取您的群成员信息，暂时无法执行该命令。")
                return
        return wrapper
    return decorator
```

为什么会这样??


> 后来我了解到, 原来是装饰器优先级的问题
> 装饰器本质上就是在运行函数前后先运行个别的函数

所以说我的代码中 command_registry.command 命令依赖注入 BaseMessageEvent 后异步函数才有的 event 参数
而先用require_sender_admin装饰在函数上则会报错, 因为没检测到 依赖注入的 event 参数

原来如此, 那就调整一下顺序吧

``` python
import random
@command_registry.command("/随机图片"，description='')
@require_sender_admin
async def random_image(self, event: BaseMessageEvent):
  random_id = random.randint(1, 5)
  await self.api.send_group_image(event.group_id,f'plugins/scpc/assets/image{random_id}.png')
```

再次运行, 又报错???
``` bash
导入模块 scpc 时出错: require_sender_admin() takes 0 positional arguments but 1 was given
```
试了半天, 最后给我自定义的装饰器加了个 `()` 就好了, 踩坑 狠狠的踩坑
 
## 总结
总之, 这次开发经理十分有趣, 问了很多大佬, 学了很多知识, 感谢善良的人们