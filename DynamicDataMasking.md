# Data masking types
```sql
CREATE DATABASE Masks
```
## Default

Mặt nạ mặc định che dấu các giá trị hoàn chỉnh trong cột chỉ định

```
Tạo table và insert giá trị
Các cột Name, BirthDate và Social_Security sẽ có default dynamic data masking.
```

```sql
USE Masks
 
DROP TABLE IF EXISTS DefaultMask;
        
CREATE TABLE DefaultMask
(
ID INT IDENTITY (1,1) PRIMARY KEY NOT NULL
,Name VARCHAR(255)	MASKED WITH (FUNCTION = 'default()') NULL
,BirthDate     DATE		MASKED WITH (FUNCTION = 'default()') NOT NULL
,Social_Security  BIGINT		MASKED WITH (FUNCTION = 'default()') NOT NULL
);
GO

INSERT INTO DefaultMask
(
Name, BirthDate, Social_Security
)
VALUES 
('James Jones',  '1998-06-01', 784562145987),
( 'Pat Rice',  '1982-08-12', 478925416938),
('George Eliot',  '1990-05-07', 794613976431);

SELECT * FROM DefaultMask
```

`tạo user mới : DefaultMaskTestUser cấp quyền select trên bảng DefaultMask`

```sql
DROP USER IF EXISTS DefaultMaskTestUser;
CREATE USER DefaultMaskTestUser WITHOUT LOGIN;
 
GRANT SELECT ON DefaultMask TO DefaultMaskTestUser;
 
EXECUTE AS USER = 'DefaultMaskTestUser';
SELECT * FROM DefaultMask;
 
REVERT;
```

## Partial

Mặt nạ mặc định ẩn mọi thứ trong cột mà nó được áp dụng. Điều gì sẽ xảy ra nếu chúng ta muốn hiển thị một phần thông tin trong cột trong khi để ẩn một số phần của nó?

Đây là lúc mặt nạ một phần có ích, chú ý là mặt nạ một phần chỉ áp dụng cho các cột kiểu string.

```
Tạo bảng và insert giá trị
Cột Name và Comment sẽ bị mask 1 phần 
Đối với cột Name thì hiển thị 2 kí tự đầu và 2 kí tự cuối, ở giữa thay bằng XXXX
Tương tự cột Comment hiển thị 5 kí tự đầu và 5 kí tự cuối, ở giữa thay bằng XXXX
```

```sql
USE Masks
 
DROP TABLE IF EXISTS PartialMask;
        
CREATE TABLE PartialMask
(
ID INT IDENTITY (1,1) PRIMARY KEY NOT NULL,
Name VARCHAR(255) MASKED WITH (FUNCTION = 'partial(2, "XXXX",2)') NULL,
Comment  NVARCHAR(255)	MASKED WITH (FUNCTION = 'partial(5, "XXXX", 5)') NOT NULL
);
GO

INSERT INTO PartialMask
(
  Name,  Comment
)
VALUES 
('James Jones',  'The tea was fantastic'),
( 'Pat Rice',  'I like these mangoes' ),
('George Eliot',  'I do not really like this');

SELECT * FROM PartialMask
```

`tạo user mới PartialMaskTestUser có quyền select trên PartialMask`

```sql
DROP USER IF EXISTS PartialMaskTestUser;
CREATE USER  PartialMaskTestUser WITHOUT LOGIN;
        
GRANT SELECT ON PartialMask TO PartialMaskTestUser;  
        
EXECUTE AS USER = 'PartialMaskTestUser';  
SELECT * FROM PartialMask
REVERT;
```
