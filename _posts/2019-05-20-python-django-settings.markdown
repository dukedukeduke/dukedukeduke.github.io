---
layout: post
title:  "django settings的加载"
date:   2019-05-20 10:27:02 +0800
comments: true
tags:
- python
- django
- setting
- settings
---
django settings是lazy加载， 即在将要使用的时候才去从配置文件中加载设置。
初始化运行的时候会生成一个全局的settings作为单例使用。

```
settings = LazySettings()
```
而在通过settings.属性的时候，则会触发从配置文件中读取设置的动作:

```
class LazySettings(LazyObject):
    ...
    
    def _setup(self, name=None):
        """
        Load the settings module pointed to by the environment variable. This
        is used the first time we need any settings at all, if the user has not
        previously configured the settings manually.
        """
        settings_module = os.environ.get(ENVIRONMENT_VARIABLE)
        if not settings_module:
            desc = ("setting %s" % name) if name else "settings"
            raise ImproperlyConfigured(
                "Requested %s, but settings are not configured. "
                "You must either define the environment variable %s "
                "or call settings.configure() before accessing settings."
                % (desc, ENVIRONMENT_VARIABLE))

        self._wrapped = Settings(settings_module)

    def __getattr__(self, name):
        if self._wrapped is empty:
            self._setup(name)
        return getattr(self._wrapped, name)
```

重定义__getattr__则会调用_setup， 从而找到自定义settings的配置文件路径，通过Settings这个类的实例， 获得属性的相关调用。
