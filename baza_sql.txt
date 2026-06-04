ALTER TABLE dbo.stg_finance_data 
ADD userId INT IDENTITY(1,1);
INSERT INTO dbo.users (id, gender, age)
SELECT userId, gender, age 
FROM dbo.stg_finance_data;


-- tabela users
CREATE TABLE dbo.users (
	id INT PRIMARY KEY,
	gender VARCHAR(50),
	age TINYINT
);

-- tabela reasonType
CREATE TABLE dbo.reasonType (
	id INT PRIMARY KEY,
	reasonType VARCHAR(100)
);

-- tabela reasonText
CREATE TABLE dbo.reasonText (
	id INT IDENTITY(1,1) PRIMARY KEY,
	reasonTypeId INT FOREIGN KEY REFERENCES dbo.reasonType(id),
	reasonText VARCHAR(100)
);

-- tabela reasonUser
CREATE TABLE dbo.reasonUser (
	userId INT FOREIGN KEY REFERENCES dbo.users(id),
	reasonId INT FOREIGN KEY REFERENCES dbo.reasonText(id),
	PRIMARY KEY(userId, reasonId)
);


CREATE TABLE dbo.userProfileFields (
	id INT PRIMARY KEY,
	fieldName VARCHAR(100)
);

INSERT INTO dbo.userProfileFields(id, fieldName) VALUES
(1, 'Factor'),
(2, 'Objective'),
(3, 'Purpose'),
(4, 'Duration'),
(5, 'Invest Monitor'),
(6, 'Expect'),
(7, 'Avenue'),
(8, 'What are your saving objectives');

SELECT * from userProfileFields

CREATE TABLE dbo.userProfileAnswer (
	id INT IDENTITY(1,1) PRIMARY KEY,
	fieldId INT FOREIGN KEY REFERENCES userProfileFields(id),
	answer VARCHAR(100)
);

CREATE TABLE dbo.userProfile (
	userId INT FOREIGN KEY REFERENCES dbo.users(id),
	answerId INT FOREIGN KEY REFERENCES dbo.userProfileAnswer(id),
	PRIMARY KEY(userId, answerId)
)

SELECT * FROM dbo.userProfileAnswer
SELECT * FROM dbo.userProfileFields
SELECT * FROM dbo.userProfile

INSERT INTO dbo.userProfileAnswer (fieldId, answer)
SELECT DISTINCT 1, factor FROM dbo.stg_finance_data WHERE factor IS NOT NULL
UNION
SELECT DISTINCT 2, objective FROM dbo.stg_finance_data WHERE objective IS NOT NULL
UNION
SELECT DISTINCT 3, Purpose FROM dbo.stg_finance_data WHERE purpose IS NOT NULL
UNION
SELECT DISTINCT 4, duration FROM dbo.stg_finance_data WHERE duration IS NOT NULL
UNION
SELECT DISTINCT 5, investMonitor FROM dbo.stg_finance_data WHERE investMonitor IS NOT NULL
UNION
SELECT DISTINCT 6, expect FROM dbo.stg_finance_data WHERE expect IS NOT NULL
UNION
SELECT DISTINCT 7, avenue FROM dbo.stg_finance_data WHERE avenue IS NOT NULL
UNION
SELECT DISTINCT 8, savingObjectives FROM dbo.stg_finance_data WHERE savingObjectives IS NOT NULL;

SELECT * FROM dbo.userProfileAnswer

-- Factor
INSERT INTO dbo.userProfile (userId, answerId)
SELECT s.userId, a.id
FROM dbo.stg_finance_data s
JOIN dbo.userProfileAnswer a ON s.factor = a.answer AND a.fieldId = 1
WHERE s.factor IS NOT NULL;

-- Objective
INSERT INTO dbo.userProfile (userId, answerId)
SELECT s.userId, a.id
FROM dbo.stg_finance_data s
JOIN dbo.userProfileAnswer a ON s.objective = a.answer AND a.fieldId = 2
WHERE s.objective IS NOT NULL;

-- Purpose
INSERT INTO dbo.userProfile (userId, answerId)
SELECT s.userId, a.id
FROM dbo.stg_finance_data s
JOIN dbo.userProfileAnswer a ON s.purpose = a.answer AND a.fieldId = 3
WHERE s.purpose IS NOT NULL;

