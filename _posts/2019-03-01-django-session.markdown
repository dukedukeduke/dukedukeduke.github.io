---
layout: post
title:  "django session处理机制"
date:   2019-03-01 08:27:02 +0800
comments: true
tags:
- django
- session
---

#### 流程
django setup 设置apps以及校验apps配置信息， 导入apps相关模块等->WSGIHandler:
WSGIHandler又分为几个步骤， 首先包装request(WSGIRequest), 
`request = self.request_class(environ)`, 然后调用get_response， 
`response = self.get_response(request)`:get_response实现了中间层request处理函数调用，

```
            for middleware_method in self._request_middleware:
                response = middleware_method(request)
                if response:
                    break

            if response is None:
                ... 
```

view 处理函数，

```
                for middleware_method in self._view_middleware:
                    response = middleware_method(request, callback, callback_args, callback_kwargs)
                    if response:
                        break
```

以及url匹配和apps函数回调，

```
            if response is None:
                wrapped_callback = self.make_view_atomic(callback)
                try:
                    response = wrapped_callback(request, *callback_args, **callback_kwargs)
                except Exception as e:
                    ...
```

以及最后response函数的调用：

```
            for middleware_method in self._response_middleware:
                response = middleware_method(request, response)
                # Complain if the response middleware returned None (a common error).
                if response is None:
                    raise ValueError(...
```

对于session， 主要在request和response的处理函数， session的request处理函数如下：

```
    def process_request(self, request):
        session_key = request.COOKIES.get(settings.SESSION_COOKIE_NAME, None)
        request.session = self.SessionStore(session_key)
```

作用主要是初始化db或者file的SessionStore类实例，
而response处理函数如下：

```
    def process_response(self, request, response):
        """
        If request.session was modified, or if the configuration is to save the
        session every time, save the changes and set a session cookie or delete
        the session cookie if the session has been emptied.
        """
        try:
            accessed = request.session.accessed
            modified = request.session.modified
            empty = request.session.is_empty()
        except AttributeError:
            pass
        else:
            # First check if we need to delete this cookie.
            # The session should be deleted only if the session is entirely empty
            if settings.SESSION_COOKIE_NAME in request.COOKIES and empty:
                response.delete_cookie(settings.SESSION_COOKIE_NAME)
            else:
                if accessed:
                    patch_vary_headers(response, ('Cookie',))
                if modified or settings.SESSION_SAVE_EVERY_REQUEST:
                    if request.session.get_expire_at_browser_close():
                        max_age = None
                        expires = None
                    else:
                        max_age = request.session.get_expiry_age()
                        expires_time = time.time() + max_age
                        expires = cookie_date(expires_time)
                    # Save the session data and refresh the client cookie.
                    # Skip session save for 500 responses, refs #3881.
                    if response.status_code != 500:
                        request.session.save()
                        response.set_cookie(settings.SESSION_COOKIE_NAME,
                                request.session.session_key, max_age=max_age,
                                expires=expires, domain=settings.SESSION_COOKIE_DOMAIN,
                                path=settings.SESSION_COOKIE_PATH,
                                secure=settings.SESSION_COOKIE_SECURE or None,
                                httponly=settings.SESSION_COOKIE_HTTPONLY or None)
        return response
```

如果请求的cookie中没有sessionid， 则session返回不作任何处理(初始化SessionStore， modified默认值是False，
所以如果不触发modified=True的操作， 是不会产生sessionid的)。

如果存在session并且sessionid 有更改， 比如login（login的views app的操作在session response之前）， 
则会保存session（如果前面已经创建并保存到self._session_key中）或者创建新的session id并保存：

```
                if modified or settings.SESSION_SAVE_EVERY_REQUEST:
                    if request.session.get_expire_at_browser_close():
                        max_age = None
                        expires = None
                    else:
                        max_age = request.session.get_expiry_age()
                        expires_time = time.time() + max_age
                        expires = cookie_date(expires_time)
                    # Save the session data and refresh the client cookie.
                    # Skip session save for 500 responses, refs #3881.
                    if response.status_code != 500:
                        request.session.save()
                        response.set_cookie(settings.SESSION_COOKIE_NAME,
                                request.session.session_key, max_age=max_age,
                                expires=expires, domain=settings.SESSION_COOKIE_DOMAIN,
                                path=settings.SESSION_COOKIE_PATH,
                                secure=settings.SESSION_COOKIE_SECURE or None,
                                httponly=settings.SESSION_COOKIE_HTTPONLY or None)
```

注意：

```
class SessionBase(object):
    ...

    def __setitem__(self, key, value):
        self._session[key] = value
        self.modified = True
```

只要触发了request.session[#key]=#value操作， 则会置`self.modified = True`

save()

```
    def save(self, must_create=False):
        """
        Saves the current session data to the database. If 'must_create' is
        True, a database error will be raised if the saving operation doesn't
        create a *new* entry (as opposed to possibly updating an existing
        entry).
        """
        obj = Session(
            session_key=self._get_or_create_session_key(),
            session_data=self.encode(self._get_session(no_load=must_create)),
            expire_date=self.get_expiry_date()
        )
        using = router.db_for_write(Session, instance=obj)
        try:
            with transaction.atomic(using=using):
                obj.save(force_insert=must_create, using=using)
        except IntegrityError:
            if must_create:
                raise CreateError
            raise
```

可以查看session在db保存的表结构：

```
class Session(models.Model):
    session_key = models.CharField(_('session key'), max_length=40,
                                   primary_key=True)
    session_data = models.TextField(_('session data'))
    expire_date = models.DateTimeField(_('expire date'), db_index=True)
    objects = SessionManager()

    class Meta:
        db_table = 'django_session'
        verbose_name = _('session')
        verbose_name_plural = _('sessions')

    def __str__(self):
        return self.session_key

    def get_decoded(self):
        return SessionStore().decode(self.session_data)
```
