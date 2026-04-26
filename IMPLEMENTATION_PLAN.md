# Tax Filing Tool (VAT_Reporting) - Implementation Plan v1

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a complete UStVA/ZM filing tool with ELSTER integration, draft management, and analytics dashboard.

**Architecture:** Python FastAPI backend + React frontend + PostgreSQL. ELSTER integration via erica REST wrapper. All data encrypted at rest.

**Tech Stack:** Python 3.12, FastAPI, PostgreSQL 16, SQLAlchemy ORM, React 19, TypeScript, TailwindCSS, Docker, pytest, pytest-cov

---

## Project Structure

```
tax-filing-tool/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py (FastAPI app)
│   │   ├── models.py (SQLAlchemy)
│   │   ├── schemas.py (Pydantic)
│   │   ├── config.py (env vars)
│   │   ├── auth/
│   │   │   ├── __init__.py
│   │   │   ├── routes.py (register, login)
│   │   │   ├── utils.py (JWT, hashing)
│   │   │   └── security.py (dependencies)
│   │   ├── documents/
│   │   │   ├── __init__.py
│   │   │   ├── routes.py (CRUD endpoints)
│   │   │   ├── service.py (business logic)
│   │   │   ├── validation.py (field validation)
│   │   │   └── calculations.py (KZ formulas)
│   │   ├── elster/
│   │   │   ├── __init__.py
│   │   │   ├── client.py (erica REST wrapper)
│   │   │   ├── xml_builder.py (XML generation)
│   │   │   └── response_parser.py
│   │   ├── exports/
│   │   │   ├── __init__.py
│   │   │   ├── pdf.py (PDF generation)
│   │   │   ├── csv.py (CSV export)
│   │   │   └── xml.py (XML export)
│   │   └── dashboard/
│   │       ├── __init__.py
│   │       ├── routes.py (analytics endpoints)
│   │       └── analytics.py (calculations)
│   ├── tests/
│   │   ├── __init__.py
│   │   ├── conftest.py (pytest fixtures)
│   │   ├── test_auth.py
│   │   ├── test_documents.py
│   │   ├── test_calculations.py
│   │   ├── test_exports.py
│   │   └── test_elster.py
│   ├── requirements.txt
│   ├── .env.example
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── App.tsx
│   │   ├── main.tsx
│   │   ├── index.css
│   │   ├── components/
│   │   │   ├── Auth/
│   │   │   │   ├── LoginForm.tsx
│   │   │   │   ├── RegisterForm.tsx
│   │   │   │   └── CertificateUpload.tsx
│   │   │   ├── Documents/
│   │   │   │   ├── DocumentList.tsx
│   │   │   │   ├── DocumentForm.tsx
│   │   │   │   ├── UStVAForm.tsx
│   │   │   │   ├── ZMForm.tsx
│   │   │   │   └── CSVImport.tsx
│   │   │   ├── Dashboard/
│   │   │   │   ├── SingleDocDashboard.tsx
│   │   │   │   ├── Highlights.tsx
│   │   │   │   ├── ComparisonView.tsx
│   │   │   │   └── TrendChart.tsx
│   │   │   └── Layout/
│   │   │       ├── Navbar.tsx
│   │   │       └── Sidebar.tsx
│   │   ├── pages/
│   │   │   ├── LoginPage.tsx
│   │   │   ├── DocumentsPage.tsx
│   │   │   ├── DashboardPage.tsx
│   │   │   └── SettingsPage.tsx
│   │   ├── services/
│   │   │   ├── api.ts (Axios instance)
│   │   │   ├── authService.ts
│   │   │   └── documentService.ts
│   │   ├── types/
│   │   │   └── index.ts
│   │   └── utils/
│   │       ├── formatting.ts
│   │       └── validation.ts
│   ├── package.json
│   ├── tsconfig.json
│   ├── vite.config.ts
│   └── Dockerfile
├── docker-compose.yml
├── .gitignore
├── README.md
└── IMPLEMENTATION_PLAN.md (this file)
```

---

## Phase 1: Project Setup + Database + Auth

### Task 1.1: Backend Project Initialization

**Files:**
- Create: `backend/requirements.txt`
- Create: `backend/.env.example`
- Create: `backend/app/__init__.py`
- Create: `backend/app/main.py`
- Create: `backend/app/config.py`

- [ ] **Step 1: Initialize Python virtual environment**

```bash
cd backend
python3.12 -m venv venv
source venv/bin/activate
```

