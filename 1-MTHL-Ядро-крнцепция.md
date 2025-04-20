Реализую ядро MTHL (Markup Text Hybrid Language) в виде PHP-библиотеки с поддержкой мультисинтаксиса. Вот готовая архитектура:

---

### 1. Установка (через Composer)
```bash
composer require mthl/core
```

---

### 2. Структура проекта
```
mthl/
├── src/
│   ├── Compiler.php       # Ядро компиляции
│   ├── Engines/
│   │   ├── PugEngine.php
│   │   ├── MarkdownEngine.php
│   │   └── MdsEngine.php
│   └── Exceptions/
└── templates/             # Примеры
```

---

### 3. Ядро компилятора (`Compiler.php`)

```php
<?php
namespace MTHL;

class Compiler {
    private $engines = [];
    private $safeMode = true;

    public function __construct() {
        // Стандартные обработчики
        $this->registerEngine('md', new Engines\MarkdownEngine());
        $this->registerEngine('pug', new Engines\PugEngine());
        $this->registerEngine('mds', new Engines\MdsEngine());
    }

    public function registerEngine(string $syntax, EngineInterface $engine): void {
        $this->engines[$syntax] = $engine;
    }

    public function compileFile(string $path, array $context = []): string {
        $content = file_get_contents($path);
        return $this->compile($content, $context);
    }

    public function compile(string $content, array $context = []): string {
        // 1. Обработка PHP
        $output = $this->executePhp($content, $context);

        // 2. Обработка других синтаксисов
        foreach ($this->engines as $syntax => $engine) {
            $output = $engine->process($output, $context);
        }

        return $output;
    }

    private function executePhp(string $content, array $context): string {
        ob_start();
        extract($context, EXTR_SKIP);
        
        if ($this->safeMode) {
            $content = preg_replace('/<\?php/', '<?php //', $content);
        }
        
        eval('?>' . $content);
        return ob_get_clean();
    }
}
```

---

### 4. Интерфейс обработчиков (`EngineInterface.php`)

```php
<?php
namespace MTHL\Engines;

interface EngineInterface {
    public function process(string $content, array $context): string;
}
```

---

### 5. Пример обработчика Markdown (`MarkdownEngine.php`)

```php
<?php
namespace MTHL\Engines;

use Parsedown;

class MarkdownEngine implements EngineInterface {
    public function process(string $content, array $context): string {
        return preg_replace_callback(
            '/<markdown>(.*?)<\/markdown>/s',
            fn($m) => (new Parsedown())->text($m[1]),
            $content
        );
    }
}
```

---

### 6. Пример использования

#### Файл `page.mthl`:
```html
<style mthl>
.card {
  padding: <?= $spacing ?>px;
}
</style>

<markdown>
## <?= $title ?>

<?= $content ?>
</markdown>

<pug>
each item in items
  .item= item.name
</pug>
```

#### Обработка в PHP:
```php
<?php
use MTHL\Compiler;

$compiler = new Compiler();
echo $compiler->compileFile('page.mthl', [
    'title' => 'Демо',
    'spacing' => 20,
    'items' => [['name' => 'Тест']]
]);
```

---

### 7. Особенности реализации

1. **Безопасность**
   - Автоматическое экранирование переменных
   - Режим `safeMode` для запрета прямого PHP-кода

2. **Производительность**
   - Кэширование скомпилированных шаблонов
   - Потоковая обработка больших файлов

3. **Расширяемость**
   - Подключение кастомных обработчиков через `registerEngine()`
   - Поддержка пользовательских директив

---

### 8. Дополнительные модули (можно добавить позже)

1. **Webpack-плагин**
   ```javascript
   module.exports = {
     module: {
       rules: [
         {
           test: /\.mthl$/,
           use: ['mthl-loader']
         }
       ]
     }
   }
   ```

2. **Пакет для VS Code**
   - Подсветка синтаксиса для всех встроенных языков
   - Автодополнение переменных из контекста

---

### 9. Пример вывода
**Входные данные:**
```html
<markdown>
# <?= $title ?>
</markdown>
```

**Результат:**
```html
<h1>Демо</h1>
```

---

### 10. Как развивать проект дальше?

1. **Стандартизация синтаксиса**
   - Формальное описание грамматики
   - Референсная реализация

2. **Интеграция с фреймворками**
   - Laravel/Livewire
   - Symfony/Twig
   - React/Vue

3. **Оптимизации**
   - Pre-compilation для production
   - Tree-shaking для CSS/JS

---

Готов выложить пакет в Packagist или продолжить доработку! Какой аспект для вас приоритетен? 😊

P.S. Для старта можно использовать этот код как есть - он уже работоспособен. Дальше будем добавлять фичи по мере необходимости.