-- Duration
INSERT INTO dbo.userProfile (userId, answerId)
SELECT s.userId, a.id
FROM dbo.stg_finance_data s
JOIN dbo.userProfileAnswer a ON s.duration = a.answer AND a.fieldId = 4
WHERE s.duration IS NOT NULL;

-- Invest Monitor
INSERT INTO dbo.userProfile (userId, answerId)
SELECT s.userId, a.id
FROM dbo.stg_finance_data s
JOIN dbo.userProfileAnswer a ON s.investMonitor = a.answer AND a.fieldId = 5
WHERE s.investMonitor IS NOT NULL;

-- Expect
INSERT INTO dbo.userProfile (userId, answerId)
SELECT s.userId, a.id
FROM dbo.stg_finance_data s
JOIN dbo.userProfileAnswer a ON s.expect = a.answer AND a.fieldId = 6
WHERE s.expect IS NOT NULL;

-- Avenue
INSERT INTO dbo.userProfile (userId, answerId)
SELECT s.userId, a.id
FROM dbo.stg_finance_data s
JOIN dbo.userProfileAnswer a ON s.avenue = a.answer AND a.fieldId = 7
WHERE s.avenue IS NOT NULL;

-- Savings objectives
INSERT INTO dbo.userProfile (userId, answerId)
SELECT s.userId, a.id
FROM dbo.stg_finance_data s
JOIN dbo.userProfileAnswer a ON s.savingObjectives = a.answer AND a.fieldId = 8
WHERE s.savingObjectives IS NOT NULL;









SELECT 
    f.fieldName AS Pytanie,
    a.answer AS Udzielona_Odpowiedz
FROM dbo.userProfile p
JOIN dbo.userProfileAnswer a ON p.answerId = a.id
JOIN dbo.userProfileFields f ON a.fieldId = f.id
WHERE p.userId = 1
ORDER BY f.id;


-- 1. DANE ZNORMALIZOWANE (Złożone przez JOIN z 3 tabel)
SELECT 
    f.fieldName AS Pytanie,
    a.answer AS Udzielona_Odpowiedz
FROM dbo.userProfile p
JOIN dbo.userProfileAnswer a ON p.answerId = a.id
JOIN dbo.userProfileFields f ON a.fieldId = f.id
WHERE p.userId = 150
ORDER BY f.id;

-- 2. DANE SUROWE (Z oryginalnej tabeli stagingowej)
SELECT 
    Factor, Objective, Purpose, Duration, 
    investMonitor, Expect, Avenue, 
    savingObjectives
FROM dbo.stg_finance_data
WHERE UserId = 150;

INSERT INTO dbo.reasonType(id, reasonType) VALUES
(1, 'Reason Equity'),
(2, 'Reason Mutual'),
(3, 'Reason Bonds'),
(4, 'Reason FD'),
(5, 'Source');

INSERT INTO dbo.reasonText (reasonTypeId, reasonText)
SELECT DISTINCT 1, reasonEquity FROM dbo.stg_finance_data WHERE reasonEquity IS NOT NULL
UNION
SELECT DISTINCT 2, reasonMutual FROM dbo.stg_finance_data WHERE reasonMutual IS NOT NULL
UNION
SELECT DISTINCT 3, reasonBonds FROM dbo.stg_finance_data WHERE reasonBonds IS NOT NULL
UNION
SELECT DISTINCT 4, reasonFd FROM dbo.stg_finance_data WHERE reasonFD IS NOT NULL
UNION
SELECT DISTINCT 5, [source] FROM dbo.stg_finance_data WHERE [source] IS NOT NULL;


-- Reason Equity
INSERT INTO dbo.reasonUser(userId,reasonId)
SELECT s.userid, r.id
FROM dbo.stg_finance_data s
JOIN dbo.reasonText r ON s.reasonEquity=r.reasonText AND r.reasonTypeId = 1
WHERE s.reasonEquity IS NOT NULL;


-- Reason Mutual
INSERT INTO dbo.reasonUser(userId,reasonId)
SELECT s.userid, r.id
FROM dbo.stg_finance_data s
JOIN dbo.reasonText r ON s.reasonMutual=r.reasonText AND r.reasonTypeId = 2
WHERE s.reasonMutual IS NOT NULL;

