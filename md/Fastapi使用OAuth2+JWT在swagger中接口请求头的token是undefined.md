# <center>Fastapi使用OAuth2+JWT在swagger中接口请求头的token是undefined

- 问题描述：

    1. Fastapi在使用OAuth2的password+JWT token鉴权保护接口
    2. Swagger UI中被保护的接口请求头的`Authorization` Bearer token变成了Bearer undefined

- 原因：

    - 在获取token的接口中返回出参的格式不正确
    - 获取token接口的出参必须是固定格式，形如：
    ```json
        {
            "access_token": "token",
            "token_type": "bearer"
        }
    ```

- 说明：

    - 在Swagger UI中对接口使用jwt token进行保护的逻是：
        1. 使用OAuth2 Password流的方式生成token后，使用上述固定格式返回`access_token`
        2. 固定获取 `access_token`，并在swagger UI内存中存储token
        3. 请求被token保护的接口，swagger ui会自动从内存中取出token并放到请求头的 `Authorization`中，即
        ```
            Authorization: Bearer actual_token
        ```
    - 所以，如果在获取token的接口中返回的不是固定的 `access_token`、`token_type`，则无法取到token值，即出现token是 `undefined`
