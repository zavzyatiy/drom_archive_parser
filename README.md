## Описание проекта

**Последнее обновление проекта: 29.01.2025.**

Данный парсер предназначен для сбора информации по архивным объявлениям, представленным на сайте <a href="https:/auto.drom.ru/archive/" target="_blank"> Дром.ру </a> - крупнейшей в России онлайн-платформе, агрегирующей предложение на вторичном рынке автомобилей, через которую проходит до 60% соответстующего трафика. Собранный датасет лежит ниже по ссылке:

_если ссылки нет, то происходит допарсинг_

## Общие слова

**Контекст:**

Ниже в тексте я использую следующие определения: cсылка - это определенным образом сконструированный url, в ответ на который выдача сайта возвращает упорядоченный по дате (или как-то иначе, но нас интересует дата) список удовлетворяющий запросу набор _объявлений_ , то есть конкретных публикаций продающихся автомобилей. Сайт представляет из себя (по ощущениям) большую SQL таблицу, работа с которой осложняется тем, что по любому из запросов сайт выдает 400 первых объявлений (если быть точным 20 объявлений на 20 страницах). Для обхода такой неприятности и был написан этот парсер. "Под капотом" у парсера - Async I/O (Дром.ру очень быстро начинает блокировать автоматизированные запросы через специальную задержку или 429 ответ сайта), Stem и Requests[SOCKS] (для возможности смены IP-адресов при блокировании со стороны сайта), набор библиотек для HTML-парсинга (lxml, Beutiful Soup) и Selenium (для эмулирования браузера в случае, когда необходимо взаимодействовать с кнопками на нем).

В коде есть коментарии, хоть и не весьма подробные.

**Структура:**

0. Файлы воспроизводятся последовательно по их названиям (zero -> first -> second). Такая процедура позволяет структурировать обращения к сайту и увеличивает скорость парсинга.
1. Файл zero_stage.ipynb представляет из себя файл для сбора ссылок всевозможных ссылок для парсинга конкретного списка автомобилей. Большая часть кода полностью автоматизированна, хотя местами и медленная. Требуется ручное вмешательство только в самом конце обработке ссылок, о чем подробнее написано в самом коде.
2. Файл first_stage.ipynb представляет из себя файл для асинхронного сбора ссылок на объявления и их даты публикаций на основе того, что полученно в пункте (1). Это сделано потому, что ссылки на конкретные объявления формируются по-другому протоколу, нежели ссылки-запросы к базе данных Дрома, с которым мы работаем в пункте (1).
3. Файл second_stage.ipynb представляет из себя файл для формирования итоговых таблиц со спаршенными данными. На этом этапе мы получаем чистые данные для дальнейшего анализа (нам они нужны для эконометрического исследования, о котором, возможно, будет сказано здесь отдельно, когда оно будет дописано, с приложением необходимого кода).

**Комментарии:**

- Предполагается, что для работы на своем устройстве пользователь загрузит из проекта все три файла в некоторую папку с подпапками "./cars", "./cars/href", "./second_href", "./all_cars".
- Также предполагается наличие у пользователя браузера **Гугл-Хром** в директории "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" (нужно для файла zero_stage.ipynb) и установленного **Tor Browser** на компьюетере (об установке и настройке отдельно ниже).

## Как установить Tor, что это и какие в этом тонкости?

0. Tor - это браузер с особой маршрутизацией запросов. Подробнее про то, как она работает и какую ПОТЕНЦИАЛЬНУЮ ОПАСНОСТЬ для вашего ПК она представляет можно почитать <a href="https://habr.com/ru/articles/357128/" target="_blank"> здесь </a>. В России на данный момент сайт Тора заблокирован, но если искать через него НЕЗАПРЕЩЕННЫЕ РЕСУРСЫ (одним из которых является Дром.ру), то за это вы НЕ НЕСЕТЕ НИКАКОЙ ОТВЕТСТВЕННОСТИ.
1. Скачать можно с <a href="https://www.torproject.org/" target="_blank"> официального сайта </a>.
2. Мосты для браузера можно получить в ОФИЦИАЛЬНОМ телеграмм-боте: ник @GetBridgesBot .
3. Добавить мосты себе в браузер: открыть браузер -> в правом верхнем углу нажать на иконку с тремя палочками -> "Настройки" -> "Подключение" -> "Вставить мосты".
4. Установим в PATH (если все происходит на Windows) путь к .exe файлу Тора.
5. Нужно придумать пароль для ControlPort и захэшировать. После пункта (4) можно написать в терминале <code>tor --hash-password <пароль></code> .
6. В файл, который находится вот так: "<директория для установки Тора>\Tor Browser\Browser\TorBrowser\Data\Tor\torrc" (torrc - нужный файл), нужно вставить вот это (файл перед этим можно открыть через Блокнот или Notepad):

<code>ControlPort 9051
HashedControlPassword <закодированный пароль></code>

7. Проверить работу браузера и порта можно так: открыть браузер, и в терминале написать команду <code> netstat -an | findstr 9050 </code> (ну и 9051 можно проверить).

## Что можно улучшить?

К сожалению, работа программы в некоторых местах предполагает ручные действия. На данный момент таковыми являются:

- Определение изначального пула автомобилей для парсинга.
- Ручная до-обработка некоторых ссылок на этапе сбора (о чем подробнее в коде на примере).
- Код не работает в случае, если на этапе первичного парсинга возникают ссылки на спецтехнику (пример: Volvo, долго с ним возился).
- Создание папок в репозитории для корректной систематизации спаршенных файлов.
- Отсутствие инструментария для копирования репозитория (я не знаю git).

Также отдельно стоит отметить bottle-neck в разделе первичного сбора ссылок: работа с эмулятором Гугл-хром через selenium не удовлетворяет требованиям к скорости (хотя и качественно справляется с поставленной задачей). Возможное решение проблемы: перепись под более быстрые библиотеки (например, Playwright).
