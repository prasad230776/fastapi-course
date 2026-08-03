# FastAPI Authentication & Authorization Essentials

A concise reference guide explaining Authentication (verifying identity) and Authorization (verifying permissions) in FastAPI. It covers password hashing, HTTP Basic Auth, Session-Based Auth, stateless JWT Auth, and Role-Based Access Control (RBAC).

---

## 1. Authentication vs. Authorization

*   **Authentication (AuthN):** Answers **"Who are you?"**
    *   *Examples:* Username/Password login, API keys, JWT tokens.
    *   *Failure Code:* `401 Unauthorized` (credentials are missing or incorrect).
*   **Authorization (AuthZ):** Answers **"What are you allowed to do?"**
    *   *Examples:* User vs. Admin permissions, ownership checks.
    *   *Failure Code:* `403 Forbidden` (user is authenticated but lacks permission).

---

## 2. Password Security (Hashing)

Passwords must never be stored in plain text. Always use a hashing algorithm like `bcrypt`.

```python
from passlib.context import CryptContext

# 1. Initialize CryptContext
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)
```

---

## 3. HTTP Basic Authentication

The client sends credentials encoded in Base64 via the `Authorization` header: `Authorization: Basic <base64(username:password)>`.

```python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBasic, HTTPBasicCredentials
from sqlmodel import Session, select
from database import get_session
from models import User
from auth_utils import verify_password

security = HTTPBasic()

def authenticate_user(
    credentials: HTTPBasicCredentials = Depends(security),
    session: Session = Depends(get_session)
) -> User:
    # 1. Fetch user from DB
    user = session.exec(select(User).where(User.username == credentials.username)).first()
    
    # 2. Verify existence and password
    if not user or not verify_password(credentials.password, user.password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid username or password.",
            headers={"WWW-Authenticate": "Basic"},
        )
    return user
```

---

## 4. Session-Based Authentication (Stateful)

The server stores active session IDs in a database and sets a session cookie on the client browser.

```
Login -> Verify -> Generate Session ID -> Store in DB -> Send Cookie (session_id)
Request -> Browser Sends Cookie -> Read Cookie -> Validate in DB -> Execute API
```

### Server-Side Implementation Snippet
```python
import secrets
from fastapi import APIRouter, Depends, HTTPException, Response, Cookie
from sqlmodel import Session
from database import get_session
from models import User, SessionTable

router = APIRouter()

# 1. Login & Set Cookie
@router.post("/login")
def login(response: Response, username: str, password_plain: str, session: Session = Depends(get_session)):
    # ... (Verify user credentials) ...
    session_id = secrets.token_hex(32)
    new_session = SessionTable(session_id=session_id, user_id=user.id)
    session.add(new_session)
    session.commit()

    # Set HTTP-only Cookie
    response.set_cookie(key="session_id", value=session_id, httponly=True)
    return {"message": "Login successful"}

# 2. Authenticate Dependency using Cookie
def get_current_user(session_id: str | None = Cookie(default=None), session: Session = Depends(get_session)) -> User:
    if not session_id:
        raise HTTPException(status_code=401, detail="Authentication required.")
        
    db_session = session.exec(select(SessionTable).where(SessionTable.session_id == session_id)).first()
    if not db_session:
        raise HTTPException(status_code=401, detail="Invalid session.")
        
    return session.get(User, db_session.user_id)

# 3. Logout (Invalidate Session)
@router.post("/logout")
def logout(response: Response, db_session = Depends(get_session), user = Depends(get_current_user)):
    # Invalidate session in DB and delete cookie
    session.delete(db_session)
    session.commit()
    response.delete_cookie(key="session_id")
    return {"message": "Logged out successfully"}
```

---

## 5. JWT Token-Based Authentication (Stateless)

A JSON Web Token (JWT) is a stateless, signed string containing user details. It has three parts: **Header.Payload.Signature**.
*   *Stateless:* The server does not store tokens in the database. It validates authenticity using a shared `SECRET_KEY`.

### Steps to Implement JWT in FastAPI

Implementing stateless JWT authentication involves six sequential stages:

1. **Install Dependencies:** Setup security packages: `pyjwt` for token encoding/decoding, and `passlib` with `bcrypt` for password hashing.
2. **Configure Hashing Utilities:** Create helper functions to hash plain text passwords on registration and verify them on login.
3. **Write Token Generation Helper (`create_access_token`):** Define a function that adds an expiration timestamp (`exp`) to the payload and signs it using a secret key: `jwt.encode(payload, SECRET_KEY, algorithm)`.
4. **Create the Login/Token Route (`/login`):** Implement a POST route that validates the password. Upon successful verification, it generates a JWT and returns it to the client: `{"access_token": token, "token_type": "bearer"}`.
5. **Implement Authentication Dependency (`get_current_user`):** Create a dependency utilizing `HTTPBearer` to extract the token from request headers (`Authorization: Bearer <token>`), verify the signature & expiry using `jwt.decode()`, retrieve the user from the database, and return the user object.
6. **Protect Endpoints:** Use FastAPI's Dependency Injection (`Depends(get_current_user)`) on any route parameters to restrict access to authenticated users only.

### Token Generation & Verification
```python
from datetime import datetime, timedelta, timezone
import jwt
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from sqlmodel import Session
from database import get_session
from models import User

SECRET_KEY = "your-super-secret-key"
ALGORITHM = "HS256"
TOKEN_EXPIRE_MINUTES = 30
security = HTTPBearer()

def create_access_token(data: dict) -> str:
    payload = data.copy()
    payload["exp"] = datetime.now(timezone.utc) + timedelta(minutes=TOKEN_EXPIRE_MINUTES)
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)

def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    session: Session = Depends(get_session)
) -> User:
    token = credentials.credentials
    try:
        # Decode and verify signature & expiry
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id = int(payload.get("sub"))
    except jwt.PyJWTError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid or expired token."
        )
    user = session.get(User, user_id)
    if not user:
        raise HTTPException(status_code=401, detail="User not found.")
    return user
```

---

## 6. Role-Based Access Control (RBAC)

RBAC controls endpoint access based on assigned user roles (e.g., `user`, `author`, `admin`).

### Role-Based Dependency Factory
```python
from fastapi import Depends, HTTPException, status
from models import User
from auth import get_current_user

# Factory function returning a dependency
def require_roles(*allowed_roles: str):
    def authorize(user: User = Depends(get_current_user)) -> User:
        if user.role not in allowed_roles:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail="Insufficient privileges."
            )
        return user
    return authorize
```

### Protecting Routes
```python
from fastapi import FastAPI, Depends
from models import User
from auth import require_roles

app = FastAPI()

# Admin-only endpoint
@app.get("/admin/dashboard")
def admin_dashboard(user: User = Depends(require_roles("admin"))):
    return {"message": f"Welcome Admin {user.username}!"}

# Author and Admin endpoint
@app.get("/posts/write")
def write_post(user: User = Depends(require_roles("admin", "author"))):
    return {"message": "Access granted to create posts."}
```
