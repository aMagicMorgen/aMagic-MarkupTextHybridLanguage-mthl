class mds MarkdownStyle  не рабочий
```php
class MDS {
    const VOID_TAGS = [
        'area', 'base', 'br', 'col', 'embed', 'hr', 'img',
        'input', 'link', 'meta', 'param', 'source', 'track', 'wbr'
    ];

    private $indentSize = 2;
    private $stack = [];
    private $output = '';

    public function parse(string $input): string {
        $lines = explode("\n", trim($input));
        $currentIndent = 0;

        foreach ($lines as $line) {
            $line = rtrim($line);
            if ($line === '') continue;

            // Определяем уровень вложенности
            preg_match('/^[\t ]*/', $line, $indentMatch);
            $indent = strlen(str_replace("\t", str_repeat(' ', $this->indentSize), $indentMatch[0]));

            // Закрываем теги при уменьшении вложенности
            while ($indent < $currentIndent) {
                $this->closeTag();
                $currentIndent -= $this->indentSize;
            }

            // Парсим текущую строку
            $node = $this->parseLine(trim($line));
            $this->renderOpeningTag($node);

            // Добавляем в стек если тег не void
            if (!in_array($node['tag'], self::VOID_TAGS)) {
                array_push($this->stack, $node['tag']);
                $currentIndent += $this->indentSize;
            }
        }

        // Закрываем оставшиеся теги
        while (!empty($this->stack)) {
            $this->closeTag();
        }

        return $this->output;
    }

    private function parseLine(string $line): array {
        $parts = [
            'tag' => 'div',
            'id' => '',
            'classes' => [],
            'attrs' => [],
            'text' => ''
        ];

        // 1. Извлекаем текст (все после первого пробела не в квадратных скобках)
        if (preg_match('/^(.*?)(?<!\S)\s+(?![^\[]*\])(.*)$/', $line, $textMatch)) {
            $parts['text'] = $textMatch[2];
            $line = $textMatch[1];
        }

        // 2. Извлекаем тег
        if (preg_match('/^([a-z][a-z0-9]*)/i', $line, $tagMatch)) {
            $parts['tag'] = strtolower($tagMatch[1]);
            $line = substr($line, strlen($tagMatch[0]));
        }

        // 3. Извлекаем ID
        if (preg_match('/^#([\w-]+)/', $line, $idMatch)) {
            $parts['id'] = $idMatch[1];
            $line = substr($line, strlen($idMatch[0]));
        }

        // 4. Извлекаем классы
        if (preg_match('/^\.([\w.-]+)/', $line, $classMatch)) {
            $parts['classes'] = explode('.', substr($classMatch[0], 1));
            $line = substr($line, strlen($classMatch[0]));
        }

        // 5. Извлекаем атрибуты
        if (preg_match('/^\[([^\]]+)\]/', $line, $attrMatch)) {
            preg_match_all('/(\w+)=(["\'])(.*?)\2/', $attrMatch[1], $attrPairs, PREG_SET_ORDER);
            foreach ($attrPairs as $pair) {
                $parts['attrs'][$pair[1]] = $pair[3];
            }
        }

        return $parts;
    }

    private function renderOpeningTag(array $node): void {
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

        $this->output .= '<' . $node['tag'];
        if (!empty($attrs)) {
            $this->output .= ' ' . implode(' ', $attrs);
        }
        $this->output .= '>';

        if (!in_array($node['tag'], self::VOID_TAGS)) {
            $this->output .= htmlspecialchars($node['text']);
        }
    }

    private function closeTag(): void {
        if (!empty($this->stack)) {
            $tag = array_pop($this->stack);
            $this->output .= '</' . $tag . '>';
        }
    }
}
```

Пример использования

```php
$mds = new MDS();
$input = "
ul.menu#main
  li.item Пункт 1
  li.item.active Пункт 2
  input[type='text' placeholder='Введите имя']

form#login.form-inline[action='/auth']
  input[type='text' name='login']
  input[type='password' name='pass']
  button.btn[type='submit'] Войти
";

echo $mds->parse($input);
```

Проблема в том, что теги что теги обрабатываются и вкладываются правильно, а text в темах не появляется. Реши проблему

Проблема заключается в том, что текст внутри тегов не добавляется правильно при рендеринге тегов. Это происходит потому, что текст добавляется только для тегов, которые не являются пустыми (`VOID_TAGS`), но при этом текст не добавляется в правильное место внутри HTML-структуры.

