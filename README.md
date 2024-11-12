# Bolshoi Golang Project

## 📖 Описание

**Bolshoi Golang** — это реализация in-memory Redis кэша, написанная на языке Go. Проект поддерживает хранение строк, списков и словарей с возможностью установки времени жизни (TTL) для ключей. 

### 🛠️ Возможности
- Хранение строк, списков и словарей
- Установка TTL на любой ключ
- Работа через telnet при помощи REST API
- Запуск через `docker-compose up`
- Возможность сохранения данных на диск
- Масштабируемость
- Результаты покрытия тестами (файл `coverage.out`)

## 🚀 Инструкция для запуска

### Требования
Необходимо иметь:
- **git**
- **docker**
- **docker-compose**

### Шаги для установки
1. Клонируйте репозиторий:
   ```bash
   git clone https://github.com/antonvlasov/geo
Перейдите в скачанную директорию:

cd geo
Запустите контейнеры:

docker-compose up
###🛠️ Инструкция по использованию

Подключение к серверу

Подключитесь к запущенному серверу по telnet к порту 7089:

telnet localhost 7089
Используйте команды для взаимодействия с сервером. Пример:

SET name Anton EX 60

###📜 Список команд

KEYS pattern

Возвращает список ключей, удовлетворяющих glob-style образцу.

SET name Anton
OK
SET age 20
OK
KEYS *
1) "name"
2) "age"
GET key

Возвращает значение по ключу key. Если по ключу нет значения, возвращается nil.

SET name Anton
OK
GET name
Anton
GET PUE
(nil)
SET key value [EX seconds]

Устанавливает значение по ключу key равным строке value.

SET key1 value1 EX 20
OK
EXPIRE key seconds

Устанавливает время жизни значения по ключу.

SET key1 1     
OK
EXPIRE key3 30
(integer) 0
EXPIRE key1 20
(integer) 1
...after some time
GET key1
(nil)
HSET key field value [field value ...]

Устанавливает поле словаря field равным value.

HSET key1 hash1 val1 hash2 val2
(integer) 2
HGET key field

Возвращает значение поля field словаря по ключу key.

HGET key1 hash1
val1
LPUSH key element [element ...]

Вставляет элементы слева в список по ключу key.

LPUSH list1 1 2 3
(integer) 3
RPUSH key element [element ...]

Вставляет элементы справа в список по ключу key.

RPUSH list1 1 2 3
(integer) 3
LPOP key [count]

Удаляет и возвращает элемент слева из списка.

LPOP list1 2
1) 1
2) 2
RPOP key [count]

Удаляет и возвращает элемент справа из списка.

RPOP list1 2
1) 10
2) 9
LSET key index element

Устанавливает значение элемента с индексом index списка по ключу key равным element.

LSET list1 3 30
OK
LGET key index

Получает значение элемента с индексом index из списка по ключу key.

LGET list1 3
30
🖥️ Клиент

Все методы доступны в виде функций с подписью:

func(conn net.Conn, args []string) error {}
Пример использования:

clientConn, err := net.DialTimeout("tcp", "localhost:7089", 0)
if err != nil {
    t.Error(err)
}

err = Set(clientConn, []string{"name", "Anton"})
if err != nil {
    t.Error(err)
}
reader := bufio.NewReader(clientConn)
bytes, err := reader.ReadBytes('\n')
fmt.Println(string(bytes))
💾 Сохранение

Для сохранения используйте команду:

SAVE savename
где savename — имя сохранения. Для загрузки данных используйте:

LOAD savename
Текущие данные заменяются на данные из сохранения.

🧪 Тесты

Реализовано покрытие тестами более 70% кода и нагрузочные тесты операций записи и чтения.

Операция записи:

BenchmarkSet             1000000               173 ns/op
Операция чтения:

BenchmarkGet             1000000                56.4 ns/op
Конкурентное чтение:

BenchmarkGetConcurrent            10000000               289 ns/op
📞 Контакты

Если у вас есть вопросы, вы можете связаться со мной по адресу: [ваш email].
