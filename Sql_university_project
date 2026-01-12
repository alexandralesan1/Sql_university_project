--2.1
-- Crearea bazei de date "ClinicaVeterinara"


USE master;
GO

IF EXISTS (SELECT name FROM sys.databases WHERE name = N'ClinicaVeterinara')
BEGIN
    ALTER DATABASE ClinicaVeterinara SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
    DROP DATABASE ClinicaVeterinara;
END
GO

CREATE DATABASE ClinicaVeterinara;
GO

USE ClinicaVeterinara;
GO


-- 1. Să se realizeze relația proprietari
CREATE TABLE proprietari (
    IdProprietar INT IDENTITY(1,1),
    IDNP CHAR(13) NOT NULL,
-- 2.3.2 
-- Să se creeze  integritate de domeniu (NOT NULL) pentru atributul Nume din relația proprietari
    Nume NVARCHAR(60) NOT NULL,
    Telefon CHAR(12) NOT NULL,
    Adresa VARCHAR(100) NOT NULL
);
GO

-- 2.3.1 
-- Să se creeze cheie primară pentru relația proprietari
ALTER TABLE proprietari
ADD CONSTRAINT CheiePrimara_proprietari PRIMARY KEY (IdProprietar);

-- 2.3.3 - Integritate de domeniu (UNIQUE)
-- Să se realizeze integritate de domeniu (UNIQUE) pentru atributele IDNP și Telefon din relația proprietari
ALTER TABLE proprietari
ADD CONSTRAINT UQ_proprietari_IDNP UNIQUE (IDNP);

ALTER TABLE proprietari
ADD CONSTRAINT UQ_proprietari_Telefon UNIQUE (Telefon);


-- 2. Să se realizeze relația pacienti
CREATE TABLE pacienti (
    IdPacient INT IDENTITY(1,1) PRIMARY KEY,
    NumeAnimal NVARCHAR(30) NOT NULL,
    DataNasterii DATE NOT NULL,
    Specie NVARCHAR(50) NOT NULL,
    Rasa NVARCHAR(50),
    ArePasaport BIT NOT NULL,
    IdProprietar INT NOT NULL,
    DataCurenta DATE DEFAULT GETDATE() NOT NULL,
    VarstaAnimal AS DATEDIFF(MONTH, DataNasterii, DataCurenta) PERSISTED,
    
);
GO

-- 2.3.4 
-- Să se realizeze verificarea datei nașterii să fie <= data curentă prin  integritate de domeniu (CHECK) pentru atributul DataNasterii din relația pacienti
ALTER TABLE pacienti
ADD CONSTRAINT CHK_DataNasterii
CHECK (DataNasterii <= GETDATE());
GO

-- 2.3.5 
-- Să se realizeze valoare implicită pentru ArePasaport utilizând integritatea structurală (DEFAULT) pentru atributul ArePasaport din relația pacienti
ALTER TABLE pacienti
ADD CONSTRAINT ArePasaport_constrangere DEFAULT 0 FOR ArePasaport;
 GO

-- 2.3.6 
-- Să se realizeze cheia compusă care este o constrângere de tuplu pentru relația pacienti
ALTER TABLE pacienti
ADD CONSTRAINT Constrangere1 UNIQUE (NumeAnimal, IdProprietar)


-- 3. Să se realizeze relația cabinete

CREATE TABLE cabinete (
    IdCabinet TINYINT IDENTITY(1,1),
    NumeCabinet NVARCHAR(100) NOT NULL,
    NumarCabinet TINYINT NOT NULL,
    NumarEtaj TINYINT NOT NULL,
    UstensileDisponibile NVARCHAR(200) NOT NULL
);
GO

-- 2.3.7 
-- Să se realizeze cheia primară pentru relația cabinet
ALTER TABLE cabinete
ADD CONSTRAINT PK_Cabinete PRIMARY KEY (IdCabinet);
GO

-- 2.3.8 
-- Să se realizeze integritate de domeniu (CHECK) pentru atributul NumarCabinet din relația cabinete
ALTER TABLE cabinete
ADD CONSTRAINT CHK_NumarCabinet CHECK (NumarCabinet BETWEEN 1 AND 255);
GO

-- 2.3.9 
-- Să se realizeze integritate de domeniu (CHECK) pentru atributul NumarEtaj din relația cabinete
ALTER TABLE cabinete
ADD CONSTRAINT CHK_NumarEtaj CHECK (NumarEtaj BETWEEN 1 AND 2);
GO


-- 4. Să se realizeze relația veterinari

CREATE TABLE veterinari (
    IdVeterinar SMALLINT IDENTITY(1,1),
    IDNP CHAR(13) NOT NULL,
    Nume NVARCHAR(100) NOT NULL,
    Specializare NVARCHAR(100) NOT NULL,
    Telefon CHAR(12) NOT NULL,
    IdCabinet TINYINT NOT NULL
);
GO

