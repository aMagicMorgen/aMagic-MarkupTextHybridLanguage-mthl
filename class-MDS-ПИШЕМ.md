#  Вот актуализированная реализация:

```php
class MDS {
    const VOID_TAGS = ['img', 'input', 'meta', 'link', 'br'];
    private $indentSize = 2;

    public function parse(string $input): string {
        $lines = explode("\n", trim($input));
        $output = '';
        $stack = [];
        $prevIndent = 0;

        foreach ($lines as $line) {
            // Пропуск пустых строк
            if (trim($line) === '') continue;

            // Определяем отступ (поддержка табов и пробелов)
            preg_match('/^[\t ]*/', $line, $indentMatch);
            $indent = strlen(str_replace("\t", "    ", $indentMatch[0]));
            $content = trim($line);

            // Закрытие предыдущих тегов
            while ($indent < $prevIndent) {
                $output .= $this->closeTag(array_pop($stack));
                $prevIndent -= $this->indentSize;
            }

            // Парсинг текущей строки
            $node = $this->parseLine($content);
            $output .= $this->renderTag($node);

            // Добавляем в стек если тег не void
            if (!in_array($node['tag'], self::VOID_TAGS)) {
                array_push($stack, $node['tag']);
                $prevIndent += $this->indentSize;
            }
        }

        // Закрываем оставшиеся теги
        while (!empty($stack)) {
            $output .= $this->closeTag(array_pop($stack));
        }

        return $output;
    }

    // ... остальные методы класса (parseLine, renderTag и т.д.) 
}
```

**Пример использования:**
```php
$html = (new MDS())->parse("
div.container
  h1#title Привет, MDS!
  img[src='logo.png']
");
```

**Преимущества переименования:**
1. **Краткость** — `MDS` быстрее писать и читать
2. **Семантика** — явная связь с Markdown Style
3. **Простота интеграции** — понятное имя для импорта
4. **Масштабируемость** — можно добавить `MDS\Parser`, `MDS\Renderer` и т.д.

Теперь ваш синтаксис обрел законченный вид и логичное именование! 🚀
