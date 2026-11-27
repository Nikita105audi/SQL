Пять "пунктов меню"

Вид	Тип среза	На какой вопрос отвечает
v_price	проекция одной таблицы (tools)	что есть в прокате и сколько стоит в день
v_open	фильтр по строкам (return_date IS NULL)	кто сейчас держит инструмент и не вернул
v_rentals_full	соединение трёх таблиц	в какой выдаче кто и что взял, с датами и залогом
v_client_deposits	агрегат с GROUP BY	кто сколько раз брал и на какую сумму залогов
v_open_full	вид поверх v_rentals_full (один уровень)	кто сейчас держит что именно, человеческими именами

Родной способ изменения вида (SQLite). В SQLite нет ни ALTER VIEW, ни CREATE OR REPLACE VIEW — проверено, оба падают синтаксической ошибкой:

ALTER VIEW v_price AS SELECT name FROM tools; → OperationalError: near "VIEW": syntax error
CREATE OR REPLACE VIEW v_price AS SELECT name FROM tools; → OperationalError: near "OR": syntax error

Родной путь — удалить и создать заново под тем же именем (в одной транзакции, чтобы откат при ошибке вернул старое определение):

sql
Копировать
BEGIN;
DROP VIEW v_price;
CREATE VIEW v_price AS SELECT name, price_per_day, category FROM tools;
COMMIT;

Проверка: SELECT name, price_per_day, category FROM v_price ... — новая колонка видна, старые на месте. Ограничение PostgreSQL (нельзя переставлять/переименовывать колонки, только дописывать в конец) к SQLite не относится — там пересоздание заменяет запрос целиком.

Фактические названия пунктов в клиенте (SQLite). В sqlite3 обозревателя объектов нет — всё делается SQL-командами из скрипта: создание CREATE VIEW … AS SELECT …;, просмотр данных SELECT … FROM имя_вида;, изменение — DROP VIEW + CREATE VIEW, удаление DROP VIEW IF EXISTS имя;. Список видов — SELECT name, type FROM sqlite_master WHERE type = 'view';. Если вы работаете в DBeaver/DBSchema: узел Views рядом с Tables, правый клик по виду — Edit View (открыть текст в редакторе), View Data (сетка со строками), Drop View (удалить); точные формулировки пунктов зависят от версии клиента — запишите их у себя тем же способом (правый клик по узлу "Представления" и по любому виду).

Что сдать. pr17.sql — шапка-оглавление, пять секций, все CREATE / SELECT / пересоздание / DROP с комментариями, в секции 4 — реальный путь SQLite с проверкой SELECT, в секции 5 — двойной DROP VIEW IF EXISTS без падения. README — таблица пяти пунктов, родной способ изменения с реальным сообщением ошибки, названия пунктов контекстного меню вашего клиента.

Если у вас не SQLite — меняются только три места, виды и секции те же:

Действие	SQLite	PostgreSQL	MySQL	SQL Server
Изменить вид	DROP VIEW + CREATE VIEW	CREATE OR REPLACE VIEW (только дописать колонку в конец)	CREATE OR REPLACE VIEW (замена целиком)	ALTER VIEW
Список видов	sqlite_master	information_schema.views	SHOW FULL TABLES WHERE Table_type = 'VIEW'	information_schema.views
Повторный DROP	IF EXISTS	IF EXISTS	IF EXISTS	IF EXISTS

Сообщение из части 3 для вашей СУБД возьмите из реального запуска: выполните "чужой" способ и впишите в README то, что вернул клиент (успех или текст ошибки) — это и есть проверка, а не пересказ документации.
