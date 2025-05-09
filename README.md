# .mdh -> .mthl
Спецификация формата MTHL (Markup Text Hybrid Language)

- ЭТО ИДЕЯ ДЛЯ БУДУЮЩЕЙ РАЗРАБТКИ!
- `*.mthl` это гибридный файл для написания на разном синтаксисе
    - `html`, `Markdown`, `Markdown Extra`, `Markdown Style` (это мой новый синтаксис на основе CSS селекторов), `PHP`
    - PUG и другие синтаксисы которые можно регистрировать и вводить для письма
- Markdown Hybrid (это мой новый подход писать без переключения на отдельные синтаксисы)
    - `*.mdh (Markdown Hibrid)` это расширение файла для синтаксиса `html`, `Markdown`, `Markdown Extra`, `Markdown Style`, `PHP` - родитель для `*.mthl`
- Запуск будет производиться через `*.php` файл через `class MHTH`

### **Спецификация формата MTHL (Markup Text Hybrid Language)**  
**Версия:** 1.0 (Draft)  
**Дата публикации:** 11.04.2025  
**Автор:** Алексей Владимирович Нечаев (23.09.1972 г.р., г. Москва, тел. +79990039023)  
**Лицензия:** [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)  

---

## **1. Введение**
**MTHL** (Markup Text Hybrid Language) — гибридный язык разметки, объединяющий различные синтаксисы (Markdown, HTML, CSS-селекторы, код) в одном файле.  
**Цель:** Упростить создание комплексных документов и веб-страниц без переключения между языками.

---

## **2. Базовый синтаксис**
### **2.1. Блоки синтаксиса**
Любой код внутри тегов `<!-- syntax --> ... <!-- /syntax -->` обрабатывается парсером, указанным в `syntax`.  
```mthl
<!-- markdown -->
## Заголовок
- Список
<!-- /markdown -->

<!-- html -->
<div class="alert">Внимание!</div>
<!-- /html -->

<!-- css -->
.card { padding: 10px; }
<!-- /css -->
```

### **2.2. Встроенные выражения**
Динамические данные через `<?= ... ?>`:  
```mthl
<!-- markdown -->
Привет, <?= $username ?>!
<!-- /markdown -->
```

---

## **3. Основные элементы**
### **3.1. Поддерживаемые синтаксисы**
| **Тип**      | **Описание**                          | **Пример**                     |
|--------------|---------------------------------------|--------------------------------|
| `markdown`   | Стандартный CommonMark                | `## Заголовок`                 |
| `html`       | HTML5                                 | `<div class="box"></div>`      |
| `css`        | CSS-правила                           | `.button { color: red; }`      |
| `mds`        | CSS-селекторы (а-ля MDS)              | `div#header.header[data-role]` |
| `php`        | PHP-код (исполняется на сервере)      | `<?= date('Y') ?>`             |
| `javascript` | Клиентский JS (в `<script>`)          | `console.log('Hello')`         |

### **3.2. Кастомизация парсера**
Добавьте обработчик для нового синтаксиса:  
```php
$parser->registerSyntax('vue', function($content) {
    return compileVueComponent($content);
});
```
Использование:  
```mthl
<!-- vue -->
<template>
  <button @click="count++">{{ count }}</button>
</template>
<!-- /vue -->
```

---

## **4. Обработка файла**
### **4.1. Алгоритм рендеринга**
1. Извлечь все блоки `<!-- syntax -->`.  
2. Обработать каждый блок соответствующим парсером.  
3. Заменить блоки результатом обработки.  
4. Собрать итоговый HTML.

### **4.2. Приоритеты**
1. PHP (`<?= ... ?>`)  
2. Пользовательские синтаксисы (например, `vue`)  
3. Стандартные синтаксисы (markdown/html/css)  

---

## **5. Вставки кода**
### **5.1. Инлайн-код**
Используйте обратные кавычки с указанием языка:  
````mthl
```php
$name = "MTHL";
echo $name;
```
````

### **5.2. Подсветка синтаксиса**
Добавьте атрибут `highlight`:  
```mthl
<!-- html highlight="prism" -->
<nav class="menu"></nav>
<!-- /html -->
```

---

## **6. Безопасность**
- **Экранирование по умолчанию:** Все динамические данные (`<?= ... ?>`) экранируются через `htmlspecialchars()`.  
- **Отключение экранирования:** Явное указание `{raw}`:  
  ```mthl
  <?= {raw} $trustedHtml ?>
  ```

---

## **7. Пример документа**
```mthl
<!DOCTYPE html>
<!-- html -->
<head>
  <!-- css -->
  .title { color: blue; }
  <!-- /css -->
</head>
<!-- /html -->

<!-- markdown -->
# <?= $pageTitle ?>
<!-- /markdown -->

<!-- mds -->
div.widget
  h3 Статистика
  ul
    li Просмотры: <?= $stats['views'] ?>
<!-- /mds -->
```

---

## **8. Реализации**
- **Эталонный парсер:** [GitHub/mthl-parser](class-MDS-MarkdownStyle.md) (PHP)  
- **Плагины:** Поддержка Vue/React через npm-пакеты.  

---

## **9. Лицензия и контакты**
- **Лицензия:** Открытая (CC BY-SA 4.0).  
- **Контакты:** Алексей Нечаев, nechaev72@list.ru, +79990039023.  

---

**© 2025 MTHL Working Group.**  
*Спецификация находится в разработке. Актуальная версия: https://mthl.org/spec.*
