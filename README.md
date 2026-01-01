# ApacheTomcat-Stop-issues
# Windows Server : 
- Edition	Windows Server 2022 Standard
- Version	21H2
- Installed on	1/26/2024
- OS build	20348.4529

# Java Version :
- java version "1.7.0_79"
- Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
- Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)

# SQL Server Version :
- Microsoft SQL Server 2016 (RTM-GDR) (KB4058560) - 13.0.1745.2 (X64)   Dec 29 2017 10:51:35   
- Copyright (c) Microsoft Corporation  Standard Edition (64-bit) on Windows Server 2022 Standard 6.3 <X64> (Build 20348: ) 
- Version : 13.0.1745.2 
- Service Pack : RTM	
- Edition : Standard Edition (64-bit)
- Apache Tomcat Version 6.0.36

# Java JVM Crash – EXCEPTION_ACCESS_VIOLATION (JDBC-ODBC Issue)

## 📌 Overview
This repository documents a **fatal JVM crash** caused by using the **deprecated JDBC-ODBC bridge** with **Java 7** in a multi-threaded reporting environment.

The application crashes with:
EXCEPTION_ACCESS_VIOLATION (0xc0000005)
Problematic frame: ntdll.dll

The crash occurs **outside the JVM**, inside native Windows code.

---

## 🧩 Environment
- **Java Version:** Java 7 (1.7.0_79)
- **JVM:** HotSpot 64-Bit Server VM
- **OS:** Windows (64-bit)
- **Server:** Apache Tomcat
- **Database Access:** JDBC-ODBC Bridge (`sun.jdbc.odbc.JdbcOdbc`)
- **Reporting Engine:** i-net Clear Reports
- **Threading:** RenderThreadPool (multi-threaded)
---

- Short & simple explanation (what happened)
- Your Java program crashed, not because of normal Java code, but because of native (C/C++) code.

## Main cause:
- You are using Java 7 (1.7.0_79) and the old JDBC-ODBC bridge
- (sun.jdbc.odbc.JdbcOdbc), which is unstable and deprecated.
- The crash happened while fetching database data during report rendering:
- sun.jdbc.odbc.JdbcOdbc.SQLFetch
- This caused a memory access violation inside Windows (ntdll.dll), so the JVM stopped.
## Why this is happening
❌ Java 7 is very old
❌ JDBC-ODBC bridge uses native code → unsafe
❌ Heavy multi-threading (RenderThreadPool)
❌ Likely large ResultSet / report data
❌ Possible buggy ODBC driver

## ❗ Root Cause
- The **JDBC-ODBC bridge** uses native C/C++ code
- It is **deprecated, unstable, and unsafe**
- Under heavy load and multi-threading, it causes **native memory access violations**
- Java heap size is NOT the issue

---
## What you should do (IMPORTANT)
✅ 1. Stop using JDBC-ODBC bridge
- It is removed after Java 8 for a reason.
👉 Replace it with a proper JDBC driver
## Examples:
- SQL Server → Microsoft JDBC Driver
- MySQL → MySQL Connector/J
- Oracle → ojdbc
- PostgreSQL → postgresql JDBC
- ❌ Avoid: sun.jdbc.odbc.JdbcOdbc


## 📉 Symptoms
- JVM crashes without Java exception handling
- `.mdmp` and `hs_err_pid*.log` files generated
- Application stops responding
- Occurs during `SQLFetch()` while generating reports

---

## ✅ Recommended Fixes

✅ 2. Upgrade Java
- Minimum recommended:Java 8
- Better: Java 11 or 17 (LTS) 
- Java 7 is end-of-life & unsafe.

- ✅ 3. Check your ODBC / DB driver
- If you must keep ODBC temporarily:
- Update ODBC driver
- Reduce concurrent report threads
- Limit result set size
- ✅ 4. Reduce report load
- Because crash happened in:
    - com.inet.report.*
    - RenderThreadPool
- Try:
    - Smaller datasets
    - Pagination
    - Fewer parallel render threads
- ✅ 5. Restart is NOT a fix
  - This is not a memory leak you can tune away.
  - Heap is fine:
  - Heap used only ~32%
  - This is a native crash.

- One-line root cause (for report / ticket)
- JVM crashed due to native memory access violation caused by deprecated JDBC-ODBC bridge on Java 7 during multi-threaded report data fetching.
  
### 1️⃣ Replace JDBC-ODBC Bridge (Critical)
Use a **native JDBC driver** instead:

| Database | Recommended Driver |
|--------|-------------------|
| SQL Server | Microsoft JDBC Driver |
| MySQL | MySQL Connector/J |
| PostgreSQL | PostgreSQL JDBC |
| Oracle | OJDBC |

❌ Avoid: java
sun.jdbc.odbc.JdbcOdbc

# Finaly 
- your Apache Tomcat 6 is unstable, and combined with Java 7, it can easily crash, especially under load or long-running reports like inet.report. Tomcat stopping once or twice a day is likely caused by one or more of these issues:

🔹 Common Causes for Tomcat Crashing
- Old Java Version
  - You’re using Java 7, which is very old. Many libraries (including JDBC drivers) have memory or compatibility issues with it.
  - Some crashes in your log (EXCEPTION_ACCESS_VIOLATION) are usually caused by native code in old JVMs.
- Old Tomcat Version
 - You’re using Tomcat 6.0.36, which is outdated and unsupported.
 - Tomcat 6 has many known memory leaks and bugs in multi-threaded environments.
- Memory Leaks / Heavy Reports
  - Your stack trace shows RenderThreadPool threads for report generation. If reports are large or many users request reports at the same time, it can exhaust heap or native memory, causing Tomcat to crash.
- JDBC/Database Driver Issues
  - Your logs show sun.jdbc.odbc.JdbcOdbc — the old JDBC-ODBC bridge.
  - It’s deprecated and unstable, especially on modern Windows or long-running servers.
- Windows Server / Service Limits
  - Sometimes Windows Server may kill Java processes if they exceed certain memory or session limits, especially on older software combinations.
 
  # 🔹 Recommended Solutions

  - Upgrade Java
    - Move from Java 7 → at least Java 8 (LTS) or Java 11 (supported).
    - This fixes many native crashes and improves stability.
- Upgrade Tomcat
    - Move from Tomcat 6 → Tomcat 9 or 10.
    - Newer versions have better thread management, memory handling, and support for modern JVMs.
- Replace JDBC-ODBC
    - Switch from JdbcOdbc to the official SQL Server JDBC driver:
        - com.microsoft.sqlserver.jdbc.SQLServerDriver
    - This is faster, stable, and compatible with modern Java versions.
- Increase Memory / Thread Pool
    - Check JAVA_OPTS for Tomcat and increase memory:
      - -Xms1024m -Xmx2048m -XX:PermSize=256m -XX:MaxPermSize=512m
    - Reduce thread pool size in your reporting engine if possible.
- Enable Logging
    - Enable Tomcat logs and GC logs to see why it stops:
      - -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:C:\tomcat\logs\gc.log
- Schedule Restarts (Temporary Fix)
    - If upgrade isn’t possible immediately, schedule Tomcat to restart once daily via Task Scheduler to avoid crashes during peak use.
-  💡 Bottom line:
  - Your current setup is outdated and unstable: Java 7 + Tomcat 6 + JDBC-ODBC + heavy reports = crashes. The best solution is to upgrade Java, Tomcat, and JDBC driver. That alone will likely stop daily crashes.
