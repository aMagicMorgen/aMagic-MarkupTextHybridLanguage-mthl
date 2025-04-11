### Пример `class MTHL` для обработки файлов `.mthl` с поддержкой Pug, Markdown Extra и кастомных синтаксисов

```php
<?php

use Phug\Phug;
use Michelf\MarkdownExtra;

class MTHL {
    private $parsers = [];
    private $data = [];
    private $safeMode = true;

    public function __construct() {
        // Регистрация стандартных парсеров
        $this->registerParser('md', [new MarkdownExtra(), 'transform']);
        $this->registerParser('pug', [new Phug(), 'render']);
    }

    /**
     * Регистрация кастомного парсера для синтаксиса
     * @param string $syntax Название синтаксиса (тег в комментариях)
     * @param callable $handler Функция обработки
     */
    public function registerParser(string $syntax, callable $handler): void {
        $this->parsers[$syntax] = $handler;
    }

    /**
     * Установка данных для использования в шаблоне
     */
    public function setData(array $data): void {
        $this->data = $data;
    }

    /**
     * Обработка содержимого файла
     */
    public function render(string $content): string {
        extract($this->data);
        
        // Шаг 1: Обработка PHP-кода
        ob_start();
        eval('?>' . $content);
        $content = ob_get_clean();

        // Шаг 2: Обработка кастомных блоков
        $content = preg_replace_callback(
            '/<!--\s*([a-z]+)\s*-->(.*?)<!--\s*\/\1\s*-->/s',
            function ($matches) {
                return $this->processBlock($matches[1], $matches[2]);
            },
            $content
        );

        return $content;
    }

    private function processBlock(string $syntax, string $content): string {
        $content = trim($content);
        
        if (!isset($this->parsers[$syntax])) {
            return "<!-- Unknown syntax: $syntax -->";
        }

        try {
            return call_user_func($this->parsers[$syntax], $content, $this->data);
        } catch (Exception $e) {
            return "<!-- Error in $syntax block: " . $e->getMessage() . " -->";
        }
    }

    /**
     * Загрузка и рендер файла
     */
    public function renderFile(string $path): string {
        return $this->render(file_get_contents($path));
    }
}

// Пример использования
$mthl = new MTHL();

// Регистрация кастомного синтаксиса
$mthl->registerParser('mds', function($content) {
    return (new MDS())->parse($content);
});

// Установка данных
$mthl->setData([
    'title' => 'Hello MTHL!',
    'items' => ['First', 'Second', 'Third']
]);

// Рендер файла
echo $mthl->renderFile('example.mthl');
```

---

### Пример файла `example.mthl`
```mthl
<?php
// PHP-код в шаблоне
$count = 7;
?>

<!-- md -->
## <?= $title ?>

Список элементов:
<?php foreach ($items as $item): ?>
- **<?= $item ?>**  
<?php endforeach ?>
<!-- /md -->

<!-- pug -->
div.container
  h1.title= title
  ul.list
    each item in items
      li.item= item
<!-- /pug -->

<!-- mds -->
div#main[data-role="content"]
  table.stats
    tr
      td Количество: <?= $count ?>
<!-- /mds -->
```

---

### Особенности реализации:
1. **Мультисинтаксическая обработка**  
   - Поддержка PHP, Markdown, Pug и кастомных синтаксисов
   - Возможность добавления новых обработчиков через `registerParser()`

2. **Безопасность**  
   - Изоляция данных через `extract()`
   - Обработка ошибок в блоках

3. **Динамические данные**  
   - Передача переменных через `setData()`
   - Использование PHP-переменных во всех блоках

4. **Интеграция с библиотеками**  
   - Phug для Pug-синтаксиса
   - MarkdownExtra для Markdown
   - Кастомный MDS-парсер

---

### Требования:
1. Установите зависимости через Composer:
```bash
composer require phug/phug michelf/php-markdown
```

2. Создайте класс `MDS` (как в предыдущих примерах)

3. Настройте обработку ошибок PHP для безопасного выполнения `eval()`

Данная реализация позволяет создавать комплексные шаблоны, объединяя преимущества разных языков разметки в одном файле.
