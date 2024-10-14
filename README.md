## 1. Тема и Целевая Аудитория

Steam — это цифровая платформа для дистрибуции видеоигр, программного обеспечения и общения, где пользователи могут покупать, скачивать, обновлять и играть в игры, а также взаимодействовать с игровым сообществом.

- Число активных пользователей
  - MAU (информация для конца 2021; с 2022 года Steam не предоставляет открытой информации по MAU и DAU): 132 млн. [^1]
  - В пике: 36,7 млн. [^2]
![image](https://github.com/user-attachments/assets/e6b06e84-8030-4fc1-bdbb-1e96cd69a344)

- Целевая аудитория
  - По странам [^3]:
![image](https://github.com/user-attachments/assets/6facf817-839b-4c68-8a38-c2fdd7343895)
![image](https://github.com/user-attachments/assets/1fc000a9-055e-4985-9151-718764085915)
  - По возрасту [^4]:
    | Возрастная группа | Процент пользователей |
    |-------------------|-----------------------|
    | 18-19 лет         | 8%                    |
    | 20-29 лет         | 37%                   |
    | 30-39 лет         | 32%                   |
    | 40-49 лет         | 15%                   |
    | 50-59 лет         | 7%                    |
  - По характеристикам систем [^5]:
![image](https://github.com/user-attachments/assets/fd10cd76-62e8-46a3-8655-3b252e6b7d05)


- Требования к функционалу
  - MVP:
    - Регистрация и авторизация
    - Магазин игр
    - Страница игры с возможностью купить игру (или активировать ключ, купленный на сторонней площадке)
    - Библиотека игр пользователя
    - Установка купленной игры
    - Профиль
  - Особенности решения:
    - Копии игр хранятся в сжатом виде. Т.е. пользователь скачивает некий архив, который на лету разархивируется в исполняемые файлы. Таким образом можно сэкономить нагрузку на сеть и хранилище в ЦОДах, но при этом повысится нагрузка на ПК пользователей.
    - Steam работает как клиентское приложение на ПК (есть и браузерная версия, которая предоставляет ограниченный функционал) и использует gRPC вместо классического HTTP, что тоже снижает нагрузку на сеть.
   

## 2. Расчёт нагрузки

### Продуктовые метрики

- MAU: 135 млн. уникальных пользователей [^1]
- DAU: 69 млн. уникальных пользователей [^1]
- Дневной пик по количеству активных пользователей: 36.7 млн. [^4]

Средний занимаемый размер хранилища пользователем:
| Какие данные      | Их размер             |
|-------------------|-----------------------|
| Аватар пользователя         | 1 МБ        |
| Скриншоты пользователя      | 5 МБ        |
| Данные пользователя         | 1 МБ        |

**Под данными пользователя подразумеваются: никнейм, список друзей, комментарии в профиле, описание профиля, витрина достижений и т.п.*

Средний занимаемый размер хранилища продуктом (видеоигрой):
| Какие данные      | Их размер                |
|-------------------|--------------------------|
| Исполняемые файлы игры (*)         | 15 ГБ   |
| Текстовые описания и хар-ки игры   | 1 МБ    |
| Медиафайлы (изображения, трейлеры) | 20 МБ   |

**На данный момент в steam около 71 тыс. игр, однако большинство из них - инди-игры, вес которых зачастую не доходит даже до 5 ГБ. Тем не менее, популярностью пользуются ААА-игры, размер которых может быть гораздо больше 15 ГБ. Поэтому с точки зрения "холодного" хранения исполняемых файлов, 15 ГБ - справедливое число, однако основная аудитория будет скачивать именно ААА-игры, поэтому под "горячее" хранение следует выбрать другое число и выбрать подходящий алгоритм для раздачи популярного контента*

Основные действия по функционалу MVP:
| Действие | Кол-во в день на одного пользователя в среднем |
|--------------------|-----------------------|
| Авторизация                           | 0,032   (1 в месяц)   |
| Регистрация                           | 0,0027  (1 в год)     | 
| Открытие и просмотр страницы магазина | 1                     |
| Покупка игры                          | 0,032 (1 в месяц)     |
| Написание отзыва на игру              | 0,021 (примерно 2/3 юзеров оставляют отзывы [^6] и оставить его можно только после покупки) |
| Просмотр профиля пользователя         | 0,5                   |
| Написание комментария в профиль       | 0,032 (1 в месяц)     |
| Загрузка скриншота                    | 0,142 (1 в неделю)    |
| Отправка заявки в друзья              | 0,032 (1 в месяц)     |
| Скачивание или обновление игры        | 0,142 (1 в неделю)    |

### Технические метрики

- Хранилище:
  На данный момент в Steam около 71 тыс. игр [^7]. Средний размер игры - 15 ГБ (пояснение см. выше). Итак, 15 ГБ * 71 000 ~ 1 040 ТБ ~ 1,01 ПБ на хранение исполняемых файлов. Для хранения информации, находяшейся на странице игры в магазине, нужно хранить тексктовые данные (включая локализации, технические характеристики и т.п.) ~ 1 МБ и медиафайлы ~ 20 МБ, тогда (20 МБ + 1 МБ) * 71 000 = 1 456 ГБ ~ 1,42 ТБ.
  Открытой информации по количеству аккаунтов в Steam на 2024 или хотя бы 2023 нет, однако известно, что к 2020 были зарегистрированы 667 млн. аккаунтов [^8], а пик игроков - 24,8 млн. [^9]. Если считать, что зависимость между пиком игроков и количеством зарегистрированных пользователей линейная, то получаем 36 700 000 / 24 800 000 * 667 000 000 = 987 млн. игроков. Тогда: 987 000 000 * (1 МБ + 5 МБ + 1 МБ) ~ 6588,93 ТБ ~ 6,43 ПБ

  Итого на хранилище: ~ 7,44 ПБ

- Сетевой трафик:
  Настоящая статистика за последние 48 часов по данным Steam [^10]:  <img width="905" alt="image" src="https://github.com/user-attachments/assets/8bb8f79b-c0e2-46af-9300-f23f2a64d6b2">

  Попробуем посчитать средний трафик для MVP, описанного выше. Основной трафик будет приходиться на просмотр магазина, профиля и скачивание \ обновление игры:
  - Просмотр страницы игры в магазине: 1 * 69 000 000 * (20 МБ + 1 МБ) / 24 / 60 / 60 = 16,37 ГБ/c = 131,02 Гбит/c
  - Просмотр профиля игрока: 0,5 * 69 000 000 * (1 МБ + 5 МБ + 1 МБ) / 24 / 60 / 60 = 2,73 ГБ/c = 21,84 Гбит/с
  - Скачивание \ обновление игры: 0,142 * 69 000 000 * 15 ГБ / 24 / 60 / 60 = 1701 ГБ/с = 1,66 ТБ/c = 13,29 Тбит/с

  То же, но в пике (по графику выше это примерно x1,5):
  - Просмотр страницы игры в магазине: 131,02 Гбит/c * 1,5 = 196,53 Гбит/с
  - Просмотр профиля игрока: 21,84 Гбит/с * 1,5 = 32,76 Гбит/с
  - Скачивание \ обновление игры: 13,29 Тбит/с * 1,5 = 19,94 Тбит/с

  Потребление трафика за сутки:
  - Просмотр страницы игры в магазине: 131,02 Гбит/c * 60 * 60 * 24 = 10,8 Пбит = 1,35 ПБ
  - Просмотр профиля игрока: 21,84 Гбит/с * 60 * 60 * 24 = 1,8 Пбит = 0,025 ПБ
  - Скачивание \ обновление игры: 13,29 Тбит/с * 60 * 60 * 24 = 1121 Пбит = 140 ПБ

- RPS

  Используя данные выше, получаем:
  | Действие      | RPS             | RPS с учётом пика (x1,5) |
  |---------------|-----------------|--------------------------|
  | Авторизация                           | 26,3 RPS   | 39,45 RPS |
  | Регистрация                           | 2,2 RPS    | 3,3 RPS   |
  | Открытие и просмотр страницы магазина | 798 RPS    | 1197 RPS  |
  | Покупка игры                          | 26,3 RPS   | 39,45 RPS |
  | Написание отзыва на игру              | 16,8 RPS   | 25,2 RPS  |
  | Просмотр профиля пользователя         | 399 RPS    | 598,5 RPS |
  | Написание комментария в профиль       | 26,3 RPS   | 39,45 RPS |
  | Загрузка скриншота                    | 113 RPS    | 169,5 RPS |
  | Отправка заявки в друзья              | 26,3 RPS   | 39,45 RPS |

## 3. Глобальная балансировка
  - **Разбиение по доменам**  
    Далее в качестве домена второго уровня будет использоваться domain.com как пример. Будем считать, что domain.com работает на раздачу фронтенда, а бекенд распределён по поддоменам, которые будут рассмотрены далее.
    В соответствии с функционалом, описанном в MVP, будет логично выделить следующие поддомены:
    - **store.domain.com**  
      Домен для раздела с магазином игр и приложений, включая отзывы, а также для внутриигровых API, отвечающих за подтверждение владения игрой. Также в рамках MVP возможно использование этого домена для процессинга: не только для оплаты банковскими картами, но и другими различными методами пополнения вроде подарочных карт. Но для дальнейшего развития будет предпочтительнее вынести это в отдельный сервис (а соответственно и домен), т.к. существуют и другие части Steam, зависящие от работы с реальными деньгами (например, торговая площадка). В этой работе в рамках MVP будем подразумевать, что процессинг работает только в магазине игр и приложений.
    - **download.domain.com**  
      Домен для раздачи цифровых копий игр и приложений, выступающий в роли CDN. Это самый загруженный по объёму трафика домен.
    - **community.domain.com**  
      Домен для игровых сообществ. В контексте функционала MVP будет служить для работы с профилями игроков и внутриигровыми приглашениями в игровые лобби, а в перспективе под этим доменом могут располагаться сообщества, игровые гайды и прочие элементы игрового сообщества.
    - **auth.domain.com**  
      Домен для авторизации и регистрации, интеграций с другими платформами, а также для функционирования части внутриигровых API. Стоит обратить внимание, что этот домен выступает внешней частью всего сервиса авторизаций - бОльшая часть функций самого сервиса так или иначе будет доступна лишь во внутренней корпоративной сети для использования другими сервисами платформы.

  **Можно сделать замечание, что Steam работает с огромным количеством медиаданных, для которых было бы логично спроектировать CDN, однако в рамках MVP функциональность профилей ограничена, а игровых сообществ нет вообще, из-за чего единственной частью, использующей много картинок и видео, является магазин. Поэтому будем считать, что для MVP реализация CDN не нужна и каждый сервис сам отвечает за распространение своих медиаданных. CDN для распространения цифровых копий существует отдельно и не предназначен для распространения медиафайлов*
  - **Расположение ЦОДов**  
    Steam работает по всему миру, поэтому ЦОДы не следует сосредотачивать в одной стране. Тем не менее, объём трафика из разных регионов сильно различается. Чтобы определить основных потребителей трафика, стоит обратиться с скриншоту выше в разделе 2.  
    <img width="450" alt="image" src="https://github.com/user-attachments/assets/8bb8f79b-c0e2-46af-9300-f23f2a64d6b2">  
    Итак, имеем следующее распределение:  
    | Регион      | Процент от общего объёма трафика |
    |-------------|----------------------------------|
    | Северная Америка      | 24,1%   |
    | Европа                | 33,3%   |
    | Азия                  | 53,5%   |
    | Россия (и СНГ)        | 6,5%    |
    | Южная Америка         | 8,6%    |
    | Австралия (и Океания) | 2,2%    |
    | Средний Восток        | 2,8%    |
    | Африка                | 0,5%    |
    | Центральная Америка   | 0,3%    |
    
    Таким образом получаем, что ЦОДы можно расположить в следующих локациях:
      - Амстердам
      - Франкфурт
      - Хельсинки
      - Лондон
      - Мадрид
      - Париж
      - Стокгольм
      - Вена
      - Варшава
      - Атланта
      - Чикаго
      - Даллас
      - Лос-Анджелес
      - Сиэтл
      - Стерлинг
      - Буэнос-Айрес
      - Лима
      - Сантьяго
      - Ченнаи
      - Мумбаи
      - Дубай
      - Гонконг
      - Сеул
      - Сингапур
      - Токио
      - Чэнду
      - Гуанчжоу
      - Шанхай
      - Тяньцзинь
      - Сидней
      - Йоханнесбург
      - Екатеринбург
  
Посмотреть на интерактивной карте: https://yandex.ru/maps/?um=constructor%3A148aa1448d3e7cbeb705f39b1b22c0e8bd088f0831540f9bdb374171b2936639&source=constructorLink  
Список был составлен на основе не только основных потребителей трафика, но и основных точек присутствия провайдеров [^11] и подводных магистральных сетей [^12]. Нужно понимать, что разные сервера могут держать на себе различные сервисы. Так, список выше подходит для размещения серверов с основной логикой платформы; для CDN же этот список стоит значительно расширить, но в рамках работы будем считать, что CDN расположены там же.  
Таким образом, при таком списке мы получим снижение латентности и улучшим доступность платформы.
  - **Расчет распределение запросов из секции "Расчет нагрузки" по типам запросов по датацентрам**
    RPS по действиям считается на основе пика RPS
    | Регион                | % от всех пользователей  | Авторизация | Регистрация | Просмотр магазина  | Покупка игры | Написание отзыва | Просмотр профиля | Отправка комментария в профиль | Загрузка скриншота | Отправка заявки в друзья |
    |-----------------------|--------------------------|-------------|-------------|--------------------|--------------|------------------|------------------|--------------------------------|--------------------|--------------------------|
    | Северная Америка      | 24,1%                    | 8,2         | 0,8         | 288,5              | 8,2          | 6,1              | 144,3            | 8,2                            | 40,8               | 8,2                      |
    | Европа                | 33,3%                    | 13,1        | 1,1         | 398,6              | 13,1         | 8,4              | 199,3            | 13,1                           | 56,4               | 13,1                     |
    | Азия                  | 53,5%                    | 21,1        | 1,8         | 640,4              | 21,1         | 13,5             | 320,2            | 21,1                           | 90,7               | 21,1                     |
    | Россия (и СНГ)        | 6,5%                     | 2,6         | 0,2         | 77,8               | 2,6          | 1,6              | 38,9             | 2,6                            | 11                 | 2,6                      |
    | Южная Америка         | 8,6%                     | 3,4         | 0,3         | 102,9              | 3,4          | 2,2              | 51,5             | 3,4                            | 14,6               | 3,4                      |
    | Австралия (и Океания) | 2,2%                     | 0,9         | 0,1         | 26,3               | 0,9          | 0,6              | 13,2             | 0,9                            | 3,7                | 0,9                      |
    | Средний Восток        | 2,8%                     | 1,1         | 0,1         | 33,5               | 1,1          | 0,7              | 16,8             | 1,1                            | 4,7                | 1,1                      |
    | Африка                | 0,5%                     | 0,2         | 0,01        | 6                  | 0,2          | 0,1              | 3                | 0,2                            | 0,8                | 0,2                      |
    | Центральная Америка   | 0,3%                     | 0,1         | 0,01        | 3,6                | 0,1          | 0,1              | 1,8              | 0,1                            | 0,5                | 0,1                      |
  
  - **Схема DNS-балансировки**  
    За счёт географических масштабов проекта здесь может применяться GeoDNS или Latency-based DNS. Т.к. сама концепция Steam не подразумевает, что нам необходимо обеспечивать очень низкий пинг, то будем использовать GeoDNS ввиду более простой реализации.

## 4. Локальная балансировка
### Схема балансировки для входящих и межсервисных запросов
В этом разделе стоит отдельно рассмотреть CDN и остальные сервисы. Специфика CDN заключается в огромном объёме передаваемого трафика, но при этом он слабо связан с другими сервисами (а если быть точным, то CDN перед началом загрузки игры лишь один раз обращается в сервис авторизации и один раз в сервис магазина для подтверждения владения пользователем игры, затем он работает полностью автономно). В то же время остальные сервисы так или иначе будут постоянно общаться между собой и, скорее всего, делить одни и те же физические БД. К тому же, для CDN нужно гораздо больше серверов и сетевых каналов, поэтому количество ЦОДов для CDN будет гораздо больше количества ЦОДов, где будут остальные сервисы. Таким образом, ЦОДы для CDN и остальных сервисов будут распределены по-разному. 

Помимо этого, будем подразумевать, что мы делаем свой кастомный CDN, схему которого можно посмотреть вместе со схемой балансировки ниже.

![image](https://github.com/user-attachments/assets/8779f0da-8090-48fa-b167-b7b9a510defa)  


### Схема отказоустойчивости
Для отказоустойчивости будут применяться инструменты k8s и nginx. Kubernetes с помощью readiness-проб обеспечит исключение из кластера упавших подов и добавление новых, а также сможет автоматически поднимать упавшие сервера. Если предположить, что ЦОДы будут не нашими, а арендованными, то kubernetes также позволит выполнять удобный auto-scaling, чтобы подстраиваться под актуальную нагрузку. Nginx умеет выполнять retry запросов после падения бэкенда, перезапускать web-сервера без downtime, равномерно распределять запросы по бэкендам, а также незаметно для пользователя перезагружать конфиг при обновлениях.

### Нагрузка по терминации SSL
На современных серверах ssl handshake может занимать от 2 мс при сокращённом рукопожатии до 20 мс при полном рукопожатии [^13], при этом это время полного ответа, в то время как сами вычисления на процессоре занимают 0.5 - 0.6 мс. При последующих расчётах будем считать, что ssl handshake занимает 3 мс, т.к. мы будет использовать сокращённые рукопожатия в nginx, то есть сервер и клиент будут согласны на переиспользование защищённой сессии, которую использовали ранее (session tickets).

В таком случае, исходя из пикового RPS по всем основным ручкам (2151.3 RPS) получаем, что на обработку SSL каждую секунду потребуется 2151.3 * 3 = 6453.9 мс = 6.5 секунд вычислительного времени. 

## 5. Логическая схема БД
### Схема  
<img width="1319" alt="image" src="https://github.com/user-attachments/assets/edf98f0f-0a25-40d5-a487-af959df315a8">

### Размеры данных и консистентность
| Таблица             | Размеры данных                                                                                                           | Консистентность |
|---------------------|--------------------------------------------------------------------------------------------------------------------------|-----------------|
| session             | (16 token + 8 user_id + 8 expire) * 135 млн. MAU = **4 ГБ**                                                              | - |
| user                | (8 id + 64 nick + 64 login + 128 pass_hash + 64 pass_salt + 8 created_at + 8 avatar) * 1 млрд пользователей = **320 ГБ** | при удалении user удаляются все его session, review, friendship_request, friendship, upload, purchase, profile_comment |
| review              | (8 id + 8 user_id + 8 game_page_id + 1024 comment + 1 vote + 8 created_at) * 125 млн. обзоров = **123 ГБ**               | при удалении или изменении review пересчитываем game_page.rating |
| friendship_request  | (8 user_from + 8 user_to + 1 status + 8 created_at) * 1 млрд. пользователей * ~3 заявки от каждого = **69 ГБ**           | - |
| upload              | (8 id + 8 user_id + 128 path) * 1 млрд. пользователей * ~3 аплоада от каждого = **402 ГБ**                               | при удалении также удаляются связанные screenshot |
| screenshot          | (8 id + 8 upload_id) * 1 млрд. пользователей = **15 ГБ**                                                                 | - |
| friendship          | (8 user1 + 8 user2 + 8 created_at) * 1 млрд. пользователей * ~2 друга у каждого = **44 ГБ**                              | - |
| purchase            | (8 id + 8 game_id + 8 user_id + 8 purchased_at) * 580 млн. покупок = **17 ГБ**                                           | - |
| profile_comment     | (8 id + 8 author_id + 8 user_id + 128 comment + 8 created_at) * 340 млн. комментариев = **50 ГБ**                        | - |
| mediafile           | (8 id + 128 path) * 100 тыс. игр * 10 медиафайлов = **129 МБ**                                                           | при удалении также удаляются связанные game_page_mediafile |
| game                | (8 id + 128 path) * 100 тыс. игр = **12 МБ**                                                                             | при удалении также удаляются связанные game_page и purchase |
| game_page           | (8 id + 8 game_id + 16 title + 10*1024 description + 256 requirements + 4 rating) * 100 тыс. игр = **1 ГБ**              | при удалении также удаляются связанные review |
| game_page_mediafile | (8 game_id + 8 mediafile_id) * 100 тыс. игр * 10 медиафайлов = **15 МБ**                                                 | - |

### Как будем хранить
Все данные в системе можно разделить на несколько категорий: структурированные данные (база данных, см. выше), файловые данные (медиафайлы, скриншоты), кеши.
- Структурированные данные: База данных содержит информацию о пользователях, друзьях, покупках, обзорах, комментариях и т.д.
- Файловые данные: Медиафайлы, скриншоты загружаются пользователями и играми, эти файлы требуют особого подхода к хранению и управлению, их мы будем хранить в обычной файловой системе.
- Кеши и буферы: Для ускорения операций чтения и записи (например, кеширование популярных страниц магазина) можно (и нужно!) использовать in-memory решение, например tarantool.

### Нагрузка на чтение/запись
Исходя из данных по RPS (запросы в секунду) и средней активности пользователей (см. предыдущие пункты):

#### Чтение:
Основная нагрузка на чтение приходится на просмотр страницы магазина (1197 RPS в пиковые моменты) и просмотр профилей пользователей (598,5 RPS).
Следовательно, кеширование популярных страниц магазина и профилей пользователей может существенно снизить нагрузку на базу данных.
Для файловых данных (скриншоты, медиафайлы) необходим быстрый доступ, будем использовать кастомный CDN, под капотом у которого будут nginx и обычное файловое хранилище, см. ДЗ 4.
#### Запись:
Основная нагрузка на запись связана с загрузкой скриншотов (169,5 RPS), покупками игр (39,45 RPS), написанием отзывов (25,2 RPS), и заявками в друзья (39,45 RPS).

### Требования к консистентности
- Сессии: Должны быть консистентными, чтобы избежать ситуаций с несколькими активными сессиями для одного пользователя.
- Пользователи: При удалении пользователя необходимо удалить все связанные данные (сессии, обзоры, покупки и т.д.), что требует поддержания целостности данных и консистентности связей.
- Отзывы и рейтинг игр: При изменении или удалении отзыва требуется немедленный пересчет рейтинга игры, что требует сильной консистентности для обновления связанных данных. Возможно, следует выполнить денормализацию.
- Комментарии и профили: Могут работать с более слабой консистентностью, обновления могут происходить с некоторой задержкой.

### Распределение нагрузки по ключам
Нагрузку по ключам необходимо распределять таким образом, чтобы избежать концентрации операций на одном узле:
- User ID: В большинстве таблиц присутствует user_id, что делает этот ключ основным для шардирования. Например, таблицы session, review, friendship, upload могут быть распределены по шардированию по user_id.
- Game ID: Таблицы, связанные с играми (например, purchase, review, game_page), могут быть шардированы по game_id, что позволит сбалансировать нагрузку на обработку информации по разным играм.
Таблицы со связями (например, friendship, friendship_request) требуют особого подхода, возможно использование двунаправленного шардирования, чтобы обе стороны связи (пользователь 1 и пользователь 2) были на одном шарде.

### Кеширование и буферы
- Популярные страницы игр: Использование кеша для хранения популярных страниц игр, данных о рейтингах и описаниях.
- Профили пользователей: Для ускорения загрузки профилей можно кешировать данные профилей наиболее активных пользователей.
- Сессии: Могут храниться в Redis с автоматическим удалением по истечении срока действия токена.

## 6. Физическая схема БД
### Схема
<img width="1319" src="https://github.com/user-attachments/assets/c751e95c-5a04-408e-a2db-84434f5f6e35">

### Таблица с описанием таблиц
| Таблица            | Поля                                                                  | Описание                                                                      |
|--------------------|-----------------------------------------------------------------------|-------------------------------------------------------------------------------|
| user               | id, nickname, login, password_hash, password_salt, created_at, avatar | Пользователи системы (идентификаторы, информация для аутентификации, аватары) |
| session            | token, user_id, expiry_datetime                                       | Хранение сессий пользователей для аутентификации                              |
| profile_comment    | id, author_id, user_id, comment, created_at                           | Комментарии к профилям пользователей                                          |
| purchase           | id, game_id, user_id, purchased_at                                    | Информация о покупках пользователей                                           |
| friendship         | user1, user2, created_at                                              | Информация о дружеских отношениях между пользователями                        |
| friendship_request | user_from, user_to, status, created_at                                | Запросы на добавление в друзья                                                |
| review             | id, user_id, game_page_id, comment, vote, created_at                  | Отзывы пользователей на игры                                                  |
| game               | id, path                                                              | Игры в системе                                                                |
| game_page          | id, game_id, title, description, requirements, rating                 | Страницы с описаниями игр                                                     |
| mediafile          | game_id, path, type                                                   | Медиафайлы, связанные с играми (например, скриншоты или трейлеры)             |
| upload             | id, user_id, type, path                                               | Загруженные пользователями файлы (например, аватары)                          |

### Индексы
Для повышения производительности запросов нужно добавить индексы на поля, которые чаще всего используются в условиях поиска или объединяются с другими таблицами.
Используя информацию по rps из предыдущих пунктов, можно выделить следующие индексы:
- user:
  - Индекс на поле login для быстрого поиска пользователей по логину
  - Индекс на поле nickname для поиска по псевдониму
  - Индекс на поле id как primary key
- purchase:
  -	Индексы на user_id и game_id для ускорения поиска покупок по пользователю и игре
- friendship:
  - Индексы на user1 и user2 для быстрого поиска друзей
- friendship_request:
  - Индексы на user_from и user_to для быстрого поиска запросов на дружбу
- review:
  - Индекс на game_page_id для поиска отзывов по играм
- game:
  - Индекс на id как primary key
- game_page:
  - Индекс на поле game_id для ускорения поиска страниц игр

### Денормализация
Для ускорения работы с БД можно применить денормализацию. Денормализуем основные таблицы, к которым часто идут обращения и для избежания операций с JOIN:
- Добавление reviews_count и votes в таблицу game_page. При каждом добавлении \ удалении \ редактировании отзыва эти поля в game_page будут также меняться, что в дальнейшем позволит при обращении к странице магазина моментально рассчитывать рейтинг игры без вызова count или avg на таблицу review.
- Добавление author_nickname и avatar в profile_comment. Это ускорит запросы на получение комментариев профиля, т.к. в таком случае можно будет избежать JOIN'ов.
- Аналогично пункту выше: добавление user_nickname_from, user_nickname_to, avatar_from, avatar_to в friendship_request, чтобы так же избежать JOIN'ов.

### Выбор СУБД
- user, profile_comment, purchase, friendship, friendship_request, review, game, game_page, mediafile, upload:
  Реляционные данные с хорошей поддержкой транзакционности, нормализации и связи – здесь хорошо подойдет PostgreSQL.
- mediafile, upload, game:
  Метаданные хранятся в PostgreSQL, но сами файлы будут располагаться в обычном файловом хранилище (ext4, xfs), которое находится за nginx.  
  В случае с game лучше использовать xfs, т.к. он больше подходит для хранения большого объёма крупных данных. Т.к. игры будут храниться в виде архивов (чаще всего крупных архивов), то xfs идеально подходит для хранения игр.  
  Для mediafile и upload (метаданные также хранятся в PostgreSQL), где файлы в основном не очень большого размера, но их много, лучше подходит Ext4.
  Для ext4 максимальный размер файловой системы составляет 1 ЭБ, а для xfs - 8 ЭБ. В настоящее время и в ближайщем будущем этого будет хватать с огромным запасом. В реальных условиях проблема будет скорее экономической, т.к. SSD- и HDD-хранилища не бесплатные.
- session:
  Обращение к этой таблице очень частое, следовательно, требуется быстрый отклик. Было бы логично использовать in-memory решение, например redis. К тому же, если мы выбираем redis или tarantool, то нужно понимать, что они хорошо работают с lua, что позволяет легко писать скрипты для nginx, соответственно работу с сессиями можно будет осуществлять на уровне nginx

### Шардирование
- user, friendship, friendship_request, purchase:
  Шардирование на основе user_id, так как user - самая крупная и основная таблица. Могут быть шардированы по диапазону идентификаторов пользователей (например, диапазоны user_id 1–1 млн на одном сервере, 1 млн+ на другом)
- game, game_page, mediafile, review:
  Шардирование по game_id для распределения нагрузок между различными серверами хранения данных об играх.

Особое внимание стоит обратить на friendship и friendship_request. В них есть поля, которые условно ссылаются на user1 и user2, при этом user1 и user2 могут находиться на разных шардах. Соответственно, стоит применить какую-то особую схему шардирования для оптимальности работы таблицы. В данном случае идеально подойдет двойное шардирование: с одной стороны, это раздувает размер БД (что не так критично, т.к. эти таблицы невелики по размеру) и затрудняет операции вставки, обновления и удаления (что тоже не столь критично, т.к. подобные операции происходят редко, см. предыдущие пункты), зато позволяет избежать запросов между шардов.

### Клиентские библиотеки и интеграции
Для работы с медиафайлами можно использовать стандартные библиотеки языков + стандартные возможности nginx.  
Для работы с PostgreSQL будем использовать sqlx для работы с sql-БД и pq в качестве драйвера для postgres. sqlx адаптирован к встроенным интерфейсам библиотеки sql в golang и позволяет удобно переключаться между разными SQL-БД, меняя лишь драйвер, что может пригодиться при необходимостии миграции.  
Для работы с redis будет использовать lua-resty-redis для написания плагинов в nginx на lua и официальную библиотеку go-redis для golang.  

### Балансировка запросов и мультиплексирование подключений
Для балансировки запросов и их мультиплексирования хорошо подойдет pgBouncer:
  - pooling: позволяет сократить количество активных подключений к PostgreSQL, одновременно поддерживая большое число клиентов. Он действует как посредник между клиентом и сервером, переиспользуя активные подключения вместо того, чтобы открывать новые. Клиенты подключаются к PgBouncer, а тот передаёт запросы уже к базе данных через пул подключений.
  - экономия ресурсов: Postgres плохо масштабируется при большом количестве активных подключений, так как каждое из них потребляет оперативную память и вычислительные ресурсы. PgBouncer позволяет ограничить количество соединений с базой данных, значительно снижая нагрузку на сервер.
  - оптимизация для микросервисной архитектуры: В архитектуре микросервисов или систем с большим количеством клиентов каждая часть системы может требовать свое подключение к базе данных. PgBouncer позволяет агрегировать эти подключения, ограничивая их до разумного числа, снижая вероятность перегрузки базы.
  - минимизация задержек: PgBouncer ускоряет работу с базой данных, так как клиентские приложения больше не ждут открытия новых подключений. Уже открытые соединения передаются в пуле, что значительно уменьшает время отклика на запросы.
  - балансировка: у нас несколько реплик PostgreSQL для чтения, потому pgBouncer можно настроить на отправку запросов на разные серверы, распределяя нагрузку. Это можно реализовать с помощью внешних балансировщиков запросов, например nginx, или средствами pgBouncer при помощи подключения к нескольким инстансам баз данных при мастер/слейв-репликах.
  - фильтрация запросов при перегрузке: pgBouncer может фильтровать или отклонять запросы в случае перегрузки или превышения лимитов на соединения. Это позволяет предотвратить ситуацию, когда сервер базы данных перегружен и становится недоступным.

### Схема резервного копирования
Для Postgres можно использовать pg_dump или point-in-time recovery с помощью write-ahead logging.  
Для redis требования ниже, т.к. в случае чего мы можем просто попросить пользователя повторно авторизоваться (хоть и нежелательно, т.к. это портит user experience), при этом здесь очень высокие требования к скорости чтения\записи, поэтому можно обойтись rdb. Он не гарантирует полного сохранения текущего состояния, зато гораздо производительнее, чем режим AOF.



[^1]: https://backlinko.com/steam-users
[^2]: https://store.steampowered.com/charts
[^3]: https://worldpopulationreview.com/country-rankings/steam-users-by-country
[^4]: https://www.demandsage.com/steam-statistics/
[^5]: https://store.steampowered.com/hwsurvey/
[^6]: https://www.retail.ru/news/issledovanie-rekomendatsii-i-otzyvy-v-internete-chitayut-99-onlayn-pokupateley-2-sentyabrya-2020-197335/
[^7]: https://gamalytic.com/blog/steam-revenue-infographic
[^8]: https://cyber.sports.ru/games/1082667359-v-steam-zaregistrirovano-okolo-667-millionov-akkauntov.html
[^9]: https://app2top.ru/analytics/120-mln-aktivny-h-pol-zovatelej-ezhemesyachno-i-31-3-mlrd-igrovy-h-chasov-steam-podvel-itogi-2020-goda-180149.html
[^10]: https://store.steampowered.com/stats/content/
[^11]: https://www.internetexchangemap.com/
[^12]: https://www.submarinecablemap.com/
[^13]: https://www.ibm.com/docs/en/cics-ts/6.x?topic=performance-ssl-handshake-overhead 