Expected: `(venv) $` prompt appears

- [ ] **Step 2: Create requirements.txt with core dependencies**

```txt
fastapi==0.104.1
uvicorn[standard]==0.24.0
sqlalchemy==2.0.23
psycopg2-binary==2.9.9
pydantic==2.5.0
pydantic-settings==2.1.0
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.6
pytest==7.4.3
pytest-cov==4.1.0
httpx==0.25.2
cryptography==41.0.7
reportlab==4.0.7
```

- [ ] **Step 3: Install dependencies**

```bash
pip install -r requirements.txt
```

Expected: All packages installed without errors

- [ ] **Step 4: Create .env.example**

```env
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/tax_filing
DATABASE_ECHO=false

# JWT
SECRET_KEY=your-secret-key-here-min-32-chars
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60

# ELSTER (erica)
ERICA_BASE_URL=http://erica-service:5000
ERICA_TIMEOUT=30

# Application
DEBUG=false
ENVIRONMENT=development
```

- [ ] **Step 5: Create app/config.py**

```python
from pydantic_settings import BaseSettings
from typing import Optional

class Settings(BaseSettings):
    # Database
    database_url: str
    database_echo: bool = False
    
    # JWT
    secret_key: str
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 60
    
    # ELSTER
    erica_base_url: str
    erica_timeout: int = 30
    
    # Application
    debug: bool = False
    environment: str = "development"
    
    class Config:
        env_file = ".env"

settings = Settings()
```

- [ ] **Step 6: Create app/__init__.py (empty)**

```python
# empty file
```

- [ ] **Step 7: Create app/main.py (skeleton)**

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.config import settings