-- 2.3.10 
-- Să se realizeze cheia primară pentru relația veterinari
ALTER TABLE veterinari
ADD CONSTRAINT PK_Veterinari PRIMARY KEY (IdVeterinar);
GO

-- 2.3.11
-- Să se realizeze integritate structurală (UNIQUE) pentru atributul IdCabinet din relația veterinari
ALTER TABLE veterinari
ADD CONSTRAINT UQ_Veterinari_IdCabinet UNIQUE (IdCabinet);
GO


-- 5. Să se realizeze relația asistenti
CREATE TABLE asistenti (
    IdAsistent SMALLINT IDENTITY(1,1) PRIMARY KEY,
    IDNP CHAR(13) NOT NULL UNIQUE,
    Nume NVARCHAR(100) NOT NULL,
    Functie NVARCHAR(50) NOT NULL,
    Telefon CHAR(12) NOT NULL,
    EstePracticant BIT NOT NULL
);
GO

-- 2.3.12 
-- Să se realizeze integritatea structurală (DEFAULT) pentru atributul EstePracticant din relația asistenti
ALTER TABLE asistenti
ADD CONSTRAINT DF_Asistenti_EstePracticant DEFAULT 0 FOR EstePracticant;
GO



-- 6. Să se realizeze relația intermediară [veterinari-asistenti] 

CREATE TABLE [veterinari-asistenti] (
    IdVeterinar SMALLINT NOT NULL,
    IdAsistent SMALLINT NOT NULL
);
GO

-- 2.3.13 
-- Să se realizeze cheia primară compusă pentru relația intermediară veterinari-asistenti compusă din două chei externe
ALTER TABLE [veterinari-asistenti]
ADD CONSTRAINT CheiePrimaraVeterinariAsistenti PRIMARY KEY (IdVeterinar, IdAsistent);
GO



-- Trigger pentru limitarea veterinarilor la maxim 2 asistenti practicanti

CREATE TRIGGER LimitaPracticanti
ON [veterinari-asistenti]
AFTER INSERT, UPDATE
AS
BEGIN
    IF EXISTS (
        SELECT va.IdVeterinar
        FROM [veterinari-asistenti] AS va
        JOIN asistenti AS a ON va.IdAsistent = a.IdAsistent
        JOIN inserted i ON i.IdVeterinar = va.IdVeterinar
        WHERE a.EstePracticant = 1
        GROUP BY va.IdVeterinar
        HAVING COUNT(*) > 2
    )
    BEGIN
        RAISERROR('Un veterinar nu poate avea mai mult de 2 asistenti practicanti.', 16, 1);
        ROLLBACK TRANSACTION;
    END
END;
GO


-- 7. Să se realizeze relația consultatii 

CREATE TABLE consultatii (
    IdConsultatie INT IDENTITY(1,1) PRIMARY KEY,
    DataConsultatiei DATETIME DEFAULT GETDATE() NOT NULL,
    IdPacient INT NOT NULL,
    IdVeterinar SMALLINT NOT NULL,
    EsteProgramat BIT DEFAULT 0 NOT NULL
);
GO

-- 2.3.14 
-- Nu se pot repeta consultații pentru același pacient și veterinar în aceeași zi
-- Să se realizeze integritatea de tuplu pentru relația consultatii
ALTER TABLE consultatii
ADD CONSTRAINT Constrangere2 UNIQUE (IdPacient, IdVeterinar, DataConsultatiei);
GO



-- Trigger pentru eliminarea consultatiilor vechi neprogramate

CREATE TRIGGER EliminaConsultatiiNeprogramateVechi
ON consultatii
AFTER INSERT, UPDATE
AS
BEGIN
    DELETE FROM consultatii
    WHERE DataConsultatiei < DATEADD(YEAR, -2, GETDATE())
      AND EsteProgramat = 0; 
END;
GO


-- Trigger pentru eliminarea consultatiilor vechi programate

CREATE TRIGGER EliminaConsultatiiProgramateVechi
ON consultatii
AFTER INSERT, UPDATE
AS
BEGIN
    DELETE FROM consultatii
    WHERE DataConsultatiei < DATEADD(YEAR, -2, GETDATE())
      AND EsteProgramat = 1;  -- elimină doar consultațiile vechi care erau programate
END;
GO



-- 8. Să se realizeze relația medicamente si vaccinari

CREATE TABLE medicamente_si_vaccinari (
    IdMedicament INT IDENTITY(1,1) PRIMARY KEY,
    IdConsultatie INT NOT NULL,
    TipAdministrare NVARCHAR(20) NOT NULL CHECK (TipAdministrare IN ('Medicament', 'Vaccin')),
    DenumireMedicament NVARCHAR(100) NOT NULL,
    Doza NVARCHAR(50) NOT NULL,
    DurataZile INT NOT NULL CHECK (DurataZile > 0),
    Cost DECIMAL(6,2) NOT NULL CHECK (Cost >= 0),
    Descriere NVARCHAR(MAX) NOT NULL
);
GO

