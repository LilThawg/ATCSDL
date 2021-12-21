```sql
-- Create an audit
USE master;
GO

CREATE SERVER AUDIT Audit
   
TO FILE
(FILEPATH = 'D:\Audit'
,MAXSIZE = 24 MB
,MAX_ROLLOVER_FILES = 2147483647
,RESERVE_DISK_SPACE = OFF
)
WITH
(QUEUE_DELAY = 1000
,ON_FAILURE = CONTINUE
)
GO
ALTER SERVER AUDIT Audit
WITH (STATE = ON);
GO

-- create database
create database Audit

-- Create a database audit specification

USE [Audit];
go

CREATE DATABASE AUDIT SPECIFICATION Audit
FOR SERVER AUDIT [Audit]
ADD (SCHEMA_OBJECT_ACCESS_GROUP),
ADD (SCHEMA_OBJECT_CHANGE_GROUP),
ADD (SUCCESSFUL_DATABASE_AUTHENTICATION_GROUP),
ADD (DATABASE_CHANGE_GROUP),
ADD (DATABASE_PERMISSION_CHANGE_GROUP),
ADD (DATABASE_PRINCIPAL_CHANGE_GROUP)
WITH (STATE = ON)
GO

-- query
create table CHUYENBAY
(
	MaCB varchar(20) not null,
	GaDi varchar(20),
	GaDen varchar(20),
	DoDai int,
	GioDi time,
	GiaDen time,
	ChiPhi int
)

create table MAYBAY
(
	MaMB int not null,
	Loai varchar(30),
	TamBay int
)

create table NHANVIEN
(
	 MaNV int not null,
	 Ten varchar(30),
	 Luong int,
)

create table CHUNGNHAN
(
	MaNV int not null,
	MaMB int not null
)

alter table CHUYENBAY add constraint PK_CHUYENBAY 
primary key (MaCB)

alter table MAYBAY add constraint MB_PK
primary key (MaMB)

alter table NHANVIEN add constraint NV_PK
primary key (MaNV)

alter table CHUNGNHAN add constraint CN_FK
foreign key (MaNV) references NHANVIEN (MaNV)

alter table CHUNGNHAN add constraint CN_MaMB_FK
foreign key (MaMB) references MAYBAY (MaMB)

alter table ChungNhan add constraint CN_PK
primary key (MaMB,MaNV)


insert into NHANVIEN (MaNV,Ten,Luong) values
(242518965,'Tran Van Son',120433)

insert into CHUYENBAY(MaCB,GaDi,GaDen,DoDai,GioDi,GiaDen,ChiPhi) values
('VN431','SGN','CAH',3693,'05:55','06:55',236)

insert into MAYBAY (MaMB,Loai,TamBay) values
(747,'Boeing 747 - 400',13488)

drop table CHUNGNHAN

drop table CHUYENBAY

CREATE LOGIN testLogin WITH PASSWORD = 'Welcome123'
USE Audit
Create USER testLogin FOR LOGIN testLogin
EXEC sp_addrolemember 'db_datareader', 'testLogin'
GRANT CREATE TABLE TO testLogin
GRANT INSERT TO testLogin
GRANT UPDATE  TO testLogin
DROP USER testLogin
DROP LOGIN testLogin


select * from sys.fn_get_audit_file(
	'D:\Audit\*.sqlaudit', default, default
)
go
```