app = FastAPI(
    title="Tax Filing Tool",
    description="UStVA/ZM filing with ELSTER integration",
    version="1.0.0"
)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173", "http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/health")
async def health_check():
    return {"status": "ok"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

- [ ] **Step 8: Test FastAPI server starts**

```bash
python -m uvicorn app.main:app --reload
```

Expected: Server runs on `http://localhost:8000`, `/health` returns `{"status": "ok"}`

- [ ] **Step 9: Commit**

```bash
git add requirements.txt .env.example app/
git commit -m "feat: initialize FastAPI backend with core dependencies"
```

---

### Task 1.2: Database Models

**Files:**
- Create: `backend/app/models.py`

- [ ] **Step 1: Write failing test**

```bash
# File: backend/tests/test_models.py
```

```python
import pytest
from app.models import User, Document
from sqlalchemy.orm import Session

def test_user_creation(db_session: Session):
    """Test creating a user and retrieving it"""
    user = User(
        email="test@example.com",
        password_hash="hashed_password"
    )
    db_session.add(user)
    db_session.commit()
    
    retrieved = db_session.query(User).filter_by(email="test@example.com").first()
    assert retrieved is not None
    assert retrieved.email == "test@example.com"

def test_document_creation(db_session: Session):
    """Test creating a document"""
    user = User(email="test@example.com", password_hash="hash")
    db_session.add(user)
    db_session.commit()
    
    doc = Document(
        user_id=user.id,
        type="USt1A",
        period="2026-01",
        status="draft",
        data={"kz81": "50000"}
    )
    db_session.add(doc)
    db_session.commit()
    
    retrieved = db_session.query(Document).filter_by(period="2026-01").first()
    assert retrieved is not None
    assert retrieved.type == "USt1A"
```

- [ ] **Step 2: Run test (expect failure)**

```bash
cd backend
pytest tests/test_models.py -v
```

Expected: `FAILED` (models don't exist yet)

- [ ] **Step 3: Create app/models.py**

```python
from sqlalchemy import Column, String, Integer, DateTime, JSON, ForeignKey, Enum
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship
from datetime import datetime
import uuid
import enum

Base = declarative_base()

class User(Base):
    __tablename__ = "users"
    
    id = Column(String, primary_key=True, default=lambda: str(uuid.uuid4()))
    email = Column(String, unique=True, nullable=False, index=True)
    password_hash = Column(String, nullable=False)
    certificate_encrypted = Column(String, nullable=True)  # PEM format, encrypted
    certificate_expires = Column(DateTime, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow, nullable=False)
    
    documents = relationship("Document", back_populates="user")

class DocumentStatus(str, enum.Enum):
    draft = "draft"
    submitted = "submitted"
    error = "error"

class DocumentType(str, enum.Enum):
    USt1A = "USt1A"
    USt1H = "USt1H"
    ZM = "ZM"

class Document(Base):
    __tablename__ = "documents"
    
    id = Column(String, primary_key=True, default=lambda: str(uuid.uuid4()))
    user_id = Column(String, ForeignKey("users.id"), nullable=False, index=True)
    type = Column(Enum(DocumentType), nullable=False)
    period = Column(String, nullable=False)  # "2026-01" or "2026-Q1"
    status = Column(Enum(DocumentStatus), default=DocumentStatus.draft)
    data = Column(JSON, default={})  # { "kz81": "50000", ... }
    elster_receipt = Column(JSON, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow, nullable=False)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    submitted_at = Column(DateTime, nullable=True)
    
    user = relationship("User", back_populates="documents")

class AuditLog(Base):
    __tablename__ = "audit_logs"
    
    id = Column(String, primary_key=True, default=lambda: str(uuid.uuid4()))
    user_id = Column(String, ForeignKey("users.id"), nullable=False, index=True)
    action = Column(String, nullable=False)  # "create", "update", "submit"
    document_id = Column(String, ForeignKey("documents.id"), nullable=True)
    details = Column(JSON, default={})
    created_at = Column(DateTime, default=datetime.utcnow, nullable=False)
```

- [ ] **Step 4: Create conftest.py for pytest fixtures**

```python
# File: backend/tests/conftest.py
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session
from app.models import Base

# Use in-memory SQLite for testing
@pytest.fixture
def db_engine():
    engine = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(engine)
    return engine

@pytest.fixture
def db_session(db_engine) -> Session:
    SessionLocal = sessionmaker(bind=db_engine)
    session = SessionLocal()
    yield session
    session.close()
```

- [ ] **Step 5: Run test (expect pass)**

```bash
pytest tests/test_models.py -v
```

Expected: `PASSED` (2 tests)

- [ ] **Step 6: Commit**

```bash
git add app/models.py tests/test_models.py tests/conftest.py
git commit -m "feat: add SQLAlchemy models for User, Document, AuditLog"
```

---

### Task 1.3: Database Connection + Alembic Migrations

**Files:**
- Create: `backend/alembic.ini`
- Create: `backend/alembic/` (Alembic directory)
- Modify: `backend/app/main.py`

- [ ] **Step 1: Install Alembic**

```bash
pip install alembic
pip install -r requirements.txt
```

Expected: alembic installed

- [ ] **Step 2: Initialize Alembic**

```bash
cd backend
alembic init alembic
```

Expected: `alembic/` directory created with versions/, env.py, script.py.mako

- [ ] **Step 3: Configure alembic/env.py**

```python
# In alembic/env.py, modify the imports and config:
from app.config import settings
from app.models import Base

# Set sqlalchemy.url
config.set_main_option("sqlalchemy.url", settings.database_url)

# Set target_metadata
target_metadata = Base.metadata

# In the run_migrations_online() function, ensure:
# - Connection uses settings.database_url
# - Transactions are configured correctly
```

- [ ] **Step 4: Create initial migration**

```bash
alembic revision --autogenerate -m "initial schema"
```

Expected: Migration file created in `alembic/versions/`

- [ ] **Step 5: Apply migration**

```bash
alembic upgrade head
```

Expected: Database schema created

- [ ] **Step 6: Update app/main.py to add DB dependency**

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session
from app.config import settings

engine = create_engine(settings.database_url, echo=settings.database_echo)
SessionLocal = sessionmaker(bind=engine)

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# Add to app startup
@app.on_event("startup")
async def startup():
    print(f"Connected to database: {settings.database_url}")

@app.on_event("shutdown")
async def shutdown():
    engine.dispose()
```

- [ ] **Step 7: Test DB connection**

```bash
python -c "from app.main import engine; print(engine.execute('SELECT 1'))"
```

Expected: No errors

- [ ] **Step 8: Commit**

```bash
git add alembic/ alembic.ini app/main.py requirements.txt
git commit -m "feat: add Alembic migrations and database connection"
```

---

### Task 1.4: JWT Authentication + User Registration/Login

**Files:**
- Create: `backend/app/auth/__init__.py`
- Create: `backend/app/auth/utils.py`
- Create: `backend/app/auth/security.py`
- Create: `backend/app/auth/routes.py`
- Create: `backend/app/schemas.py`

- [ ] **Step 1: Write failing tests**

```python
# File: backend/tests/test_auth.py
import pytest
from fastapi.testclient import TestClient
from app.main import app
from sqlalchemy.orm import Session

client = TestClient(app)

def test_register_user(db_session: Session):
    """Test user registration"""
    response = client.post("/api/auth/register", json={
        "email": "user@example.com",
        "password": "SecurePassword123!"
    })
    assert response.status_code == 201
    assert response.json()["email"] == "user@example.com"

def test_register_duplicate_email(db_session: Session):
    """Test duplicate email rejection"""
    client.post("/api/auth/register", json={
        "email": "user@example.com",
        "password": "SecurePassword123!"
    })
    response = client.post("/api/auth/register", json={
        "email": "user@example.com",
        "password": "DifferentPassword123!"
    })
    assert response.status_code == 400

def test_login_success(db_session: Session):
    """Test successful login"""
    # Register first
    client.post("/api/auth/register", json={
        "email": "user@example.com",
        "password": "SecurePassword123!"
    })
    
    # Login
    response = client.post("/api/auth/login", json={
        "email": "user@example.com",
        "password": "SecurePassword123!"
    })
    assert response.status_code == 200
    assert "access_token" in response.json()
    assert response.json()["token_type"] == "bearer"

def test_login_wrong_password(db_session: Session):
    """Test login with wrong password"""
    client.post("/api/auth/register", json={
        "email": "user@example.com",
        "password": "SecurePassword123!"
    })
    
    response = client.post("/api/auth/login", json={
        "email": "user@example.com",
        "password": "WrongPassword123!"
    })
    assert response.status_code == 401
```

- [ ] **Step 2: Run tests (expect failure)**

```bash
pytest tests/test_auth.py -v
```

Expected: All tests FAIL (endpoints don't exist)

- [ ] **Step 3: Create app/schemas.py**

```python
from pydantic import BaseModel, EmailStr
from typing import Optional
from datetime import datetime

class UserRegisterRequest(BaseModel):
    email: EmailStr
    password: str

class UserLoginRequest(BaseModel):
    email: EmailStr
    password: str

class UserResponse(BaseModel):
    id: str
    email: str
    created_at: datetime
    
    class Config:
        from_attributes = True

class TokenResponse(BaseModel):
    access_token: str
    token_type: str
    user: UserResponse

class DocumentBase(BaseModel):
    type: str  # "USt1A", "USt1H", "ZM"
    period: str  # "2026-01" or "2026-Q1"

class DocumentCreate(DocumentBase):
    data: dict = {}

class DocumentUpdate(BaseModel):
    data: dict

class DocumentResponse(DocumentBase):
    id: str
    status: str
    created_at: datetime
    submitted_at: Optional[datetime]
    
    class Config:
        from_attributes = True
```

- [ ] **Step 4: Create app/auth/utils.py**

```python
from passlib.context import CryptContext
from datetime import datetime, timedelta
from jose import JWTError, jwt
from app.config import settings

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=settings.access_token_expire_minutes)
    
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, settings.secret_key, algorithm=settings.algorithm)
    return encoded_jwt

