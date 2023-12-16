# <center>Python-在fastapi的中间件中获取request body会阻塞应用程序

> 背景：fastapi 应用程序的自定义中间件log_request_middleware通过切面自动打印请求发起、请求响应的日志，由于日志中需要填充请求体（即request body）信息，导致只要是POST方式且带有 request body 入参的接口都会被阻塞，无法正常响应（接口一直处于 pending）

## 1. 问题分析

- 问题现象：只要是POST方式且带有 request body 入参的接口都会被阻塞，无法正常响应（接口一直处于 pending）
- 问题原因：在中间件中获取 request body 会阻塞应用程序
- 根本原因：request.body()只能被获取一次，log_request_middleware中获取了一次后，后续的请求无法获取到request.body(), 进而阻塞程序，接口一直处于 pending
- 解决方案：在中间件中获取 request body 后，通过闭包将 request.body() 保存在内存中，后续的请求直接从内存中获取 request body

## 2. 具体实现
```python
import json
import time
import traceback

from fastapi import Request, Response
from starlette.middleware.base import RequestResponseEndpoint, BaseHTTPMiddleware
from starlette.concurrency import iterate_in_threadpool
from starlette.types import Message

from loguru import logger

from app.api.responses.api_response import ResponseEnum, ApiResponse


class LogRequestMiddleware(BaseHTTPMiddleware):
    # the problem that request.body() can only be read once
    # which above problem will block the application if the request with body
    # get the request body in advance and use closure function to store it in memory
    # for more reference: https://github.com/tiangolo/fastapi/issues/394
    @staticmethod
    async def set_body(request: Request):
        receive_ = await request._receive()    # 1st: request._receive() will store the request body in request.stream()

        async def receive() -> Message:    # 2nd: closure function will read the request body and keep it in memory
            return receive_

        request._receive = receive

    # 3rd: subsequent request.body() will read the request body from memory anyhow

    async def dispatch(
        self, request: Request, call_next: RequestResponseEndpoint
    ) -> Response:
        """
        A custom middleware to log HTTP request and response information, which derived from BaseHTTPMiddleware
        :param request: starlette.requests
        :param call_next: RequestResponseEndpoint
        :return: starlette.response
        """
        start_time = time.time()
        await self.set_body(request)
        try:
            # Log the request coming in
            logger.info({
                "operation": "Log Request",
                "method": request.method,
                "path": request.url.path,
                "params": await request.json() or request.query_params.__str__(),
                "headers": request.headers.items()
            })
            response: Response = await call_next(request)

            # get the actual response content of the request.
            response_body = [chunk async for chunk in response.body_iterator]
            response.body_iterator = iterate_in_threadpool(iter(response_body))
            response_body = (b''.join(response_body)).decode()

            # Log the response going out
            if "success:" in response_body and json.loads(response_body)["success"] is True:
                logger.info({
                    "operation": "Request Succeed",
                    "duration": f"{(time.time() - start_time) * 1000:.3f}ms",
                    "method": request.method,
                    "path": request.url.path,
                    "params": await request.json() or request.query_params.__str__(),
                    "response": response_body
                })
            else:
                logger.info({
                    "operation": "Request Failed",
                    "duration": f"{(time.time() - start_time) * 1000:.3f}ms",
                    "method": request.method,
                    "path": request.url.path,
                    "params": await request.json() or request.query_params.__str__()
                })
            return response
        except Exception as e:
            # Log the exception and return a globally consistent "500 Server Error" response.
            logger.error(f'"operation": "Log Exception"  '
                         f'"method": {request.method}  '
                         f'"path": {request.url.path}  '
                         f'"error": {str(e)}  '
                         f'"traceback": {traceback.format_exc()}')
            return ApiResponse.error(ResponseEnum.SERVER_ERROR)

```

- 这里核心是通过`set_body`方法，将`request.body()`提前获取并保存在内存中，后续的请求直接从内存中获取`request.body()`