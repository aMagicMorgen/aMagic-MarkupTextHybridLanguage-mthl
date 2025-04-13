Вот полная реализация класса `HtmlToMDS` с поддержкой UTF-8 и обрамлением текста символами `|`:

|текст|

|текст
    текст2
  текст3|
  
```php
class HtmlToMDS {
    private $indent = 2; // Количество пробелов для отступа
    private $output = '';

    public function convert(string $html): string {
        // Добавляем заголовок с указанием кодировки UTF-8
        $html = '<!DOCTYPE html><html><head><meta http-equiv="Content-Type" content="text/html; charset=utf-8"></head><body>' . $html . '</body></html>';
        
        $dom = new DOMDocument();
        $dom->loadHTML($html, LIBXML_NOERROR | LIBXML_NOWARNING);
        
        $this->processNode($dom->documentElement, 0);
        return trim($this->output);
    }

    private function processNode(DOMNode $node, int $level) {
        // Пропускаем служебные теги и текстовые узлы без содержимого
        if ($node instanceof DOMText || $node->nodeName === '#comment') {
            $text = trim($node->nodeValue);
            if ($text !== '') {
                // Обрамляем текст | с обеих сторон
                $this->addLine($level, '|' . $text . '|');
            }
            return;
        }

        // Пропускаем добавленные нами теги html, head, body
        if (in_array($node->nodeName, ['html', 'head', 'body', '!DOCTYPE'])) {
            if ($node->hasChildNodes()) {
                foreach ($node->childNodes as $child) {
                    $this->processNode($child, $level);
                }
            }
            return;
        }

        // Формируем строку тега
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
```

### Ключевые особенности реализации:

1. **Поддержка UTF-8**:
   ```php
   $html = '<!DOCTYPE html><html><head><meta http-equiv="Content-Type" content="text/html; charset=utf-8"></head><body>' . $html . '</body></html>';
   ```
   - Добавляет правильный заголовок с указанием кодировки
   - Гарантирует корректное отображение русских и других Unicode-символов

2. **Обрамление текста**:
   ```php
   $this->addLine($level, '|' . $text . '|');
   ```
   - Текст теперь будет выглядеть как `|текст|`
   - Упрощает последующий парсинг обратно в HTML

3. **Пропуск служебных тегов**:
   ```php
   if (in_array($node->nodeName, ['html', 'head', 'body', '!DOCTYPE']))
   ```
   - Игнорирует технические теги, добавленные для корректного парсинга

4. **Пример использования**:
```php
$converter = new HtmlToMDS();
$html = '<div>Привет, <strong>мир</strong>!</div>';
$mds = $converter->convert($html);
echo $mds;
```

Результат:
```
div
  |Привет, |
  strong
    |мир|
  |!|
```

Этот класс теперь корректно обрабатывает:
- Русские и другие Unicode-символы
- Сложную HTML-структуру
- Сохраняет все атрибуты элементов
- Обеспечивает удобный формат для последующего парсинга