def decode_access_token(token: str) -> str:
    try:
        payload = jwt.decode(token, settings.secret_key, algorithms=[settings.algorithm])
        user_id: str = payload.get("sub")
        if user_id is None:
            return None
        return user_id
    except JWTError:
        return None
```

Add to requirements.txt:
```
email-validator==2.1.0
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
```

- [ ] **Step 5: Create app/auth/security.py**

```python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthCredentials
from app.auth.utils import decode_access_token
from app.models import User
from sqlalchemy.orm import Session

security = HTTPBearer()

def get_current_user(credentials: HTTPAuthCredentials = Depends(security), db: Session = Depends(get_db)) -> User:
    token = credentials.credentials
    user_id = decode_access_token(token)
    
    if not user_id:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials",
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    user = db.query(User).filter(User.id == user_id).first()
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="User not found"
        )
    
    return user
```

- [ ] **Step 6: Create app/auth/routes.py**

```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session
from datetime import timedelta
from app.schemas import UserRegisterRequest, UserLoginRequest, UserResponse, TokenResponse
from app.models import User
from app.auth.utils import hash_password, verify_password, create_access_token
from app.main import get_db

router = APIRouter(prefix="/api/auth", tags=["auth"])

@router.post("/register", response_model=TokenResponse, status_code=201)
async def register(request: UserRegisterRequest, db: Session = Depends(get_db)):
    # Check if user exists
    existing_user = db.query(User).filter(User.email == request.email).first()
    if existing_user:
        raise HTTPException(status_code=400, detail="Email already registered")
    
    # Create user
    user = User(
        email=request.email,
        password_hash=hash_password(request.password)
    )
    db.add(user)
    db.commit()
    db.refresh(user)
    
    # Generate token
    access_token = create_access_token(data={"sub": user.id})
    
    return {
        "access_token": access_token,
        "token_type": "bearer",
        "user": UserResponse.from_orm(user)
    }