-- 2.3.15 - Integritate de tuplu
-- Nu se poate repeta același medicament sau vaccin pentru aceeași consultație
-- Să se realizeze integritatea de tuplu pentru relația medicamente_si_vaccinari
ALTER TABLE medicamente_si_vaccinari
ADD CONSTRAINT Constrangere3 UNIQUE (IdConsultatie, DenumireMedicament, TipAdministrare);
GO


-- Trigger pentru eliminarea medicamentelor/vaccinurilor asociate consultatiilor vechi

CREATE TRIGGER EliminaMedicamenteVaccinuriVechi
ON medicamente_si_vaccinari
AFTER INSERT, UPDATE
AS
BEGIN
    DELETE mv
    FROM medicamente_si_vaccinari mv
    JOIN consultatii c ON mv.IdConsultatie = c.IdConsultatie
    WHERE c.DataConsultatiei < DATEADD(YEAR, -2, GETDATE())
      AND (mv.TipAdministrare = 'Medicament' OR mv.TipAdministrare = 'Vaccin');
END;
GO


-- 2.4
-- 2.4.1
-- Adaugarea cheii externe prin ALTER TABLE pentru relatia pacienti
ALTER TABLE pacienti
ADD CONSTRAINT CheieExternaPacientiProprietari FOREIGN KEY (IdProprietar)
REFERENCES proprietari(IdProprietar)
ON DELETE CASCADE ON UPDATE CASCADE;
GO

-- 2.4.2
-- Adaugarea cheii externe prin ALTER TABLE pentru relatia veterinari
ALTER TABLE veterinari
ADD CONSTRAINT CheieExternaVeterinariCabinete FOREIGN KEY (IdCabinet)
REFERENCES cabinete(IdCabinet)
ON DELETE CASCADE ON UPDATE CASCADE;
GO

-- 2.4.2
-- Adaugarea cheii externe prin ALTER TABLE pentru relatia intermediară veterinari-asistenti
ALTER TABLE [veterinari-asistenti]
ADD CONSTRAINT CheieExternaVeterinari FOREIGN KEY (IdVeterinar)
REFERENCES veterinari(IdVeterinar)
ON DELETE CASCADE ON UPDATE CASCADE;
GO

-- 2.4.3
-- Adaugarea cheii externe prin ALTER TABLE pentru relatia intermediară veterinari-asistenti
ALTER TABLE [veterinari-asistenti]
ADD CONSTRAINT CheieExternaAsistenti FOREIGN KEY (IdAsistent)
REFERENCES asistenti(IdAsistent)
ON DELETE CASCADE ON UPDATE CASCADE;
GO

-- 2.4.4
-- Adaugarea cheii externe prin ALTER TABLE pentru relatia consultatii
ALTER TABLE consultatii
ADD CONSTRAINT CheieExternaConsultatiiPacienti FOREIGN KEY (IdPacient)
REFERENCES pacienti(IdPacient)
ON DELETE CASCADE ON UPDATE CASCADE;
GO

-- 2.4.5
-- Adaugarea cheii externe prin ALTER TABLE pentru relatia consultatii
ALTER TABLE consultatii
ADD CONSTRAINT CheieExternaConsultatiiVeterinari FOREIGN KEY (IdVeterinar)
REFERENCES veterinari(IdVeterinar)
ON DELETE CASCADE ON UPDATE CASCADE;
GO

-- 2.4.6
-- Adaugarea cheii externe prin ALTER TABLE pentru relatia medicamente_si_vaccinari
ALTER TABLE medicamente_si_vaccinari
ADD CONSTRAINT CheieExternaMedicamenteConsultatii FOREIGN KEY (IdConsultatie)
REFERENCES consultatii(IdConsultatie)
ON DELETE CASCADE ON UPDATE CASCADE;
GO



-- 2.5
--2.5.1
-- Să se însereze tupluri în relația proprietari
INSERT INTO proprietari (IDNP, Nume, Telefon, Adresa)
VALUES 
('1980501123456','Popescu Ion','+37368123456','Str. Florilor 10'),
('1971205123457','Ionescu Maria','+37369111222','Str. Stefan 5'),
('1980304123458','Vlad Adrian','+37368222333','Str. Mihai Viteazul 12'),
('1990401123459','Radu Elena','+37369123478','Str. Al. Ioan 7'),
('1980715123460','Ciobanu Andrei','+37369233456','Str. Traian 15'),
('1990310123461','Marin Ana','+37360111234','Str. Bucuresti 22'),
('1980825123462','Dumitru Victor','+37360222345','Str. Independentei 5'),
('1990123123463','Petre Gabriela','+37360333456','Str. Mihai Eminescu 3'),
('1980607123464','Stefan Ioana','+37360444567','Str. Stefan cel Mare 9'),
('1990115123465','Moraru Ion','+37360555678','Str. Unirii 11'),
('1980925123466','Gherman Oana','+37360666789','Str. Tineretului 8'),
('1990222123467','Toma Vlad','+37360777890','Str. Libertatii 14'),
('1980512123468','Neagu Roxana','+37360888901','Str. Florilor 21'),
('1990412123469','Florea Denis','+37360999012','Str. Stefan 19'),
('1980125123470','Costea Ana','+37360111223','Str. Eminescu 16');
GO

