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

![image]()

**3. Нормализация логов**

Теперь нужно нормализовать данные, привести их к единой форме. Например, можно разделить логи по уровням или компонентам и сохранить их в отдельные таблицы.

```
# Нормализация логов по уровню логирования
def normalize_logs_by_level(df):
    return df.groupby('log_level')

# Нормализация по компоненту
def normalize_logs_by_component(df):
    return df.groupby('component')

# Пример использования
log_levels = normalize_logs_by_level(log_df)
log_components = normalize_logs_by_component(log_df)

# Сохранение нормализованных данных в таблицы
conn = sqlite3.connect('logs.db')

for level, group in log_levels:
    group.to_sql(f'logs_{level}', conn, if_exists='replace', index=False)

for component, group in log_components:
    group.to_sql(f'logs_{component}', conn, if_exists='replace', index=False)

conn.close()
```

**4. Статистический анализ и визуализация**

Для анализа будем использовать библиотеки Pandas и Matplotlib. Проведем анализ частоты логов по уровням логирования и компонентам.

```
import matplotlib.pyplot as plt

# Анализ частоты логов по уровням
log_level_counts = log_df['log_level'].value_counts()
log_level_counts.plot(kind='bar', title='Распределение по уровням логирования')
plt.xlabel('Уровень логирования')
plt.ylabel('Количество')
plt.show()

# Анализ частоты логов по компонентам
log_component_counts = log_df['component'].value_counts().head(10)  # ТОП-10 компонентов
log_component_counts.plot(kind='bar', title='Топ-10 компонентов по количеству логов')
plt.xlabel('Компонент')
plt.ylabel('Количество')
plt.show()
```

![image]()

![image]()

**Вывод:**

Первый график показывает распределение логов по уровням (INFO, ERROR и т.д.).
Второй график отображает топ-10 компонентов, по которым создается наибольшее количество логов.