@router.post("/login", response_model=TokenResponse)
async def login(request: UserLoginRequest, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.email == request.email).first()
    
    if not user or not verify_password(request.password, user.password_hash):
        raise HTTPException(status_code=401, detail="Invalid email or password")
    
    access_token = create_access_token(data={"sub": user.id})
    
    return {
        "access_token": access_token,
        "token_type": "bearer",
        "user": UserResponse.from_orm(user)
    }
```

- [ ] **Step 7: Register routes in app/main.py**

```python
from app.auth.routes import router as auth_router

app.include_router(auth_router)
```

- [ ] **Step 8: Fix imports in conftest.py**

```python
# backend/tests/conftest.py
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session
from fastapi.testclient import TestClient
from app.models import Base
from app.main import app, get_db

@pytest.fixture
def db_engine():
    engine = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(engine)
    return engine

@pytest.fixture
def db_session(db_engine) -> Session:
    SessionLocal = sessionmaker(bind=db_engine)
    session = SessionLocal()
    yield session
    session.close()

@pytest.fixture
def client(db_session):
    def override_get_db():
        return db_session
    
    app.dependency_overrides[get_db] = override_get_db
    yield TestClient(app)
    app.dependency_overrides.clear()
```

- [ ] **Step 9: Run tests**

```bash
pytest tests/test_auth.py -v
```

Expected: All tests PASS

- [ ] **Step 10: Commit**

```bash
git add app/auth/ app/schemas.py app/main.py tests/test_auth.py requirements.txt
git commit -m "feat: implement JWT authentication with register/login endpoints"
```

---

## Phase 2: Document CRUD + Validation

### Task 2.1: Document Validation & Calculations

**Files:**
- Create: `backend/app/documents/validation.py`
- Create: `backend/app/documents/calculations.py`
- Create: `backend/tests/test_calculations.py`

- [ ] **Step 1: Write test for KZ calculations**

```python
# File: backend/tests/test_calculations.py
import pytest
from app.documents.calculations import UStVACalculator

def test_ustva_calculation_simple():
    """Test basic USt 1 A calculation"""
    data = {
        "kz81": 50000,  # 19% revenue
        "kz86": 30000,  # 7% revenue
        "kz66": 10000,  # Input tax
        "kz61": 2000,   # Input tax from EU
        "kz62": 500,    # Import VAT
    }
    
    calc = UStVACalculator(data)
    result = calc.calculate()
    
    # KZ 19 (Output tax) = KZ81*0.19 + KZ86*0.07 = 50000*0.19 + 30000*0.07 = 9500 + 2100 = 11600
    assert result["kz19"] == 11600
    
    # KZ 65 (Remaining) = KZ19 - (KZ66 + KZ61 + KZ62) = 11600 - 12500 = -900
    assert result["kz65"] == -900

def test_ustva_with_zero_values():
    """Test with zero/null values"""
    data = {
        "kz81": 100000,
        "kz86": 0,
        "kz66": 15000,
    }
    
    calc = UStVACalculator(data)
    result = calc.calculate()
    
    assert result["kz19"] == 19000  # 100000*0.19
    assert result["kz65"] == 4000   # 19000 - 15000
```

Expected: FAIL (UStVACalculator doesn't exist)

- [ ] **Step 2: Create app/documents/calculations.py**

```python
from typing import Dict, Optional

