# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **simple Java web application** built to learn JSP, Servlets, and JDBC basics. It uses:
- **Struts 1.3** framework (configured in `struts-config.xml`)
- **JSP** for views with **JSTL** and custom tag (`wrapper.tag`)
- **Servlets** with `@WebServlet` annotations for request handling
- **JDBC** with MySQL (ClearDB on Heroku) for persistence
- **Eclipse** IDE with **Apache Tomcat v8.5** runtime
- **Java 8** target

## Architecture

### Package Structure
```
src/com/lanihuang/simplewebapp/
├── beans/          # Domain models (Product, UserAccount)
├── conn/           # Database connection utilities
├── filter/         # Servlet filters (JDBC, Cookie, Encoding)
├── servlet/        # Servlet controllers (Home, Product CRUD, Login, UserInfo)
└── utils/          # Database operations (DBUtils), request helpers (MyUtils)
```

### Web Content
```
WebContent/
├── WEB-INF/
│   ├── lib/                # JAR dependencies (Struts, JSTL, MySQL connector, etc.)
│   ├── tags/wrapper.tag    # Page layout tag with Bootstrap navbar
│   ├── views/              # JSP pages (home, login, product CRUD, user info)
│   ├── web.xml             # Servlet filter mappings
│   └── struts-config.xml   # Struts 1.x action mappings & form beans
├── styles.css              # Custom styles
└── META-INF/MANIFEST.MF
```

### Key Components

**Filters** (`web.xml`):
- `jdbcFilter` - Opens/closes DB connection per request, manages transactions
- `cookieFilter` - Handles remember-me cookie logic
- `encodingFilter` - Sets UTF-8 encoding

**Database** (`ConnectionUtils.java`, `MySQLConnUtils.java`):
- Hardcoded ClearDB Heroku credentials in `MySQLConnUtils.java:13-16` — **security risk**
- Connection stored in request via `MyUtils.storeConnection()`
- Auto-commit disabled; commit/rollback in `JDBCFilter.doFilter()`

**Struts Configuration** (`struts-config.xml`):
- Form beans: `LoginForm`, `ProductForm` (referenced but not in source)
- Actions mapped to `/home.do`, `/login.do`, `/doLogin.do`, `/productList.do`, `/createProduct.do`, etc.
- Uses Tiles plugin for layout (`tiles-defs.xml` referenced but not present)

**Servlets** (annotated with `@WebServlet`):
- Handle same URLs as Struts actions (potential conflict)
- Forward to JSPs under `/WEB-INF/views/`

## Development Commands

This is an **Eclipse project without Maven/Gradle**. Build and run via Eclipse:

1. **Import** into Eclipse as "Existing Project into Workspace"
2. **Configure Targeted Runtimes**: Add Apache Tomcat v8.5
3. **Build**: Project → Build Automatically (or Project → Build Project)
4. **Run**: Right-click project → Run As → Run on Server → Tomcat v8.5
5. **Access**: `http://localhost:8080/SimpleWebApp/` (context root from `.settings/org.eclipse.wst.common.component`)

### Dependencies
All JARs are committed in `WebContent/WEB-INF/lib/`:
- Struts 1.3.10 (core, taglib)
- Commons: beanutils, chain, collections, digester, logging, validator
- JSTL 1.2 (API + impl)
- MySQL Connector/J 5.1.40
- ORO 2.0.8

No package manager — add JARs manually to `WEB-INF/lib/` and refresh project.

## Security Notes

- **Hardcoded DB credentials** in `src/com/lanihuang/simplewebapp/conn/MySQLConnUtils.java:13-16` — do not commit real credentials
- Passwords stored in plaintext in database (`User_Account` table)
- No password hashing, no prepared statement for all queries (though most use `PreparedStatement`)

## Common Tasks

### Add a New Feature
1. Create servlet in `src/com/lanihuang/simplewebapp/servlet/`
2. Add JSP in `WebContent/WEB-INF/views/`
3. Update `struts-config.xml` if using Struts actions
4. Add DB logic in `DBUtils.java` if needed

### Database Changes
- Update `DBUtils.java` for new queries
- Modify `Product.java` / `UserAccount.java` for schema changes
- Run DDL manually against MySQL

### Debugging
- Check Tomcat console for `System.out.println()` from filters/servlets
- `JDBCFilter` logs "Open Connection for: <path>" for servlet requests

## Project Configuration Files

| File | Purpose |
|------|---------|
| `.project` | Eclipse project metadata |
| `.classpath` | Java build path (src, JRE, Tomcat runtime, web container) |
| `.settings/org.eclipse.wst.common.component` | Deployment assembly (WebContent→/, src→/WEB-INF/classes) |
| `.settings/org.eclipse.wst.common.project.facet.core.xml` | Facets: Java 1.8, Dynamic Web Module 3.1, JSDT Web 1.0 |
| `web.xml` | Filter mappings, welcome files |
| `struts-config.xml` | Struts action mappings, form beans, Tiles config |

## Git Notes

- `.gitignore` only excludes `/build/`
- JARs in `WEB-INF/lib/` are committed (not ideal; consider Maven/Gradle)
- `.settings/.claude/settings.json` contains API keys — should be in `.claude/settings.local.json` instead

## Known Issues

1. **Struts actions referenced but missing**: `struts-config.xml` references `com.lanihuang.simplewebapp.action.*` classes that don't exist in source — servlets handle those URLs instead
2. **Tiles config missing**: `tiles-defs.xml` referenced in `struts-config.xml:117` but not present
3. **Mixed Struts + Servlet approach**: Both handle same URLs; may cause conflicts
4. **No tests**: No test framework or test files present
5. **Old dependencies**: Struts 1.x is EOL (2013), MySQL connector 5.1.x is old