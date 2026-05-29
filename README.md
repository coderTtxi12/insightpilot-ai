# Natural Language Analytics Dashboard

InsightPilot AI is a natural language analytics platform that lets business users ask questions in plain English and automatically generates executable Python code, database queries, and dynamic visualizations from structured business data.

---

## 🎯 Overview

This system allows restaurant owners to ask questions in plain English like:
- *"Show me sales comparison between Downtown and Airport locations"*
- *"What were my top 5 selling products last week?"*
- *"Graph hourly sales for Friday vs Saturday at all stores"*

**The system automatically:**
1. Parses the natural language query
2. Generates Python code to query the database
3. Executes the code in a secure environment
4. Creates dynamic visualizations using Python libraries - the agent can generate **any type of visualization** (bar charts, line graphs, pie charts, heatmaps, scatter plots, tables, metrics, etc.) because it writes and executes custom code
5. Returns the visualization to the dashboard

---

## 🏗️ Architecture

### System Components

```
┌─────────────────┐
│   Frontend       │  Next.js Dashboard (TypeScript)
│   (Port 3000)    │  - Natural language input
└────────┬────────┘  - Dynamic visualization rendering
         │
         │ HTTP/REST
         ▼
┌─────────────────┐
│   Backend API   │  FastAPI (Python)
│   (Port 8000)   │  - Query processing
└────────┬────────┘  - Session management
         │
         │ OpenAI API
         ▼
┌─────────────────┐
│  Coding Agent   │  Agentic Loop
│                 │  - Code generation
│                 │  - Tool execution
│                 │  - Context compression
└────────┬────────┘
         │
         │ Code Execution
         ▼
┌─────────────────┐
│ Code Executor   │  Isolated Docker Container
│ (Port 8001)     │  - Secure code execution
│                 │  - Database access
└────────┬────────┘
         │
         │ SQL Queries
         ▼
┌─────────────────┐
│   Supabase      │  PostgreSQL Database
│   (Production)  │  - Normalized restaurant data
└─────────────────┘
```

### Data Flow

1. **User Query** → Frontend sends natural language query
2. **LLM Processing** → Backend uses OpenAI to interpret query
3. **Code Generation** → Agent generates Python code with database queries
4. **Code Execution** → Isolated executor runs code safely
5. **Data Retrieval** → Executor queries Supabase database
6. **Visualization** → Results formatted as charts/tables
7. **Response** → Frontend renders interactive widgets

---

## 🤖 The Coding Agent: Core Innovation

### Why a Coding Agent?

Traditional approaches map queries directly to SQL, but this is brittle:
- **Limited flexibility**: Can't handle complex data transformations
- **No iteration**: Can't refine queries based on results
- **Fixed patterns**: Struggles with ambiguous or novel requests

**My solution**: An agentic system that **generates and executes Python code** dynamically.

### How It Works

The coding agent uses a **ReAct (Reasoning + Acting) loop**:

```
┌─────────────────────────────────────────────────────────┐
│ 1. User Query: "Compare sales by location"            │
└─────────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────┐
│ 2. LLM Reasoning:                                       │
│    - Need to query orders table                        │
│    - Group by location                                  │
│    - Calculate total sales                             │
│    - Compare results                                   │
└─────────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────┐
│ 3. Code Generation:                                     │
│    ```python                                            │
│    import pandas as pd                                  │
│    from db_helper import get_db_connection              │
│                                                         │
│    conn = get_db_connection()                           │
│    df = pd.read_sql("""                                 │
│        SELECT location_name,                            │
│               SUM(total_amount) as sales                │
│        FROM orders                                      │
│        GROUP BY location_name                           │
│    """, conn)                                           │
│    print(df.to_json())                                  │
│    ```                                                  │
└─────────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────┐
│ 4. Code Execution (Isolated Sandbox)                    │
│    - Runs in Docker container                           │
│    - Limited permissions                                │
│    - Database read-only access                         │
└─────────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────┐
│ 5. Result Processing:                                   │
│    - Parse JSON output                                  │
│    - Determine chart type (bar, line, pie, etc.)        │
│    - Generate visualization                             │
└─────────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────┐
│ 6. Response to User:                                    │
│    - Interactive chart widget                           │
│    - Text summary                                       │
│    - Data table (if applicable)                         │
└─────────────────────────────────────────────────────────┘
```

### Agentic Loop Details

The agent can iterate multiple times:

