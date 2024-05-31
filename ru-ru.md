# HexOS GBus - Глобальная шина HexOS

## Что это такое?

Глобальная шина Hexos - это глобальная система очередей намерений, используемая для обмена информацией между различными сущностями ОС.

## Как использовать?

Вы можете поставить намерения на шину, и ОС позаботится о остальном.

## Локальные шины

Вы можете создавать локальные шины, и поставить намерения на них. Вы можете добавлять несколько сущностей к шине, и ОС будет отправлять намерения на все эти сущности.

### Создание локальной шины

```cpp
/*/
/* Простой пример широковещателя локальной шины
/*/

#include <hexos-consts>
#include <hexos-bus>

hexos::bus::LocalBus lbus;

int main() {
    lbus.init("dbus-name");
    lbus.count(2); // принимаем максимум 2 сущностей (включая инициатор)
    lbus.require(2); // потребуем минимум 2 сущностей (включая инициатор)
    lbus.accept(); // принимаем сущности
    lbus.wait(); // ждем, пока сущности подключены

    // поставим намерения на шину
    lbus.put(hexos::bus::Intent(HEXOS_BROADCAST_TEXT, "Hello world!")); // отправляем текстовое сообщение на все сущности
    lbus.put(hexos::bus::Intent(HEXOS_BROADCAST_COMMAND, "show textbox")); // отправляем команду на все сущности

    lbus.put(hexos::bus::Intent(HEXOS_DISCONNECT)); // отключить шину
    lbus.wait(); // ждем, пока все сущности отключены
    lbus.destroy(); // удалить шину
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
    lbus.connect("dbus-name"); // подключиться к шине

    if (!lbus.success) {
        // ошибка
        return -1;
    }

    while (true) {
        hexos::bus::Intent intent = lbus.get(); // получить намерение из шины
        if (intent.type == HEXOS_BROADCAST_TEXT)
            // обработать текстовое сообщение
            message = intent.data;
        else if (intent.type == HEXOS_BROADCAST_COMMAND)
            // обработать команду
            if (std::string(intent.data) == "show textbox")
                // показать текстовое поле
                hexos::gui::showTextbox(message);
        else if (intent.type == HEXOS_DISCONNECT)
            // обработать отключение шины
            break;
    }

    lbus.close(); // закрыть соединение
}

```

## Виртуальные шины (очереди, предоставляемые операционной системой)

Вы можете создавать виртуальные шины, и поставить любые данные на них. Вы можете добавлять несколько сущностей к шине, и ОС будет отправлять намерения на все эти сущности.

### Создание виртуальной шины

```cpp
/*/
/* Простой пример широковещателя-приемника виртуальной шины
/*/

#include <hexos-consts>
#include <hexos-bus>

hexos::bus::VirtualBus vbus;

int main() {
    vbus.init("dbus-name");

    vbus.put("Hello world!"); // отправляем строковое сообщение на все сущности
    vbus.put("Blah blah blah");
    vbus.put("Hi there!");

    while (vbus.notEmpty()) {
        std::string message = vbus.get(); // получить строковое сообщение из шины
        hexos::gui::showTextbox(message);
    }

    vbus.close(); // закрыть соединение
}

```

## Ошибки

Когда вы инициализируете шину, вы можете получить ошибку, и ее можно проверить.

```cpp
/*/
/*  Мы специально создадим шину с зарезервированным именем
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-bus>

hexos::bus::VirtualBus vbus;

int main() {
    vbus.init("osgbus0"); // попробуем создать шину с именем "osgbus0"
    if (!vbus.success) {
        // Упс, ошибка!
        hexos::gui::showTextbox("Error: " + std::string(vbus.errorMessage));
        return -1;
    }
}

```

покажет:

```
Error: Bus with name "osgbus0" already exists
```

У вас есть несколько констант для проверки ошибок:

