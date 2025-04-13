# Вот полная реализация класса HtmlToMDS с поддержкой UTF-8 и обрамлением только слева текста символом | text PHP 7.4

Вот исправленная версия класса `MDS` с заменой `str_starts_with()` на альтернативную проверку:

```php
class MDS {
    private $lines = [];
    private $currentLine = 0;

    public function toHtml(string $mds): string {
        // Нормализация строк: удаление пустых строк и лишних пробелов
        $this->lines = array_values(array_filter(array_map('trim', explode("\n", $mds))));
        $this->currentLine = 0;
        
        $dom = new DOMDocument('1.0', 'UTF-8');
        $dom->formatOutput = true;
        
        // Создаем корневой элемент-контейнер
        $container = $dom->createElement('div');
        $dom->appendChild($container);
        
        // Обрабатываем все строки
        while ($this->currentLine < count($this->lines)) {
            $this->parseNode($dom, $container, 0);
        }
        
        // Возвращаем HTML без корневого div-контейнера
        $html = '';
        foreach ($container->childNodes as $node) {
            $html .= $dom->saveHTML($node);
        }
        return trim($html);
    }

    private function parseNode(DOMDocument $dom, DOMNode $parent, int $currentIndent) {
        if ($this->currentLine >= count($this->lines)) return;
        
        $line = $this->lines[$this->currentLine];
        $indent = $this->getIndentLevel($line);
        
        // Если отступ меньше текущего - выходим (это должен обработать родитель)
        if ($indent < $currentIndent) return;
        
        // Удаляем отступы из строки
        $content = substr($line, $currentIndent * 2);
        
        // Обработка текстового узла (замена str_starts_with)
        if (strpos($content, '| ') === 0) {
            $text = substr($content, 2);
            $parent->appendChild($dom->createTextNode($text . ' '));
            $this->currentLine++;
            
            // Добавляем последующие текстовые строки без | с тем же отступом
            while ($this->currentLine < count($this->lines)) {
                $nextLine = $this->lines[$this->currentLine];
                $nextIndent = $this->getIndentLevel($nextLine);
                
                if ($nextIndent != $currentIndent) break;
                
                if (strpos(trim($nextLine), '| ') === 0) {
                    $text = substr(trim($nextLine), 2);
                    $parent->appendChild($dom->createTextNode($text . ' '));
                } else {
                    $parent->appendChild($dom->createTextNode(trim($nextLine) . ' '));
                }
                $this->currentLine++;
            }
            return;
        }
        
        // Обработка HTML-элемента
        $element = $this->parseElement($dom, $content);
        $parent->appendChild($element);
        $this->currentLine++;
        
        // Рекурсивная обработка дочерних элементов
        while ($this->currentLine < count($this->lines)) {
            $nextIndent = $this->getIndentLevel($this->lines[$this->currentLine]);
            
            if ($nextIndent <= $currentIndent) break;
            
            $this->parseNode($dom, $element, $currentIndent + 1);
        }
    }

    // Остальные методы класса остаются без изменений
    private function parseElement(DOMDocument $dom, string $line): DOMElement {
        // Разбираем селектор на тег, id, классы и атрибуты
        preg_match('/^([a-zA-Z0-9]+)/', $line, $tagMatch);
        $tag = $tagMatch[1] ?? 'div';
        
        $element = $dom->createElement($tag);
        
        // ID (#header)
        if (preg_match('/#([a-zA-Z0-9_-]+)/', $line, $idMatch)) {
            $element->setAttribute('id', $idMatch[1]);
        }
        
        // Классы (.class1.class2)
        if (preg_match_all('/\.([a-zA-Z0-9_-]+)/', $line, $classMatches)) {
            $element->setAttribute('class', implode(' ', $classMatches[1]));
        }
        
        // Атрибуты ([name='value' type='text'])
        if (preg_match('/\[(.*?)\]/', $line, $attrMatch)) {
            $attributes = $this->parseAttributes($attrMatch[1]);
            foreach ($attributes as $name => $value) {
                $element->setAttribute($name, $value);
            }
        }
        
        return $element;
    }

    private function parseAttributes(string $attrString): array {
        $attributes = [];
        $pairs = preg_split('/\s+(?=[a-zA-Z-]+=)/', $attrString);
        
        foreach ($pairs as $pair) {
            if (preg_match('/([a-zA-Z-]+)="([^"]*)"|\'([^\']*)\'/', $pair, $match)) {
                $value = $match[2] ?? $match[3] ?? '';
                $attributes[$match[1]] = $value;
            } elseif (preg_match('/([a-zA-Z-]+)=([^\s\'"]+)/', $pair, $match)) {
                $attributes[$match[1]] = $match[2];
            } elseif (preg_match('/^[a-zA-Z-]+$/', $pair)) {
                // Атрибут без значения (например, "required")
                $attributes[$pair] = $pair;
            }
        }
        
        return $attributes;
    }

    private function getIndentLevel(string $line): int {
        $spaces = strspn($line, ' ');
        return (int)($spaces / 2);
    }
}
```

Основные изменения:
1. Заменил все вызовы `str_starts_with($str, '| ')` на `strpos($str, '| ') === 0`
2. Оставил всю остальную логику без изменений

Это решение будет работать в PHP версий ниже 8.0. Функция `strpos()` возвращает позицию первого вхождения подстроки или `false`, если подстрока не найдена. Сравнение `=== 0` означает, что подстрока найдена в начале строки.

Если вам нужно обрабатывать многобайтовые строки (например, с русскими символами), можно использовать `mb_strpos()` вместо `strpos()`, но в данном случае это не обязательно, так как мы ищем просто символ `|` с пробелом.
