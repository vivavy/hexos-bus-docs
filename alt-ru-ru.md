# HexOS GBus - Глобальная шина HexOS

## Что это такое?

Глобальная шина HexOS (GBus) представляет собой систему глобальных очередей намерений, предназначенную для обмена информацией между различными компонентами операционной системы. Она позволяет процессам и приложениям взаимодействовать, отправляя и принимая сообщения.

## Как использовать?

Вы можете поставить намерения на шину, и операционная система сама позаботится о доставке их адресатам.

## Локальные шины

Вы можете создавать локальные шины для обмена сообщениями между ограниченным числом сущностей. Это полезно для создания небольших, изолированных каналов связи.

### Пример использования локальной шины

```cpp
/*/
/* Простой пример широковещателя локальной шины
/*/

#include <hexos-consts>
#include <hexos-bus>

hexos::bus::LocalBus lbus;

int main() {
    lbus.init("dbus-name"); // Инициализация шины с именем "dbus-name"
    lbus.count(2); // Установка максимального количества сущностей (включая инициатора)
    lbus.require(2); // Установка минимального количества сущностей (включая инициатора)
    lbus.accept(); // Прием сущностей
    lbus.wait(); // Ожидание подключения всех сущностей

    // Отправка намерений на шину
    lbus.put(hexos::bus::Intent(HEXOS_BROADCAST_TEXT, "Hello world!")); // Отправка текстового сообщения
    lbus.put(hexos::bus::Intent(HEXOS_BROADCAST_COMMAND, "show textbox")); // Отправка команды

    lbus.put(hexos::bus::Intent(HEXOS_DISCONNECT)); // Отключение шины
    lbus.wait(); // Ожидание отключения всех сущностей
    lbus.destroy(); // Удаление шины
}

```

```cpp
/*/
/* Простой пример приемника локальной шины
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-bus>
#include <string>

hexos::bus::LocalBus lbus;
std::string message;

int main() {
    lbus.connect("dbus-name"); // Подключение к шине

    if (!lbus.success) {
        // Ошибка
        return -1;
    }

    while (true) {
        hexos::bus::Intent intent = lbus.get(); // Получение намерения из шины
        if (intent.type == HEXOS_BROADCAST_TEXT)
            // Обработка текстового сообщения
            message = intent.data;
        else if (intent.type == HEXOS_BROADCAST_COMMAND)
            // Обработка команды
            if (std::string(intent.data) == "show textbox")
                // Показать текстовое поле
                hexos::gui::showTextbox(message);
        else if (intent.type == HEXOS_DISCONNECT)
            // Обработка отключения шины
            break;
    }

    lbus.close(); // Закрытие соединения
}

```

## Виртуальные шины

Виртуальные шины позволяют создавать очереди сообщений, управляемые операционной системой. Это полезно для более сложных сценариев обмена данными.

### Пример использования виртуальной шины

```cpp
/*/
/* Простой пример широковещателя-приемника виртуальной шины
/*/

#include <hexos-consts>
#include <hexos-bus>

hexos::bus::VirtualBus vbus;

int main() {
    vbus.init("dbus-name");

    vbus.put("Hello world!"); // Отправка строкового сообщения
    vbus.put("Blah blah blah");
    vbus.put("Hi there!");

    while (vbus.notEmpty()) {
        std::string message = vbus.get(); // Получение строкового сообщения
        hexos::gui::showTextbox(message);
    }

    vbus.close(); // Закрытие соединения
}

```

## Ошибки

При инициализации шины может возникнуть ошибка. Это можно проверить и обработать соответствующим образом.

```cpp
/*/
/* Пример создания шины с зарезервированным именем, чтобы вызвать ошибку
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-bus>

hexos::bus::VirtualBus vbus;

int main() {
    vbus.init("osgbus0"); // Попытка создать шину с зарезервированным именем "osgbus0"
    if (!vbus.success) {
        // Ошибка
        hexos::gui::showTextbox("Error: " + std::string(vbus.errorMessage));
        return -1;
    }
}

```

Константы для проверки ошибок:

```cpp
// hexos-consts part
#define HEXOS_BUS_ERROR_NONE 0
#define HEXOS_BUS_ERROR_NAME_EXISTS 1
#define HEXOS_BUS_ERROR_ALREADY_CONNECTED 2
#define HEXOS_BUS_ERROR_NOT_CONNECTED 3
#define HEXOS_BUS_ERROR_INVALID_DATA 4
#define HEXOS_BUS_ERROR_VIOLATION 5
#define HEXOS_BUS_ERROR_UNKNOWN 6
#define HEXOS_BUS_ERROR_TIMEOUT 7
```

## Глобальная шина GBus

Глобальная шина GBus используется для обмена информацией между различными процессами и системами в операционной системе.

### Пример подключения к глобальной шине GBus