```cpp
// hexos-const part
#define HEXOS_BUS_ERROR_NONE 0
#define HEXOS_BUS_ERROR_NAME_EXISTS 1
#define HEXOS_BUS_ERROR_ALREADY_CONNECTED 2
#define HEXOS_BUS_ERROR_NOT_CONNECTED 3
#define HEXOS_BUS_ERROR_INVALID_DATA 4
#define HEXOS_BUS_ERROR_VIOLATION 5
#define HEXOS_BUS_ERROR_UNKNOWN 6
#define HEXOS_BUS_ERROR_TIMEOUT 7
```

## Глобальная шина Gbus

Глобальная шина Gbus - это шина, используемая для обмена информацией между различными сущностями ОС. Она используется для обмена информацией между различными процессами и системами API.

### Подключение к глобальной шине Gbus

```cpp
/*/
/*  Мы хотим подключиться к глобальной шине Gbus
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-bus>

hexos::bus::GlobalBus gbus;

int main() {
    gbus.connect(); // подключиться к глобальной шине Gbus

    if (!gbus.success) {
        // Упс, ошибка!
        hexos::gui::showTextbox("Error: " + std::string(gbus.errorMessage));
        return -1;
    }

    // мы хотим отправить намерение на всех сущностях в системе
    gbus.put(HEXOS_BROADCAST_TEXT, "Hello world!", append_credentials = true);
    /*                                             ^^^^^^^^^^^^^^^^^^
     *                                             добавлять дополнительные данные
     *                                             к сообщению
     *
     *                                             это добавит дополнительные данные
     *                                             пользователя к сообщению, такие
     *                                             как тип автора и идентификатор
     */

    gbus.close(); // закрыть соединение
}

```

### Получение сообщений из глобальной шины Gbus

```cpp
/*/
/*  Мы хотим получить сообщение из глобальной шины Gbus
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-bus>

hexos::bus::GlobalBus gbus;

int main() {
    gbus.connect(); // Подключиться к глобальной шине Gbus

    if (!gbus.success) {
        // Ошибка
        hexos::gui::showTextbox("Error: " + std::string(gbus.errorMessage));
        return -1;
    }

    while (gbus.notEmpty()) {
        hexos::bus::Intent intent = gbus.get(); // Получить намерение из шины
        if (intent.type == HEXOS_BROADCAST_TEXT)
            // обработать текстовое сообщение
            hexos::gui::showTextbox(intent.data);
        else if (intent.type == HEXOS_DISCONNECT)
            // обработать отключение шины
            break;
    }

    gbus.close(); // закрыть соединение
}

```

## Разница между syscall системной API (HWSAPI) и Gbus системной API (SWSAPI)

### syscall системной API (HWSAPI)

HWSAPI - это *HardWare System API*, использует `syscall` инструкцию для связи с ядром.

```cpp
/*/
/*  We want to call system function
/*/

#include <hexos-consts>
#include <hexos-assembly>

int main() {
    hexos::assembly::syscall(HEXOS_DEBUG_OUT, "Hello world!");
}

////////////////////  ALTERNATIVE  ////////////////////
/*/
/*  We want to call system function
/*/

#include <hexos-debug>

int main() {
    // hexos::internals::SWSAPI is false by default
    hexos::debug::out("Hello world!");
}

```

### Gbus системной API (SWSAPI)

SWSAPI - это *SoftWare System API*, использует Gbus системную API для связи с ядром.

Для тех, кто интересуется, что разница между прямым HWSAPI доступом к функциям и software-wrapped HWSAPI,
Gbus не использует `syscall` инструкцию, оно перемещается в статическую RAM-адресную память как GPIO.


## diffence between syscall system API (HWSAPI) and Gbus system API (SWSAPI)

### syscall system API (HWSAPI)

HWSAPI is a *HardWare System API*, it uses `syscall` instruction to communicate with kernel.

```cpp
/*/
/*  We want to call system function
/*/

#include <hexos-consts>
#include <hexos-assembly>

int main() {
    hexos::assembly::syscall(HEXOS_DEBUG_OUT, "Hello world!");
}

////////////////////  ALTERNATIVE  ////////////////////
/*/
/*  We want to call system function
/*/

#include <hexos-debug>

int main() {
    // hexos::internals::SWSAPI is false by default
    hexos::debug::out("Hello world!");
}

```

