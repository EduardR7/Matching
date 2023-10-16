![image](https://github.com/EduardR7/Matching/assets/126398449/8ef414ad-3402-48b2-bb75-2ade5b45d46e)
# Мэтчинг товаров в маркетплейсе

## Описание задачи
Заказчику необходимо внедрить систему поиска похожих товаров (мэтчинг).
На вход приходит вектор значений последнего скрытого слоя нейросети (эмбединги) нового товара. На выходе нужно получить похожий товар из базы

Имеющиеся данные:
- База эмбедингов в ~3М строк
- Тренировочная выборка в 100к строк: новые товары, которым проставлено совпадение из базы
- Валидационная выборка в 100к строк

В работе проведено тестирование 4 алгоритмов для генерации обучающей выборки. Выбран оптимальный

## Результаты работы и выводы
- Построена двухэтапная модель (поиск `FAISS` и ранжирование с `CatboostClassifier`)
- `FAISS` хорошо работает с расстояниями и скалированными векторами
- Датасет имеет несколько признаков, которые нарушают кластеры, но хорошо разделяют таргеты

`FAISS искал векторы без "странных признаков". Классификатор ранжировал результаты со всеми признаками`

- Для ранжирования попробованы несколько подходов формирования `обучающей выборки`:
    - обучение на расстояниях до рандомных векторов - слишком простая тренировочная выборка. близкие векторы разделять трудно
    - обучение на расстоях до близких векторов - слишком сложная обучающая выборка
    - обучение на сконкатенированных векторах - результаты хорошие, но недостаточные
    - обучение на сконкатенированных векторах с добавлением расстояний как доп фич - пока что лучшая модель
    
Вывод:
- Не недооценивать важность `EDA`

#### Результат
- Метрика acc@5 на валидации на 5 лучших кандидатах из 50 отранжированных составила 75.122
- ММетрика acc@5 на валидации на 5 лучших кандидатах из 100 отранжированных составила 75.079 - видимо, нужно обучать на соответствующей выборке (так модель научится разделять более непохожие векторы). Нужно это или нет - второй вопрос. Т.к. увеличивается время инференса.

- Время работы алгоритма для инференса выборки из 100_000 кандидатов на i5-9600K
    - для 50 соседей: 32 минуты (10 минут поиск `FAISS` и 22 минуты на инференс(включая генерацию фич)
    - для 100 соседей: 52 минуты (10 минут поиск `FAISS` и 42 минуты на инференс(включая генерацию фич)

#### Что нужно доделать:
- Поискать иные подходы (возможно, считать разницу между признаками)
- Поэкспериментировать с другими скейлерами, например `QuantileTransformer`
- Попробовать заменить `FAISS` на `Quadrant` или `Annoy`
- Сделать код ранжирования эффективнее (Отрабатывает долго. Вероятно, из-за множества инициализаций)
- Тюнить модель
- Купить больше оперативной памяти =)
