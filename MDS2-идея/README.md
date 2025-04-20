# вернёмся к css. 

Ме понравился подход с js // Функция для применения стилей к элементам
```    
    function applyTailwindStyles() {
      // Общие стили для тегов
      const tagStyles = {
        'body': 'font-sans bg-gray-50 text-gray-800',
        'h1': 'text-4xl font-bold mb-6 text-center',
        'h2': 'text-3xl font-bold mb-8 text-center',
        'h3': 'text-2xl font-bold mb-4',
        'p': 'text-lg mb-4',
        'a': 'text-primary hover:underline',
        'button': 'bg-primary text-white px-6 py-3 rounded-lg hover:bg-primary/90 transition',
        'section': 'py-16 px-4',
        'header': 'bg-white shadow-sm py-4 sticky top-0',
        'footer': 'bg-gray-800 text-white py-12',
        'input': 'border rounded px-4 py-2 w-full mb-4',
        'textarea': 'border rounded px-4 py-2 w-full mb-4'
      }
```
это меня подтолкнуло на мысль, что в php при генерации html будет удобно использовать такой массив тег=[атрибуты, атрибуты2, атрибуты3] причём например это div но их может быть много разных в и потому к разным div будут поставляться свои элементы.