### Gbus system API (SWSAPI)

SWSAPI - это *SoftWare System API*, использует Gbus системную API для связи с ядром.

Для тех, кто интересуется, что разница между прямым HWSAPI доступом к функциям и software-wrapped HWSAPI,
Gbus не использует `syscall` инструкцию, оно перемещается в статическую RAM-адресную память как GPIO.

```cpp
/*/
/*  Мы хотим вызвать системную функцию с помощью SWSAPI
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-bus>

hexos::bus::GlobalBus gbus;

int main() {
    gbus.connect(); // Подключиться к глобальной шине Gbus

    if (!gbus.success) {
        // Ошибка
        hexos::gui::showTextbox("Error: " + std::string(gbus.errorMessage));
        return -1;
    }

    gbus.put(HEXOS_EXECSWSAPI, HEXOS_DEBUG_OUT, "Hello world!");

    while (gbus.notEmpty()) {
        hexos::bus::Intent intent = gbus.get(); // Получить намерение из шины
        if (intent.type == HEXOS_DISCONNECT)
            // обработать отключение шины
            break;
    }

    gbus.close(); // закрыть соединение
}

```

Так же, используя HWSAPI, вы не можете обмениваться информацией между различными системами, но используя Gbus, вы можете.
Так как вы хотите использовать, например, дебаггер, вы можете настроить шифрование на шине для селективной отправки отладочной информации только к отладчику,
и использовать SWSAPI вместо HWSAPI для связи с ядром.

Пример:

```cpp
/*/
/*  Мы хотим вызвать системную функцию с помощью SWSAPI для отладки
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-internals>
#include <hexos-debug>

int main() {
    hexos::internals::SWSAPIKey("debug"); // установить ключ шифрования
    hexos::internals::SWSAPI(true); // установить SWSAPI флаг на true

    if (!gbus.success) {
        // Ошибка
        hexos::gui::showTextbox("Error: " + std::string(gbus.errorMessage));
        return -1;
    }

    // мы можем использовать этот формат для представления системной вызова, но будет просто установить флаг `SWSAPI` на true
    // gbus.put(HEXOS_EXECSWSAPI, HEXOS_DEBUG_OUT, "Hello world!");
    hexos::debug::out("Hello world!");

    while (gbus.notEmpty()) {
        hexos::bus::Intent intent = gbus.get(); // Получить намерение из шины    
        if (intent.type == HEXOS_DISCONNECT)
            // обработать отключение шины
            break;
    }

    gbus.close(); // закрыть соединение
}

```

код дебаггера:

```cpp
/*/
/*  Мы хотим вызвать системную функцию с помощью SWSAPI для отладки
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-bus>

hexos::bus::GlobalBus gbus;

int main() {
    gbus.setDecryptionKey("debug"); // установить ключ шифрования
    gbus.connect(); // Подключиться к глобальной шине Gbus

    if (!gbus.success) {
        // Ошибка
        hexos::gui::showTextbox("Error: " + std::string(gbus.errorMessage));
        return -1;
    }

    while (gbus.notEmpty()) {
        hexos::bus::Intent intent = gbus.get(); // Получить намерение из шины    
        hexos::debug::out(intent.data.repr()); // вывести бинарный представление системной вызова
    }

    gbus.close(); // закрыть соединение
}

```

## Шифрование Gbus

Если вы установите ключ шифрования, то все сообщения будут зашифрованы с помощью этого ключа, вы можете прочитать все сообщения из шины.
Если вы установите ключ дешифрования, то вы можете только прочитать сообщения, которые были зашифрованы с помощью этого ключа, ваши сообщения не будут зашифрованы с помощью этого ключа.

P.S.: Вы можете установить ключ шифрования и ключ дешифрования одновременно, и использовать их для прочитания и записи сообщений. (Этот метод рекомендуется для создания подсетевых каналов, которые будут проходить через Gbus), но лучше использовать локальные шины для этого целей.

Не забывайте, что вы можете создать несколько клиентов Gbus в одном процессе, поэтому вы можете получить доступ к глобальной области и зашифрованной локальной области.
