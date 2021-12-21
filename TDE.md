#### Mã hóa cột
Mã hóa cột TDE cho phép mã hoá từng cột riêng lẻ được sử dụng để bảo vệ dữ liệu nhạy cảm, chẳng hạn như thẻ tín dụng và số an sinh xã hội, được lưu trữ trong các cột của bảng.

Giống như mã hoá dữ liệu thông thường, trong SQL server cũng có 2 cách để mã hoá dữ liệu :
1. Mã hoá và dữ liệu vẫn có thể giải ngược lại được (encrypt and keep the data decryptable) : có nghĩa là sử dụng khoá (nói chung cả khoá bí mật và khoá công khai) để mã hoá và giải mã.
2. Mã hoá và dữ liệu không thể giải được (encrypt and make the data non-decryptable) : có nghĩa là mã hoá một chiều bằng hàm băm, không thể giải ngược lại được.

**Use-Case 1 – encrypt and keep the data decryptable**
``` sql
USE
AdventureWorks2014;
GO

SELECT * FROM [Sales].[CreditCard]

/*
vì cột CardNumber mà để là bản rõ như này thì rất không an toàn
do vậy ta hãy mã hoá nó
  
đối với trường hợp 1, ta cần làm một số hành động này trước tiên:
-Create and backup Master Key (tạo và sao lưu khoá chính)
-Create and backup Certificate (Tạo và sao lưu chứng thư)
-Create a Symmetric Key (tạo khoá đối xứng)
*/

--1) Create Master key:
CREATE MASTER KEY
ENCRYPTION BY PASSWORD = '$trongPa$$word'; --choose a strong password and keep it in a safe place!
GO

--Check the master key has been created:
SELECT * FROM sys.symmetric_keys
GO

--2) Backup Master Key:
BACKUP MASTER KEY TO FILE = 'D:\EncryptionBackups\AW2014MasterKeyBackup'
ENCRYPTION BY PASSWORD = '$trongPa$$word';  
GO 

--3) Create certificate
CREATE CERTIFICATE AW2014Certificate
WITH SUBJECT = 'CreditCard_Encryption',
EXPIRY_DATE = '20991231';
GO

--4) Backup certificate
BACKUP CERTIFICATE AW2014Certificate TO FILE = 'D:\EncryptionBackups\AW2014CertificateBackup'   
GO  

--5) Create symmetric key
CREATE SYMMETRIC KEY AW2014SymKey
WITH ALGORITHM = AES_256
ENCRYPTION BY CERTIFICATE AW2014Certificate;
GO

--Ok vậy là ta đã setup xong
  
--============
--ENCRYPTION--
--============

--Add a varbinary datatype encrypted column
ALTER TABLE [Sales].[CreditCard]
ADD CardNumberEnc VARBINARY(250) NULL
GO
--Check the table
SELECT * FROM [Sales].[CreditCard]
GO

--Open the symmetric key
OPEN SYMMETRIC KEY AW2014SymKey
DECRYPTION BY CERTIFICATE AW2014Certificate;

--Encrypt existing data
UPDATE [Sales].[CreditCard]
SET CardNumberEnc = EncryptByKey(Key_GUID('AW2014SymKey'), CardNumber, 1, CONVERT(VARBINARY, CreditCardID))
--Check the table
SELECT * FROM [Sales].[CreditCard]
GO

--Make the new column Non-Nullable
ALTER TABLE [Sales].[CreditCard]
ALTER COLUMN CardNumberEnc VARBINARY(250) NOT NULL
GO
--Check the table
SELECT * FROM [Sales].[CreditCard]
GO

--Drop old column
DROP INDEX [AK_CreditCard_CardNumber] ON [Sales].[CreditCard]
GO
ALTER TABLE [Sales].[CreditCard]
DROP COLUMN CardNumber
--Check the table
SELECT * FROM [Sales].[CreditCard]
GO

--Rename
EXEC sp_rename
'Sales.CreditCard.CardNumberEnc', 'CardNumber', 'COLUMN';  
GO
--Check the table
SELECT * FROM [Sales].[CreditCard]
GO

--Close the symmetric key
CLOSE SYMMETRIC KEY AW2014SymKey;
GO

--=========================================================

--============
--DECRYPTION--
--============

--Add a nvarchar datatype decrypted column
ALTER TABLE [Sales].[CreditCard]
ADD CardNumberDec NVARCHAR(250) NULL
GO
--Check the table
SELECT * FROM [Sales].[CreditCard]
GO

--Open the symmetric key
OPEN SYMMETRIC KEY AW2014SymKey
DECRYPTION BY CERTIFICATE AW2014Certificate;

--Decrypt existing data
UPDATE [Sales].[CreditCard]
SET CardNumberDec = DecryptByKey(CreditCard.[CardNumber], 1, CONVERT(varbinary, CreditCardID))
--Check the table
SELECT * FROM [Sales].[CreditCard]
GO

--Make the new column Non-Nullable
ALTER TABLE [Sales].[CreditCard]
ALTER COLUMN CardNumberDec NVARCHAR(250) NOT NULL
GO
--Check the table
SELECT * FROM [Sales].[CreditCard]
GO

--Drop old column
ALTER TABLE [Sales].[CreditCard]
DROP COLUMN CardNumber
--Check the table
SELECT * FROM [Sales].[CreditCard]
GO

--Rename 
EXEC sp_rename
'Sales.CreditCard.CardNumberDec', 'CardNumber', 'COLUMN';  
GO
--Check the table
SELECT * FROM [Sales].[CreditCard]
GO

CLOSE SYMMETRIC KEY AW2014SymKey;
GO
 
--=========================================================
```

**Use-Case 2 – encrypt and make the data non-decryptable**
``` sql
USE
AdventureWorks2014;
GO
 
--Add a varbinary datatype encrypted column
ALTER TABLE [Sales].[CreditCard]
ADD CardNumberEnc VARBINARY(250) NULL
GO
--Check the table
SELECT * FROM [Sales].[CreditCard]
 
--Encrypt existing data
UPDATE [Sales].[CreditCard]
SET CardNumberEnc = HASHBYTES('SHA2_256', CardNumber) --SHA2_256 is the encryption algorithm
--Check the table
SELECT * FROM [Sales].[CreditCard]
 
--Make the new column Non-Nullable
ALTER TABLE [Sales].[CreditCard]
ALTER COLUMN CardNumberEnc VARBINARY(250) NOT NULL
GO
--Check the table
SELECT * FROM [Sales].[CreditCard]
 
--Drop old column

ALTER TABLE [Sales].[CreditCard]
DROP COLUMN CardNumber
 
 --Rename
EXEC sp_rename
'Sales.CreditCard.CardNumberEnc', 'CardNumber', 'COLUMN';  
GO
 --Check the table
SELECT * FROM [Sales].[CreditCard]

```



