---
title:  "Стандартная библиотека"
layout: "chapter"
---

Эта глава — своего рода «швейцарский нож» разработчика. Стандартная библиотека Go (она же **stdlib**) настолько мощная, что во многих случаях вам не понадобятся сторонние зависимости.

Реальный мир программирования — это умение эффективно использовать готовые кирпичики. В Go стандартная библиотека считается одной из лучших в индустрии: она лаконична, быстра и безопасна.

## Работа со строками

Пакет `strings` содержит всё необходимое для манипуляций с текстом.

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    fmt.Println(
        strings.Contains("test", "es"),  // true
        strings.Count("test", "t"),      // 2
        strings.HasPrefix("test", "te"), // true
        strings.Join([]string{"a", "b"}, "-"), // "a-b"
        strings.Replace("aaaa", "a", "b", 2),  // "bbaa"
        strings.Split("a-b-c-d-e", "-"),       // []string{"a","b","c","d","e"}
        strings.ToUpper("test"),               // "TEST"
    )
}

```

> [!TIP] Производительность
> Если вам нужно склеить тысячи строк в цикле, не используйте `+` или `Join`. Используйте **`strings.Builder`** — это гораздо быстрее и эффективнее расходует память.

---

## Ввод / Вывод (I/O)

Два главных интерфейса в Go — это `io.Reader` (чтение) и `io.Writer` (запись).

* `os.File`, `net.Conn`, `bytes.Buffer` — все они реализуют эти интерфейсы.
* Функция `io.Copy(dst, src)` позволяет перекачивать данные из любого "читателя" в любого "писателя".

---

## Файловая система

> [!WARNING] Важное обновление
> Функции из старого пакета `ioutil` теперь находятся в пакетах **`os`** (работа с файлами) и **`io`** (вспомогательные функции).

### Чтение и запись файлов

Самый простой способ прочитать файл целиком в 2026 году:

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    // Раньше было ioutil.ReadFile, теперь os.ReadFile
    bs, err := os.ReadFile("test.txt")
    if err != nil {
        return
    }
    fmt.Println(string(bs))
}

```

### Создание файла

```go
file, err := os.Create("test.txt")
if err == nil {
    defer file.Close()
    file.WriteString("Привет, Go!")
}

```

### Обход папок (Walk)

Для рекурсивного обхода папок лучше использовать `filepath.WalkDir` (он быстрее, чем старый `Walk`):

```go
filepath.WalkDir(".", func(path string, d os.DirEntry, err error) error {
    fmt.Println(path, d.IsDir())
    return nil
})

```

---

## Ошибки (Errors)

Современный Go поощряет "оборачивание" ошибок для сохранения контекста:

```go
import "fmt"

func main() {
    err := fmt.Errorf("ошибка в модуле X: %w", baseError)
}

```

Спецификатор `%w` позволяет позже проверить "корень" ошибки через `errors.Is` или `errors.As`.

---

## Сортировка

Начиная с Go 1.21, для простых срезов лучше использовать новый пакет **`slices`**:

```go
import "slices"

func main() {
    ints := []int{4, 2, 1, 5}
    slices.Sort(ints) // Быстро и просто (Generics!)
}

```

Для сложных структур по-прежнему используется пакет `sort`, где нужно реализовать интерфейс с методами `Len`, `Less` и `Swap`.

---

## Сетевое программирование

### HTTP Сервер

Go позволяет поднять сервер буквально в несколько строк:

```go
package main

import (
    "net/http"
    "io"
)

func hello(w http.ResponseWriter, r *http.Request) {
    io.WriteString(w, "<h1>Hello World!</h1>")
}

func main() {
    // В Go 1.22+ роутинг стал мощнее (поддержка методов GET/POST в HandleFunc)
    http.HandleFunc("GET /hello", hello)
    http.ListenAndServe(":9000", nil)
}

```

### RPC (Remote Procedure Call)

Хотя стандартный `net/rpc` всё еще в библиотеке, в современной разработке для взаимодействия между сервисами чаще используют **gRPC** или **Twitch RPC**. Стандартный пакет хорош для простых внутренних инструментов на Go.

---

## Командная строка (Flags)

Для создания CLI-утилит используется пакет `flag`:

```go
max := flag.Int("max", 100, "Максимальное значение")
flag.Parse()
fmt.Println(*max) // Обратите внимание: флаг возвращает указатель

```

---

## Синхронизация: Mutex

Если горутины обращаются к одной переменной, нужно использовать **`sync.Mutex`**, чтобы избежать состояния гонки (race condition).

```go
var (
    mu    sync.Mutex
    count int
)

func increment() {
    mu.Lock()
    count++
    mu.Unlock()
}

```

> [!TIP] defer с Mutex
> Всегда пишите `mu.Lock(); defer mu.Unlock()`. Это гарантирует разблокировку, даже если в коде случится паника.

---

## Задачи

* [ ] **I/O:** Напишите программу-аналог `cat`, которая читает файл и выводит его в консоль, используя `io.Copy` и `os.Stdout`.
* [ ] **Хэши:** Сравните два файла по их контрольной сумме SHA-256 (пакет `crypto/sha256`).
* [ ] **HTTP:** Создайте сервер, который по адресу `/time` отдает текущее время в формате JSON.
* [ ] **Flags:** Напишите утилиту, которая принимает флаг `-name` и выводит "Привет, [name]!".

## Полезные ссылки

1. [Go Packages (pkg.go.dev)](https://pkg.go.dev/std) — официальная документация всех пакетов.
2. [Go by Example: Strings](https://gobyexample.com/string-functions)
3. [Go Blog: I/O Package update](https://www.google.com/search?q=https://go.dev/blog/io-util-deprecated) — почему `ioutil` больше не нужен.
4. [Standard Library Cookbook](https://refactoring.guru/design-patterns/go) — паттерны проектирования в Go.

---

```