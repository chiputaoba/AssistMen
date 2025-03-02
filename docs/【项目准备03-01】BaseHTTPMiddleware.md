```python
class BaseHTTPMiddleware:
    def __init__(self, app: ASGIApp, dispatch: DispatchFunction | None = None) -> None:
        self.app = app
        self.dispatch_func = self.dispatch if dispatch is None else dispatch

    async def __call__(self, scope: Scope, receive: Receive, send: Send) -> None:
        if scope["type"] != "http":
            await self.app(scope, receive, send)
            return

        request = _CachedRequest(scope, receive)
        wrapped_receive = request.wrapped_receive
        response_sent = anyio.Event()

        async def call_next(request: Request) -> Response:
            app_exc: Exception | None = None

            async def receive_or_disconnect() -> Message:
                if response_sent.is_set():
                    return {"type": "http.disconnect"}

                async with anyio.create_task_group() as task_group:

                    async def wrap(func: typing.Callable[[], typing.Awaitable[T]]) -> T:
                        result = await func()
                        task_group.cancel_scope.cancel()
                        return result

                    task_group.start_soon(wrap, response_sent.wait)
                    message = await wrap(wrapped_receive)

                if response_sent.is_set():
                    return {"type": "http.disconnect"}

                return message

            async def send_no_error(message: Message) -> None:
                try:
                    await send_stream.send(message)
                except anyio.BrokenResourceError:
                    # recv_stream has been closed, i.e. response_sent has been set.
                    return

            async def coro() -> None:
                nonlocal app_exc

                with send_stream:
                    try:
                        await self.app(scope, receive_or_disconnect, send_no_error)
                    except Exception as exc:
                        app_exc = exc

            task_group.start_soon(coro)

            try:
                message = await recv_stream.receive()
                info = message.get("info", None)
                if message["type"] == "http.response.debug" and info is not None:
                    message = await recv_stream.receive()
            except anyio.EndOfStream:
                if app_exc is not None:
                    raise app_exc
                raise RuntimeError("No response returned.")

            assert message["type"] == "http.response.start"

            async def body_stream() -> typing.AsyncGenerator[bytes, None]:
                async for message in recv_stream:
                    assert message["type"] == "http.response.body"
                    body = message.get("body", b"")
                    if body:
                        yield body
                    if not message.get("more_body", False):
                        break

                if app_exc is not None:
                    raise app_exc

            response = _StreamingResponse(status_code=message["status"], content=body_stream(), info=info)
            response.raw_headers = message["headers"]
            return response

        streams: anyio.create_memory_object_stream[Message] = anyio.create_memory_object_stream()
        send_stream, recv_stream = streams
        with recv_stream, send_stream, collapse_excgroups():
            async with anyio.create_task_group() as task_group:
                response = await self.dispatch_func(request, call_next)
                await response(scope, wrapped_receive, send)
                response_sent.set()
                recv_stream.close()

    async def dispatch(self, request: Request, call_next: RequestResponseEndpoint) -> Response:
        raise NotImplementedError()  # pragma: no cover

```

# 解释这个类的功能，入参、出参，call_next在其中是什么作用
`BaseHTTPMiddleware` 是 FastAPI 中用于处理 HTTP 中间件的基类。中间件是在请求处理过程中插入的代码，用于在请求到达路由处理函数之前或响应返回给客户端之前执行一些操作，例如日志记录、身份验证、请求预处理等。

### 类功能概述
`BaseHTTPMiddleware` 类提供了一个通用的结构来实现 HTTP 中间件。它允许用户定义一个自定义的 `dispatch` 方法，该方法将在请求处理过程中被调用。中间件可以对请求进行预处理，然后调用下一个中间件或最终的路由处理函数，最后对响应进行后处理。

### 入参解释
```python
def __init__(self, app: ASGIApp, dispatch: DispatchFunction | None = None) -> None:
```
- `app: ASGIApp`：这是一个 ASGI（Asynchronous Server Gateway Interface）应用程序实例，通常是 FastAPI 应用本身。中间件会将请求传递给这个应用程序进行处理。
- `dispatch: DispatchFunction | None = None`：这是一个可选的调度函数，用于处理请求和响应。如果未提供该参数，则默认使用类中的 `dispatch` 方法。

### 出参解释
`BaseHTTPMiddleware` 类本身没有显式的返回值。但是，它的 `__call__` 方法会处理 ASGI 协议的请求和响应，最终将响应发送给客户端。

### `__call__` 方法解释
```python
async def __call__(self, scope: Scope, receive: Receive, send: Send) -> None:
```
- `scope: Scope`：这是一个包含请求信息的字典，如请求方法、URL、头信息等。
- `receive: Receive`：这是一个异步函数，用于接收客户端发送的数据。
- `send: Send`：这是一个异步函数，用于向客户端发送响应数据。

### `call_next` 的作用
`call_next` 是一个在 `__call__` 方法中定义的异步函数，它的作用是将请求传递给下一个中间件或最终的路由处理函数，并返回处理后的响应。在 `dispatch` 方法中，我们可以调用 `call_next` 来继续请求的处理流程。

以下是 `call_next` 在代码中的具体作用：
```python
async def call_next(request: Request) -> Response:
    # ...
    await self.app(scope, receive_or_disconnect, send_no_error)
    # ...
    response = _StreamingResponse(status_code=message["status"], content=body_stream(), info=info)
    return response
```
- `call_next` 函数接收一个 `Request` 对象作为参数，并返回一个 `Response` 对象。
- 在 `call_next` 函数内部，它调用 `self.app` 来处理请求，`self.app` 通常是下一个中间件或最终的路由处理函数。
- 处理完成后，`call_next` 函数将生成一个 `Response` 对象并返回。

在 `dispatch` 方法中，我们可以这样使用 `call_next`：
```python
async def dispatch(self, request: Request, call_next: RequestResponseEndpoint) -> Response:
    # 请求预处理
    # ...
    response = await call_next(request)
    # 响应后处理
    # ...
    return response
```
在这个示例中，`dispatch` 方法首先对请求进行预处理，然后调用 `call_next` 函数将请求传递给下一个中间件或路由处理函数，最后对响应进行后处理并返回。

### `dispatch` 方法解释
```python
async def dispatch(self, request: Request, call_next: RequestResponseEndpoint) -> Response:
    raise NotImplementedError()  # pragma: no cover
```
- `dispatch` 方法是一个抽象方法，需要在子类中实现。它接收两个参数：`request` 表示当前的请求对象，`call_next` 是一个用于调用下一个中间件或路由处理函数的函数。
- `dispatch` 方法的返回值是一个 `Response` 对象，表示处理后的响应。

通过继承 `BaseHTTPMiddleware` 类并实现 `dispatch` 方法，我们可以创建自定义的 HTTP 中间件。例如：
```python
from fastapi import FastAPI, Request
from fastapi.middleware.base import BaseHTTPMiddleware
from fastapi.responses import Response

class CustomMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next) -> Response:
        # 请求预处理
        print(f"Received request: {request.method} {request.url}")
        # 调用下一个中间件或路由处理函数
        response = await call_next(request)
        # 响应后处理
        print(f"Returning response with status code: {response.status_code}")
        return response

app = FastAPI()
app.add_middleware(CustomMiddleware)

@app.get("/")
def read_root():
    return {"Hello": "World"}
```
在这个示例中，`CustomMiddleware` 类继承自 `BaseHTTPMiddleware` 类，并实现了 `dispatch` 方法。在 `dispatch` 方法中，我们对请求进行了预处理和后处理，并调用 `call_next` 函数将请求传递给下一个中间件或路由处理函数。
