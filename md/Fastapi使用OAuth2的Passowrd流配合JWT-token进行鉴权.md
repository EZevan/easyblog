# <center>Fastapi使用OAuth2的Passowrd流配合JWT-token进行鉴权

> 说明：FastAPI 在 fastapi.security 模块中提供了诸多安全性的工具，简化了各种安全机制的使用方法，可用于处理安全性、身份认证和授权等问题；FastAPI 作为服务端托管服务时，对用户的鉴权校验可以使用 `session token` 和 `jwt token`两种方式，本文针对后者进行阐述。


## 介绍

- FastAPI 提供了多种工具，可帮助我们以标准的方式轻松、快速地处理安全性，而无需研究和学习所有的安全规范，这相比花费大量的精力和代码处理安全性和身份认证很有好了

### OAuth2

- OAuth2 是一个规范，它定义了多种处理身份认证和授权的方法。它是一个相当广泛的规范，涵盖了一些复杂的使用场景，包括使用「第三方」（如：github、微信、微博等）进行身份认证的方法。

### OpenAPI

- OpenAPI 是一种规范，用于定义 RESTful API 的描述文件, FastAPI基于 OpenAPI，其 swagger 即 OpenAPI 规范的一种实现
- OpenAPI 有一种定义多个安全「方案」的方法，可以利用所有这些基于标准的工具，包括这些交互式文档系统
- OpenAPI 定义了以下安全方案：
    - apiKey：一个特定于应用程序的密钥，可以来自:
        - 查询参数
        - 请求头
        - cookie
    - http：标准的 HTTP 身份认证系统，包括：
        - bearer: 一个值为 Bearer 加令牌字符串的 Authorization 请求头。这是从 OAuth2 继承的
        - HTTP Basic 认证方式
        - HTTP Digest，等等
    - oauth2：所有的 OAuth2 处理安全性的方式（称为「流程」）
        - 以下几种流程适合构建 OAuth 2.0 身份认证的提供者（例如 Google，GitHub 等）： * implicit * clientCredentials * authorizationCode
        - 但是有一个特定的「流程」可以完美地用于直接在同一应用程序中处理身份认证，即 `Password`流，接下来开始介绍OAuth2的 Password 流如何应用在 FastAPI 中

### JWT

- JWT 即 "Json Web Tokens"：将JSON对象编码为不带空格的一长串的字符串的一种标准，如下：

    ```
    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
    ```

