# breakout-bot

Крипто-бот для торговли пробоями на бессрочных фьючерсах Bybit с валютой котировки USDT.

<a id="documentation"></a>
## Документация

- [Торговая стратегия: этапы и открытые вопросы](docs/strategy/README.md)

Этапы стратегии:

1. [Отбор инструментов](docs/strategy/01-instrument-selection.md)
2. [Анализ инструментов](docs/strategy/02-instrument-analysis.md) — анализ H1 и поиск ситуации на M5.
3. [Подготовка входа](docs/strategy/03-entry-preparation.md)
4. [Открытие позиции и ограничение риска](docs/strategy/04-position-and-risk.md)
5. [Сопровождение и выход](docs/strategy/05-position-management.md)

## Скрипты TradingView

Код Pine Script находится в [PyScript/](PyScript/), документация — в `PyScript/docs/` и `docs/library/`.

- [Прототип стратегии](PyScript/docs/pine-prototype.md)
- [Индикатор pivot](docs/library/pivots.md)
- [Pivot: определение и общий контракт](PyScript/docs/library/pivot.md)
- [Pivot: реализация библиотеки в Pine Script](PyScript/docs/library/pivot-pine.md)
- [Индикатор экстремумов и уровней](PyScript/docs/pivot-levels.md)
