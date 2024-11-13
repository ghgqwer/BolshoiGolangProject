# In-Memory Key-Value Database

Данный проект представляет собой in-memory key-value базу данных с HTTP интерфейсом, обеспечивающим возможность персистентного хранения и автоматического удаления устаревших записей. Состояние storage хранится в PostgresDB.

Для удобства работы с базой данных реализован фронтенд, который демонстрирует пример работы со скалярами. Добавлен Swagger для всех эндпоинтов.

![Иллюстрация к проекту](https://github.com/jon/coolproject/raw/master/image/image.png)

## Типы хранимых данных

Все ключи в базе данных являются строками. Каждый ключ может иметь только одно значение, которое может быть одного из следующих типов:

- Скаляр
- Словарь
- Массив

### Скаляр

Скаляр - это единичное значение типа либо строка, либо целое число. Тип скаляра можно узнать (строка/число). Более подробно о форматах передачи ответов и запросов см. в разделе [интерфейс](#интерфейс).

#### Операции со скалярами

- **GET key**: Возвращает значение по ключу key. Если значение отсутствует, возвращается nil. Если тип значения не скаляр, возвращается ошибка.
  Пример:
```
SET name "Anton"
OK
SET number 42
OK
GET name
"Anton"
GET PUE
(nil)
GET number
42
  ```

- **SET key value [EX seconds]**: Устанавливает значение по ключу key. Если value указано в кавычках, это строка; иначе это число. Дополнительный параметр EX seconds задает время жизни значения.
  Пример:
```
SET key1 "value1" EX 20
OK
SET key2 value1 EX 20
Requested value is not a number
```

### Словарь

Словарь хранит в своих полях скаляры, где ключи являются строковыми значениями.

#### Операции со словарями

- **HSET key field value [field value ...]**: Устанавливает поле словаря. Возвращает количество затронутых полей.
  Пример:
  ```
  SET key1 1
  OK
  HSET key1 hash1 "val1"
  Requested field is of type scalar
  HSET key1 hash1 "val1" hash2 "val2"
  (integer) 2
  ```

- **HGET key field**: Возвращает значение поля field словаря по ключу key.
  Пример:
  ```
  SET name Anton
  OK
  HSET key1 hash1 "val1" hash2 "val2"
  (integer) 2

  HGET key1 hash1
  "val1"
  HGET key1 hash2 
  "val2"
  HGET key1 hash3
  (nil)
  HGET key434 hash2
  (nil)
  HGET name hash
  Requested field is of type scalar
  ```

### Массив

Массив позволяет хранить упорядоченные массивы скаляров по заданному ключу в базе данных.

#### Операции по работе с массивами

- **LPUSH key element [element ...]**: Вставляет элементы в начало списка по ключу `key`. Если элементов несколько, они добавляются поочередно. При отсутствии существующего значения по ключу создаётся новый список.  
  **Пример:**
  ```plaintext
  LPUSH list1 1 2 3
  (integer) 3
  LPOP list1 0 -1
  1) 3
  2) 2
  3) 1
  ```

- **RPUSH key element [element ...]**: Вставляет элементы в конец списка по ключу `key`. Если элементов несколько, они добавляются по порядку. При отсутствии значения создаётся новый список.  
  **Пример:**
  ```plaintext
  RPUSH list1 1 2 3
  (integer) 3
  LPOP list1 0 -1
  1) 1
  2) 2
  3) 3
  ```

- **RADDTOSET key element [element ...]**: Добавляет элементы в конец списка по ключу `key`, при этом добавляются только уникальные элементы, которых ещё нет в списке. Если список не существует, он создаётся.  
  **Пример:**
  ```plaintext
  RPUSH list1 1 2 3
  (integer) 3
  RADDTOSET list1 3 5 8 4 8
  LPOP list1 0 -1
  1) 1
  2) 2
  3) 3
  4) 5
  5) 8
  6) 4
  ```

- **LPOP key [count]**: Удаляет и возвращает элементы из начала списка. Параметр `count` определяет количество удаляемых элементов. Если количество удаляемых элементов превышает общее количество, возвращается всё доступное.  
  **Пример:**
  ```plaintext
  RPUSH list1 1 2 3 4 5 6 7 8 9 10
  (integer) 10
  LPOP list1 2
  1) 1
  2) 2
  ```

- **RPOP key [count]**: Удаляет и возвращает элементы из конца списка по ключу. Параметр `count` аналогичен `LPOP`.  
  **Пример:**
  ```plaintext
  RPUSH list1 1 2 3 4 5 6 7 8 9 10
  (integer) 10
  RPOP list1 2
  1) 10
  2) 9
  ```

- **LSET key index element**: Устанавливает значение элемента по индексу `index` в списке по ключу `key`. Если элемента с указанным индексом нет, возвращается ошибка.  
  **Пример:**
  ```plaintext
  RPUSH list1 0 1 2 3 4 5 6 7 8 9
  (integer) 10
  LSET list1 3 30
  OK
  LGET list1 3
  30
  LSET list1 20 2
  Index out of range
  ```

- **LGET key index**: Получает значение элемента из списка по индексу `index` и ключу `key`. Если элемента не существует, возвращается ошибка.  
  **Пример:**
  ```plaintext
  RPUSH list1 0 1 2 3 4 5 6 7 8 9
  (integer) 10
  LGET list1 3
  30
  LSET list1 20 2
  Index out of range
  ```
### Дополнительные операции

- **EXPIRE key seconds**: Задает время жизни ключа в секундах. После его истечения ключ удаляется.
  Пример:
  ```
  SET key1 1   
  OK
  EXPIRE key3 30
  (integer) 0
  EXPIRE key1 20
  (integer) 1
  ...after 20 seconds
  GET key1
  (nil)
  ```

## Сохранение данных

База данных периодически сохраняет свое состояние на диск для восстановления после сбоев. Состояние сохраняется в формате JSON в PostgresDB.

## Интерфейс

База данных предоставляет HTTP интерфейс: для получения данных используется GET, для записи - POST. Каждая операция имеет свой эндпоинт HTTP. 

## Docker-compose

Для развертывания приложения и базы данных Postgres используйте docker-compose.