--2.5.1
-- Selecarea tuturor tuplurilor din relația proprietari
SELECT * FROM proprietari;


--2.5.2
-- Să se însereze tupluri în relația cabinete
INSERT INTO cabinete (NumeCabinet, NumarCabinet, NumarEtaj, UstensileDisponibile)
VALUES
('Cabinet Oftalmologie',1,1,'Oftalmoscop, Lame, Solutii'),
('Cabinet Chirurgie',2,1,'Scalpel, Forceps, Monitoare'),
('Cabinet Cardiologie',3,1,'ECG, Stetoscoape, Medicamente'),
('Cabinet Dermatologie',4,1,'Lampă UV, Creme, Solutii'),
('Cabinet Oncologie',5,1,'Citometru, Scalpels, Analize'),
('Cabinet Endocrinologie',6,1,'Analize hormonale, Echipamente'),
('Cabinet Ortopedie',7,1,'Ghipsuri, Radiografie, Laser'),
('Cabinet Stomatologie',8,2,'Dentist, Radiografie dentara, Lame'),
('Cabinet Nutritie',9,2,'Analize, Suplimente, Tabel'),
('Cabinet Neurologie',10,2,'EEG, Analize, Medicamente'),
('Cabinet Gastroenterologie',11,2,'Endoscop, Solutii, Lame'),
('Cabinet Vaccinari',12,2,'Vaccinuri, Seringi, Dezinfectant'),
('Cabinet Urgente',13,2,'Defibrilator, Medicamente, Truse'),
('Cabinet Preventie',14,2,'Consultatii, Analize, Suplimente'),
('Cabinet Boli Infectioase',15,2,'Izolare, Antibiotice, Echipament');
GO

--2.5.2
-- Selecarea tuturor tuplurilor din relația cabinete
SELECT * FROM cabinete;


--2.5.3
-- Să se însereze tupluri în relația veterinari
INSERT INTO veterinari (IDNP, Nume, Specializare, Telefon, IdCabinet)
VALUES
('1980501123456','Dr. Andrei Petrescu','Oftalmologie','+37369123456',1),
('1971205123457','Dr. Elena Rusu','Chirurgie','+37369234567',2),
('1980304123458','Dr. Vlad Ionescu','Cardiologie','+37369345678',3),
('1990401123459','Dr. Radu Elena','Dermatologie','+37369456789',4),
('1980715123460','Dr. Ciobanu Andrei','Oncologie','+37369567890',5),
('1990310123461','Dr. Marin Ana','Endocrinologie','+37369678901',6),
('1980825123462','Dr. Dumitru Victor','Ortopedie','+37369789012',7),
('1990123123463','Dr. Petre Gabriela','Stomatologie','+37369890123',8),
('1980607123464','Dr. Stefan Ioana','Nutritie','+37369901234',9),
('1990115123465','Dr. Moraru Ion','Neurologie','+37360012345',10),
('1980925123466','Dr. Gherman Oana','Gastroenterologie','+37360123456',11),
('1990222123467','Dr. Toma Vlad','Vaccinari','+37360234567',12),
('1980512123468','Dr. Neagu Roxana','Urgente','+37360345678',13),
('1990412123469','Dr. Florea Denis','Preventie','+37360456789',14),
('1980125123470','Dr. Costea Ana','Boli Infectioase','+37360567890',15);
GO

--2.5.3
-- Selecarea tuturor tuplurilor din relația veterinari
SELECT * FROM veterinari;


--2.5.4
-- Să se însereze tupluri în relația asistenti
INSERT INTO asistenti (IDNP, Nume, Functie, Telefon, EstePracticant)
VALUES
('1980501123456','Vasile Ana','Asistent principal','+37360123456',0),
('1971205123457','Moraru Ion','Practicant','+37360234567',1),
('1980304123458','Gherman Oana','Practicant','+37360345678',1),
('1990401123459','Popa Maria','Asistent principal','+37360456789',0),
('1980715123460','Istrate Alex','Practicant','+37360567890',1),
('1990310123461','Ciobanu Ioana','Asistent principal','+37360678901',0),
('1980825123462','Dumitru Vlad','Practicant','+37360789012',1),
('1990123123463','Petre Gabriela','Asistent principal','+37360890123',0),
('1980607123464','Stefan Adrian','Practicant','+37360901234',1),
('1990115123465','Moraru Ana','Asistent principal','+37361012345',0),
('1980925123466','Gherman Victor','Practicant','+37361123456',1),
('1990222123467','Toma Roxana','Asistent principal','+37361234567',0),
('1980512123468','Neagu Denis','Practicant','+37361345678',1),
('1990412123469','Florea Ioana','Asistent principal','+37361456789',0),
('1980125123470','Costea Vlad','Practicant','+37361567890',1);
GO