- JWT 是没有加密的，任何人可以将其内容解析出来，但它是签名的，因此，当收到我们发出的令牌时，就可以验证确实是自己所发出的令牌。
- 这样，我们可以给令牌附加一个过期时间，当令牌过期后，用户必须重新登录校验后来获取新的令牌；由于每一次的令牌都带有签名，如果用户试图修改令牌的过期时间，签名信息将会被修改，这样令牌就会验证失败。
- 如果想要看到 JWT 详细内容，可以在这里查看：[jwt.io](https://jwt.io)

## 使用

### 安装依赖

- 官方推荐使用`python-jose`来生成并校验JWT token
    ```bash
    pip install "python-jose[cryptography]"
    ```
- [Python-jose](https://github.com/mpdavis/python-jose)要求安装一个加密后端作为依赖

### Password 哈希

- “哈希”意味着将一些内容(在本例中是密码)转换为看起来像胡言乱语的字节序列(字符串)；无论何时传递完全相同的内容(完全相同的密码)，都会得到完全相同的哈希值。

- 但是，我们无法将哈希值转换回实际的密码

#### 为什么要使用密码哈希

- 试想一下，如果数据库被泄露，存储在表中的密码是明文，任何人都可以很容易通过账户密码访问系统，风险极大；所以如果表中的密码是哈希密文，就算数据库遭泄露，也不用担心密码被人知晓

### 安装 `passlib`

- `passlib`用来处理密码哈希的一个好用的python包。它支持许多安全的散列算法及使用它们的实用程序，官方推荐使用的算法是`Bcrypt`，所以安装带有`Bcrypt`依赖的`passlib`

    ```bash
    pip install "passlib[bcrypt]"
    ```

### 哈希并验证密码

1. 导入`passlib`，并创建PassLib "context"
    - 注：passlib context 支持多种哈希算法，包括仅允许验证它们的废弃旧算法

    ```py
    from passlib.context import CryptContext

    pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
    ```

2. 创建实用函数用来对用户输入的密码进行哈希

    ```py
    from passlib.context import CryptContext

    pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

    def get_password_hash(password):
        return pwd_context.hash(password)
    ```

3. 创建另一个函数用来验证用户输入的密码与存储的哈希密码是否一致
    ```py
    from passlib.context import CryptContext

    pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

    def get_password_hash(password):
        return pwd_context.hash(password)

    def verify_password(plain_password, hashed_password):
        return pwd_context.verify(plain_password, hashed_password)
    ```

4. 最后一个函数用来验证并返回有效用户，以下是完整示例
    - 注：get_user 函数中通过依赖注入获取 user repository 对象后调用 get_by_user_name 仓储方法获取用户实体对象

    ```py
    import os
    from datetime import datetime, timedelta, timezone
    from typing import Dict, Annotated, Set
    from passlib.context import CryptContext

    from fastapi import Depends, Request, Response
    from fastapi.security import OAuth2PasswordBearer, utils
    from jose import JWTError, jwt
    from starlette.middleware.base import RequestResponseEndpoint, BaseHTTPMiddleware

    from app.api.responses.response_enum import ResponseEnum
    from app.api.errors.business_exception import AuthenticationException

    from app.db.repositories.user_repo import UsersRepository
    from app.api.dependencies.database import GetRepository

    from app.models.domain.users import User
    from app.models.schemas.users import UserInResponse
    from app.models.schemas.jwt import Token, TokenData

    # use command line "openssl rand -hex 32" to generate a random secret key
    SECRET_KEY = "26b3480404abb2e70db1ec2dd4e20f00a64d74658660f6071c6f5cd4e5884f7f"
    ALGORITHM = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES = int(os.getenv("ACCESS_TOKEN_EXPIRE_MINUTES", 60))

    # create a OAuth2 authentication scheme and specify the endpoint URL to get the token as "/token"
    oauth2_scheme = OAuth2PasswordBearer(
        tokenUrl="auth/token" if os.getenv("ENV") == "dev" else "/api/auth/token",
        scheme_name="JWT"
    )

    blacklist: Set[str] = set()

    def get_user(
        username, 
        user_repo: user_repo: UsersRepository = Depends(GetRepository(User, UsersRepository))):
        return user_repo.get_by_user_name(name=username)

    pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

    # hash password
    def get_password_hash(password):
        return pwd_context.hash(password)

    # verify and match user password and hash password
    def verify_password(plain_password, hashed_password):
        return pwd_context.verify(plain_password, hashed_password)

    # verify user and return it
    def authenticate_user(username, password):
        user = get_user(username)
        if not user:
            return false
        if not verify_password(password,user.password):
            return fase
        return user

    def create_access_token(data: dict, expires_delta: timedelta | None = None) -> str:
        """
        create a new access token
        :param data: the data to be encoded in the token
        :param expires_delta: the expiration time of the token
        :return: the encoded token
        """
        to_encode = data.copy()
        if expires_delta:
            # note: the expiration time of jose jwt generated will be utc time plus current timezone hours
            # so the expiration time should be set to utc time
            expire = datetime.now(timezone.utc) + expires_delta
        else:
            expire = datetime.now(timezone.utc) + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
        to_encode.update({"exp": expire})
        encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
        return encoded_jwt


    def create_refresh_token(data: dict, expires_delta: timedelta | None = None) -> str:
        """
        create a new refresh token
        :param data: the data to be encoded in the token
        :param expires_delta: the expiration time of the refresh token
        :return: the encoded token
        """
        to_encode = data.copy()
        if expires_delta:
            expire = datetime.now(timezone.utc) + expires_delta
        else:
            expire = datetime.now(timezone.utc) + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
        to_encode.update({"exp": expire})
        encoded_refresh_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
        return encoded_refresh_jwt


    def decode_token(token: str) -> Dict:
        """
        decode the jwt token without verifying the expiration time by default.
        :param token: the token to be decoded
        :return: the decoded token
        """
        return jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM], options={"verify_exp": False})


    def remove_token(token: str) -> None:
        """
        remove the jwt token
        :param token: the token to be removed
        :return: None
        """
        # remove the expired token from the blacklist to ensure the blacklist is not too large
        if len(blacklist) > 0:
            for blacklist_token in blacklist:
                expire_time = decode_token(blacklist_token).get("exp")
                if datetime.fromtimestamp(expire_time) <= datetime.now():
                    blacklist.remove(blacklist_token)

        # add the token to the blacklist
        expires = decode_token(token).get("exp")
        if datetime.fromtimestamp(expires) > datetime.now():
            blacklist.add(token)
    ```

### 处理 JWT tokens

1. 创建一个随机的密钥来给 JWT token 签名

    ```bash
    #  使用如下命令创建随机密钥
    $ openssl rand -hex 32

    09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7
    ```
    - 注：复制上述命令生成的随机密钥

2. 使用给JWT token签名的算法创建一个变量ALGORITHM，并将其设置为“HS256”；创建一个过期变量

    ```py
    # use command line "openssl rand -hex 32" to generate a random secret key
    SECRET_KEY = "26b3480404abb2e70db1ec2dd4e20f00a64d74658660f6071c6f5cd4e5884f7f"
    ALGORITHM = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES = 60
    ```

3. 创建实用函数生成 JWT token

    ```py
    def create_access_token(data: dict, expires_delta: timedelta | None = None) -> str:
        """
        create a new access token
        :param data: the data to be encoded in the token
        :param expires_delta: the expiration time of the token
        :return: the encoded token
        """
        to_encode = data.copy()
        if expires_delta:
            # note: the expiration time of jose jwt generated will be utc time plus current timezone hours
            # so the expiration time should be set to utc time
            expire = datetime.now(timezone.utc) + expires_delta
        else:
            expire = datetime.now(timezone.utc) + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
        to_encode.update({"exp": expire})
        encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
        
        return encoded_jwt
    ```

4. 创建刷新 token 的函数

    ```py
    def create_refresh_token(data: dict, expires_delta: timedelta | None = None) -> str:
        """
        create a new refresh token
        :param data: the data to be encoded in the token
        :param expires_delta: the expiration time of the refresh token
        :return: the encoded token
        """
        to_encode = data.copy()
        if expires_delta:
            expire = datetime.now(timezone.utc) + expires_delta
        else:
            expire = datetime.now(timezone.utc) + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
        to_encode.update({"exp": expire})
        encoded_refresh_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

        return encoded_refresh_jwt
    ```

5. 创建移除 token 的函数用来处理用户登出后使 token 失效

    ```py
    # step1：先声明一个 token 黑名单用来存放手动失效的 token
    blacklist: Set[str] = set()

    # step2：定义一个解析 JWT token 的函数
    def decode_token(token: str) -> Dict:
        """
        decode the jwt token without verifying the expiration time by default.
        :param token: the token to be decoded
        :return: the decoded token
        """
        return jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM], options={"verify_exp": False})

    # step3：将 token 放入黑名单实现使 token 失效
    def remove_token(token: str) -> None:
        """
        remove the jwt token
        :param token: the token to be removed
        :return: None
        """
        # remove the expired token from the blacklist to ensure the blacklist is not too large
        if len(blacklist) > 0:
            for blacklist_token in blacklist:
                expire_time = decode_token(blacklist_token).get("exp")
                if datetime.fromtimestamp(expire_time) <= datetime.now():
                    blacklist.remove(blacklist_token)

        # add the token to the blacklist
        expires = decode_token(token).get("exp")
        if datetime.fromtimestamp(expires) > datetime.now():
            blacklist.add(token)
    ```

### 创建依赖项函数校验 JWT token

1. 创建OAuth2身份验证方案并指定端点URL以获取令牌，如下示例通过`.env`配置不同环境使用不同的端点 url

    ```py
    # create a OAuth2 authentication scheme and specify the endpoint URL to get the token as "/token"
    oauth2_scheme = OAuth2PasswordBearer(
        tokenUrl="auth/token" if os.getenv("ENV") == "dev" else "/api/auth/token"
    )
    ```

2. 通过一个获取当前用户函数接收登录后生成的 JWT token，解码并验证传入的 token，随后返回当前用户信息

    - 注：get_current_user 函数作为全局接口的依赖项，在接收到接口请求即验证当前 token 是否被移除（是否登出），随后解码并验证 token 

    ```py
    # create a OAuth2 authentication scheme and specify the endpoint URL to get the token as "/token"
    oauth2_scheme = OAuth2PasswordBearer(
        tokenUrl="auth/token" if os.getenv("ENV") == "dev" else "/api/auth/token"
    )

    async def get_current_user(
            token: Annotated[str, Depends(oauth2_scheme)],
            user_repo: UsersRepository = Depends(GetRepository(User, UsersRepository))
    ) -> UserInResponse:
        """
        get current user by jwt token
        :param token: the jwt token in the request header
        :param user_repo: the user repository instance
        :return: the user entity object
        """
        credentials_exception = AuthenticationException(
            response_msg_enum=ResponseEnum.AUTHORIZATION_ERROR,
            message="token is null or not valid",
            args={"headers": {"WWW-Authenticate": "Bearer"}}
        )
        try:
            if token in blacklist:
                raise credentials_exception
            payload = decode_token(token)
            username, expire_time = payload.values()
            if username is None:
                raise credentials_exception
            token_data = TokenData(username=username)
        except JWTError:
            raise credentials_exception
        if datetime.fromtimestamp(expire_time) < datetime.now():    # verify the token expiration time firstly
            raise credentials_exception
        user = await user_repo.get_by_user_name(name=token_data.username)
        if user is None:    # verify the user failed and raise an exception
            raise credentials_exception

        return UserInResponse.model_validate(user)
    ```

3. 将`get_current_user`作为全局接口依赖项

    ```py
    router = APIRouter()

    router.include_router(user.router, tags=["user"], prefix="/user", dependencies=[Depends(get_current_user)])
    ```

### 通过端点接口 `/token` 生成真实的 JWT token

- 实际用户登录时，通过校验用户名、密码有效后，即生成实际的 JWT token 并将其放入 cookies 或者 Request Headers
    
    - 注：下面的示例代码以`ldap`账号进行验证，故没有使用到对密码 hash 验证，实际如果使常规的用户名密码验证，只需将 ldap 验证替换成验证用户及密码哈希即可。

    ```py
    @router.post("/token", name="获取token", include_in_schema=False, tags=["auth"])
    async def login_for_access_token(form_data: Annotated[OAuth2PasswordRequestForm, Depends()]) -> dict:
        """
        login by ldap and get token by username and password
        :param form_data: the form data of the request, which contains the username and password
        :return: the result of login, which contains the jwt token
        """
        username = form_data.username
        password = form_data.password

        # the logic of ldap authentication
        result = await ldap_authenticate(username=username, password=password)

        if not result:
            raise AuthenticationException(ResponseEnum.PASSWORD_ERROR,
                                        message=ResponseEnum.PASSWORD_ERROR.get_message(),
                                        args=[username, password])

        access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
        access_token = create_access_token(
            data={"sub": username}, expires_delta=access_token_expires
        )

        return {"access_token": access_token, "token_type": "Bearer"}


    @router.post(path="/login", response_model=GenericResponse[LoginSuccess], name="登录验证LDAP并返回当前用户信息", tags=["auth"])
    async def login(
            user_info: UserInLogin,
            user_repo: UsersRepository = Depends(GetRepository(User, UsersRepository)),
    ) -> JSONResponse:
        """
        generate jwt token by username and password, the entry endpoint of the authentication
        :param user_info: the form data of the request, which contains the username and password
        :param user_repo: the user repository instance
        :return: the result of login, which contains the jwt token and the user info
        """
        if not user_info.username or not user_info.password:
            raise ParamsValidationException(ResponseEnum.PARAM_REQUIRED, message=ResponseEnum.PARAM_REQUIRED.get_message(),
                                            args=["username", "password"])

        user_db = None

        data = OAuth2PasswordRequestForm(username=user_info.username, password=user_info.password)
        result = await login_for_access_token(data)
        access_token, token_type = result.values()

        # sync user info to the database
        if access_token:    # authenticate successfully and sync user info
            ldap_user = await ldap_user_info(username=user_info.username)
            if ldap_user:
                try:
                    user = await user_repo.get_by_user_name(name=ldap_user["sAMAccountName"])
                    if not user.enabled:
                        raise AuthenticationException(
                            ResponseEnum.PERMISSION_ERROR,
                            message="Current user is not active",
                            args=[user_info.username]
                        )
                    if user:    # update user info
                        user_update = UserInUpdate(
                            id=user.id,
                            email=ldap_user["mail"],
                            full_name=ldap_user["displayName"],
                            mobile=str(ldap_user["telephoneNumber"]),
                            pwd_error_count=0,
                            last_login_time=datetime.now()
                        )
                        user_db = await user_repo.update(obj_in=user_update, updated_by=user_update.id)
                    else:       # create user info
                        user_create = UserInCreate(
                            user_name=ldap_user["sAMAccountName"],
                            password="ldap user password",
                            email=ldap_user["mail"],
                            full_name=ldap_user["displayName"],
                            mobile=str(ldap_user["telephoneNumber"]),
                            user_type=UserTypeEnum.PROJECT_MAINTAINER.value,
                            pwd_expire_time=datetime(2099, 12, 31, 23, 59, 59)
                        )
                        user_db = await user_repo.create(obj_in=user_create, created_by="sync ldap")
                except Exception as e:
                    raise BusinessException(
                        response_msg_enum=ResponseEnum.SERVER_ERROR,
                        message="Sync ldap user info to the database failed",
                        args=[str(e.args)]
                    )
            else:
                logger.error("authenticating ldap user successfully, but no user info found in ldap with the username.")
        else:
            user_db = await user_repo.get_by_user_name(name=user_info.username)
            if user_db:  # update the number of password errors
                if user_db.pwd_error_count < 5:
                    user_update = UserInUpdate(
                        id=user_db.id,
                        pwd_error_count=user_db.pwd_error_count + 1
                    )
                    await user_repo.update(obj_in=user_update, updated_by=user_update.id)
                else:
                    raise AuthenticationException(
                        ResponseEnum.LOCKED_ERROR,
                        message="the number of password errors exceeds 5 times, the user would be locked.",
                        args=[user_info.username, user_info.password])
            raise AuthenticationException(ResponseEnum.PASSWORD_ERROR,
                                        message=ResponseEnum.PASSWORD_ERROR.get_message(),
                                        args=[user_info.username, user_info.password])

        resp = LoginSuccess(
            token=access_token,
            token_type=token_type,
            id=user_db.id,
            user_name=user_db.user_name,
            email=user_db.email,
            full_name=user_db.full_name,
            mobile=user_db.mobile,
            profile_pic_id=user_db.profile_pic_id,
            user_type=user_db.user_type,
            pwd_error_count=user_db.pwd_error_count,
            pwd_expire_time=user_db.pwd_expire_time,
            last_login_time=user_db.last_login_time,
            locked=user_db.locked,
            enabled=user_db.enabled,
            disable_time=user_db.disable_time,
            version=user_db.version
        )

        response = ApiResponse.success(resp)
        response.headers["Authorization"] = f"Bearer {access_token}"
        # use 'Authorization' header, instead of using set_cookie method to set the cookie
        # cookie_expire_time = datetime.fromtimestamp(decode_token(access_token).get("exp")).strftime("%Y-%m-%d %H:%M:%S")
        # response.set_cookie(
        #     key="ACCESS_TOKEN",
        #     value=f"{token_type}%20{access_token}",
        #     httponly=True,
        #     samesite=None,
        #     expires=cookie_expire_time
        # )
        return response
    ```

### 检查效果

1.  运行fastapi入口程序并访问文档： http://127.0.0.1:8000/docs， 可以看到如下swagger文档已经生效 `Authorize`鉴权，在没有登录获取 token 时，接口都是未锁状态

![001](../assets/img/Fastapi使用OAuth2的Passowrd流配合JWT-token进行鉴权/001.png)

2. 点击`Authorize`即调用端点 url `/token`接口，使用用户密码鉴权

![002](../assets/img/Fastapi使用OAuth2的Passowrd流配合JWT-token进行鉴权/002.png)

3. 登录后，可以看到注入了`get_current_user`依赖项的接口`/user`上都已鉴权"上锁"

![003](../assets/img/Fastapi使用OAuth2的Passowrd流配合JWT-token进行鉴权/003.png)

4. 此时请求`/user/getUserInfo/{id}`接口，请求头会带上 `Authorization`: `Bearer Token` 

![004](../assets/img/Fastapi使用OAuth2的Passowrd流配合JWT-token进行鉴权/004.png)


### 补充刷新`token`实现滑动过期时间的效果

1. 使用HTTP中间件拦截器实现刷新`token`

    ```py
    class RefreshTokenMiddleware(BaseHTTPMiddleware):
        """
        the middleware to refresh the jwt token
        """
        async def dispatch(self, request: Request, call_next: RequestResponseEndpoint) -> Response:
            """
            refresh the jwt token
            :param call_next: RequestResponseEndpoint
            :param request: the request object
            :return: None
            """
            response = await call_next(request)
            authorization = request.headers.get("Authorization")
            scheme, param = utils.get_authorization_scheme_param(authorization)
            if scheme.lower() == "bearer" and param and param not in ["null", "undefined"]:
                username, expire_time = decode_token(param).values()
                refresh_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)

                # refresh token only when the token is not expired.
                if datetime.fromtimestamp(expire_time) > datetime.now():
                    refresh_token = create_refresh_token(data={"sub": username}, expires_delta=refresh_token_expires)
                    response.headers["Authorization"] = f"Bearer {refresh_token}"

        return response
    ```

    - 继承`BaseHTTPMiddleware`并实现`dispatch`方法，在该方法中
        - 首先通过接口请求头获取`Authorization` token
        - 验证有效的 `Bearer` token
        - 解码`token`并判断是否过期
        - `token`未过期则刷新 token 并通过 `Response Headers` 返回