1. **Initial Query** → Generate code
2. **Execute Code** → Get results
3. **Analyze Results** → If incomplete or error:
   - **Refine code** based on error messages
   - **Adjust query** based on data structure
   - **Retry execution**
4. **Final Output** → Return visualization

**Why this works better:**
- **Adaptive**: Adjusts approach based on actual data
- **Self-correcting**: Fixes errors automatically
- **Flexible**: Can handle unexpected data structures
- **Powerful**: Full Python ecosystem available (pandas, matplotlib, etc.)

---

## 🧠 Context Compression: Managing Long Conversations

### The Problem

As users ask multiple questions, the conversation history grows:
- Each message adds tokens to the LLM context
- OpenAI has token limits (e.g., 60,000 tokens)
- Long histories become expensive and slow
- **But**: We need to remember past context for follow-up questions

### The Solution: Intelligent Compression

Instead of truncating history, we **compress it intelligently**:

```
┌─────────────────────────────────────────────────────────┐
│ Before Compression:                                      │
│ - 50 messages                                            │
│ - 45,000 tokens                                          │
│ - Full conversation history                              │
└─────────────────────────────────────────────────────────┘
                    │
                    ▼ (When >70% of token limit)
┌─────────────────────────────────────────────────────────┐
│ Compression Process:                                    │
│ 1. Identify compressible messages (early in history)   │
│ 2. Send to LLM with compression prompt                 │
│ 3. Extract structured snapshot:                        │
│    - Overall goal                                        │
│    - Key knowledge                                       │
│    - File system state                                   │
│    - Recent actions                                      │
│    - Current plan                                        │
└─────────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────┐
│ After Compression:                                      │
│ - 2 messages (snapshot + acknowledgment)                │
│ - ~2,000 tokens                                          │
│ - All essential context preserved                        │
│ - Recent messages kept in full                           │
└─────────────────────────────────────────────────────────┘
```

### Why This Approach?

**Traditional truncation** (keep last N messages):
- ❌ Loses important early context
- ❌ User might reference something from earlier
- ❌ Agent "forgets" its own decisions

**My compression approach**:
- ✅ Preserves essential information
- ✅ Maintains user goals and constraints
- ✅ Keeps recent context in full detail
- ✅ Reduces token usage by ~95%

### Compression Threshold

We compress when usage exceeds **70% of token limit** (42,000 / 60,000 tokens):
- **Why 70%?** Leaves room for new queries and responses
- **What gets compressed?** Early messages (oldest first)
- **What stays?** Recent messages and tool results

---

## 📊 Data Architecture

### Source Data

The system processes **6 JSON files** from 3 different POS systems:

| Source | Files | Challenge |
|--------|-------|-----------|
| **Toast POS** | `toast_pos_export.json` | Single nested structure |
| **DoorDash** | `doordash_orders.json` | Delivery-specific fields |
| **Square POS** | 4 files (catalog, orders, payments, locations) | Split across multiple files |

**Data Quality Issues:**
- Inconsistent product names ("Hash Browns" vs "Hashbrowns")
- Typos ("Griled Chiken", "expresso")
- Category variations ("🍔 Burgers" vs "Burgers")
- Different date/time formats

### Normalized Schema

Designed a unified schema that handles all sources:

```
locations (id, name, address, ...)
    │
    ├── orders (id, location_id, order_date, order_type, ...)
    │       │
    │       ├── order_items (id, order_id, product_id, quantity, price, ...)
    │       │
    │       └── payments (id, order_id, payment_type, amount, ...)
    │
    └── products (id, location_id, name, category, normalized_name, ...)
```

**Why this design?**
- **Normalized**: Reduces data duplication
- **Flexible**: Handles different order types (dine-in, delivery, pickup)
- **Queryable**: Optimized for analytics queries
- **Extensible**: Easy to add new data sources

### ETL Process

1. **Extract**: Parse JSON files with source-specific logic
2. **Transform**: 
   - Normalize product names (fuzzy matching)
   - Standardize dates/times
   - Convert currencies
   - Handle missing fields
3. **Load**: Insert into Supabase with referential integrity

**ETL Functions** (PostgreSQL):
- `normalize_product_name()`: Fuzzy matching for product names
- `calculate_order_totals()`: Aggregations
- `detect_duplicate_orders()`: Data quality checks

---

## 🚀 Easy Setup (Local Development)

### Prerequisites

- Docker & Docker Compose
- Git

### Quick Start (Less than 5 Minutes)

