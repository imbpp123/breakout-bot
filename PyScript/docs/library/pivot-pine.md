# Pivot: реализация в Pine Script

## Назначение

Справка описывает раздел Pivot [PivotLevels.pine](../../libraries/PivotLevels.pine). Определение и инварианты находятся в [общем контракте](pivot.md). Этот документ содержит только особенности Pine Script v6 и текущей реализации.

## Тип и экспортируемые функции

`Pivot` содержит `float price`, `int index`, `int side`, `int knownAt`. HIGH представлен числом `1`, LOW — `-1`. Создание вручную: `levels.Pivot.new(price, index, side, knownAt)`. Конструктор не проверяет инварианты. Поля ATR нет.

```pine
export pivot_collectHigh(array<Pivot> highPivots, float source, simple int span, bool confirmed, int index)
export pivot_collectLow(array<Pivot> lowPivots, float source, simple int span, bool confirmed, int index)
export pivot_pruneWindow(array<Pivot> pivots, int firstIndex)
```

Коллекции — изменяемые массивы потребителя. `source` — серия цен, `span` — положительное постоянное число соседей. Для обычного графика передаются `barstate.isconfirmed` и `bar_index`. Возвращаемые значения не используются: функции изменяют массивы.

Сбор HIGH вызывает `ta.pivothigh(source, span, span)`, сбор LOW — `ta.pivotlow(source, span, span)`. Каждая функция вызывается один раз на каждом баре, включая открытый, вне условий закрытия и отрисовки. Приватная `f_pivot_append()` добавляет точку только при `confirmed = true` и найденной цене, отличной от `na`. Сохраняются цена, `index - span`, сторона и `knownAt = index`.

Повторный вызов на том же закрытом баре может добавить дубликат. Отдельной проверки входных параметров нет. `pivot_pruneWindow()` удаляет элементы с начала массива и останавливается на первой точке с `index >= firstIndex`; неупорядоченный вход не поддерживается.

## Поведение встроенных функций

Текущие сборщики делегируют выбор равных цен и обработку пропусков стандартным функциям Pine. Если результат поиска равен `na`, точка не добавляется.

Общий контракт требует сохранять каждый равный максимум или минимум, прошедший проверку, исключать полностью равное окно и не формировать pivot, если в его окне есть хотя бы одно отсутствующее значение. В Pine отсутствующая цена представлена значением `na`.

Соответствие текущих сборщиков этим правилам не подтверждено. Перед выпуском необходимы проверки обоих концов плато, полностью равного окна и `na` на месте кандидата, левого или правого соседа. При несовпадении сборщики должны быть изменены.

Задержка обнаружения pivot через правые соседние свечи показана в [официальном примере TradingView](https://www.tradingview.com/pine-script-docs/visuals/plots/). В общем контракте она описана независимо от API.

## Пример подключения

Фрагмент для вызывающего индикатора. Номер `/6` нужно сверить с фактической публикацией.

```pine
import imbpp123/PivotLevels/6 as levels

int historyBars = input.int(1000, "History window, bars", minval = 50, maxval = 5000)
int pivotSpan = input.int(5, "Pivot bars on each side", minval = 1, maxval = 50)

var array<levels.Pivot> highPivots = array.new<levels.Pivot>()
var array<levels.Pivot> lowPivots = array.new<levels.Pivot>()

float highSource = math.max(open, close)
float lowSource = math.min(open, close)
levels.pivot_collectHigh(highPivots, highSource, pivotSpan, barstate.isconfirmed, bar_index)
levels.pivot_collectLow(lowPivots, lowSource, pivotSpan, barstate.isconfirmed, bar_index)

if barstate.isconfirmed
    int firstIndex = bar_index - historyBars + 1
    levels.pivot_pruneWindow(highPivots, firstIndex)
    levels.pivot_pruneWindow(lowPivots, firstIndex)
```

Для каждого инструмента и таймфрейма нужна отдельная пара массивов. Библиотека не запрашивает другой таймфрейм и не подтверждает его закрытие за потребителя. Настройки, серии и отрисовка остаются снаружи библиотеки.

## Версии и совместимость

В текущем исходнике нет `Pivot.atr`, `pivot_collect()`, `pivot_collectSides()` и `pivot_clean()`. Старый `pivot_levels.pine` остаётся на опубликованной `/5`; его вызовы и конструкторы требуют адаптации перед сменой импорта. ATR для зон больше не передаётся через Pivot.

Изменение локального файла не обновляет опубликованную библиотеку. После публикации нужно указать владельца и фактическую версию импорта. `/6` в текущем индикаторе — ожидаемый номер, публикация не подтверждена.

## Состояние проверки

Встроенные тесты [pivots.pine](../../pivots.pine) используют те же экспортируемые функции, что и расчёт графика. Они проверяют подтверждение, независимые стороны, пару одной свечи, отсутствующие серии, цену и индексы, порядок и очистку окна. Для выполнения всех сценариев сбора нужно минимум девять баров.

Компиляция и выполнение тестов новой версии в TradingView пока не подтверждены: локального компилятора Pine в проекте нет. Перед выпуском скомпилировать библиотеку, опубликовать её, сверить импорт, выполнить встроенные тесты и сравнить точки с работавшей версией на одинаковых данных.