--2.5.4
-- Selecarea tuturor tuplurilor din relația asistenti
SELECT * FROM asistenti;


--2.5.5
-- Să se însereze tupluri în relația cabinete
INSERT INTO pacienti (NumeAnimal, DataNasterii, Specie, Rasa, ArePasaport, IdProprietar)
VALUES
('Bobby','2019-06-15','Caine','Labrador',1,1),
('Mimi','2021-03-02','Pisica','Siameza',0,2),
('Max','2020-01-10','Caine','Golden Retriever',1,3),
('Luna','2018-08-22','Pisica','Persana',0,4),
('Charlie','2022-05-05','Caine','Beagle',1,5),
('Bella','2021-12-12','Caine','Pug',0,6),
('Simba','2017-09-19','Pisica','Maine Coon',1,7),
('Rocky','2019-11-30','Caine','Boxer',1,8),
('Lilly','2020-07-07','Pisica','British Shorthair',0,9),
('Jack','2021-03-15','Caine','Bulldog',1,10),
('Nala','2022-02-28','Pisica','Ragdoll',0,11),
('Oscar','2018-04-01','Caine','Schnauzer',1,12),
('Daisy','2019-09-10','Pisica','Bengaleza',0,13),
('Toby','2020-06-23','Caine','Cocker Spaniel',1,14),
('Luna','2021-10-18','Pisica','Norvegiana',0,15);
GO

--2.5.5
-- Selecarea tuturor tuplurilor din relația pacienti
SELECT * FROM pacienti;


--2.5.6
-- Să se însereze tupluri în relația intermediară veterinari-asistenti
INSERT INTO [veterinari-asistenti] (IdVeterinar, IdAsistent)
VALUES
(1,1),(1,2),(2,3),(2,4),(3,5),
(3,6),(4,7),(4,8),(5,9),(5,10),
(6,11),(6,12),(7,13),(7,14),(8,15);
GO

--2.5.6
-- Selecarea tuturor tuplurilor din relația intermediară veterinari-asistenti
SELECT * FROM [veterinari-asistenti];


--2.5.7
-- Să se însereze tupluri în relația consultatii
INSERT INTO consultatii (DataConsultatiei, IdPacient, IdVeterinar, EsteProgramat)
VALUES
(DATEADD(DAY,-30,GETDATE()),1,1,0),
(DATEADD(DAY,-60,GETDATE()),2,2,1),
(DATEADD(DAY,-90,GETDATE()),3,3,0),
(DATEADD(DAY,-120,GETDATE()),4,4,1),
(DATEADD(DAY,-150,GETDATE()),5,5,0),
(DATEADD(DAY,-180,GETDATE()),6,6,1),
(DATEADD(DAY,-210,GETDATE()),7,7,0),
(DATEADD(DAY,-240,GETDATE()),8,8,1),
(DATEADD(DAY,-270,GETDATE()),9,9,0),
(DATEADD(DAY,-300,GETDATE()),10,10,1),
(DATEADD(DAY,-330,GETDATE()),11,11,0),
(DATEADD(DAY,-360,GETDATE()),12,12,1),
(DATEADD(DAY,-390,GETDATE()),13,13,0),
(DATEADD(DAY,-420,GETDATE()),14,14,1),
(DATEADD(DAY,-450,GETDATE()),15,15,0);
GO

--2.5.7
-- Selecarea tuturor tuplurilor din relația consultatii
SELECT * FROM consultatii;


