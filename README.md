# AD5M Postprocess

Единый скрипт постобработки G-кода OrcaSlicer для **Flashforge Adventurer 5M / 5M Pro**
с прошивкой [zmod](https://github.com/ghzserg/zmod) (+ Moonraker / Fluidd / Spoolman).

Заменяет несколько отдельных скриптов (время печати, MD5) одним, который выполняет
все шаги в правильном порядке.

## Что делает

За один проход:

1. Удаляет старые строки `; MD5:` в начале файла — скрипт можно запускать повторно.
2. Копирует `; estimated printing time (normal mode) = ...` в заголовок файла,
   чтобы штатный экран принтера показывал оставшееся время.
3. Проверяет метаданные, по которым Moonraker / Fluidd / Spoolman определяют слайсер
   и филамент. Если поле есть в файле, но не попадает в последние 512 КБ (где его
   ищет Moonraker), дописывает копию в конец файла в блок
   `; AD5M_PP_METADATA_START` … `; AD5M_PP_METADATA_END`.
4. Последним шагом считает MD5 и ставит `; MD5:<hash>` первой строкой
   (формат `CHECK_MD5` / `FORCE_MD5` из zmod).

Окончания строк (LF/CRLF) сохраняются, файл записывается атомарно.

## Требования

Python 3.7+ — только стандартная библиотека, ничего устанавливать не нужно.

Для Windows есть вариант без Python — `ad5m_postprocess.exe` (см. [Сборка exe](#сборка-exe-для-windows)).

## Подключение в OrcaSlicer

**Процесс → Прочее → Скрипты постобработки**:

```text
Windows:      "C:\Python312\python.exe" "C:\Scripts\ad5m_postprocess.py";
Linux/macOS:  /usr/bin/python3 /home/user/scripts/ad5m_postprocess.py;
Windows exe:  "C:\Scripts\ad5m_postprocess.exe";
```

Старые скрипты времени и MD5 из списка нужно удалить — этот скрипт их заменяет,
а MD5 должен считаться строго последним.

Сообщения скрипта выводятся в stderr с префиксом `[ad5m_postprocess]`.
При ошибке скрипт завершается с кодом 1, и OrcaSlicer покажет ошибку при экспорте.

## Проверка готового файла

```sh
python ad5m_postprocess.py --check file.gcode
```

Ничего не меняет, выводит:

- совпадает ли MD5 в первой строке с содержимым;
- есть ли время печати в заголовке;
- определит ли Moonraker слайсер;
- какие поля филамента увидит Moonraker.

Код возврата: `0` — всё в порядке, `2` — есть проблемы.

## Тесты

```sh
python -m unittest discover -s tests -v
```

## Сборка exe для Windows

```sh
python -m pip install pyinstaller
pyinstaller --onefile --console --clean --name ad5m_postprocess ad5m_postprocess.py
```

Готовый файл — `dist\ad5m_postprocess.exe` (~8 МБ, Python внутри). Аргументы те же,
что у скрипта, включая `--check`. Первый запуск может занять секунду — onefile-exe
распаковывается во временную папку.

## Если Fluidd не показывает филамент

Убедитесь, что в OrcaSlicer не отключена запись блока конфигурации в G-код:
без него в файле нет `filament_type`, `filament_settings_id` и `filament used [g]`,
и скрипт выведет предупреждение.
