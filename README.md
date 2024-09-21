### Задание 1. СУБД

### Кейс
Крупная строительная компания, которая также занимается проектированием и девелопментом, решила создать 
правильную архитектуру для работы с данными. Ниже представлены задачи, которые необходимо решить для
каждой предметной области. 

Какие типы СУБД, на ваш взгляд, лучше всего подойдут для решения этих задач и почему? 
 
1.1. Бюджетирование проектов с дальнейшим формированием финансовых аналитических отчётов и прогнозирования рисков.
СУБД должна гарантировать целостность и чёткую структуру данных.

1.1.* Хеширование стало занимать длительно время, какое API можно использовать для ускорения работы? 

1.2. Под каждый девелоперский проект создаётся отдельный лендинг, и все данные по лидам стекаются в CRM к 
маркетологам и менеджерам по продажам. Какой тип СУБД лучше использовать для лендингов и для CRM? 
СУБД должны быть гибкими и быстрыми.

1.2.* Можно ли эту задачу закрыть одной СУБД? И если да, то какой именно СУБД и какой реализацией?

1.3. Отдел контроля качества решил создать базу по корпоративным нормам и правилам, обучающему материалу 
и так далее, сформированную согласно структуре компании. СУБД должна иметь простую и понятную структуру.

1.3.* Можно ли под эту задачу использовать уже существующую СУБД из задач выше и если да, то как лучше это 
реализовать?

1.4. Департамент логистики нуждается в решении задач по быстрому формированию маршрутов доставки материалов 
по объектам и распределению курьеров по маршрутам с доставкой документов. СУБД должна уметь быстро работать
со связями.

1.4.* Можно ли к этой СУБД подключить отдел закупок или для них лучше сформировать свою СУБД в связке с СУБД 
логистов?

1.5.* Можно ли все перечисленные выше задачи решить, используя одну СУБД? Если да, то какую именно?

*Приведите ответ в свободной форме.*

---

### Решение 1

- 1.1 Реляционная СУБД: PostgreSQL, Microsoft SQL server.
    - Обеспечивает целостность данных, поддерживает сложные запросы, сложные структуры данных и взаимосвязи.
- 1.2 
    - Для лендингов лучше подходит NoSQL СУБД, т.к. поддерживает хранение документов, обеспечивает высокую производительность при больших объемах данных.
    - Для CRM подходит реляцционная СУБД с поддержкой четкой структуры и транзакций.
    1.2.* Для обеих задач можно использовать СУБД PostgreSQL, комбинируя реляционные таблицы и JSON-документы.
- 1.3
    - Если не требуется строгое структурирование и данные можно хранить в виде докуметов - NoSQL, в другом случае - реляционная база данных.
- 1.4
    - Подходит графовая СУБД, напр. Neo4j. Работает с графовыми структурами.
- 1.5.* Для всех задач может подойти одна достаточная мощная СУБД с обширным функционалом, напр. PostgreSQL. СУБД имеет возможность работы с документами и сторонними расширениями для работы с графами. Но если требуется максимальная производительность, масштабируемость и имеется достаточное кол-во ресурсов - лучше всего использовать отдельные продукты, ориентированные на каждую отдельную задачу.

### Задание 2. Транзакции

2.1. Пользователь пополняет баланс счёта телефона, распишите пошагово, какие действия должны произойти для того, чтобы 
транзакция завершилась успешно. Ориентируйтесь на шесть действий.

### Решение 2

- Ввод данных пользователем: номер телефона, сумма пополнения, метод оплаты.
- Проверка данных системой: валидный ли номер телефона, верно ли указаны данные.
- Создание запроса: запрос передается в платежную систему.
- Обработка запроса: проверка наличия платежных средств.
- Списание средств: платежная система списывает средства, отправляет подтверждение платежа оператору
- Обновление баланса: оператор получил подтверждение, баланс средств обновляется.

2.1.* Какие действия должны произойти, если пополнение счёта телефона происходило бы через автоплатёж?

- Запуск автоплатежа по выбранному расписанию, создание запроса на пополнение.
- Шаги, аналогичные ручному пополнению.

Мне не известны методы взаимодействия при выполнении автоплатежа. К тому же имеет значение, с какой стороны он выполняется (со стороны мобильного оператора или же создан в приложении банка). При разных условиях этапы должны отличаться..

---

### Задание 3. SQL vs NoSQL

3.1. Напишите пять преимуществ SQL-систем по отношению к NoSQL. 
3.1.* Какие, на ваш взгляд, преимущества у NewSQL систем перед SQL и NoSQL.

### Решение 3

3.1

- Строгая схема - фиксированные схемы данных, структурируемость
- Целостность информации - транзакции, подходит для хранения критически важных данных
- Поддержка сложных запросов, сложных взаимосвязей между элементами данных
- Распространенность, широкая поддержка и совместимость
- Большие возможности по интеграции, составлению отчетов и аналитики

3.1.*

- Масштабируемость
- Поддержка ACID (в отличии от NoSQL)
- Производительность
- Гибкость
- Совместимость с синтаксисом SQL
---

### Задание 4. Кластеры

Необходимо производить большое количество вычислений при работе с огромным количеством данных, под эту задачу 
выделено 1000 машин. 

На основе какого критерия будете выбирать тип СУБД и какая модель *распределённых вычислений* 
здесь справится лучше всего и почему?

### Решение 4

Критерии выбора - масштабируемость и производительность. Так же поддержка распределенных вычислений и надежность.

Модель MapReduce, так как соответствует основным критериям выбора и подходит для распределенной обработки больших объемов данных.
