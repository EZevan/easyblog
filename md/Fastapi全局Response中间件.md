# <center>FastAPI使用中间件实现全局统一Response

> 说明：

> 1. 统一的后端response方便前端调用，不需要额外关心每个接口的响应形式，以及异常、错误处理等
> 2. 使用统一的response，提高前端使用的易读性，可理解性
> 3. reponse中对异常做统一处理，对异常的排查定位更加方便和准确

## FastAPI实现全局统一的response

```py
from fastapi.responses import JSONResponse

import ResponseEnum


class APIReponse
@staticmethod
def success(data: any) -> JSONResponse:
    content = {
        "success": True,
        "errors": None,
        "data": data,
    }

    return JSONResponse(content=content, status_code=200)


@staticmethod
def error(response_enum: ResponseEnum, **extra_args) -> JSONResponse:
    if extra_args:
        content = {
            "success": False,
            "errors": {
                "code": response_enum.get_code(),
                **extra_args
            },
            "data": None
        }
    else:
        content = {
            "success": False,
            "errors":{
                "code": response_enum.get_code(),
                "message": response_enum.get_message()
            },
            "data": None
        }

    return JSONResponse(content=content, status_code=200)
```

## FastAPI在启动应用程序时注册中间件，实现记录请求、响应日志，同时返回全局统一的response

```py
from fastapi import Request, Response
from starlette.middleware.base import RequestResponseEndpoint, BaseHTTPMiddleware
from starlette.concurrency import iterate_in_threadpool

from loguru import logger

from domore.app.api.responses.api_response import ResponseEnum, ApiResponse


class LogRequestMiddleware(BaseHTTPMiddleware):
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
        try:
            # Log the request coming in
            logger.info({
                "operation": "Log Request",
                "method": request.method,
                "path": request.url.path,
                "params": await request.body() or request.query_params.__str__(),
                "headers": request.headers.items()
            })
            response: Response = await call_next(request)

            # get the actual response content of the request.
            response_body = [chunk async for chunk in response.body_iterator]
            response.body_iterator = iterate_in_threadpool(iter(response_body))
            response_body = (b''.join(response_body)).decode()

            # Log the response going out
            if "success" in response_body and json.loads(response_body)["success"] is False:
                logger.info({
                    "operation": "Request Failed",
                    "duration": f"{(time.time() - start_time) * 1000:.3f}ms",
                    "method": request.method,
                    "path": request.url.path,
                    "params": await request.body() or request.query_params.__str__()
                })
            else:
                logger.info({
                    "operation": "Request Succeed",
                    "duration": f"{(time.time() - start_time) * 1000:.3f}ms",
                    "method": request.method,
                    "path": request.url.path,
                    "params": await request.body() or request.query_params.__str__(),
                    "response": response_body
                })
            
            return response
        except Exception as e:
            # Log the exception and return a globally consistent "500 Server Error" response.
            logger.error(f'"operation": "Log Exception"  '
                         f'"method": request.method  '
                         f'"path": {request.url.path}  '
                         f'"error": {str(e)}  '
                         f'"traceback": {traceback.format_exc()}')
            return ApiResponse.error(ResponseEnum.SERVER_ERROR)

```

## Router(类似java的Controller)接口返回的时候调用APIResponse

```py
@router.get("/path/getInfo")
async def get_info(name: str = None):

    "implementation logic"
    content = user_info...

    return APIResponse.success(json.dumps(content))
```

这样调用/path/getInfo接口返回的内容形如以下内容：

```py
{
    "success": true,
    "errors": false,
    "data": {
        "userName": "zhangsan",
        "age": 19,
        ...
    }
}
```