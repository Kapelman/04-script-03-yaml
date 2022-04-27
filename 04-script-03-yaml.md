### Как сдавать задания

Вы уже изучили блок «Системы управления версиями», и начиная с этого занятия все ваши работы будут приниматься ссылками на .md-файлы, размещённые в вашем публичном репозитории.

Скопируйте в свой .md-файл содержимое этого файла; исходники можно посмотреть [здесь](https://raw.githubusercontent.com/netology-code/sysadm-homeworks/devsys10/04-script-03-yaml/README.md). Заполните недостающие части документа решением задач (заменяйте `???`, ОСТАЛЬНОЕ В ШАБЛОНЕ НЕ ТРОГАЙТЕ чтобы не сломать форматирование текста, подсветку синтаксиса и прочее, иначе можно отправиться на доработку) и отправляйте на проверку. Вместо логов можно вставить скриншоты по желани.

# Домашнее задание к занятию "4.3. Языки разметки JSON и YAML"


## Обязательная задача 1
Мы выгрузили JSON, который получили через API запрос к нашему сервису:
```
    { "info" : "Sample JSON output from our service\t",
        "elements" :[
            { "name" : "first",
            "type" : "server",
            "ip" : 7175 
            }
            { "name" : "second",
            "type" : "proxy",
            "ip : 71.78.22.43
            }
        ]
    }
```
  Нужно найти и исправить все ошибки, которые допускает наш сервис
Решение:
  - напишем небольшой PY скрипт
```
import json

file_path='item1.json'
with open(file_path,'r') as open_file:
     b=json.load(open_file)
     print(json.dumps(b, indent=2, sort_keys='false'))

```
- исправим JSON
```
{ "info" : "Sample JSON output from our service\t",
        "elements" :[
            { "name" : "first",
            "type" : "server",
            "ip" : "7175"
            },
            { "name" : "second",
            "type" : "proxy",
            "ip" : "71.78.22.43"
            }
        ]
    }
```
- Проверим, все работает
```
vagrant@vagrant:~/py_script$ python main4.py
{
  "elements": [
    {
      "ip": "7175",
      "name": "first",
      "type": "server"
    },
    {
      "ip": "71.78.22.43",
      "name": "second",
      "type": "proxy"
    }
  ],
  "info": "Sample JSON output from our service\t"
}
```

## Обязательная задача 2
В прошлый рабочий день мы создавали скрипт, позволяющий опрашивать веб-сервисы и получать их IP. К уже реализованному функционалу нам нужно добавить возможность записи JSON и YAML файлов, описывающих наши сервисы. Формат записи JSON по одному сервису: `{ "имя сервиса" : "его IP"}`. Формат записи YAML по одному сервису: `- имя сервиса: его IP`. Если в момент исполнения скрипта меняется IP у сервиса - он должен так же поменяться в yml и json файле.

### Ваш скрипт:
```python
import os
import socket
import datetime
import time
import json
import yaml


dns_list = ['drive.google.com','mail.google.com','google.com']

if os.path.exists('web-services.yaml'):
    with open('web-services.yaml') as f:

        dns_ip_prev = yaml.safe_load(f)
        dns_ip_cur = dict()
        for dns_name in dns_list:
            dns_ip_cur[dns_name] = ''
else:
    dns_ip_prev = dict()
    dns_ip_cur = dict()
    for dns_name in dns_list:
        dns_ip_prev[dns_name] = ''
        dns_ip_cur[dns_name] = ''

for dns_name in dns_list:
    ip1 = socket.gethostbyname(dns_name)
    prev_ip = dns_ip_prev[dns_name]
    if  prev_ip != ip1 and dns_ip_prev[dns_name] != '':
        dns_ip_cur[dns_name] = ip1
        print('[ERROR] ' + dns_name +' '+'IP mismatch: ' + dns_ip_prev[dns_name] +' ' + dns_ip_cur[dns_name])
    else:
        dns_ip_prev[dns_name] = ip1
        dns_ip_cur [dns_name] = ip1
        print(dns_name + ' - ' + dns_ip_prev[dns_name])

with open('web-services.yaml', 'w') as opened_file:
    yaml.dump(dns_ip_cur, opened_file, explicit_start=True,explicit_end=True, canonical=False,default_flow_style=False)

with open('web-services.json', 'w') as opened_file_json:
    json.dump(dns_ip_cur, opened_file_json)

```

### Вывод скрипта при запуске при тестировании:
```
vagrant@vagrant:~/py_script$ python main5.py
drive.google.com - 173.194.73.194
[ERROR] mail.google.com IP mismatch: 74.125.131.18 74.125.131.17
[ERROR] google.com IP mismatch: 173.194.73.139 173.194.73.102
```

### json-файл(ы), который(е) записал ваш скрипт:
```json
{"google.com": "173.194.73.102", "drive.google.com": "173.194.73.194", "mail.google.com": "74.125.131.17"}
```

### yml-файл(ы), который(е) записал ваш скрипт:
```yaml
---
drive.google.com: 173.194.73.194
google.com: 173.194.73.102
mail.google.com: 74.125.131.17
...
```

## Дополнительное задание (со звездочкой*) - необязательно к выполнению

Так как команды в нашей компании никак не могут прийти к единому мнению о том, какой формат разметки данных использовать: JSON или YAML, нам нужно реализовать парсер из одного формата в другой. Он должен уметь:
   * Принимать на вход имя файла
   * Проверять формат исходного файла. Если файл не json или yml - скрипт должен остановить свою работу
   * Распознавать какой формат данных в файле. Считается, что файлы *.json и *.yml могут быть перепутаны
   * Перекодировать данные из исходного формата во второй доступный (из JSON в YAML, из YAML в JSON)
   * При обнаружении ошибки в исходном файле - указать в стандартном выводе строку с ошибкой синтаксиса и её номер
   * Полученный файл должен иметь имя исходного файла, разница в наименовании обеспечивается разницей расширения файлов

### Ваш скрипт:
```python
???
```

### Пример работы скрипта:
???