```bash
# 1. Clone the repository
git clone https://github.com/coderTtxi12/clave-take-home.git
cd clave-take-home

# 2. Create .env file (copy from .env.example)
cp .env.example .env
# Edit .env with your:
# - OpenAI API key

# 3. Start all services
docker compose up -d

# 4. Run Alembic migrations ✅
docker exec -it restaurant_analytics_api alembic upgrade head

# 5. Create read-only user (if it doesn't exist) ✅
docker exec -it restaurant_analytics_db psql -U postgres -d restaurant_analytics -f /sql/create_readonly_user.sql

# 6. Install ETL functions ✅
docker exec -it restaurant_analytics_db psql -U postgres -d restaurant_analytics -f /sql/etl_functions.sql

# 7. Load data ✅
docker exec -it restaurant_analytics_api bash -c "cd /app/scripts && python3 load_all_data.py --clear"

# 8. Access the application
# Frontend: http://localhost:3000
# API: http://localhost:8000
```

### Services

| Service | Port | Description |
|---------|------|-------------|
| Frontend | 3000 | Next.js dashboard |
| Backend API | 8000 | FastAPI server |
| Code Executor | 8001 | Isolated Python executor |
| Redis | 6379 | Session storage |
| PostgreSQL | 5432 | Local database (if not using Supabase) |

### Database Setup

**Option A: Local PostgreSQL (Docker)**
- Already included in `docker-compose.yml`
- No additional setup needed

