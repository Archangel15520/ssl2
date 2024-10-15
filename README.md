# SSSL-PR-2 Vasiljew Grigoriy Maksimovich BBMO-02-23

Группа: ББМО-02-23

Студент: Васильев Г.М.

Вариант:

3 % 16 + 1 = 4

**1. Описание структуры полей логов и разработка парсера
Логи имеют структуру строк вида:**

```
# <date> <time> <milliseconds> <log_level> <component>: <message>
```

date — дата в формате YYMMDD.

time — время в формате HHMMSS.

milliseconds — миллисекунды после времени.

log_level — уровень логирования, например, INFO.

component — компонент, к которому относится лог (например, dfs.FSNamesystem).

message — текст сообщения.

**Пример логов в файле HDFS_2k.log**

![image]()

**2. Парсинг лога и сохранение в СУБД**

Мы будем парсить лог-файл и выделять информацию для последующего анализа. Парсер будет извлекать ключевые поля из каждого сообщения.

```
import re
import pandas as pd
import sqlite3

# Функция для парсинга одной строки лога
def parse_log_line(line):
    pattern = r'(\d{6}) (\d{6}) (\d+) (\w+) ([\w\.\$]+): (.+)'
    match = re.match(pattern, line)
    if match:
        return {
            'date': match.group(1),
            'time': match.group(2),
            'milliseconds': match.group(3),
            'log_level': match.group(4),
            'component': match.group(5),
            'message': match.group(6)
        }
    return None

# Функция для парсинга файла логов
def parse_log_file(filepath):
    parsed_logs = []
    with open(filepath, 'r') as file:
        for line in file:
            parsed_line = parse_log_line(line)
            if parsed_line:
                parsed_logs.append(parsed_line)
    return pd.DataFrame(parsed_logs)

# Парсинг лог файла
log_df = parse_log_file('HDFS_2k.log')

# Сохранение данных в SQLite
conn = sqlite3.connect('logs.db')
log_df.to_sql('logs', conn, if_exists='replace', index=False)
conn.close()

# Вывод первых строк для проверки
log_df.head()
```

