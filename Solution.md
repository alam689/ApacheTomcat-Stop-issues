# Windows Server Environment Upgrade Checklist

This checklist outlines **safe steps to upgrade your environment** (Windows Server 2022, Java, Tomcat, SQL Server) without breaking your reporting system.

---

## **Environment Overview**

* **Windows Server:** 2022 Standard, 21H2, OS Build 20348.4529
* **Java:** 1.7.0_79
* **SQL Server:** 2016 Standard Edition (13.0.1745.2)
* **Apache Tomcat:** 6.0.36

---

## **Step 1: Preparation**

* Notify users about potential downtime (few hours).
* Create a backup folder: `D:\Backup_YYYYMMDD`

---

## **Step 2: Backup Everything**

### SQL Server

```
Open SSMS → Right-click database → Tasks → Back Up → Save to D:\Backup_YYYYMMDD\DB
```

### Tomcat

```
Copy entire Tomcat folder (e.g., C:\Tomcat6) → D:\Backup_YYYYMMDD\Tomcat6
Backup webapps, conf, lib, logs
```

### Application Files

```
Copy .war files or custom scripts → D:\Backup_YYYYMMDD\Apps
```

✅ Ensure backups are readable and complete.

---

## **Step 3: Install Modern Java**

1. Download **Java 17 LTS** from [Adoptium](https://adoptium.net/)
2. Install to `C:\Java\jdk-17`
3. Update Environment Variables:

```
setx JAVA_HOME "C:\Java\jdk-17"
setx PATH "%JAVA_HOME%\bin;%PATH%"
```

4. Verify:

```
java -version
```

---

## **Step 4: Install Modern Tomcat**

1. Download **Tomcat 9/10** from [Tomcat Downloads](https://tomcat.apache.org/download-90.cgi)
2. Extract to `C:\Tomcat9`
3. Copy application and config files:

   * `webapps` → `C:\Tomcat9\webapps`
   * `conf\server.xml` → `C:\Tomcat9\conf\server.xml` (review changes)
   * `conf\web.xml` → `C:\Tomcat9\conf\web.xml`
     ⚠️ Update deprecated syntax
4. Copy required JARs to `C:\Tomcat9\lib`

---

## **Step 5: Update JDBC Driver**

1. Download SQL Server JDBC Driver from [Microsoft](https://learn.microsoft.com/en-us/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server)
2. Copy JAR to Tomcat lib (`C:\Tomcat9\lib`)
3. Update database connection strings in your applications if needed:

```java
Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");
String url = "jdbc:sqlserver://SERVER_NAME:1433;databaseName=DB_NAME";
Connection con = DriverManager.getConnection(url, "username", "password");
```

---

## **Step 6: Adjust Tomcat Memory Settings**

Create `setenv.bat` in `C:\Tomcat9\bin`:

```bat
set JAVA_OPTS=-Xms1024m -Xmx2048m -XX:+UseG1GC -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=C:\Tomcat9\logs\heapdump.hprof
```

---

## **Step 7: Test New Setup**

1. Start Tomcat:

```cmd
cd C:\Tomcat9\bin
catalina.bat start
```

2. Open `http://localhost:8080`
3. Verify all applications and reports function correctly

---

## **Step 8: Switch Production**

1. Stop old Tomcat service:

```cmd
net stop "Apache Tomcat 6"
```

2. Install and start new Tomcat service (optional):

```cmd
cd C:\Tomcat9\bin
service.bat install
net start "Apache Tomcat 9"
```

3. Monitor logs: `C:\Tomcat9\logs\catalina.out`

---

## **Step 9: Monitor System**

* Check logs daily for 1–2 weeks
* Verify all reports are generated correctly
* Monitor SQL Server performance and GC logs

---

## **Step 10: Archive Old Environment**

* Rename old folders:

```
C:\Tomcat6 → C:\Tomcat6_backup
C:\Java\jdk1.7.0_79 → C:\Java\jdk1.7_backup
```

* Keep backups until the new system is stable

---

## **Optional Future Upgrade**

* SQL Server 2016 → SQL Server 2019/2022
* Windows Server 2022 is up-to-da
