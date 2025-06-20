# 📘 Oracle DBA Guide
> รวมคำสั่งและแนวทางปฏิบัติสำคัญ พร้อมคำอธิบายและตัวอย่าง

---

## 🧭 สารบัญ
1. [การเชื่อมต่อ Oracle](#1-การเชื่อมต่อ-oracle)
2. [User และ Privilege](#2-user-และ-privilege)
3. [การจัดการ Tablespace](#3-การจัดการ-tablespace)
4. [Backup & Recovery](#4-backup--recovery)
5. [ตรวจสอบสถานะ DB](#5-ตรวจสอบสถานะ-db)
6. [Performance Tuning](#6-performance-tuning)
7. [Redo, Archive, Logs](#7-redo-archive-logs)
8. [Jobs และ Scheduler](#8-jobs-และ-scheduler)
9. [SQL Scripts สำคัญ](#9-sql-scripts-สำคัญ)
10. [Data Dictionary Views](#10-data-dictionary-views)
11. [Procedure, Function, Trigger](#11-procedure-function-trigger)

---

## 1. การเชื่อมต่อ Oracle
> ใช้สำหรับเข้าสู่ระบบฐานข้อมูล Oracle ด้วยคำสั่ง `sqlplus`
> ใช้สำหรับเข้าสู่ระบบฐานข้อมูล Oracle

```bash
sqlplus sys@ORCL as sysdba
sqlplus username/password@SID
```

---

## 2. User และ Privilege
> ใช้สำหรับสร้างและจัดการผู้ใช้งาน รวมถึงกำหนดสิทธิ์ต่าง ๆ เช่น การเข้าถึงข้อมูล การจัดการ object
> สร้าง/จัดการผู้ใช้ และการให้สิทธิ์การเข้าถึง

### สร้างผู้ใช้ใหม่
```sql
CREATE USER takoyaki IDENTIFIED BY 1234 DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp;
```
### ให้สิทธิ์
```sql
GRANT CONNECT, RESOURCE TO takoyaki;
GRANT DBA TO takoyaki;
```
### เปลี่ยนรหัส / ล็อค / ปลดล็อก
```sql
ALTER USER takoyaki IDENTIFIED BY newpass;
ALTER USER takoyaki ACCOUNT LOCK;
ALTER USER takoyaki ACCOUNT UNLOCK;
```

---

## 3. การจัดการ Tablespace
> ใช้สำหรับจัดสรรและบริหารพื้นที่จัดเก็บข้อมูลในฐานข้อมูล (Tablespace)
> ใช้สำหรับจัดสรรพื้นที่เก็บข้อมูล

```sql
CREATE TABLESPACE app_data 
DATAFILE '/u01/oradata/ORCL/app_data01.dbf' SIZE 100M AUTOEXTEND ON NEXT 10M MAXSIZE 1G;
```

ตรวจสอบ:
```sql
SELECT TABLESPACE_NAME, STATUS FROM DBA_TABLESPACES;
```

ขยาย:
```sql
ALTER DATABASE DATAFILE '/u01/oradata/ORCL/app_data01.dbf' RESIZE 500M;
```

---

## 4. Backup & Recovery
> สำรองและกู้คืนข้อมูลเพื่อความปลอดภัยของข้อมูล โดยใช้ทั้ง Cold Backup และ RMAN
> สำรองและกู้คืนข้อมูล (Cold Backup, RMAN)

```bash
# Cold backup
SHUTDOWN IMMEDIATE
cp /u01/oradata /backup

# RMAN
rman target /
BACKUP DATABASE PLUS ARCHIVELOG;
```

---

## 5. ตรวจสอบสถานะ DB
> ตรวจสอบสถานะของ Instance และ Database เช่น เปิดใช้งานหรือไม่ อยู่ในโหมดใด
```sql
SELECT INSTANCE_NAME, STATUS FROM V$INSTANCE;
SELECT DATABASE_NAME, OPEN_MODE FROM V$DATABASE;
```

---

## 6. Performance Tuning
> ตรวจสอบปัญหาด้านประสิทธิภาพ เช่น Session ที่ช้า หรือ SQL ที่ใช้เวลานาน
> ตรวจสอบ Session, SQL ช้า, Lock

```sql
SELECT * FROM V$SESSION;
SELECT * FROM V$LOCK WHERE BLOCK > 0;
SELECT * FROM V$SQLAREA WHERE ELAPSED_TIME > 10000000;
```

---

## 7. Redo, Archive, Logs
> ตรวจสอบไฟล์ Redo log, archive log ที่ใช้ในการกู้คืนข้อมูลเมื่อเกิดความเสียหาย
```sql
ARCHIVE LOG LIST;
SELECT GROUP#, STATUS FROM V$LOG;
```

---

## 8. Jobs และ Scheduler
> ใช้สำหรับสร้างงานที่ทำอัตโนมัติตามเวลาที่กำหนด เช่น backup ทุกวัน
> ใช้ตั้งเวลาให้ DB ทำงานอัตโนมัติ

```sql
BEGIN
  DBMS_SCHEDULER.CREATE_JOB (
    job_name        => 'job_backup',
    job_type        => 'PLSQL_BLOCK',
    job_action      => 'BEGIN backup_proc(); END;',
    start_date      => SYSTIMESTAMP,
    repeat_interval => 'FREQ=DAILY; BYHOUR=2',
    enabled         => TRUE);
END;
/
```

---

## 9. SQL Scripts สำคัญ
> สคริปต์ SQL ที่ DBA ใช้บ่อย เช่น ตรวจสอบพื้นที่, ลบ session ที่ค้าง
```sql
-- พื้นที่ใช้จริงของ Tablespace
SELECT TABLESPACE_NAME, SUM(BYTES)/1024/1024 AS MB_USED FROM DBA_DATA_FILES GROUP BY TABLESPACE_NAME;

-- Kill session
ALTER SYSTEM KILL SESSION 'sid,serial#' IMMEDIATE;
```

---

## 10. Data Dictionary Views
> วิวพิเศษที่ใช้ตรวจสอบ metadata ของระบบ เช่น ผู้ใช้ ตาราง session object ต่าง ๆ

| View | อธิบาย |
|------|--------|
| DBA_USERS | ข้อมูลผู้ใช้ทั้งหมด |
| DBA_TABLESPACES | รายชื่อ Tablespace |
| V$SESSION | session ที่ใช้งาน |
| ALL_OBJECTS | object ที่มีทั้งหมด |
| USER_SOURCE | source code ของโปรแกรม PL/SQL |

---

## 11. Procedure, Function, Trigger
> องค์ประกอบ PL/SQL ที่ใช้สร้างกระบวนการทางธุรกิจหรือกระตุ้นการทำงานอัตโนมัติในฐานข้อมูล

### 🧪 Stored Procedure
> กลุ่มคำสั่ง SQL ที่เก็บไว้ในระบบ ใช้เรียกใช้งานซ้ำได้
> กลุ่มคำสั่ง SQL ที่รันได้ซ้ำ

```sql
CREATE OR REPLACE PROCEDURE greet IS
BEGIN
  DBMS_OUTPUT.PUT_LINE('Hello Oracle!');
END;
/
```

### 🧩 Function
> เหมือน Procedure แต่จะคืนค่าผลลัพธ์กลับมา (เช่นใช้ใน SELECT)
> คืนค่าผลลัพธ์จาก SQL

```sql
CREATE OR REPLACE FUNCTION get_tax(p_salary NUMBER) RETURN NUMBER IS
BEGIN
  RETURN p_salary * 0.07;
END;
/
```

```sql
SELECT get_tax(10000) FROM DUAL;
```

### ⚡ Trigger
> ทำงานอัตโนมัติเมื่อเกิดเหตุการณ์บางอย่าง เช่น insert/update/delete บนตาราง
> รันอัตโนมัติเมื่อมีการเปลี่ยนแปลงข้อมูล

```sql
CREATE OR REPLACE TRIGGER trg_before_insert_emp
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
  :NEW.created_date := SYSDATE;
END;
/
```

---
