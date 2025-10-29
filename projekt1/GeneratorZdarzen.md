# Generator zdarzeń
Celem projektu jest tworzenie zdarzeń (JSON) w NiFi i przesyłanie je do tabeli w postgresie - małe zapoznanie się z NiFi
## 1. Stworzenie tabeli w PG
```
CREATE TABLE projekt1.generator_wydarzen (
id uuid PRIMARY KEY,
ts timestamptz NOT NULL,
tekst text,
wartosc integer,
notatka text
);
```
Przez adminer zrobiłem nowy schemat do projektu i dodałem tabelę - uzupełnieniem jej zajmię się już NiFi
## 2. Stworzenie Controller Services
**WAŻNE** - należy zwrócić uwagę czy tworzy się je globalnie czy dla danego flow
### DBCPConnectionPool
<img width="841" height="517" alt="image" src="https://github.com/user-attachments/assets/836a9c29-fe7b-42ef-a5b7-af1952dca0c7" />

Interesuje nas tak naprawdę pięć pól
1. Database Connection URL ```jdbc:postgresql://postgres:5432/pgdb?sslmode=disable&currentSchema=projekt1```
  - postgres - hostname serwera PG
  - 5432 - port
  - pgdb - nazwa bazy danych
  - sslmode=disable - połączenie bez SSL
  - currentSchema - schemat z naszą tabelą
2. Database Driver Class Name ```org.postgresql.Driver```
  - nazwa klasy sterownika
3. Database Driver Location(s) ```org.postgresql.Driver```
  - lokalizacja sterownika dostępna dla NiFi - [przeze mnie używany](https://repo1.maven.org/maven2/org/postgresql/postgresql/42.7.4/postgresql-42.7.4.jar)
  4. Database User
  5. Password
### JsonTreeReader 
<img width="845" height="638" alt="image" src="https://github.com/user-attachments/assets/08994562-1b4d-447f-8160-90d661d27e14" />

Tutaj dwa
  1. Schema Access Strategy
   - podajemy skąd dostanie informacje
  2. Schema Text
```
  {
  "type":"record","name":"Wydarzenie","fields":[
    {"name":"id","type":"string"},
    {"name":"ts","type":{"type":"long","logicalType":"timestamp-millis"}},
    {"name":"tekst","type":["null","string"],"default":null},
    {"name":"wartosc","type":["null","int"],"default":null},
    {"name":"notatka","type":["null","string"],"default":null}]
  }
```
  - w jakiej formie będą
## 3. FLOW
### GenerateFlowFile
<img width="840" height="633" alt="image" src="https://github.com/user-attachments/assets/754b272f-20df-4b97-b038-4c09c4862df9" />

```
{
  "id": "${UUID()}",
  "ts": ${now():toNumber()},
  "tekst": "${random()}",
  "wartosc": ${now():toNumber():mod(100)},
  "notatka": "notatka"
}
```

Co minute tworzymy 1 FlowFile z JSONem który zawiera strukturę naszej tabeli i umieszcza w nim wartości. Relacją success łączymy ją z kolejnym procesorem
### PutDatabaseRecord
<img width="840" height="633" alt="image" src="https://github.com/user-attachments/assets/6b763e9b-d92b-492b-8391-b50d0032599a" />

Czyta treść otrzymanego JSONa i mapuje z niego dane na kolumny z naszej tabeli insertując je do środka. Relacje ustawiamy na terminate
*ten procesor możemy podłączyć relacją failure do jakiegoś LogMessage ale flow jest dosyć krótkie więc nie ma takiej potrzeby
## Wyniki
<img width="1092" height="160" alt="image" src="https://github.com/user-attachments/assets/6ce9e6b1-ce50-4a9a-96e1-528abaa60b0a" />


     

