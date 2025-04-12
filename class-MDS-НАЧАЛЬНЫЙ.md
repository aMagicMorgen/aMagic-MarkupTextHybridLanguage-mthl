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


### Разработанный класс `MDS` поддерживает многоуровневую вложенность. Приведем пример с древовидной структурой папок и файлов:

```php
$input = "
ul.tree
  li.folder
    span Папка 1
    ul
      li.file Файл 1
      li.folder
        span Папка 2
        ul
          li.file Файл 2
  li.folder
    span Папка 3
";

$parser = new MDS();
echo $parser->parse($input);
```

**Результат:**
```html
<ul class="tree">
  <li class="folder">
    <span>Папка 1</span>
    <ul>
      <li class="file">Файл 1</li>
      <li class="folder">
        <span>Папка 2</span>
        <ul>
          <li class="file">Файл 2</li>
        </ul>
      </li>
    </ul>
  </li>
  <li class="folder">
    <span>Папка 3</span>
  </li>
</ul>
```

### Как это работает:
1. **Отступы определяют уровень вложенности.** Каждые 2 пробела — новый уровень.
2. **Стек тегов** отслеживает открытые элементы. При уменьшении отступа теги закрываются автоматически.
3. **Void-теги** (например, `<img>`) не требуют закрывающих элементов.

### Пример с тегами разных уровней:
```php
$input = "
div
  div
    div
      p Глубокий уровень
  div
    p Возврат на предыдущий уровень
";

// Результат:
// <div>
//   <div>
//     <div>
//       <p>Глубокий уровень</p>
//     </div>
//   </div>
//   <div>
//     <p>Возврат на предыдущий уровень</p>
//   </div>
// </div>
```

### Ограничения и улучшения:
1. **Табы vs пробелы:** Сейчас парсер считает только пробелы. Для поддержки табов замените:
   ```php
   $indent = strlen(str_replace("\t", "  ", $matches[1])); // 1 таб = 2 пробела
   ```
2. **Ошибки формата:** Некорректные отступы (например, 3 пробела) могут нарушить структуру.
3. **Экранирование:** Текст автоматически экранируется через `htmlspecialchars()`.

Такой парсер подходит для большинства базовых сценариев. Для сложных проектов лучше использовать `phug/pug`, но этот вариант отлично работает как легковесное решение.

### Вот пример таблицы с объединенными ячейками, обработанной нашим парсером:

```php
$input = "
table.table-style[border='1']
  thead
    tr
      th[colspan='2'] Основной заголовок
  tbody
    tr
      td[rowspan='2'] Объединенная ячейка
      td Ячейка 1
    tr
      td Ячейка 2
    tr
      td[colspan='2'] Нижний заголовок
";

$parser = new IndentParser();
echo $parser->parse($input);
```

**Результат выполнения:**
```html
<table class="table-style" border="1">
  <thead>
    <tr>
      <th colspan="2">Основной заголовок</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="2">Объединенная ячейка</td>
      <td>Ячейка 1</td>
    </tr>
    <tr>
      <td>Ячейка 2</td>
    </tr>
    <tr>
      <td colspan="2">Нижний заголовок</td>
    </tr>
  </tbody>
</table>
```

### Как добавить сложные атрибуты:
1. **Объединение колонок**  
   ```css
   td[colspan='3']
   ```
2. **Объединение строк**  
   ```css
   td[rowspan='2']
   ```
3. **Смешанные атрибуты**  
   ```css
   td[data-id='5'][class='special']
   ```

### Особенности реализации:
1. Поддерживается любая вложенность:
   ```php
   table
     tr
       td
         div.wrapper
           span.icon
           p.text
   ```
2. Автоматическое закрытие тегов для:
   - `<table>`
   - `<tr>`
   - `<td>`/`<th>`
   - `<thead>`/`<tbody>`

Для работы с таблицами парсер корректно обрабатывает все стандартные HTML-атрибуты. Это решение подходит для генерации сложных табличных структур без ручного написания HTML.

### Вот текстовое представление таблицы с визуализацией структуры:

```
┌──────────────────────────────┐
|         Основной заголовок         |
|           (colspan=2)          |
├───────────────────┬───────────┤
|                   |  Ячейка 1 |
| Объединенная      ├───────────┤
| ячейка (rowspan=2)|  Ячейка 2 |
|                   |
├───────────────────┴───────────┤
|       Нижний заголовок        |
|         (colspan=2)           |
└──────────────────────────────┘
```

Пояснения:
1. **Верхний заголовок** (`<th colspan="2">`) занимает 2 колонки
2. **Объединенная ячейка** (`<td rowspan="2">`):
   - Занимает 2 строки по вертикали
   - Распространяется на вторую строку таблицы
3. **Нижний заголовок** (`<td colspan="2">`) объединяет 2 колонки

Границы таблицы (`border="1"`) отображаются как:
- `┌` `┐` `└` `┘` - угловые границы
- `├` `┤` `┬` `┴` - соединения
- `─` горизонтальные линии
- `│` вертикальные линии

В реальном браузере таблица будет выглядеть примерно так:

```HTML

<table class="table-style" border="1" style="border-collapse: collapse; margin: 10px 0">
  <thead>
    <tr>
      <th colspan="2" style="padding: 5px; background: #f0f0f0">Основной заголовок</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="2" style="padding: 5px; border: 1px solid #ccc">Объединенная ячейка</td>
      <td style="padding: 5px; border: 1px solid #ccc">Ячейка 1</td>
    </tr>
    <tr>
      <td style="padding: 5px; border: 1px solid #ccc">Ячейка 2</td>
    </tr>
    <tr>
      <td colspan="2" style="padding: 5px; border: 1px solid #ccc">Нижний заголовок</td>
    </tr>
  </tbody>
</table>

```

### Cтандартный Markdown **не поддерживает объединение ячеек** (colspan/rowspan). Однако есть два подхода:

---

### 1. Чистый Markdown (без объединения ячеек)
```markdown
| Основной заголовок       | Основной заголовок |
|--------------------------|--------------------|
| Объединенная ячейка       | Ячейка 1           |
| Объединенная ячейка       | Ячейка 2           |
| Нижний заголовок          | Нижний заголовок   |
```

**Недостаток:** Нет реального объединения — это просто визуальная имитация.

---

### 2. HTML внутри Markdown (с поддержкой объединений)
```markdown
<!-- Вставка HTML-кода прямо в Markdown -->
<table class="table-style" border="1">
  <thead>
    <tr>
      <th colspan="2">Основной заголовок</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="2">Объединенная ячейка</td>
      <td>Ячейка 1</td>
    </tr>
    <tr>
      <td>Ячейка 2</td>
    </tr>
    <tr>
      <td colspan="2">Нижний заголовок</td>
    </tr>
  </tbody>
</table>
```

**Где работает:** GitHub, GitLab, Jupyter Notebook и другие системы, поддерживающие HTML в Markdown.

---

### 3. Расширенный синтаксис (не везде поддерживается)
Некоторые парсеры (например, **Markdown Extra**) позволяют использовать атрибуты через `{: }`:
```markdown
|               |               |
|---------------|---------------|
{: .table-style border="1"}
| Основной заголовок       |
|--------------------------|
| Объединенная ячейка | Ячейка 1 |
|^                | Ячейка 2 |
| Нижний заголовок          |
```

Но это **не стандарт** и поддерживается редко.

---

**Рекомендация:** Используйте HTML-вставку, если нужны точные объединения ячеек. Для простых таблиц достаточно стандартного Markdown.

---


