---
layout: post
title:  "django 与Wsgi接口"
date:   2019-02-28 19:20:02 +0800
comments: true
tags:
- django
- wsgi
---

django本身是一个application框架， 自身也带有测试的wsgiref， 正式部署， 可以用uwsgi来代替wsgiref。
关于wsgi server和django的接口：
在配置uwsgi的配置文件中， 往往指定wsgi文件的路径， 比如：

```module = django.core.handlers.wsgi:WSGIHandler()```

或者：

```
wsgi-file = /var/project/production_tracking_nexa/django_bootstrap/django_bootstrap/wsgi.py
```

第二个， wsgi.py文件， 如下：

```
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "django_bootstrap.settings")

application = get_wsgi_application()
```

而：

```
def get_wsgi_application():
    """
    The public interface to Django's WSGI support. Should return a WSGI
    callable.

    Allows us to avoid making django.core.handlers.WSGIHandler public API, in
    case the internal WSGI implementation changes or moves in the future.

    """
    django.setup()
    return WSGIHandler()
```

所以两者等价的。
而关于WSGIHandler，观察一下__call__(self, environ, start_response)， 可以发现这与WSGI协议定义的接口一模一样。

```
class WSGIHandler(base.BaseHandler):
    initLock = Lock()
    request_class = WSGIRequest

     def __call__(self, environ, start_response):
        # Set up middleware if needed. We couldn't do this earlier, because
        # settings weren't available.
        if self._request_middleware is None:
            with self.initLock:
                try:
                    # Check that middleware is still uninitialized.
                    if self._request_middleware is None:
                        self.load_middleware()
                except:
                    # Unload whatever middleware we got
                    self._request_middleware = None
                    raise

        set_script_prefix(get_script_name(environ))
        signals.request_started.send(sender=self.__class__, environ=environ)
        try:
            request = self.request_class(environ)
        except UnicodeDecodeError:
            logger.warning('Bad Request (UnicodeDecodeError)',
                exc_info=sys.exc_info(),
                extra={
                    'status_code': 400,
                }
            )
            response = http.HttpResponseBadRequest()
        else:
            response = self.get_response(request)

        response._handler_class = self.__class__

        status = '%s %s' % (response.status_code, response.reason_phrase)
        response_headers = [(str(k), str(v)) for k, v in response.items()]
        for c in response.cookies.values():
            response_headers.append((str('Set-Cookie'), str(c.output(header=''))))
        start_response(force_str(status), response_headers)
        if getattr(response, 'file_to_stream', None) is not None and environ.get('wsgi.file_wrapper'):
            response = environ['wsgi.file_wrapper'](response.file_to_stream)
        return response
```

其实就是wsgi的的handler，request经过nginx和uwsgi的处理和分派之后， 就会到django应用， 找到相应的application来处理请求(当然会先经过中间件的request以及后续经过中间件的response)， 而WSGIHandler就是application的接口。
WSGIRequest用来封装request参数：

```
class WSGIRequest(http.HttpRequest):
    def __init__(self, environ):
        script_name = get_script_name(environ)
        path_info = get_path_info(environ)
        if not path_info:
            # Sometimes PATH_INFO exists, but is empty (e.g. accessing
            # the SCRIPT_NAME URL without a trailing slash). We really need to
            # operate as if they'd requested '/'. Not amazingly nice to force
            # the path like this, but should be harmless.
            path_info = '/'
        self.environ = environ
        self.path_info = path_info
        # be careful to only replace the first slash in the path because of
        # http://test/something and http://test//something being different as
        # stated in http://www.ietf.org/rfc/rfc2396.txt
        self.path = '%s/%s' % (script_name.rstrip('/'),
                               path_info.replace('/', '', 1))
        self.META = environ
        self.META['PATH_INFO'] = path_info
        self.META['SCRIPT_NAME'] = script_name
        self.method = environ['REQUEST_METHOD'].upper()
        _, content_params = cgi.parse_header(environ.get('CONTENT_TYPE', ''))
        if 'charset' in content_params:
            try:
                codecs.lookup(content_params['charset'])
            except LookupError:
                pass
            else:
                self.encoding = content_params['charset']
        self._post_parse_error = False
        try:
            content_length = int(environ.get('CONTENT_LENGTH'))
        except (ValueError, TypeError):
            content_length = 0
        self._stream = LimitedStream(self.environ['wsgi.input'], content_length)
        self._read_started = False
        self.resolver_match = None
```

在WSGIHandler中， 正常的请求， 是调用：

`response = self.get_response(request)`

来进行request处理的，
而get_response， 则根据请求的参数， 正则匹配urls.py中定义的处理函数，然后用回调，调用该函数。

```callback, callback_args, callback_kwargs = resolver_match```

```
wrapped_callback = self.make_view_atomic(callback)
response = wrapped_callback(request, *callback_args, **callback_kwargs)
```

当然， 这过程当中包含了对中间层函数的运行， 以及模板的处理， 具体参考源码。
