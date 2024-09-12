## 1. Тема и Целевая Аудитория

Steam — это цифровая платформа для дистрибуции видеоигр, программного обеспечения и общения, где пользователи могут покупать, скачивать, обновлять и играть в игры, а также взаимодействовать с игровым сообществом.

- Число активных пользователей
  - MAU (информация для конца 2021; с 2022 года Steam не предоставляет открытой информации по MAU и DAU): 132 млн. [^1]
  - В пике: 36.7 млн. [^2]
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

**На данный момент в steam более 70 тыс. игр, однако большинство из них - инди-игры, вес которых зачастую не доходит даже до 5 ГБ. Тем не менее, популярностью пользуются ААА-игры, размер которых может быть гораздо больше 15 ГБ. Поэтому с точки зрения "холодного" хранения исполняемых файлов, 15 ГБ - справедливое число, однако основная аудитория будет скачивать именно ААА-игры, поэтому под "горячее" хранение следует выбрать другое число и выбрать подходящий алгоритм для раздачи популярного контента*

[^1]: https://backlinko.com/steam-users
[^2]: https://store.steampowered.com/charts
[^3]: https://worldpopulationreview.com/country-rankings/steam-users-by-country
[^4]: https://www.demandsage.com/steam-statistics/
[^5]: https://store.steampowered.com/hwsurvey/



