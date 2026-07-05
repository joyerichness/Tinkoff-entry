# MiniGrid RSSM + VLM-style Scorer

Задание со вступительных в магистратуру Тинькофф

Среда - `MiniGrid-Empty-Random-6x6-v0`. Цель задается текстом: `agent at the green goal`

В ноутбуке был предусмотрен CLIP VIT (`openai/clip-vit-base-patch32`) как основной image-text scorer
RSSM обучается на компактном состоянии, поэтому это скорее демонстрация идеи, чем полноценный Dreamer from pixels

## Что сравнивается

- `random` - случайные действия
- `wm_no_vlm` - планирование в world model без VLM-score
- `wm_vlm` - то же планирование, но с добавлением score будущих кадров относительно текстовой цели.

## Результаты

Оценка проводилась на 30 эпизодах с seed `100..129`

| Метод | Success rate | Mean return | Mean steps |
|---|---:|---:|---:|
| random | 0.30 | 0.148 | 43.47 |
| wm_no_vlm | 0.70 | 0.477 | 27.37 |
| wm_vlm | 0.73 | 0.439 | 29.70 |

## Выводы
- Добавление CLIP/VLM-score слегка увеличило вероятность достижения цели, но сделало поведение менее эффективным по числу шагов. Вероятно, VLM-score помогает иногда выбрать траекторию, ведущую к цели, но сам score шумный для MiniGrid-картинок и не всегда хорошо ранжирует более короткие пути
- Оба подхода явно лучше случайного блуждания
- Результаты чувствительны к NUM_CANDIDATES, HORIZON, числу seed’ов и весу VLM-score

## Failure modes
- Текст `agent at the green goal` для CLIP неоднозначен: модель не обязана понимать, что красный круг - это agent, а зеленая клетка - goal
- VLM в целом не сильно приспособлена к синтетике minigrid'а, поэтому может даже идти в минус
- Результат заметно зависит от HORIZON, NUM_CANDIDATES, веса VLM-score и числа seed’ов.

## Future work

- Обучить RSSM по RGB-кадрам через CNN encoder/decoder
- Заменить random shooting на CEM
- Дообучить VLM-скорер на MiniGrid-кадрах

## Артефакты

Основные артефакты сохраняются в папку `artifacts/`:

- `artifacts/summary.csv` - агрегированные метрики;
- `artifacts/episode_results.csv` - результаты по каждому seed;
- `artifacts/success_rate.png` - сравнение success rate;
- `artifacts/final_frames.png` - финальные кадры примеров;
- `artifacts/*_episode.gif` - GIF с примерами эпизодов;
- `artifacts/rssm_world_model.pt` - веса world model
