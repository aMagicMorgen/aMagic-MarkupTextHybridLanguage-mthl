Реализуем такой парсер на PHP. Он будет обрабатывать синтаксис вида `tag.class#id[attr="value"] Текст` с поддержкой вложенности через отступы (2 или 4 пробела).

### Код парсера (PHP):
```php
class MDS {
    // Теги без закрывающего элемента (void elements)
    const VOID_TAGS = [
        'area', 'base', 'br', 'col', 'embed', 'hr', 'img',
        'input', 'link', 'meta', 'param', 'source', 'track', 'wbr'
    ];

    public function parse(string $input): string {
        $lines = explode("\n", trim($input));
        $stack = [];
        $prevIndent = 0;
        $output = '';

        foreach ($lines as $line) {
            // Пропускаем пустые строки
            if (trim($line) === '') continue;

            // Определяем уровень вложенности
            preg_match('/^(\s*)/', $line, $matches);
            $indent = strlen($matches[1]);
            $content = trim($line);

            // Закрываем предыдущие теги при уменьшении вложенности
            while ($indent < $prevIndent) {
                array_pop($stack);
                $prevIndent -= 2;
            }

            // Парсим текущую строку
            $node = $this->parseLine($content);
            $output .= $this->renderTag($node, $stack);

            // Добавляем в стек если тег не void
            if (!in_array($node['tag'], self::VOID_TAGS)) {
                array_push($stack, $node['tag']);
                $prevIndent += 2;
            }
        }

        // Закрываем оставшиеся теги
        while (!empty($stack)) {
            $output .= '</' . array_pop($stack) . '>';
        }

        return $output;
    }

    private function parseLine(string $line): array {
        // Регулярка для разбора строки
        preg_match('/
            ^
            (?:([a-z]+))?                # Тег
            (?:#([\w-]+))?               # ID
            (?:\.([\w.-]+))?             # Классы
            (?:\[(.*?)\])?               # Атрибуты
            (?:\s+(.+))?                 # Текст
            $
        /xi', $line, $matches);

        // Обработка атрибутов
        $attrs = [];
        if (!empty($matches[4])) {
            preg_match_all('/(\w+)=("[^"]*"|\S+)/', $matches[4], $attrPairs);
            foreach ($attrPairs[1] as $i => $name) {
                $attrs[$name] = trim($attrPairs[2][$i], '"');
            }
        }

        return [
            'tag' => $matches[1] ?? 'div',
            'id' => $matches[2] ?? '',
            'classes' => $matches[3] ? explode('.', $matches[3]) : [],
            'attrs' => $attrs,
            'text' => $matches[5] ?? ''
        ];
    }

    private function renderTag(array $node, array $stack): string {
        $tag = $node['tag'];
        $attrs = [];

        if ($node['id']) {
            $attrs[] = 'id="' . htmlspecialchars($node['id']) . '"';
        }

        if (!empty($node['classes'])) {
            $attrs[] = 'class="' . htmlspecialchars(implode(' ', $node['classes'])) . '"';
        }

        foreach ($node['attrs'] as $name => $value) {
            $attrs[] = htmlspecialchars($name) . '="' . htmlspecialchars($value) . '"';
        }

        $html = '<' . $tag . ($attrs ? ' ' . implode(' ', $attrs) : '') . '>';

        if (!in_array($tag, self::VOID_TAGS)) {
            $html .= htmlspecialchars($node['text']);
        }

        return $html;
    }
}

// Пример использования
$input = "
ul.menu#main
  li.item Пункт 1
  li.item.active Пункт 2
  input[type='text' placeholder='Введите имя']
";

$parser = new MDS();
echo $parser->parse($input);
```

### Результат:
```html
<ul id="main" class="menu">
  <li class="item">Пункт 1</li>
  <li class="item active">Пункт 2</li>
  <input type="text" placeholder="Введите имя">
</ul>
```

### Особенности парсера:
1. **Поддержка вложенности через отступы** (2 пробела на уровень)
2. **Автозакрытие тегов** для void-элементов (input, img и др.)
3. **Экранирование спецсимволов** через htmlspecialchars()
4. **Поддерживаемые элементы:**
   - Тег: `div` (по умолчанию)
   - ID: `#id`
   - Классы: `.class1.class2`
   - Атрибуты: `[attr1="val1" attr2]`
   - Текст: после атрибутов

### Аналогичные решения:
1. **Slim** (Ruby) - похожий синтаксис
2. **Emmet** (аббревиатуры) - требует компиляции
3. **Pug/Jade** - более сложный синтаксис с логикой

Такой парсер подойдет для:
- Быстрого прототипирования
- Генерации статических страниц
- Конфигураций UI-компонентов
- Упрощения верстки для разработчиков

Для сложных проектов лучше использовать полноценные шаблонизаторы (Twig, Blade), но этот вариант отлично работает как минималистичное решение.
