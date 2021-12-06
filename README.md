## Mã hoá trong SQL SERVER

Mã hóa dữ liệu trong suốt (Transparent data encrytion – TDE) là một trong những cơ chế an toàn, cho phép mã hóa dữ liệu nhạy cảm được lưu trữ trong bảng và không gian bảng. Dữ liệu được mã hóa và giải mã trong suốt đối với người dùng và các ứng dụng có quyền truy cập vào dữ liệu. Để ngăn chặn việc giải mã trái phép, TDE lưu trữ các khóa mã hóa trong mô-đun an toàn bên ngoài cơ sở dữ liệu (CSDL).

### Các loại mã hóa TDE
Có 2 loại mã hoá TDE là : Mã hóa cột và mã hóa không gian bảng.
Trong khi mã hóa cột TDE cho phép mã hóa dữ liệu nhạy cảm được lưu trữ trong các cột của bảng được chọn, thì mã hóa không gian bảng TDE cho phép mã hóa tất cả dữ liệu được lưu trữ trong một vùng bảng.
Cả mã hóa cột và mã hóa không gian bảng TDE đều sử dụng kiến trúc dựa trên khóa hai tầng. Ngay cả khi dữ liệu mã hóa được truy xuất, dữ liệu cũng không được hiển thị cho đến khi quá trình giải mã được ủy quyền xảy ra, quá trình này là tự động cho người dùng được ủy quyền truy cập vào bảng.

#### Mã hóa cột TDE
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
DROP INDEX [AK_CreditCard_CardNumber] ON [Sales].[CreditCard]
GO
ALTER TABLE [Sales].[CreditCard]
DROP COLUMN CardNumber
 
 --Rename
EXEC sp_rename
'Sales.CreditCard.CardNumberEnc', 'CardNumber', 'COLUMN';  
GO
 --Check the table
SELECT * FROM [Sales].[CreditCard]

```

#### Mã hóa không gian bảng TDE
Mã hóa không gian bảng TDE cho phép mã hóa toàn bộ vùng bảng. Tất cả các đối tượng được tạo trong không gian bảng sẽ được mã hóa tự động. Mã hóa không gian bảng TDE rất hữu ích nếu trong trường hợp muốn bảo vệ dữ liệu nhạy cảm trong các bảng. Trường hợp này, không cần thực hiện phân tích chi tiết từng cột trong bảng để xác định các cột cần mã hóa.
Mã hóa không gian bảng TDE là một thay thế tốt cho mã hóa cột TDE nếu các bảng chứa dữ liệu nhạy cảm trong nhiều cột hoặc nếu muốn bảo vệ toàn bộ bảng chứ không chỉ các cột riêng lẻ
Mã hóa không gian bảng TDE cũng cho phép phạm vi chỉ mục quét dữ liệu trong không gian bảng được mã hóa. Điều này là không thể với mã hóa cột TDE.
Lưu ý, dữ liệu mã hóa được bảo vệ trong các hoạt động như JOIN và SORT. Điều này có nghĩa là dữ liệu được an toàn khi nó được di chuyển đến các vùng bảng tạm thời. Dữ liệu trong nhật ký hoàn tác và làm lại cũng được bảo vệ.