**Option B: Supabase (Production)**
1. Create project at [supabase.com](https://supabase.com)
2. Get connection string from Settings → Database
3. Update `.env` with Supabase credentials
4. Use `docker-compose.prod.yml` instead

---

## 📁 Project Structure

```
clave-take-home/
├── data/                          # Source data files (JSON from POS systems)
│   └── sources/
│       ├── toast_pos_export.json  # Toast POS data
│       ├── doordash_orders.json   # DoorDash orders
│       └── square/                # Square POS data (4 files)
│           ├── catalog.json
│           ├── locations.json
│           ├── orders.json
│           └── payments.json
│
├── my-api/                        # Backend API (FastAPI)
│   ├── app/                       # Main application code
│   │   ├── api/routes/           # API endpoints
│   │   │   ├── coding_agent.py   # Main coding agent endpoint
│   │   │   └── health.py          # Health check endpoint
│   │   ├── core/                  # Core configuration
│   │   │   ├── config.py         # Settings and environment variables
│   │   │   └── logging.py        # Logging configuration
│   │   ├── models/                # Pydantic models and database models
│   │   │   ├── coding_agent.py   # Request/response models
│   │   │   └── database.py       # SQLAlchemy models
│   │   ├── services/              # Business logic
│   │   │   ├── coding_agent_service.py  # Agentic loop implementation
│   │   │   └── session_manager.py        # Redis session management
│   │   ├── utils/                 # Utility functions
│   │   │   ├── code_executor.py  # Code execution client
│   │   │   ├── image_processor.py # Image extraction/processing
│   │   │   └── tools.py           # Agent tools (execute_code, etc.)
│   │   └── main.py                # FastAPI application entry point
│   │
│   ├── code-executor/             # Isolated code execution service
│   │   ├── executor.py            # FastAPI service for code execution
│   │   ├── db_helper.py          # Read-only database connection
│   │   └── Dockerfile             # Container definition
│   │
│   ├── scripts/                    # ETL scripts
│   │   ├── load_all_data.py       # Master ETL orchestrator
│   │   ├── load_toast_data.py     # Toast POS ETL
│   │   ├── load_doordash_data.py  # DoorDash ETL
│   │   ├── load_square_data.py    # Square POS ETL
│   │   └── etl_utils.py           # Shared ETL utilities
│   │
│   ├── alembic/                   # Database migrations
│   │   ├── env.py                 # Alembic environment config
│   │   └── versions/              # Migration files
│   │
│   ├── sql/                        # SQL scripts
│   │   ├── etl_functions.sql      # PostgreSQL functions for ETL
│   │   └── create_readonly_user.sql  # Read-only user creation
│   │
│   ├── prompts/                    # LLM prompts
│   │   ├── prompts.py             # System prompts for agent
│   │   └── DB_SCHEMA.md           # Database schema documentation
│   │
│   ├── run_alembic_migrations.sh  # Script: Run database migrations
│   ├── install_etl_functions.sh   # Script: Install PostgreSQL functions
│   ├── load_data.sh               # Script: Load ETL data
│   ├── setup_production.sh        # Script: Complete production setup
│   ├── requirements.txt           # Python dependencies
│   └── Dockerfile                 # Backend API container
│
├── my-dashboard/                   # Frontend (Next.js)
│   ├── src/
│   │   ├── app/                   # Next.js App Router
│   │   │   ├── page.tsx          # Main page component
│   │   │   ├── layout.tsx        # Root layout
│   │   │   └── api/              # Next.js API routes (proxy)
│   │   │       └── coding-agent/query/route.ts
│   │   ├── components/            # React components
│   │   │   ├── Header/           # App header with theme toggle
│   │   │   ├── WelcomeSection/   # Initial welcome screen
│   │   │   ├── MessageList/      # Active conversation view
│   │   │   ├── Message/          # Individual message component
│   │   │   ├── InputForm/        # Chat input form
│   │   │   ├── FilePreview/      # File attachment preview
│   │   │   └── TypingIndicator/   # Loading indicator
│   │   ├── hooks/                 # Custom React hooks
│   │   │   ├── useChat.ts        # Chat state management
│   │   │   └── useTheme.ts       # Theme management
│   │   └── types/                 # TypeScript type definitions
│   ├── package.json               # Node.js dependencies
│   ├── next.config.ts             # Next.js configuration
│   └── Dockerfile                 # Frontend container
│
├── docs/                           # Documentation
│   ├── EXAMPLE_QUERIES.md         # Example natural language queries
│   └── SCHEMA_HINTS.md            # Database schema hints
│
├── docker-compose.yml              # Docker Compose for local development
├── DB_SCHEMA.md                   # Database schema documentation
└── README.md                      # This file
```

---

## 📝 Key Scripts

### Database Migration Script

**File**: `my-api/run_alembic_migrations.sh`

```bash
./my-api/run_alembic_migrations.sh
```

**What it does:**
- Runs Alembic migrations to create/update database schema
- Automatically detects database type (local PostgreSQL or Supabase) from `.env`
- Handles environment variable loading and validation
- Installs Python dependencies if missing (alembic, sqlalchemy, psycopg2)

**Usage:**
- `./run_alembic_migrations.sh upgrade head` - Apply all migrations
- `./run_alembic_migrations.sh current` - Show current version
- `./run_alembic_migrations.sh history` - Show migration history

**Why Alembic?**
- Version control for schema changes
- Reproducible deployments
- Rollback capability
- Team collaboration on schema changes

### ETL Functions Installation Script

**File**: `my-api/install_etl_functions.sh`

```bash
./my-api/install_etl_functions.sh
```

**What it does:**
- Installs PostgreSQL functions for data normalization and ETL operations
- Functions include:
  - `get_or_create_category()`: Normalize and deduplicate categories
  - `get_location_id_by_source()`: Map source-specific location IDs
  - `validate_etl_data()`: Data quality checks
- Works with both local PostgreSQL and Supabase
- Uses Docker `psql` if local `psql` is not available

**Why PostgreSQL functions?**
- **Performance**: Runs close to data (no network overhead)
- **Consistency**: Same logic for ETL and queries
- **Reusability**: Can be called from Python or SQL
- **Data integrity**: Enforced at database level

### Data Loading Script

**File**: `my-api/load_data.sh`

```bash
./my-api/load_data.sh [--clear]
```

**What it does:**
1. Validates all source JSON files exist
2. Checks database schema is initialized
3. Verifies ETL functions are installed
4. Runs master ETL pipeline (`load_all_data.py`)
5. Loads data from all sources in correct order:
   - Toast POS → DoorDash → Square POS

**Options:**
- `--clear` or `-c`: Deletes all existing data before loading (fresh start)
- Without flag: Appends data (may create duplicates if re-run)

**ETL Process:**
- **Toast**: Single nested JSON → normalized orders, items, payments
- **DoorDash**: Delivery orders → unified order format
- **Square**: 4 separate files → unified schema with referential integrity

### Production Setup Script

**File**: `my-api/setup_production.sh`

```bash
./my-api/setup_production.sh
```

**What it does:**
- Orchestrates complete production setup in one command:
  1. Runs Alembic migrations
  2. Creates read-only database user
  3. Installs ETL functions
  4. Prompts for data loading (with `--clear` option)

**Features:**
- Robust error handling with clear messages
- Automatic dependency installation
- Works with both local PostgreSQL and Supabase
- Interactive prompts for data loading confirmation

**Use case**: First-time setup or fresh deployment

### Other Scripts

**`my-api/run_migrations.sh`**: Alternative migration runner (legacy)

**`my-api/TEST_COMMANDS.sh`**: Test commands for development/debugging

**`my-api/run.py`**: Alternative application entry point (development)

---

## 🛠️ Technology Stack

### Frontend
- **Next.js 14** (App Router) - React framework
- **TypeScript** - Type safety
- **Recharts** - Chart library
- **Tailwind CSS** - Styling

### Backend
- **FastAPI** - Python web framework
- **OpenAI API** - LLM for code generation
- **SQLAlchemy** - ORM for database
- **Alembic** - Database migrations

### Infrastructure
- **Docker** - Containerization
- **Supabase** - PostgreSQL database (production)
- **Redis** - Session management
- **Docker Compose** - Local development

### Why These Choices?

**FastAPI over Flask/Django:**
- Async support for concurrent requests
- Automatic API documentation
- Type hints for better IDE support

**Next.js App Router:**
- Server components for better performance
- Built-in API routes (though we use separate backend)
- Excellent TypeScript support

**Supabase:**
- Real PostgreSQL (not a toy database)
- Easy to evaluate schema design
- Production-ready from day one

---

## 🎨 Design Decisions

### 1. Separate Code Executor Service

**Why?** Security and isolation.

- Code execution is **dangerous** (arbitrary Python code)
- Isolated Docker container prevents:
  - File system access
  - Network access (except database)
  - Resource exhaustion
- **Alternative considered**: In-process execution → Rejected (too risky)

### 2. Agentic Loop vs Direct SQL

**Why agentic?** Flexibility and adaptability.

- Can handle ambiguous queries
- Self-corrects errors
- Can use full Python ecosystem (pandas, matplotlib)
- **Alternative considered**: Query templates → Too rigid

### 3. Context Compression

**Why compress instead of truncate?** Preserves important context.

- User goals and constraints remembered
- Agent's own decisions preserved
- Recent context kept in full
- **Alternative considered**: Simple truncation → Loses too much

### 4. Normalized Database Schema

**Why normalized?** Data integrity and query flexibility.

- Single source of truth for products/locations
- Easy to add new data sources
- Efficient for analytics queries
- **Alternative considered**: Denormalized → Too much duplication

---

## 📈 What Makes This Production-Ready

1. **Error Handling**: Comprehensive try/catch with user-friendly messages
2. **Security**: Isolated code execution, input validation
3. **Scalability**: Stateless API, Redis for sessions
4. **Monitoring**: Structured logging, health checks
5. **Database**: Proper migrations, indexes, constraints
6. **Documentation**: Code comments, API docs, README

---

## 🔮 Future Improvements

Given more time:

1. **Caching**: Cache query results for common questions
2. **Query Optimization**: Analyze and optimize generated SQL
3. **Query History**: Save and replay previous queries
4. **Data Refresh**: Automated ETL pipeline for new data
5. **Fullstack Agent**: Complete the fullstack agent implementation - a powerful tool that enables users to create any type of data analytics report through natural language, similar to how Lovable works for app development but specialized for data analytics workflows

---

## 📚 More Documentation

For comprehensive details about the database architecture, normalization process, and schema design, see:

### [**Database Schema Reference**](./DB_SCHEMA.md)

This document contains:

- **📊 Database Schema Overview**: Complete explanation of how data from three different POS systems (Toast, DoorDash, Square) is unified into a single normalized schema
- **🔄 Normalization Process**: Detailed breakdown of the multi-layer normalization approach:
  - Python ETL scripts (`etl_utils.py`) with normalization functions
  - PostgreSQL functions (`etl_functions.sql`) for category and location mapping
  - Fuzzy matching using `pg_trgm` extension
- **📈 Visual Database Diagram**: ASCII diagram showing all relationships (1:N, 1:1, N:1, self-references)
- **📋 Complete Table Reference**: Detailed documentation of all tables, columns, relationships, and indexes
- **🔍 Query Guidance**: Best practices for common query patterns (sales, dates, locations, products, payments)
- **📝 Important Notes**: CASCADE deletes, triggers, JSONB usage, denormalization strategy

**Key Topics Covered:**
- How product names are normalized across sources ("Hashbrowns" → "Hash Browns")
- How categories are unified ("🍔 Burgers" → "Burgers", "Drinks" → "Beverages")
- Location mapping strategy using JSONB `source_ids`
- Product mapping table for maintaining source-specific IDs
- DoorDash merchant payout handling
- Date/time field usage patterns