-- Reason Bonds
INSERT INTO dbo.reasonUser(userId,reasonId)
SELECT s.userid, r.id
FROM dbo.stg_finance_data s
JOIN dbo.reasonText r ON s.reasonBonds=r.reasonText AND r.reasonTypeId = 3
WHERE s.reasonBonds IS NOT NULL;

-- Reasoon FD
INSERT INTO dbo.reasonUser(userId,reasonId)
SELECT s.userid, r.id
FROM dbo.stg_finance_data s
JOIN dbo.reasonText r ON s.reasonFd=r.reasonText AND r.reasonTypeId = 4
WHERE s.reasonFd IS NOT NULL;

-- Source
INSERT INTO dbo.reasonUser(userId,reasonId)
SELECT s.userid, r.id
FROM dbo.stg_finance_data s
JOIN dbo.reasonText r ON s.[source]=r.reasonText AND r.reasonTypeId = 5
WHERE s.[source] IS NOT NULL;


CREATE TABLE investmentType (
	id INT PRIMARY KEY,
	investmentType VARCHAR(100)
	);


CREATE TABLE investmentScore (
	userId INT FOREIGN KEY REFERENCES dbo.users(id),
	investmentTypeId INT FOREIGN KEY REFERENCES dbo.investmentType(id),
	score TINYINT,
	PRIMARY KEY (userId, investmentTypeId)
);

SELECT * FROM dbo.investmentScore
SELECT * FROM dbo.investmenttype


INSERT INTO dbo.investmentType (id, investmentType) VALUES
(1, 'Mutual Funds'),
(2, 'Equity Market'),
(3, 'Debentures'),
(4, 'Government Bonds'),
(5, 'Fixed Deposits'),
(6, 'PPF'),
(7, 'Gold');



-- Mutual Funds
INSERT INTO dbo.investmentScore(userId,investmentTypeId,score)
SELECT userId, 1, mutualFunds FROM dbo.stg_finance_data; 

-- Equity Market
INSERT INTO dbo.investmentScore(userId,investmentTypeId,score)
SELECT userId, 2, equityMarket FROM dbo.stg_finance_data 

-- Debentures 
INSERT INTO dbo.investmentScore (userId, investmentTypeId, score)
SELECT userId, 3, debentures 
FROM dbo.stg_finance_data;

-- Government Bonds 
INSERT INTO dbo.investmentScore (userId, investmentTypeId, score)
SELECT userId, 4, governmentBonds 
FROM dbo.stg_finance_data;

-- Fixed Deposits 
INSERT INTO dbo.investmentScore (userId, investmentTypeId, score)
SELECT userId, 5, fixedDeposits 
FROM dbo.stg_finance_data;

-- PPF 
INSERT INTO dbo.investmentScore (userId, investmentTypeId, score)
SELECT userId, 6, ppf 
FROM dbo.stg_finance_data;

-- Gold
INSERT INTO dbo.investmentScore (userId, investmentTypeId, score)
SELECT userId, 7, gold 
FROM dbo.stg_finance_data;


CREATE TABLE dbo.investmentText (
	id INT PRIMARY KEY,
	investmentText VARCHAR(100)
);

INSERT INTO dbo.InvestmentText(id,investmentText) VALUES
(1,'Investment Avenues'),
(2,'Stock Market');

SELECT * FROM dbo.userInvestmentProfile

CREATE TABLE dbo.userInvestmentProfile (
	userId INT FOREIGN KEY REFERENCES dbo.users(id),
	investmentId INT FOREIGN KEY REFERENCES dbo.investmentText(id),
	[value] BIT,
	PRIMARY KEY (userId, investmentId)
)


-- Investment Avenues
INSERT INTO dbo.userInvestmentProfile(userId, investmentId, [value]) 
SELECT userId, 1, investmentAvenues
FROM dbo.stg_finance_data;

-- Stock Market
INSERT INTO dbo.userInvestmentProfile(userId, investmentId, [value]) 
SELECT userId, 2, stockMarket
FROM dbo.stg_finance_data;



CREATE TABLE dbo.userProfileFields (
	id INT PRIMARY KEY,
	fieldName VARCHAR(100)
);

INSERT INTO dbo.userProfileFields(id, fieldName) VALUES
(1, 'Factor'),
(2, 'Objective'),
(3, 'Purpose'),
(4, 'Duration'),
(5, 'Invest Monitor'),
(6, 'Expect'),
(7, 'Avenue'),
(8, 'What are your saving objectives');

