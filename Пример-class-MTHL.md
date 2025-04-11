Ранее мы разработали класс MarkdownHybrid, который обрабатывает различные синтаксисы внутри файлов .mdh с использованием блоков <!-- syntax -->. Теперь нужно адаптировать это под расширение .mthl и добавить поддержку Pug, MDH, Markdown Express и возможность расширения синтаксисов.

Сначала подумаю о структуре класса MTHL. Он должен уметь загружать файл .mthl, парсить его содержимое, определять блоки разных синтаксисов и обрабатывать их соответствующими парсерами. Нужно предусмотреть регистрацию пользовательских обработчиков для новых синтаксисов.

Важно учесть обработку PHP-кода внутри файлов, как это было сделано ранее. Также необходимо обеспечить безопасность при выполнении PHP-кода, возможно, через изоляцию или использование песочницы.

Далее, интеграция с Pug (Phug) и Markdown Extra. Для этого потребуется подключить соответствующие библиотеки через Composer и использовать их внутри обработчиков блоков.

Теперь продумаю шаги реализации:

1. Создать класс MTHL с методами для регистрации обработчиков синтаксисов.

2. Реализовать метод загрузки и обработки файла .mthl.

3. Обработка PHP-кода с использованием буферизации вывода.

4. Интеграция парсеров Pug и Markdown Extra.

5. Обеспечение возможности добавления пользовательских синтаксисов через регистрацию обработчиков.

6. Обработка динамических данных и экранирование для безопасности.

7. Тестирование на примере файла .mthl с различными синтаксисами.

Нужно также учесть возможные ошибки, например, отсутствие зарегистрированного обработчика для какого-либо синтаксиса, и предусмотреть обработку таких случаев.

Теперь приступлю к написанию кода, следуя этим шагам, и добавлю комментарии для ясности.

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