class UStVACalculator:
    """Calculate KZ values for USt 1 A based on input data"""
    
    # Steuersätze (tax rates)
    RATES = {
        "kz81": 0.19,   # 19%
        "kz86": 0.07,   # 7%
        "kz87": 0.0,    # 0% PV
        "kz35": None,   # Other rates (user input)
    }
    
    def __init__(self, data: Dict[str, float]):
        self.data = {k: float(v) if v else 0 for k, v in data.items()}
    
    def calculate(self) -> Dict[str, float]:
        """Calculate all derived KZ values"""
        result = self.data.copy()
        
        # Output tax (Umsatzsteuer) = sum of (revenue * rate)
        output_tax = 0
        output_tax += self.data.get("kz81", 0) * self.RATES["kz81"]
        output_tax += self.data.get("kz86", 0) * self.RATES["kz86"]
        output_tax += self.data.get("kz87", 0) * self.RATES["kz87"]
        # Add other tax rates if applicable
        result["kz19"] = output_tax
        
        # Input tax total
        input_tax_total = (
            self.data.get("kz66", 0) +  # Inland input tax
            self.data.get("kz61", 0) +  # EU acquisition input tax
            self.data.get("kz62", 0) +  # Import VAT
            self.data.get("kz67", 0) +  # §13b input tax
            self.data.get("kz63", 0) +  # Average rate input tax
            self.data.get("kz59", 0) -  # New vehicle input tax
            self.data.get("kz64", 0)    # Input tax adjustment
        )
        
        # Remaining amount (KZ 65)
        result["kz65"] = output_tax - input_tax_total
        
        # Tax prepayment (KZ 83)
        result["kz83"] = result["kz65"] - self.data.get("kz69", 0) - self.data.get("kz39", 0)
        
        return result
```

- [ ] **Step 3: Run test**

```bash
pytest tests/test_calculations.py -v
```

Expected: PASS

- [ ] **Step 4: Create app/documents/validation.py**

```python
from pydantic import BaseModel, validator
from typing import Dict, Optional

class UStVAValidation:
    """Validate UStVA form data"""
    
    REQUIRED_KZ = ["kz81"]  # Minimum required fields
    NUMERIC_KZ = ["kz81", "kz86", "kz87", "kz35", "kz36", "kz66", "kz61", "kz62"]
    
    @staticmethod
    def validate_data(data: Dict[str, any]) -> tuple[bool, Optional[str]]:
        """Validate document data, return (is_valid, error_message)"""
        
        # Check required fields
        for kz in UStVAValidation.REQUIRED_KZ:
            if kz not in data or data[kz] is None:
                return False, f"Missing required field {kz}"
        
        # Check numeric fields
        for kz in UStVAValidation.NUMERIC_KZ:
            if kz in data:
                try:
                    float(data[kz])
                except (ValueError, TypeError):
                    return False, f"Field {kz} must be numeric"
        
        # Check value ranges (no negative revenues)
        if data.get("kz81", 0) < 0:
            return False, "KZ 81 (revenue) cannot be negative"
        
        return True, None

class ZMValidation:
    """Validate ZM (Zusammenfassende Meldung) data"""
    
    REQUIRED_FIELDS = ["steuernummer"]
    
    @staticmethod
    def validate_data(data: Dict[str, any]) -> tuple[bool, Optional[str]]:
        for field in ZMValidation.REQUIRED_FIELDS:
            if field not in data or not data[field]:
                return False, f"Missing required field: {field}"
        
        return True, None
```

- [ ] **Step 5: Commit**

```bash
git add app/documents/calculations.py app/documents/validation.py tests/test_calculations.py
git commit -m "feat: add KZ calculation logic and field validation"
```

---

### Task 2.2: Document CRUD Endpoints

**Files:**
- Create: `backend/app/documents/__init__.py`
- Create: `backend/app/documents/routes.py`
- Create: `backend/app/documents/service.py`
- Create: `backend/tests/test_documents.py`

- [ ] **Step 1: Write failing tests**

```python
# File: backend/tests/test_documents.py
import pytest
from fastapi.testclient import TestClient
from app.main import app
from sqlalchemy.orm import Session

def test_create_ustva_draft(client: TestClient, db_session: Session):
    """Test creating a USt 1 A draft"""
    # Register and login first
    client.post("/api/auth/register", json={
        "email": "user@example.com",
        "password": "SecurePassword123!"
    })
    login_resp = client.post("/api/auth/login", json={
        "email": "user@example.com",
        "password": "SecurePassword123!"
    })
    token = login_resp.json()["access_token"]
    
    # Create document
    response = client.post(
        "/api/documents",
        json={
            "type": "USt1A",
            "period": "2026-01",
            "data": {"kz81": 50000}
        },
        headers={"Authorization": f"Bearer {token}"}
    )
    
    assert response.status_code == 201
    assert response.json()["status"] == "draft"
    assert response.json()["type"] == "USt1A"

