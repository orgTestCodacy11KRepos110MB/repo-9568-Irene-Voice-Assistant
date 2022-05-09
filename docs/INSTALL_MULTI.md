# Мультиинсталляция в режиме клиент-сервер

Мультиинсталляция позволяет управлять Ириной:
- с нескольких микрофонов
- с разных машин
- из Телеграма

**Общая логика:**

- Клиент отвечает за распознавание речи / подачу команд
- Сервер Ирины отвечает за выполнение команд

**[клиент] Speech-To-Text (распознавание голоса)** ---> _[веб-сервер Ирины] runva_webapi -> plugin -> Text-To-Speech (генерация голосового ответа)_

Можно сконфигурировать несколько клиентов для подачи команд; также команды можно подавать с клавиатуры или другим способом.


## Настройка

- Вам нужно запустить сервер Ирины - запустите **runva_webapi.py**.
Так запустится Ирина в формате веб-сервера с REST API, к которой могут обращаться
разные клиенты (клиенты нужны для каждой машины, каждого микрофона и т.д.)

Потребуется дополнительно установка 
```
pip install fastapi
pip install "uvicorn[standard]"
```

- Сконфигурируйте, где находится сервер Ирины. Настройки сервера - host,port,log_level настраиваются в **options/webapi.json**

- Запустите хотя бы один клиент. Клиенты можно взять здесь:
https://github.com/janvarev/Remote-Irene 

Самый простой способ - взять уже готовый **run_remoteva_vosk.py**. Можно взять его скомпилированную 
версию https://github.com/janvarev/Remote-Irene#compiled

Клиент надо также сконфигурировать - установите адрес сервера в **options.json**.

Готово!

При необходимости вы можете запустить больше клиентов - все они будут обращаться к единому серверу Ирины.

## Варианты настроек

### Где и как озвучивать?

Клиент может определить, как он хочет, чтобы озвучивался результат работы Ирины.

В файле options.json, отредактируйте:

```
ttsFormat = "saywav"
``` 
Варианты:
- "none" (TTS реакции будут на сервере) (звук на сервере)
- "saytxt" (сервер вернет текст, TTS будет на клиенте) (звук на клиенте)
- "saywav" (TTS на сервере, сервер отрендерит WAV и вернет клиенту, клиент его проиграет) (звук на клиенте) **наиболее универсальный для клиента**

С версии 4.1 Ирины немного поменялся формат результата вызова (для saytxt), 
но теперь поддерживаются мультиформаты через ",".

Например: "none,saywav" - как вызовет озвучку на сервере, так и передаст WAV-файл на клиент. Можно использовать
любые комбинации none,saytxt,saywav

### Несколько микрофонов

Если вы хотите использовать несколько микрофонов на одной машине,
проще всего запустить несколько VOSK-клиентов.

Я запускаю уже собранные клиенты командой в BAT-файле:
```
run_remoteva_vosk.exe -d=1
```

где после -d указывается номер устройства ввода (микрофона) для распознавания.

Например, для двух микрофонов вам потребуется два BAT-файла:

```
run_remoteva_vosk.exe -d=1
```
```
run_remoteva_vosk.exe -d=2
```

Конкретный ID девайсов надо смотреть помощью команды -l или понять путем перебора.

### Распознавание голоса на сервере

С версии 5.2 можно еще сильнее упростить клиент для работы на маломощных машинах, типа Raspberry.

Для этого надо запустить дополнительный сервер, который позволит распознавать данные с микрофона.

На сервере
- Запустите `runva_webapi.py`. Это обычный WEBAPI Ирины
- Запустите `vosk_asr_server.py`. Это VOSK-сервер, распознающий данные с микрофона 
(будет использовать PORT WEBAPI + 1)

На клиенте
- Запустите `run_remoteva_voskrem.py`. Он требует минимального набора библиотек,
и отсылает данные микрофона на vosk_asr_server, а результат на runva_webapi.

Конфигурируется аналогично run_remoteva_vosk.

**Альтернативный вариант**

Вы можете запустить VOSK ASR Server через докер. 
Так можно обойти проблемы с установкой VOSK, если у вас такие есть. 

Детали: https://alphacephei.com/vosk/server

Тогда в качестве параметра надо будет указать
`python run_remoteva_voskrem.py -u ws://localhost:2700`, чтобы распознавание
шло по другому URL/порту, а не вместе с Webapi Ирины. 

### Телеграм-клиент

Можно запустить Телеграм-клиент для доступа к серверу Ирины через телеграм-бота.
Подробнее: https://github.com/janvarev/Remote-Irene#%D1%82%D0%B5%D0%BB%D0%B5%D0%B3%D1%80%D0%B0%D0%BC-%D0%BA%D0%BB%D0%B8%D0%B5%D0%BD%D1%82

### Свои клиенты

При желании вы можете написать свой клиент, это несложно.

Документация по вызовам может быть найдена в коде **runva_webapi.py**

Также можно посмотреть документацию fastapi - там есть веб-интерфейс для тестовых вызовов функций.

Примеры реализаций клиентов можно найти на https://github.com/janvarev/Remote-Irene