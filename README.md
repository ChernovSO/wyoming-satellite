# Wyoming Satellite (RZhD Fork)

Пропатченная версия [Wyoming Satellite](https://github.com/rhasspy/wyoming-satellite) для голосового ассистента РЖД.

## Доработки

### ✅ Follow-up окно после ответа ассистента

После завершения TTS-ответа (событие `tts-played`) Satellite запускает **follow-up окно** (длительность настраивается параметром `--follow-up-seconds`):

- Если **в течение окна** обнаруживается начало речи — запускается `RunPipeline` с `start_stage=ASR`.
- Речь записывается до окончания голосового ввода (VAD), затем передаётся в Home Assistant.
- Если пользователь **не заговорил в течение окна** — Satellite возвращается к ожиданию wake word (`Detection`).

Встроено нативно в основной цикл событий Satellite..

---

### ✅ Перебивание ассистента повторным wake word

Если в процессе воспроизведения ответа ассистента пользователь **повторно произносит wake word**, TTS прерывается.

Реализовано через обёртку над `snd-command`:

- `tts_play.sh` запускает `aplay`, сохраняет PID и пишет в его stdin.
- При срабатывании повторного `wake word`, скрипт `tts_kill.sh` вызывается через `--detection-command`, убивает плеер.
- Благодаря этому TTS прекращается, и Home Assistant начинает новый `RunPipeline`.

Этот механизм **не требует модификации исходного кода Wyoming Satellite**, только корректных `--snd-command`, `--detection-command` и связанных скриптов.

---

## Docker и сборка

Для сборки и запуска используется `Dockerfile`, в котором:

- Устанавливается Python 3.10, PulseAudio и ALSA.
- Клонируется форк `wyoming-satellite` (этот репозиторий).
- Устанавливаются зависимости в `venv`.
- Копируются кастомные скрипты (`tts_play.sh`, `tts_kill.sh`).

Dockerfile и конфигурация `docker-compose.yml` находятся в основном проекте:
[`fpk-ha-config`](https://git.hard.aisa.ru/rzd/fpk-ha-config)

---