--2.5.8
-- Să se însereze tupluri în relația medicamente_si_vaccinari
INSERT INTO medicamente_si_vaccinari (IdConsultatie, TipAdministrare, DenumireMedicament, Doza, DurataZile, Cost, Descriere)
VALUES
(1,'Medicament','Amoxicillin','250 mg',7,20.00,'Antibiotic pentru infectii usoare'),
(2,'Vaccin','Rabies Vaccine','1 doz',1,15.00,'Vaccin antirabic pentru caini si pisici'),
(3,'Medicament','Carprofen','50 mg',5,25.00,'Anti-inflamator pentru caini'),
(4,'Vaccin','Feline Leukemia Vaccine','1 doz',1,18.00,'Vaccin FIV/FELV pentru pisici'),
(5,'Medicament','Metronidazole','250 mg',7,22.00,'Tratament gastrointestinal'),
(6,'Vaccin','Distemper Vaccine','1 doz',1,20.00,'Vaccin impotriva parvovirozei'),
(7,'Medicament','Prednisone','10 mg',10,30.00,'Tratament anti-inflamator'),
(8,'Vaccin','Canine Parvovirus Vaccine','1 doz',1,17.00,'Vaccin impotriva parvovirozei canine'),
(9,'Medicament','Itraconazole','100 mg',14,35.00,'Tratament antifungic'),
(10,'Vaccin','Bordetella Vaccine','1 doz',1,16.00,'Vaccin pentru tusea canina'),
(11,'Medicament','Enrofloxacin','100 mg',7,28.00,'Antibiotic pentru caini si pisici'),
(12,'Vaccin','Leptospirosis Vaccine','1 doz',1,19.00,'Vaccin impotriva leptospirozei'),
(13,'Medicament','Furosemide','20 mg',5,12.00,'Diuretic pentru insuficienta cardiaca'),
(14,'Vaccin','Parainfluenza Vaccine','1 doz',1,18.00,'Vaccin impotriva parainfluenței'),
(15,'Medicament','Cephalexin','250 mg',10,25.00,'Antibiotic pentru infectii usoare');
GO

--2.5.8
-- Selecarea tuturor tuplurilor din relația medicamente_si_vaccinari
SELECT * FROM medicamente_si_vaccinari;




--3.1.1 
-- Să se obțină numele proprietarilor și veterinarilor fără duplicate prin operația union.

SELECT  proprietari.Nume FROM proprietari
UNION
SELECT veterinari.Nume FROM veterinari;

--3.1.2 
--Să se obțină ID-urile veterinarilor care au un asistent practicant

SELECT IdVeterinar FROM [veterinari-asistenti]
INTERSECT
SELECT IdAsistent  FROM asistenti
WHERE EstePracticant = 1;

--3.1.3 
--Să se obțină ID-urile asistenților și numele lor care nu sunt practicanți

SELECT IdAsistent, Nume FROM asistenti
EXCEPT
SELECT IdAsistent, Nume FROM asistenti
WHERE EstePracticant = 1;


--3.1.4
-- Să se obțină toate animalele care nu au consultații programate

SELECT * FROM pacienti
WHERE IdPacient NOT IN (
    SELECT IdPacient FROM consultatii WHERE EsteProgramat = 1
);

--3.1.5
-- Să se obțină toate combinațiile posibile de a repartiza asistenții la veterinari

--Varianta 1
SELECT v.Nume, a.Nume FROM veterinari AS v, asistenti AS a;

--Varianta 2
SELECT v.Nume, a.Nume FROM veterinari AS v 
CROSS JOIN asistenti AS a;

--3.1.6
-- Să se obțină pacienții care au pașaport și afișați doar numele lor

--Varianta 1
SELECT NumeAnimal FROM pacienti
WHERE ArePasaport = 1;

--Varianta 2
SELECT NumeAnimal FROM pacienti
WHERE ArePasaport IN (1);

--3.1.7
--Să se obțină numele animalelor și denumirile medicamentelor pe care le-au primit, 
--dar numai pentru medicamentele care au durata mai mare de 7 zile.

--Varianta 1
SELECT p.NumeAnimal, m.DenumireMedicament, m.Descriere, m.DurataZile
FROM pacienti p
JOIN medicamente_si_vaccinari m
ON p.IdPacient = m.IdConsultatie
WHERE m.DurataZile > 7;

--Varianta 2
SELECT p.NumeAnimal, m.DenumireMedicament, m.Descriere, m.DurataZile
FROM pacienti p
INNER JOIN medicamente_si_vaccinari m
ON p.IdPacient = m.IdConsultatie
WHERE m.DurataZile > 7;

--3.1.8
--Să se obțină numele animalelor și datele consultațiilor lor, 
--pentru fiecare pacient care are cel puțin o consultație.

SELECT p.NumeAnimal, c.DataConsultatiei
FROM pacienti p
INNER JOIN consultatii c ON p.IdPacient=c.IdPacient
AND p.VarstaAnimal IS NOT NULL;


--3.1.9
--Să se obțină numele animalelor care au primit cel puțin 
--un medicament sau vaccin.

SELECT NumeAnimal FROM pacienti
WHERE IdPacient IN (
    SELECT IdConsultatie
    FROM medicamente_si_vaccinari
);

-- 3.1.10
--Să se obțină denumirea, doza și durata medicamentelor sau vaccinurilor de la 1 pana la 10 zile, cu doza care contine 250 mg, împreună cu consultațiile lor dacă există, 
--afișând totuși și medicamentele care nu au nicio consultație asociată.

