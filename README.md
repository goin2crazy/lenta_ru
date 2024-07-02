### Решение команды HackLadies от специалиста по машинному обучению goin2crazy

# Данные

В начале работы с проектом было обнаружено, что в отличие от данных для проектов компьютерного зрения, данные для проекта по обработке естественного языка не содержат в себе результатов для полноценного обучения модели, что нарушает концепцию supervised-learning, на которой построена большая часть мира глубокого обучения.

### Задача

- Взять текст
- Найти в нем названия организаций
- Определить, как текст отзывается об этой организации (позитивно, нейтрально, негативно)

### Данные

- Текст

# Автоматическая Разметка Данных

Проблему поиска названий организаций в тексте удалось решить с помощью использования мультиязыковых моделей NER (Named Entity Recognition). Модель, которую мы использовали, может находить не только названия организаций, но и имена людей, локации и т.д.

Первая часть задачи решена. Осталась вторая часть - определение отзыва об организации, человеке и т.д. По своей сути это задача определения "настроения" (sentiment) текста. Однако при обычной загрузке текста через модель мы вычислим настроение общего текста, без фокусирования на отдельный элемент, что некорректно для нужных для хакатона результатов.

Для этого при разметке данных (определении настроения текста, сфокусированного на отдельной организации) использовалась предобученная модель, которая вычисляла настроение текста только в предложении (и еще 2-3 предложения после), где упоминалась организация. В случае упоминания организации в тексте несколько раз вычисляется средний показатель настроений.

В итоге один размеченный элемент данных выглядел примерно так:

```json
{
  "word": "Ванька Кринжовый",
  "obj": "PER",
  "sentiment": "0.9",
  "indexes_in_text": [(1, 10)]
}
```

# Тренировка Конечной Модели

Можно было просто оставить концепцию такой, но определение настроения только в парочке предложений текста может быть неэффективно для понимания текста в общем. Появилась новая задача - научить модель предсказания фокусироваться на отдельных именах.

Для этого был взят размеченный датасет с текстом. Далее была обучена модель для распознавания настроения с промптом:

```
"[focus: {name}]
{text}"
```

Пример:

```
"[focus: Ванька Кринжовый]
Сегодня, губернатор области Ванька Кринжовый сделал сальто в..."
```

Это позволило научить модель общего распознавания настроения текста фокусироваться на отдельных именах и решило проблему понимания общего смысла текста.

# Чем же уникально это решение?

1. **Автоматическое определение объектов**: Использование мультиязыковых моделей NER для автоматического определения людей, организаций, локаций и т.д.

2. **Определение настроения текста об отдельных организациях**: Модель учитывает контекст вокруг упоминания организации, что позволяет точно определить настроение именно по отношению к этой организации.

3. **Использование легких и быстрых моделей**: Применение предобученных моделей для быстрого и точного анализа текста.

4. **Гибкость и адаптивность**: Модель может легко адаптироваться к различным языковым и культурным контекстам благодаря использованию мультиязыковых моделей и продуманной разметке данных.

5. **Улучшение качества анализа текста**: Фокусирование модели на отдельных именах и организациях обеспечивает более точный и детализированный анализ настроения текста.