def test_get_document(client: TestClient):
    """Test retrieving a document"""
    # ... (create document first)
    response = client.get(f"/api/documents/{doc_id}", headers={"Authorization": f"Bearer {token}"})
    assert response.status_code == 200

def test_list_documents(client: TestClient):
    """Test listing user's documents"""
    response = client.get("/api/documents", headers={"Authorization": f"Bearer {token}"})
    assert response.status_code == 200
    assert isinstance(response.json(), list)

def test_update_document(client: TestClient):
    """Test updating a draft"""
    response = client.patch(
        f"/api/documents/{doc_id}",
        json={"data": {"kz81": 60000}},
        headers={"Authorization": f"Bearer {token}"}
    )
    assert response.status_code == 200
    assert response.json()["data"]["kz81"] == 60000

def test_delete_document(client: TestClient):
    """Test deleting a draft"""
    response = client.delete(f"/api/documents/{doc_id}", headers={"Authorization": f"Bearer {token}"})
    assert response.status_code == 204
```

- [ ] **Step 2: Run tests (expect failure)**

```bash
pytest tests/test_documents.py -v
```

Expected: FAIL (endpoints don't exist)

- [ ] **Step 3: Create app/documents/__init__.py (empty)**

```python
# empty
```

- [ ] **Step 4: Create app/documents/service.py**

```python
from sqlalchemy.orm import Session
from app.models import Document, DocumentStatus, DocumentType, AuditLog
from app.documents.validation import UStVAValidation, ZMValidation
from app.documents.calculations import UStVACalculator
from typing import List, Dict, Optional
import json

class DocumentService:
    @staticmethod
    def create_document(user_id: str, doc_type: str, period: str, data: Dict, db: Session) -> Document:
        """Create a new document draft"""
        
        # Validate input
        if doc_type == "USt1A":
            is_valid, error = UStVAValidation.validate_data(data)
        elif doc_type == "ZM":
            is_valid, error = ZMValidation.validate_data(data)
        else:
            is_valid, error = False, f"Unknown document type: {doc_type}"
        
        if not is_valid:
            raise ValueError(error)
        
        # Create document
        doc = Document(
            user_id=user_id,
            type=DocumentType[doc_type],
            period=period,
            status=DocumentStatus.draft,
            data=data
        )
        
        db.add(doc)
        db.commit()
        db.refresh(doc)
        
        # Log action
        DocumentService._audit_log(user_id, "create", doc.id, {"type": doc_type, "period": period}, db)
        
        return doc
    
    @staticmethod
    def get_document(user_id: str, doc_id: str, db: Session) -> Optional[Document]:
        """Get a document (must belong to user)"""
        return db.query(Document).filter(
            Document.id == doc_id,
            Document.user_id == user_id
        ).first()
    
    @staticmethod
    def list_documents(user_id: str, db_session: Session, doc_type: Optional[str] = None) -> List[Document]:
        """List all documents for a user"""
        query = db_session.query(Document).filter(Document.user_id == user_id)
        if doc_type:
            query = query.filter(Document.type == DocumentType[doc_type])
        return query.order_by(Document.created_at.desc()).all()
    
    @staticmethod
    def update_document(user_id: str, doc_id: str, data: Dict, db: Session) -> Document:
        """Update a draft document"""
        doc = DocumentService.get_document(user_id, doc_id, db)
        if not doc:
            raise ValueError(f"Document {doc_id} not found")
        
        if doc.status != DocumentStatus.draft:
            raise ValueError(f"Cannot update submitted document")
        
        # Validate new data
        if doc.type.value == "USt1A":
            is_valid, error = UStVAValidation.validate_data(data)
        else:
            is_valid, error = True, None
        
        if not is_valid:
            raise ValueError(error)
        
        doc.data = data
        db.commit()
        db.refresh(doc)
        
        DocumentService._audit_log(user_id, "update", doc.id, {}, db)
        
        return doc
    
    @staticmethod
    def delete_document(user_id: str, doc_id: str, db: Session) -> bool:
        """Delete a draft document"""
        doc = DocumentService.get_document(user_id, doc_id, db)
        if not doc:
            raise ValueError(f"Document {doc_id} not found")
        
        if doc.status != DocumentStatus.draft:
            raise ValueError(f"Cannot delete submitted document")
        
        db.delete(doc)
        db.commit()
        
        DocumentService._audit_log(user_id, "delete", doc_id, {}, db)
        
        return True
    
    @staticmethod
    def _audit_log(user_id: str, action: str, doc_id: Optional[str], details: Dict, db: Session):
        """Log user action"""
        log = AuditLog(
            user_id=user_id,
            action=action,
            document_id=doc_id,
            details=details
        )
        db.add(log)
        db.commit()