SELECT mv.DenumireMedicament, mv.Doza, mv.DurataZile FROM medicamente_si_vaccinari AS mv
LEFT JOIN consultatii AS c
ON mv.IdConsultatie = c.IdConsultatie
WHERE mv.DurataZile BETWEEN 1 AND 10 AND mv.Doza LIKE '250%';

--3.1.11
-- Să se obțină numele animalelor care se incepe cu litera L și datele consultațiilor lor, 
--afișând toate consultațiile chiar dacă unele nu au pacient asociat.

SELECT p.NumeAnimal, c.DataConsultatiei FROM pacienti p
RIGHT JOIN consultatii c
ON p.IdPacient = c.IdPacient
WHERE NumeAnimal LIKE 'L%'
ORDER BY p.NumeAnimal;


--3.1.12
-- Să se obțină lista tuturor veterinarilor împreună cu cabinetele lor, 
--afișând totuși și veterinarilor fără cabinet și cabinetele fără veterinar,
--care sunt la etajul 1 sau este cabinet ortopedic

SELECT v.Nume AS Veterinar, c.NumeCabinet AS Cabinet
FROM veterinari v
FULL OUTER JOIN cabinete c
ON v.IdCabinet = c.IdCabinet
WHERE c.NumarEtaj = 1 OR c.NumeCabinet LIKE '%Orto%';


--3.1.13
--Să se obțină primii 3 veterinari care au exact un asistent care nu este practicant.

-- Varianta 1

SELECT TOP 3 v.Nume
FROM veterinari v
JOIN [veterinari-asistenti] va ON v.IdVeterinar = va.IdVeterinar
JOIN asistenti a ON va.IdAsistent = a.IdAsistent
WHERE a.EstePracticant = 0
GROUP BY v.IdVeterinar, v.Nume
HAVING COUNT(a.IdAsistent) = 1;


--Varianta 2
SELECT TOP 3 v.Nume
FROM veterinari v
INNER JOIN [veterinari-asistenti] va ON v.IdVeterinar = va.IdVeterinar
INNER JOIN asistenti a ON va.IdAsistent = a.IdAsistent
WHERE a.EstePracticant = 0
GROUP BY v.IdVeterinar, v.Nume
HAVING COUNT(a.IdAsistent) = 1;



--3.2.1
--Să se obțină durata minimă, durata maximă și durata medie a tratamentelor administrate pacienților.

SELECT 
MIN(DurataZile) AS Durata_Minima,
MAX(DurataZile) AS Durata_Maxima,
AVG(DurataZile) AS Durata_Medie
FROM medicamente_si_vaccinari;

--3.2.2
--Să se obțină suma Id-urilor asistenților care sunt practicanți, 
--afișând în același timp numele și statutul fiecărui asistent practicant.

SELECT 
SUM(asistenti.IdAsistent) AS Id_Asistenti, asistenti.Nume, asistenti.EstePracticant
FROM asistenti WHERE asistenti.EstePracticant = 1
GROUP BY asistenti.Nume, asistenti.EstePracticant;


--3.2.3
--Să se obțină numărul total de pacienți care dețin pașaport.

SELECT 
COUNT(ArePasaport) AS Numar_Pasapoarte
FROM pacienti WHERE ArePasaport = 1;



--3.3.1
--Să se obțină, pentru pacienții născuți înainte de anul 2020, numărul de pacienți grupați 
--în funcție de deținerea pașaportului, afișând doar acele grupuri care conțin mai mult de un pacient.

SELECT ArePasaport, COUNT(*) AS Numar
FROM pacienti
WHERE DataNasterii < '2020-01-01'
GROUP BY ArePasaport
HAVING COUNT(*) > 1;


--3.3.2
--Să se obțină numele fiecărui veterinar și numărul de 
--asistenți practicanți pe care îi are, afișând doar 
--veterinarii care au cel puțin un asistent practicant.

SELECT v.Nume, COUNT(a.IdAsistent) AS Nr_Asistenti
FROM veterinari v
JOIN [veterinari-asistenti] va ON v.IdVeterinar = va.IdVeterinar
JOIN asistenti a ON va.IdAsistent = a.IdAsistent
WHERE a.EstePracticant = 1
GROUP BY v.Nume
HAVING COUNT(a.IdAsistent) >= 1;


--3.4.1
-- Să se afișeze pacienții născuți înaintea celui mai tânăr pacient cu pașaport.

SELECT NumeAnimal, DataNasterii
FROM pacienti
WHERE DataNasterii < (SELECT MAX(DataNasterii) FROM pacienti WHERE ArePasaport = 1);


--3.4.2
-- Să se afișeze pacienții care au primit medicamente

SELECT NumeAnimal
FROM pacienti
WHERE IdPacient IN (SELECT IdConsultatie FROM medicamente_si_vaccinari);

--3.4.3
-- Să se afișeze medicamentele care au durată mai mare decât toate medicamentele cu cost sub 20

