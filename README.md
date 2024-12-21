# DAMDINOV_LAB5

Задание 1.

![image](https://github.com/user-attachments/assets/6cc14521-c97f-414d-b93f-040af82e5e7d)

![image](https://github.com/user-attachments/assets/ac52b023-f821-4c91-bfd9-9e2fe3982b5b)

![image](https://github.com/user-attachments/assets/03466106-f366-4219-8dfd-4c8dd7b7f15a)

![image](https://github.com/user-attachments/assets/6598d27d-daa8-4638-865e-21ad13ffbb38)

![image](https://github.com/user-attachments/assets/4a3c5521-c078-4773-a73c-8ed4020d6445)

![Uploading image.png…]()

![Uploading image.png…]()

Задание 2.
Этот код представляет собой пример программы на языке C, который использует несколько уязвимостей и слабых мест. Рассмотрим его компоненты:

Использование функции fgets с неправильным размером буфера:
```
fgets(buf, 133, stdin);
```
Функция fgets не может записать больше символов, чем указано в параметре, то есть максимальный размер буфера будет 128 байт (по размеру массива buf). Однако, при вызове fgets указано, что она может прочитать 133 символа. Это может привести к выходу за границы буфера и переполнению памяти.

Потенциальная угроза: переполнение буфера, что может привести к переписыванию памяти программы и, как следствие, к выполнению произвольного кода (например, если атакующий вводит данные, чтобы изменить указатель функции).

Возможность изменения указателя на функцию:
```
void (*func)() = sup;
```
Указатель на функцию func инициализируется функцией sup. Однако, эта переменная уязвима для переполнения буфера, так как атакующий может переписать func и заставить программу вызвать любую другую функцию, например, shell().

Реализация функции shell:
```
void shell() {
    setreuid(geteuid(), geteuid());
    system("/bin/bash");
}
```
Эта функция вызывает /bin/bash с правами пользователя, установленными через setreuid. Потенциально это уязвимо для злоумышленников, если они могут вызвать эту функцию с повышенными правами, поскольку могут получить доступ к командной строке с правами другого пользователя.

Потенциальная угроза: злоумышленник может вызвать эту функцию через переполнение буфера, получив доступ к оболочке с привилегиями.

Отсутствие проверки на ошибки: код не проверяет возвращаемые значения из функций, таких как fgets, что может привести к неожиданному поведению в случае ошибок при чтении ввода.

Исправленный код 
```
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>

void shell() {
    setreuid(geteuid(), geteuid());
    system("/bin/bash");
}
void sup() {
    printf("Hey dude ! Waaaaazzaaaaaaaa ?!\n");
}
void main() {
    int var;
    void (*func)() = sup;
    char buf[128];
    if (fgets(buf, sizeof(buf), stdin) == NULL) {
        perror("fgets failed");
        exit(1);
    }
    if (strlen(buf) >= sizeof(buf)) {
        printf("Input buffer overflow detected!\n");
        exit(1);
    }
    func(); 
}
```