```

- [ ] **Step 5: Create app/documents/routes.py**

```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session
from app.schemas import DocumentCreate, DocumentUpdate, DocumentResponse
from app.auth.security import get_current_user
from app.models import User
from app.documents.service import DocumentService
from app.main import get_db

router = APIRouter(prefix="/api/documents", tags=["documents"])

@router.post("", response_model=DocumentResponse, status_code=201)
async def create_document(
    request: DocumentCreate,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """Create a new document draft"""
    try:
        doc = DocumentService.create_document(
            current_user.id,
            request.type,
            request.period,
            request.data,
            db
        )
        return doc
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))

@router.get("/{doc_id}", response_model=DocumentResponse)
async def get_document(
    doc_id: str,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """Get a single document"""
    doc = DocumentService.get_document(current_user.id, doc_id, db)
    if not doc:
        raise HTTPException(status_code=404, detail="Document not found")
    return doc

@router.get("", response_model=list[DocumentResponse])
async def list_documents(
    doc_type: str = None,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """List all documents for current user"""
    docs = DocumentService.list_documents(current_user.id, db, doc_type)
    return docs

@router.patch("/{doc_id}", response_model=DocumentResponse)
async def update_document(
    doc_id: str,
    request: DocumentUpdate,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """Update a draft document"""
    try:
        doc = DocumentService.update_document(current_user.id, doc_id, request.data, db)
        return doc
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))

@router.delete("/{doc_id}", status_code=204)
async def delete_document(
    doc_id: str,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """Delete a draft document"""
    try:
        DocumentService.delete_document(current_user.id, doc_id, db)
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))
```

- [ ] **Step 6: Register routes in app/main.py**

```python
from app.documents.routes import router as documents_router

app.include_router(documents_router)
```

- [ ] **Step 7: Run tests**

```bash
pytest tests/test_documents.py -v
```

Expected: PASS (all 5+ tests)

- [ ] **Step 8: Commit**

```bash
git add app/documents/ tests/test_documents.py app/main.py app/schemas.py
git commit -m "feat: add document CRUD endpoints with validation"
```

---

## Phase 3: ELSTER Integration

*(Continued in next section due to length...)*

---

## Execution Notes

This plan is designed for **subagent-driven-development**:

**For each task:**
1. Read the task completely
2. Execute all steps in order
3. Run verifications (tests, server check)
4. Commit with exact message
5. Report completion

**Testing:**
- `pytest tests/ -v --cov=app --cov-report=html` to check coverage
- `pytest tests/test_X.py::test_Y -v` for single test

**Running server:**
- `python -m uvicorn app.main:app --reload` (development)

**Database:**
- `.env` must have `DATABASE_URL` set
- `alembic upgrade head` before running

---

## Success Checkpoints

After Phase 1:
- [ ] Server runs on localhost:8000
- [ ] `/health` returns 200
- [ ] Database connected
- [ ] `/api/auth/register` + `/api/auth/login` work

After Phase 2:
- [ ] All document CRUD endpoints work
- [ ] Can create/read/update/delete drafts
- [ ] Validation rejects invalid data
- [ ] Tests pass with >90% coverage

After Phase 3:
- [ ] ELSTER submission works
- [ ] Receipts stored in DB
- [ ] Error handling catches ELSTER errors

After Phase 4+:
- [ ] Frontend forms render correctly
- [ ] Dashboard shows data
- [ ] Exports generate PDF/CSV/XML

---

## Remaining Phases (Summarized)

**Phase 3: ELSTER Integration** (8 tasks)
- Certificate upload + encryption
- erica REST client
- XML generation
- Submission endpoint
- Error handling + queuing

**Phase 4: Frontend Setup** (6 tasks)
- React project init
- Auth pages
- Document pages
- Form components

**Phase 5: Exports** (5 tasks)
- PDF generation (ReportLab)
- CSV export (BMF format)
- XML export

**Phase 6: Dashboard** (7 tasks)
- Single-document highlights
- Multi-document comparison
- Trend analysis
- Charts (Recharts)

**Phase 7: Testing & Deployment** (4 tasks)
- Integration tests
- Docker Compose
- Cloud deployment
- CI/CD pipeline

