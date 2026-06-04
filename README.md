# Bazy_danych_projekt_hotel
Projekt na zaliczenie - System zarządzania hotelem

Autorka: Zuzanna Jalińska  
Logowanie na serwerze: `s208355` 
Nazwa bazy: `projekt_hotel`    
Adres interfejsu (WWW): [http://172.20.2.84/~s208355/hotel.php](http://172.20.2.84/~s208355/hotel.php)  

## 1. Opis bazy danych:
Projekt przedstawia relacyjną bazę danych, która wspomaga pracę recepcji hotelu. System przechowuje informacje o gościach, pokojach i pracownikach. Tabela `Rezerwacje` jest głównym punktem systemu, łączy ona wyżej wspomniane tabele za pomocą mechanizmu kluczy obcych (`FOREIGN KEY`)  
Dodatkowo, w celu monitorowania finansów została stworzona tabela `Opłaty`, która jest bezpośrednio związana z rezerwacjami.  
W tabeli `Archiwum_rezerwacji` przy użyciu triggera, baza automatycznie rejestruje historię anulowanych pobytów i zapisuje je.  

## 2. Diagram relacyjny:
Uwaga: Pobrany diagram w formie pliku .png znajduje się w katalogu głównym repozytorium pod nazwą `diagram_relacyjny.png`

## 3. Diagram związków encji:
Uwaga: Pobrany diagram w formie pliku .png znajduje się w katalogu głównym repozytorium pod nazwą `diagram_zwiazkow_encji.png`

## 4. Definicje tabel w języku SQL
   
  ### 4.1 Tabela `Goscie`  
  CREATE TABLE Goscie (  
    id SERIAL PRIMARY KEY,  
    imie VARCHAR(50) NOT NULL,  
    nazwisko VARCHAR(50) NOT NULL,  
    nr_dowodu VARCHAR(20) UNIQUE NOT NULL );   
  
  ### 4.2 Tabela `Pracownicy`  
  CREATE TABLE Pracownicy (  
    id SERIAL PRIMARY KEY,  
    imie VARCHAR(50) NOT NULL,  
    nazwisko VARCHAR(50) NOT NULL,  
    stanowisko VARCHAR(50) NOT NULL );  
  
  ### 4.3 Tabela `Pokoje`  
  CREATE TABLE Pokoje (  
    id SERIAL PRIMARY KEY,  
    numer VARCHAR(10) UNIQUE NOT NULL,  
    rodzaj VARCHAR(30) NOT NULL,  
    cena DECIMAL(10,2) NOT NULL,  
    status VARCHAR(20) NOT NULL );  
  
  ### 4.4 Tabela `Rezerwacje`  
  CREATE TABLE Rezerwacje (  
    id SERIAL PRIMARY KEY,  
    id_goscia INT NOT NULL,  
    id_pokoju INT NOT NULL,  
    id_pracownika INT NOT NULL,  
    data_od DATE NOT NULL,  
    data_do DATE NOT NULL,  
    FOREIGN KEY (id_goscia) REFERENCES Goscie(id),  
    FOREIGN KEY (id_pokoju) REFERENCES Pokoje(id),  
    FOREIGN KEY (id_pracownika) REFERENCES Pracownicy(id) );   
  
  ### 4.5 Tabela `Oplaty`  
  CREATE TABLE Oplaty (  
    id SERIAL PRIMARY KEY,  
    id_rezerwacji INT NOT NULL,  
    kwota DECIMAL(10,2) NOT NULL,  
    status_platnosci VARCHAR(20) DEFAULT 'Nieoplacone',  
    FOREIGN KEY (id_rezerwacji) REFERENCES Rezerwacje(id) ON DELETE CASCADE );   
    
  ### 4.6 Tabela `Archiwum_rezerwacji`  
  CREATE TABLE Archiwum_rezerwacji (  
    id SERIAL PRIMARY KEY,  
    id_starej_rezerwacji INT,  
    informacja_o_rezerwacji TEXT,  
    data_archiwizacji TIMESTAMP DEFAULT CURRENT_TIMESTAMP );  

## 5. Definicja widoku:  
   CREATE VIEW Widok_Rezerwacji_Gosci AS  
   SELECT  
    Rezerwacje.id AS id_rezerwacji,  
    Goscie.imie, Goscie.nazwisko,  
    Pokoje.numer AS numer_pokoju,  
    Rezerwacje.data_od,  
    Rezerwacje.data_do  
   FROM Rezerwacje  
   JOIN Goscie ON Rezerwacje.id_goscia = Goscie.id  
   JOIN Pokoje ON Rezerwacje.id_pokoju = Pokoje.id;  

## 6. Definicja triggera:  
  CREATE OR REPLACE FUNCTION funkcja_archiwizuj()  
  RETURNS TRIGGER AS $$  
  BEGIN  
    INSERT INTO Archiwum_rezerwacji (id_starej_rezerwacji, informacja_o_rezerwacji)  
    VALUES (  
      OLD.id,  
      CONCAT('Usunieto rezerwacje o ID: ', OLD.id, '. Pobyt mial trwac od: ', OLD.data_od, ' do: ', OLD.data_do)  
    );  
    RETURN OLD;  
  END;  
  $$ LANGUAGE plpgsql;  
  
  CREATE TRIGGER archiwizuj_usuniecie  
  AFTER DELETE ON Rezerwacje  
  FOR EACH ROW  
  EXECUTE FUNCTION funkcja_archiwizuj();  