```cpp
/*/
/* Пример подключения к глобальной шине GBus
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-bus>

hexos::bus::GlobalBus gbus;

int main() {
    gbus.connect(); // Подключение к глобальной шине GBus

    if (!gbus.success) {
        // Ошибка
        hexos::gui::showTextbox("Error: " + std::string(gbus.errorMessage));
        return -1;
    }

    // Отправка намерения на всех сущностях в системе
    gbus.put(HEXOS_BROADCAST_TEXT, "Hello world!", append_credentials = true);

    gbus.close(); // Закрытие соединения
}

```

### Пример получения сообщений из глобальной шины GBus

```cpp
/*/
/* Пример получения сообщения из глобальной шины GBus
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-bus>

hexos::bus::GlobalBus gbus;

int main() {
    gbus.connect(); // Подключение к глобальной шине GBus

    if (!gbus.success) {
        // Ошибка
        hexos::gui::showTextbox("Error: " + std::string(gbus.errorMessage));
        return -1;
    }

    while (gbus.notEmpty()) {
        hexos::bus::Intent intent = gbus.get(); // Получение намерения из шины
        if (intent.type == HEXOS_BROADCAST_TEXT)
            // Обработка текстового сообщения
            hexos::gui::showTextbox(intent.data);
        else if (intent.type == HEXOS_DISCONNECT)
            // Обработка отключения шины
            break;
    }

    gbus.close(); // Закрытие соединения
}

```

## Разница между системными API: HWSAPI и GBus API (SWSAPI)

### HWSAPI (HardWare System API)

HWSAPI использует инструкцию `syscall` для взаимодействия с ядром.

```cpp
/*/
/* Вызов системной функции через HWSAPI
/*/

#include <hexos-consts>
#include <hexos-assembly>

int main() {
    hexos::assembly::syscall(HEXOS_DEBUG_OUT, "Hello world!");
}

////////////////////  ALTERNATIVE  ////////////////////
/*/
/* Вызов системной функции через hexos-debug
/*/

#include <hexos-debug>

int main() {
    hexos::debug::out("Hello world!");
}

```

### SWSAPI (SoftWare System API)

SWSAPI использует GBus для взаимодействия с ядром.

```cpp
/*/
/* Вызов системной функции через SWSAPI
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-bus>

hexos::bus::GlobalBus gbus;

int main() {
    gbus.connect(); // Подключение к глобальной шине GBus

    if (!gbus.success) {
        // Ошибка
        hexos::gui::showTextbox("Error: " + std::string(gbus.errorMessage));
        return -1;
    }

    gbus.put(HEXOS_EXECSWSAPI, HEXOS_DEBUG_OUT, "Hello world!");

    while (gbus.notEmpty()) {
        hexos::bus::Intent intent = gbus.get(); // Получение намерения из шины
        if (intent.type == HEXOS_DISCONNECT)
            // Обработка отключения шины
            break;
    }

    gbus.close(); // Закрытие соединения
}

```

Использование GBus позволяет обмениваться информацией между различными системами, а также настраивать шифрование для безопасного обмена данными.

## Шифрование в GBus

Вы можете установить ключ шифрования для зашифрованной передачи данных через шину. Если вы установите ключ дешифрования, вы сможете читать только сообщения, зашифрованные с этим ключом. Возможна од

новременная установка ключа шифрования и дешифрования для создания защищенных каналов связи.

Пример использования шифрования:

```cpp
/*/
/* Вызов системной функции с помощью SWSAPI для отладки
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-internals>
#include <hexos-debug>

int main() {
    hexos::internals::SWSAPIKey("debug"); // Установка ключа шифрования
    hexos::internals::SWSAPI(true); // Установка флага SWSAPI на true

    hexos::debug::out("Hello world!"); // Вызов отладочной функции

    // Пример обработки намерений
    while (gbus.notEmpty()) {
        hexos::bus::Intent intent = gbus.get(); // Получение намерения из шины    
        if (intent.type == HEXOS_DISCONNECT)
            // Обработка отключения шины
            break;
    }

    gbus.close(); // Закрытие соединения
}

```

Код отладчика:

```cpp
/*/
/* Код для обработки зашифрованных сообщений
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-bus>

hexos::bus::GlobalBus gbus;

int main() {
    gbus.setDecryptionKey("debug"); // Установка ключа дешифрования
    gbus.connect(); // Подключение к глобальной шине GBus

    if (!gbus.success) {
        // Ошибка
        hexos::gui::showTextbox("Error: " + std::string(gbus.errorMessage));
        return -1;
    }

    while (gbus.notEmpty()) {
        hexos::bus::Intent intent = gbus.get(); // Получение намерения из шины    
        hexos::debug::out(intent.data.repr()); // Вывод бинарного представления системного вызова
    }

    gbus.close(); // Закрытие соединения
}

```

Таким образом, GBus предоставляет гибкий и безопасный способ обмена данными между различными компонентами операционной системы, а также позволяет легко интегрировать системные вызовы и шифрование сообщений.