Ниже приведено исправленное решение:

```php
class MDS {
    const VOID_TAGS = [
        'area', 'base', 'br', 'col', 'embed', 'hr', 'img',
        'input', 'link', 'meta', 'param', 'source', 'track', 'wbr'
    ];

    private $indentSize = 2;
    private $stack = [];
    private $output = '';

    public function parse(string $input): string {
        $lines = explode("\n", trim($input));
        $currentIndent = 0;

        foreach ($lines as $line) {
            $line = rtrim($line);
            if ($line === '') continue;

            // Определяем уровень вложенности
            preg_match('/^[\t ]*/', $line, $indentMatch);
            $indent = strlen(str_replace("\t", str_repeat(' ', $this->indentSize), $indentMatch[0]));

            // Закрываем теги при уменьшении вложенности
            while ($indent < $currentIndent) {
                $this->closeTag();
                $currentIndent -= $this->indentSize;
            }

            // Парсим текущую строку
            $node = $this->parseLine(trim($line));

            // Добавляем текст в вывод, если это не пустой тег
            if (!in_array($node['tag'], self::VOID_TAGS)) {
                $this->output .= htmlspecialchars($node['text']);
            }

            $this->renderOpeningTag($node);

            // Добавляем в стек если тег не void
            if (!in_array($node['tag'], self::VOID_TAGS)) {
                array_push($this->stack, $node['tag']);
                $currentIndent += $this->indentSize;
            }
        }

        // Закрываем оставшиеся теги
        while (!empty($this->stack)) {
            $this->closeTag();
        }

        return $this->output;
    }

    private function parseLine(string $line): array {
        $parts = [
            'tag' => 'div',
            'id' => '',
            'classes' => [],
            'attrs' => [],
            'text' => ''
        ];

        // 1. Извлекаем текст (все после первого пробела не в квадратных скобках)
        if (preg_match('/^(.*?)(?<!\S)\s+(?![^\[]*\])(.*)$/', $line, $textMatch)) {
            $parts['text'] = $textMatch[2];
            $line = $textMatch[1];
        }

        // 2. Извлекаем тег
        if (preg_match('/^([a-z][a-z0-9]*)/i', $line, $tagMatch)) {
            $parts['tag'] = strtolower($tagMatch[1]);
            $line = substr($line, strlen($tagMatch[0]));
        }

        // 3. Извлекаем ID
        if (preg_match('/^#([\w-]+)/', $line, $idMatch)) {
            $parts['id'] = $idMatch[1];
            $line = substr($line, strlen($idMatch[0]));
        }

        // 4. Извлекаем классы
        if (preg_match('/^\.([\w.-]+)/', $line, $classMatch)) {
            $parts['classes'] = explode('.', substr($classMatch[0], 1));
            $line = substr($line, strlen($classMatch[0]));
        }

        // 5. Извлекаем атрибуты
        if (preg_match('/^\[([^\]]+)\]/', $line, $attrMatch)) {
            preg_match_all('/(\w+)=(["\'])(.*?)\2/', $attrMatch[1], $attrPairs, PREG_SET_ORDER);
            foreach ($attrPairs as $pair) {
                $parts['attrs'][$pair[1]] = $pair[3];
            }
        }

        return $parts;
    }

    private function renderOpeningTag(array $node): void {
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

        $this->output .= '<' . $node['tag'];
        if (!empty($attrs)) {
            $this->output .= ' ' . implode(' ', $attrs);
        }
        if (in_array($node['tag'], self::VOID_TAGS)) {
            $this->output .= ' />'; // Закрываем пустые теги сразу
        } else {
            $this->output .= '>'; // Для непустых тегов открываем
        }
    }

    private function closeTag(): void {
        if (!empty($this->stack)) {
            $tag = array_pop($this->stack);
            $this->output .= '</' . $tag . '>';
        }
    }
}

// Пример использования
$mds = new MDS();
$input = "
ul.menu#main
  li.item Пункт 1
  li.item.active Пункт 2
  input[type='text' placeholder='Введите имя']

form#login.form-inline[action='/auth']
  input[type='text' name='login']
  input[type='password' name='pass']
  button.btn[type='submit'] Войти
";

echo $mds->parse($input);
```

В этом исправленном коде текст добавляется в вывод перед открытием тега, что позволяет правильно позиционировать текст внутри HTML-структуры. Кроме того, пустые теги теперь сразу закрываются в методе `renderOpeningTag`.

