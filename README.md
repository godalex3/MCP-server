This source code defines a Model Context Protocol (MCP) server named `operateMysql` that exposes tools for inspecting and interacting with a MySQL 8.0 database. It serves these tools via a Starlette web application using Server-Sent Events (SSE).

---

**Overview & Architecture**

* **Core Protocol**: MCP (Model Context Protocol) implemented via the `mcp` library.


* **Transport**: SSE (`SseServerTransport`) served via Starlette on `[http://0.0.0.0:9000](http://0.0.0.0:9000)`.


* **Database Interface**: Uses `mysql.connector` to query the target database based on environment configuration.



---

**Environment Configuration (`get_db_config`)**

Loads environment variables using `dotenv`. The following configuration variables are required:

| Environment Variable | Description | Default Value | Required |
| --- | --- | --- | --- |
| `MYSQL_HOST` | Database host address

 | `localhost`<br> | No

 |
| `MYSQL_PORT` | Database port

 | `3306`<br> | No

 |
| `MYSQL_USER` | Database username

 | N/A | **Yes**<br> |
| `MYSQL_PASSWORD` | Database password

 | N/A | **Yes**<br> |
| `MYSQL_DATABASE` | Database name

 | N/A | **Yes**<br> |

---

**Core Functions**

* **`execute_sql(query: str) -> list[TextContent]`**
Executes semicolon-separated SQL statements. For read queries, it formats results as CSV string outputs. For write/mutation queries, it executes `commit()` and returns the affected row count. Individual errors within multi-statement queries are captured and returned in place without aborting the loop.


* **`get_table_name(text: str) -> list[TextContent]`**
Searches `information_schema.TABLES` for tables matching a given comment keyword (`text`) within the configured database.


* **`get_table_desc(text: str) -> list[TextContent]`**
Searches `information_schema.COLUMNS` to retrieve column names and comments for a comma-separated list of tables.


* **`get_heart_beat() -> list[TextContent]`**
Queries the `sellbox_heartbeat_count2` table for machine heartbeat records (filtered by `company_code='3006'` and `total > 0`).


* **`get_sales_count() -> list[TextContent]`**
Queries the `sellbox_heartbeat_count` table for recent sales records.


* **`get_lock_tables() -> list[TextContent]`**
Queries `performance_schema` and `information_schema` tables to identify locked tables and blocking transactions.



---

**Exposed MCP Tools**

The MCP server explicitly registers and accepts four tools:

| Tool Name | Input Arguments | Description |
| --- | --- | --- |
| `execute_sql` | `query` (string, required)

 | Executes raw SQL on the connected MySQL 8.0 instance.

 |
| `get_table_name` | `text` (string, required)

 | Searches for table names based on Chinese table comments.

 |
| `get_table_desc` | `text` (string, required)

 | Retrieves column schema details for comma-separated table names.

 |
| `get_heart_beat` | *None*<br> | Fetches heartbeat metrics and daily differences for company code `3006`.

 |

(Note: `get_sales_count` and `get_lock_tables` are implemented in the code but are not registered in `list_tools()` or routed inside `call_tool()`).

---

**API Routes & Server Setup**

* **`GET /sse`**: Endpoints for establishing the SSE stream connection.


* **`POST /messages/`**: Endpoint mounted for client-to-server SSE message handling.