SELECT * from userProfileFields

CREATE TABLE dbo.userProfileAnswer (
	id INT IDENTITY(1,1) PRIMARY KEY,
	fieldId INT FOREIGN KEY REFERENCES userProfileFields(id),
	answer VARCHAR(100)
);

CREATE TABLE dbo.userProfile (
	userId INT FOREIGN KEY REFERENCES dbo.users(id),
	answerId INT FOREIGN KEY REFERENCES dbo.userProfileAnswer(id),
	PRIMARY KEY(userId, answerId)
)

SELECT * FROM dbo.userProfileAnswer
SELECT * FROM dbo.userProfileFields
SELECT * FROM dbo.userProfile

INSERT INTO dbo.userProfileAnswer (fieldId, answer)
SELECT DISTINCT 1, factor FROM dbo.stg_finance_data WHERE factor IS NOT NULL
UNION
SELECT DISTINCT 2, objective FROM dbo.stg_finance_data WHERE objective IS NOT NULL
UNION
SELECT DISTINCT 3, Purpose FROM dbo.stg_finance_data WHERE purpose IS NOT NULL
UNION
SELECT DISTINCT 4, duration FROM dbo.stg_finance_data WHERE duration IS NOT NULL
UNION
SELECT DISTINCT 5, investMonitor FROM dbo.stg_finance_data WHERE investMonitor IS NOT NULL
UNION
SELECT DISTINCT 6, expect FROM dbo.stg_finance_data WHERE expect IS NOT NULL
UNION
SELECT DISTINCT 7, avenue FROM dbo.stg_finance_data WHERE avenue IS NOT NULL
UNION
SELECT DISTINCT 8, savingObjectives FROM dbo.stg_finance_data WHERE savingObjectives IS NOT NULL;

SELECT * FROM dbo.userProfileAnswer

-- Factor
INSERT INTO dbo.userProfile (userId, answerId)
SELECT s.userId, a.id
FROM dbo.stg_finance_data s
JOIN dbo.userProfileAnswer a ON s.factor = a.answer AND a.fieldId = 1
WHERE s.factor IS NOT NULL;

-- Objective
INSERT INTO dbo.userProfile (userId, answerId)
SELECT s.userId, a.id
FROM dbo.stg_finance_data s
JOIN dbo.userProfileAnswer a ON s.objective = a.answer AND a.fieldId = 2
WHERE s.objective IS NOT NULL;

-- Purpose
INSERT INTO dbo.userProfile (userId, answerId)
SELECT s.userId, a.id
FROM dbo.stg_finance_data s
JOIN dbo.userProfileAnswer a ON s.purpose = a.answer AND a.fieldId = 3
WHERE s.purpose IS NOT NULL;

-- Duration
INSERT INTO dbo.userProfile (userId, answerId)
SELECT s.userId, a.id
FROM dbo.stg_finance_data s
JOIN dbo.userProfileAnswer a ON s.duration = a.answer AND a.fieldId = 4
WHERE s.duration IS NOT NULL;

-- Invest Monitor
INSERT INTO dbo.userProfile (userId, answerId)
SELECT s.userId, a.id
FROM dbo.stg_finance_data s
JOIN dbo.userProfileAnswer a ON s.investMonitor = a.answer AND a.fieldId = 5
WHERE s.investMonitor IS NOT NULL;

-- Expect
INSERT INTO dbo.userProfile (userId, answerId)
SELECT s.userId, a.id
FROM dbo.stg_finance_data s
JOIN dbo.userProfileAnswer a ON s.expect = a.answer AND a.fieldId = 6
WHERE s.expect IS NOT NULL;

-- Avenue
INSERT INTO dbo.userProfile (userId, answerId)
SELECT s.userId, a.id
FROM dbo.stg_finance_data s
JOIN dbo.userProfileAnswer a ON s.avenue = a.answer AND a.fieldId = 7
WHERE s.avenue IS NOT NULL;

-- Savings objectives
INSERT INTO dbo.userProfile (userId, answerId)
SELECT s.userId, a.id
FROM dbo.stg_finance_data s
JOIN dbo.userProfileAnswer a ON s.savingObjectives = a.answer AND a.fieldId = 8
WHERE s.savingObjectives IS NOT NULL;


SELECT * FROM dbo.userProfile

