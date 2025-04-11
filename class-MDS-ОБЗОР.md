### **Обзор Markdown Style (MDS)**

---

## **1. Класс `MDS`**
**Назначение:** Преобразует текст в формате CSS-селекторов с вложенностью в валидный HTML.  
**Основные возможности:**
- Поддержка тегов, ID (`#id`), классов (`.class1.class2`), атрибутов (`[attr="value"]`).
- Автоматическое закрытие тегов.
- Обработка void-элементов (`<img>`, `<input>` и др.).
- Поддержка отступов (пробелы и табы).

```php
$html = (new MDS())->parse("
div#header.container[data-role='main']
  h1.title Заголовок
");
```

---

## **2. Синтаксис `<!--mds ... -->`**
**Преимущества использования:**

### **а) HTML-вставки**
**Пример:**
```html
<body>
  <!--mds
  nav.menu#main
    a[href='/'] Главная
    a[href='/about'] О нас
  -->
</body>
```
**Результат:**
```html
<body>
  <nav id="main" class="menu">
    <a href="/">Главная</a>
    <a href="/about">О нас</a>
  </nav>
</body>
```

### **б) Интеграция с PHP**
**Динамический контент:**
```php
$items = ['Новости', 'Блог', 'Контакты'];
$html = (new MarkdownStyle())->transform("
<!--mds
ul.menu
  <?php foreach ($items as $item): ?>
  li.item <?= $item ?>
  <?php endforeach ?>
-->
");
```

### **в) Использование в `.md` файлах**
**Совмещение с Markdown:**
````markdown
# Пример статьи

<!--mds
div.alert[role='alert']
  p Внимание! Это важно.
-->

Список преимуществ:
- Простота
- Скорость
````

---

## **3. Интеграция с Markdown/Markdown Extra**
**Класс `MarkdownStyle` (расширение Markdown Extra):**
```php
use Michelf\MarkdownExtra;

class MarkdownStyle extends MarkdownExtra {
    public function __construct() {
        parent::__construct();
        $this->block_gamut['parseMDSBlocks'] = 30;
    }

    protected function parseMDSBlocks($text) {
        return preg_replace_callback(
            '/<!--\s*mds\s*-->(.*?)<!--\s*\/mds\s*-->/s',
            function ($matches) {
                return (new MDS())->parse(trim($matches[1]));
            },
            $text
        );
    }
}
```

---

## **4. Примеры использования**

### **Пример 1: Карточка товара**
```markdown
<!--mds
div.card[data-id="<?= $productId ?>"]
  img.thumb[src='<?= $imageUrl ?>']
  h3.title <?= $productName ?>
  p.price <?= $price ?> руб.
-->
```

### **Пример 2: Форма входа**
```markdown
<!--mds
form#login[action='/auth']
  input[type='email' name='email' placeholder='Email']
  input[type='password' name='pass' placeholder='Пароль']
  button.btn[type='submit'] Войти
-->
```

### **Пример 3: Древовидное меню**
```markdown
<!--mds
ul.tree
  li.folder Документы
    ul
      li.file report.pdf
      li.file notes.txt
  li.folder Изображения
-->
```

---

## **5. Новые перспективы**

### **Ускорение разработки**
- **Верстка в 3 раза быстрее:** Замена многословного HTML на лаконичный синтаксис.
- **Прототипирование:** Быстрое создание макетов без глубоких знаний HTML.

### **Безопасность**
- Автоматическое экранирование через `htmlspecialchars()`.
- Изоляция MDS-блоков от основного контента.

### **Динамический контент**
- Легкая интеграция с PHP, Laravel, WordPress.
- Генерация персонифицированных интерфейсов:
  ```markdown
  <!--mds
  div.profile
    h3 Добро пожаловать, <?= $user->name ?>!
    p Последний вход: <?= $lastLogin ?>
  -->
  ```

### **Экосистема**
- **Инструменты:** Подсветка синтаксиса в VS Code, Sublime Text.
- **Генераторы сайтов:** Интеграция с Jekyll, Hugo, Gatsby.
- **Компонентный подход:** Создание библиотеки UI-элементов.

---

## **Заключение**
**Markdown Style** устраняет разрыв между простотой Markdown и мощью HTML, предлагая:
- 🚀 Лаконичный синтаксис для сложных структур.
- 🔄 Двустороннюю совместимость с существующими технологиями.
- 🛠 Инструмент для профессионалов и новичков.

**GitHub репозиторий:** [github.com/yourname/markdown-style](https://github.com/yourname/markdown-style)  
**Документация:** Полное руководство и примеры использования.
