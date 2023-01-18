# Домашнее задание к занятию 11.1. «Базы данных, их типы». Паромов Роман.

## Задание 1 СУБД

###
1.1. Бюджетирование проектов с дальнейшим формированием финансовых аналитических отчётов и прогнозирования рисков. СУБД должна гарантировать целостность и чёткую структуру данных.

  Подойдет любая реляционная СУБД.
  
1.1.* Хеширование стало занимать длительно время, какое API можно использовать для ускорения работы?

Можно использовать memcache, а лучше redis

1.2. Под каждый девелоперский проект создаётся отдельный лендинг, и все данные по лидам стекаются в CRM к маркетологам и менеджерам по продажам. Какой тип СУБД лучше использовать для лендингов и для CRM? СУБД должны быть гибкими и быстрыми.

Выбрал бы что то из PA СУБД, например kasandra для лендинга и mongoDB для CRM.

1.2.*  Думаю mongoDB справилась бы и с лендингом и с СRM.

1.3. Отдел контроля качества решил создать базу по корпоративным нормам и правилам, обучающему материалу и так далее, сформированную согласно структуре компании. СУБД должна иметь простую и понятную структуру.

Подошла бы любая реляционная СУБД.

1.4. Департамент логистики нуждается в решении задач по быстрому формированию маршрутов доставки материалов по объектам и распределению курьеров по маршрутам с доставкой документов. СУБД должна уметь быстро работать со связями.

Думаю подойдет графовая СУБД, например SPARQL

1.4.* Можно ли к этой СУБД подключить отдел закупок или для них лучше сформировать свою СУБД в связке с СУБД логистов?

Предполагаю что лучше своя СУБД в связке с СУБД логистов

## Задание 2. Транзакции
2.1. Пользователь пополняет баланс счёта телефона, распишите пошагово, какие действия должны произойти для того, чтобы транзакция завершилась успешно. Ориентируйтесь на шесть действий.

Обращение к БД с балансом карты, проверка реквизитов получателя, перевод денежных средсв, изменение баланса на карте, измение баланса на счете получателя, подтверждение транзакции.

## Задание 3. SQL vs NoSQL

3.1. Напишите пять преимуществ SQL-систем по отношению к NoSQL.

Простота использования, надежность, простота установки и настройки, работы с измененными данными на клиенте без изменения в БД, доступность.

## Здание 4. Кластеры
Необходимо производить большое количество вычислений при работе с огромным количеством данных, под эту задачу выделено 1000 машин.
На основе какого критерия будете выбирать тип СУБД и какая модель распределённых вычислений здесь справится лучше всего и почему?

Думаю нужна будет высокая доступность и реплицируемость данных в формате master-slave. Не совсем понял вопрос про "модель распределенных вычислений"
  