SELECT DenumireMedicament, DurataZile, Cost
FROM medicamente_si_vaccinari
WHERE DurataZile > ALL (SELECT DurataZile FROM medicamente_si_vaccinari WHERE Cost < 20);


--3.4.4
-- Să se afișeze pacienții cu vârsta mai mică decât vârsta oricărui pacient care are pașaport

SELECT NumeAnimal, VarstaAnimal
FROM pacienti
WHERE VarstaAnimal < ANY (SELECT VarstaAnimal FROM pacienti WHERE ArePasaport = 1);


--3.4.5
-- Să se afișeze veterinarii care au cel puțin un asistent practicant

SELECT Nume
FROM veterinari v
WHERE EXISTS (
    SELECT 1
    FROM [veterinari-asistenti] va
    JOIN asistenti a ON va.IdAsistent = a.IdAsistent
    WHERE va.IdVeterinar = v.IdVeterinar AND a.EstePracticant = 1
);


--3.4.6
-- Să se afișeze pacienții care au consultatii cu un anumit veterinar

SELECT NumeAnimal
FROM pacienti p
WHERE EXISTS (
    SELECT 1
    FROM consultatii c
    WHERE c.IdPacient = p.IdPacient AND c.IdVeterinar = 1
);


--3.4.7
-- Să se obțină durata medie a medicamentelor și vaccinurilor 
--pentru fiecare tip, doar pentru tratamente mai lungi de 5 zile

SELECT TipAdministrare, AVG(DurataZile) AS DurataMedie
FROM (
    SELECT TipAdministrare, DurataZile
    FROM medicamente_si_vaccinari
    WHERE DurataZile > 5
) AS Sub
GROUP BY TipAdministrare
HAVING AVG(DurataZile) > 7;



--3.4.8
-- Să se afișeze fiecare pacient împreună cu numărul de 
--consultații programate în ultimele 180 de zile

SELECT 
    NumeAnimal,
    (SELECT COUNT(*) 
     FROM consultatii c 
     WHERE c.IdPacient = p.IdPacient 
       AND c.DataConsultatiei BETWEEN DATEADD(DAY,-180,GETDATE()) AND GETDATE()
       AND c.EsteProgramat = 1
    ) AS NrConsultatiiProgramate
FROM pacienti p
WHERE VarstaAnimal BETWEEN 1 AND 100;


SELECT * FROM pacienti;
--4.1.1 
-- Să se șteargă maxim 3 pacienți care au pasaport

DELETE TOP (3) FROM pacienti
WHERE ArePasaport = 1;  


SELECT * FROM veterinari;
--4.1.2 
-- Să se șteargă veterinarul cu cele mai puține consultații

DELETE FROM veterinari
WHERE IdVeterinar = (
    SELECT TOP 1 IdVeterinar
    FROM consultatii
    GROUP BY IdVeterinar
    ORDER BY COUNT(*) ASC
);  

SELECT * FROM veterinari;

SELECT * FROM pacienti;
--4.1.3 
-- Să se actualizeze primii 10 pacienți care au pasaport și setează-l la 0

UPDATE TOP (10) pacienti
SET ArePasaport = 0
WHERE ArePasaport = 1;  

--4.1.4 
-- Să se actualizeze costul medicamentelor asociate consultațiilor neprogramate

UPDATE medicamente_si_vaccinari
SET Cost = Cost + 10
WHERE IdConsultatie IN (
    SELECT IdConsultatie
    FROM consultatii
    WHERE EsteProgramat = 0
); 



--4.2.1 
-- Să se insereze în tabelul pacienti doar animalele care nu au pasaport și care nu există deja
 
INSERT INTO pacienti (NumeAnimal, DataNasterii, Specie, Rasa, ArePasaport, IdProprietar)
SELECT 'Lucky', '2023-05-01', 'Caine', 'Beagle', 0, 1
WHERE NOT EXISTS (
    SELECT 1 
    FROM pacienti 
    WHERE NumeAnimal = 'Lucky' AND IdProprietar = 1
);

 


--4.2.2 
-- Înserarea rezultatului unei interogări SELECT asupra unei relații (relație-sursă) într-o relație nouă NEW_Table formată instantaneu (SELECT INTO)
-- Să se creeze un tabel nou cu consultațiile neprogramate și medicamentele asociate


DROP TABLE IF EXISTS NEW_ConsultatiiMedicamenteNeprogramate;

SELECT 
    c.IdConsultatie, 
    c.DataConsultatiei, 
    m.DenumireMedicament, 
    m.Cost
INTO NEW_ConsultatiiMedicamenteNeprogramate
FROM consultatii c
JOIN medicamente_si_vaccinari m 
    ON c.IdConsultatie = m.IdConsultatie
WHERE c.EsteProgramat = 0;







