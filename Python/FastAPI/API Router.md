## Section 1: Core Concepts and Usage

### 1.1 What is `APIRouter`?

The `APIRouter` class is a tool provided by FastAPI to organize and group related API endpoints, known as _path operations_.1 As an application grows, placing all endpoints in a single file becomes unmanageable.

`APIRouter` allows you to structure your application across multiple files, similar to "Blueprints" in Flask.3 This modular approach improves readability, maintainability, and reusability.5

### 1.2 Basic Usage Pattern

Using an `APIRouter` involves three main steps: creating the router, defining path operations on it, and including it in the main `FastAPI` application.

Step 1: Create the Router

In a separate file (e.g., app/routers/users.py), import and instantiate APIRouter.3

Python

```
# In app/routers/users.py
from fastapi import APIRouter

router = APIRouter()
```

Step 2: Define Path Operations

Use the router instance to decorate functions, just as you would with a FastAPI app instance.3

Python

```
# In app/routers/users.py
from fastapi import APIRouter

router = APIRouter()

@router.get("/users/")
def read_users():
    return
```

Step 3: Include the Router in the Main App

In your main application file (e.g., main.py), import the router object and use app.include_router() to merge its routes into the main application.2

Python

```
# In main.py
from fastapi import FastAPI
from app.routers import users

app = FastAPI()

app.include_router(users.router)
```

## Section 2: Configuration with Constructor Parameters

The `APIRouter` constructor accepts several parameters to apply common configurations to all path operations within that router, reducing code repetition.3

- **`prefix`**: Adds a URL path prefix to every endpoint in the router. For example, `prefix="/users"` turns a path of `"/me"` into `/users/me`.2
    
- **`tags`**: A list of strings used to group endpoints in the interactive API documentation (e.g., Swagger UI), improving organization.2
    
- **`dependencies`**: A list of dependencies that will be executed for every request to any endpoint within the router. This is ideal for implementing authentication or other shared logic.2
    
- **`responses`**: Defines additional status code responses that will be documented for all endpoints in the router, such as a common `404 Not Found` response.2
    
- **`deprecated`**: A boolean that, when set to `True`, marks all endpoints in the router as deprecated in the OpenAPI schema.1
    
- **`include_in_schema`**: A boolean that, when set to `False`, hides all of the router's endpoints from the generated OpenAPI schema and documentation.1
    
- **`route_class`**: Allows specifying a custom `APIRoute` subclass to override core request/response handling for advanced use cases.1
    

### `APIRouter` Parameter Reference Table

|Parameter|Type|Default|Purpose|
|---|---|---|---|
|`prefix`|`str`|`""`|A URL path prefix for all routes in the router.1|
|`tags`|`Optional[List[Union[str, Enum]]]`|`None`|A list of tags to group endpoints in the API docs.1|
|`dependencies`|`Optional]`|`None`|A sequence of dependencies to apply to all routes.1|
|`responses`|`Optional]`|`None`|Additional responses to include in the OpenAPI schema.1|
|`default_response_class`|`Type`|`Default(JSONResponse)`|The default class to use for responses.1|
|`deprecated`|`Optional[bool]`|`None`|Marks all routes in the router as deprecated in the schema.1|
|`include_in_schema`|`bool`|`True`|Whether to include the router's routes in the schema.1|
|`route_class`|`Type`|`APIRoute`|A custom route class for advanced request/response handling.1|

## Section 3: Advanced Usage Patterns

### 3.1 Nesting Routers

An `APIRouter` can include another `APIRouter`. This allows for creating hierarchical API structures, which is useful for versioning (e.g., `/api/v1/users`) or organizing large domains.3 Prefixes and other parameters from parent routers are combined with those of the child router.3

Python

```
# In routers/v1/users.py
from fastapi import APIRouter
users_router = APIRouter(prefix="/users", tags=["Users v1"])
#... routes...

# In main.py
from fastapi import FastAPI
from.routers.v1 import users_router

v1_router = APIRouter(prefix="/v1")
v1_router.include_router(users_router)

app = FastAPI()
app.include_router(v1_router, prefix="/api")
```

### 3.2 Router-Level Dependencies with Return Values

A dependency defined in `APIRouter(dependencies=...)` will execute, but its return value is not passed to the path operation functions. To share a computed value (like a user object) from a router-level dependency, attach it to the `request.state` object. The endpoint can then access this value from the `request` object.

Python

```
from fastapi import APIRouter, Depends, Request

async def get_current_user(request: Request):
    # Logic to get user and attach to request state
    request.state.user = {"username": "admin"}

# The dependency is executed for all routes in this router
router = APIRouter(dependencies=)

@router.get("/me")
async def read_current_user(request: Request):
    # Access the user data attached by the dependency
    return request.state.user
```

### 3.3 Custom Route Classes (`route_class`)

For logic that should apply only to a specific router's endpoints (as opposed to global middleware), you can provide a custom `route_class`. By subclassing `fastapi.routing.APIRoute`, you can override its methods to implement specialized logic, such as custom request body handling, decompression, or automatic request timing.8

Python

```
import time
from typing import Callable
from fastapi import APIRouter, Request, Response
from fastapi.routing import APIRoute

class TimedRoute(APIRoute):
    def get_route_handler(self) -> Callable:
        original_handler = super().get_route_handler()
        async def custom_handler(request: Request) -> Response:
            start_time = time.time()
            response = await original_handler(request)
            duration = time.time() - start_time
            response.headers = str(duration)
            return response
        return custom_handler

# All routes in this router will now have the timing logic
timed_router = APIRouter(route_class=TimedRoute)
```

### 3.4 `include_router` vs. `mount`

It is critical to distinguish between `include_router` and `mount`:

- **`include_router`** is for **modularization**. It merges routes from an `APIRouter` into a single FastAPI application. All endpoints share the same configuration and appear in one unified OpenAPI documentation.2
    
- **`mount`** is for **composition**. It attaches a completely independent ASGI application (like another FastAPI app) at a specific path. The sub-application has its own isolated configuration and separate OpenAPI documentation.9
    

|Feature|`app.include_router(router)`|`app.mount("/path", sub_app)`|
|---|---|---|
|**Purpose**|**Modularization**: To structure a single application.|**Composition**: To combine independent applications.|
|**OpenAPI Schema**|**Single / Merged**: All routes appear in one `/docs`.|**Multiple / Separate**: Each app has its own `/docs`.|
|**Dependency Scope**|**Shared**: Uses the main app's dependency system.|**Isolated**: The sub-app has its own dependencies.|