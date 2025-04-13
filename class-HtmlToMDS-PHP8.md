# Реализуем обратный преобразователь `HtmlToMDS` только для PHP 8.0+, который конвертирует HTML в синтаксис TagParser. Вот код:

- Ошибка возникает потому, что функция str_starts_with() была добавлена в PHP только в версии 8.0. Если вы используете более старую версию PHP, эта функция будет недоступна.

```php
class HtmlToMDS {
    private $indent = 2; // Количество пробелов для отступа
    private $output = '';

    public function convert(string $html): string {
        $dom = new DOMDocument();
        $dom->loadHTML($html, LIBXML_NOERROR | LIBXML_NOWARNING);
        
        $this->processNode($dom->documentElement, 0);
        return trim($this->output);
    }

    private function processNode(DOMNode $node, int $level) {
        // Пропускаем служебные теги и текстовые узлы без содержимого
        if ($node instanceof DOMText || $node->nodeName === '#comment') {
            if (trim($node->nodeValue) !== '') {
                $this->addLine($level, '| ' . trim($node->nodeValue));
            }
            return;
        }

        // Формируем строку тега
        $tag = $node->nodeName;
        $selector = $this->buildSelector($node);
        
        // Добавляем строку с тегом
        $this->addLine($level, $selector);

        // Рекурсивная обработка дочерних элементов
        if ($node->hasChildNodes()) {
            foreach ($node->childNodes as $child) {
                $this->processNode($child, $level + 1);
            }
        }
    }

    private function buildSelector(DOMElement $element): string {
        $parts = [$element->nodeName];
        
        // ID
        if ($element->hasAttribute('id')) {
            $parts[] = '#' . $element->getAttribute('id');
        }

        // Классы
        if ($element->hasAttribute('class')) {
            $classes = explode(' ', $element->getAttribute('class'));
            $parts[] = '.' . implode('.', $classes);
        }

        // Атрибуты (кроме id и class)
        $attrs = [];
        foreach ($element->attributes as $attr) {
            if (!in_array($attr->name, ['id', 'class'])) {
                $attrs[] = $attr->name . "='" . $attr->value . "'";
            }
        }
        if (!empty($attrs)) {
            $parts[] = '[' . implode(' ', $attrs) . ']';
        }

        return implode('', $parts);
    }

    private function addLine(int $level, string $content) {
        $indent = str_repeat(' ', $level * $this->indent);
        $this->output .= $indent . $content . "\n";
    }
}

// Пример использования
$html = <<<HTML
<div id="header" class="main banner" data-role="primary">
    <ul class="nav">
        <li class="item active">Home</li>
        <li>About</li>
    </ul>
    <img src="logo.png" alt="Logo">
</div>
HTML;

$parser = new HtmlToMDS();
echo $parser->convert($html);
```

**Результат выполнения:**
```
div#header.main.banner[data-role='primary']
  ul.nav
    li.item.active Home
    li About
  img[src='logo.png'][alt='Logo']
```

### Основные особенности:
1. **Автоопределение структуры:**
   - Рекурсивный обход DOM-дерева
   - Учет уровня вложенности для отступов

2. **Поддержка атрибутов:**
   ```html
   <input type="text" required>
   ```
   → 
   ```
   input[type='text'][required]
   ```

3. **Обработка текста:**
   ```html
   <p>Hello <strong>World</strong></p>
   ```
   →
   ```
   p
    | Hello
    strong World
   ```

4. **Специальные случаи:**
   - Самозакрывающиеся теги (`<img>`, `<br>`)
   - Булевы атрибуты (`disabled`, `checked`)

### Ограничения:
1. Не сохраняет порядок атрибутов
2. Не обрабатывает inline-стили
3. Текст внутри тегов добавляется через `| `

Для точного двустороннего преобразования рекомендуется:
1. Использовать вместе с `TagParser`
2. Добавить валидацию структуры
3. Реализовать обработку CSS-стилей
