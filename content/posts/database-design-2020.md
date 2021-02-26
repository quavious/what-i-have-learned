+++
title = "데이터베이스 설계 강의 기록 2020-11-17"
date = "2021-02-26T19:12:24+09:00"
author = "Neon.Owl"
authorTwitter = "" #do not include @
cover = ""
tags = ["Database", "Design", "데이터베이스", "설계"]
keywords = ["Database", "Design", "데이터베이스", "설계"]
description = ""
showFullContent = false
+++

# DB Design

1. dbpl
1. transacted sql

PostgreSQL, MariaDB

### Web Server Development with Database

Embedded SQL - C Hard Coding  
ODBC\*, JDBC - Open Database Connectivity - Standard API, Multi DBMS available  
(Data Source)  
object call style - .NET, Object Linking DB

Web Server with different data types

Application must handle the processes of multi database  
Data Source, Program, Driver Manager, DBMS Driver  
DBMS Driver - single type, multi type  
DBMS Driver sends query to DBMS to execute SQL to DB  
ODBC - conformance levels affects features of API  
Core, Level 1(Retrieve lots of information), Level 2(Process a scrollable cursor)

SQL Grammer  
minimum - Simple expressions  
Core - command alter, create index and view, Aggregate functions  
Extended - command update, delete, scalar functions, Stored Procedures

.NET Framework

OLE DB  
Microsoft OLE object standard. COM objects, OOP  
Data producer and consumer  
interface, the set of objects and the properies and methods, exposed  
implementation how an objects accomplishes its tasks
rowser, same with cursor  
Data Provider

## ADO.NET

REST API(interface) - Web Applications - ADO.NET - DBMS - DB

Dataset is in-memory database such as Redis  
fast speed due to stored in memory  
Datasets managed by different DBMS
