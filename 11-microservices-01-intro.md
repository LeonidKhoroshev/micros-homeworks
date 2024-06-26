# Домашнее задание к занятию «Введение в микросервисы» - Хорошев Леонид

## Задача

Руководство крупного интернет-магазина, у которого постоянно растёт пользовательская база и количество заказов, рассматривает возможность переделки своей внутренней ИТ-системы на основе микросервисов. 

Вас пригласили в качестве консультанта для оценки целесообразности перехода на микросервисную архитектуру. 

Опишите, какие выгоды может получить компания от перехода на микросервисную архитектуру и какие проблемы нужно решить в первую очередь.

#### Для определения целесообразности перехода на микросервисную архитектуру необходимо проанализировать следующие параметры:
- рост нагрузки. Нам необходимо определить, есть ли у монолита предел масштабирования и насколько текущее состояние системы к нему приближено. Для этого необходимо определить темп роста количества заказов и транзакций в интернет-магазине и сравнить примерную стоимость реализации проекта перехода на микросервисную архитектуру с затратами на ресурсы, достаточные для "развития" монолита (приобретение серверного оборудования, аренда облачных ресурсов). При анализе необходимо учитывать, что стоимость вычислитнельных ресурсов не линейна (процессор в два раза мощнее может стоить не в два, а в четыре раза дороже);
- рост системы. Необходимо понимать насколько быстро развивается функционал интернет-магазина, сколько добавляется новых "фич" (способы оплаты, системиы бонусных баллов, сравнение товаров и т.д.) и насколько они усложняют кодовую базу монолита. Как правило с развитием проекта становится сложнее добавлять новые функций без снижения отказоустойчивости с сохранением работоспособности монолита на прежнем уровне. Разработка становится дороже;
- потребность в градации в данных. С точки зрения бизнес-логики работы интернет-магазина необходим какой-то модуль, доступ к которому должен быть быстрее, чем к остальным. Например, требуется, чтобы в интернет-магазине доступ к популярным товарам или уже собранной корзине стал быстрее. В монолите данный функционал можно выполнить, только обеспечив быстрый доступ ко всем данным (корзине, каталогу и др.), что слишком затратно;
- если по результатам анализа стало ясно, что монолит не сможет удовлетвоять требованиям бизнеса (с учетом темпов его развития), то необходимо перепроектировать систему на основе микросервисов. Так как процесс разделения приложения на отдельные составляющие довольно сложный и требует много времени и ресурсов, на которые бизнес не готов по причине вероятных издержек, разделенине необходимо выполнить в несколько этапов, когда работает монолит, а данные постепенно мигрируют в микросервисы.

#### Проблемы, которые необходимо решить в первую очередь:
- определиться с микросервисной архитектурой и порядком перехода на неё (какие сервисы выделять из монолита в первую очередь, а кикие потом), на основе каких технолоний строить новую систему (собственная или приобретенная у стороннего вендора it платформа или, например создание кластера Kubernetes и подъем сервисов в контейнерах). Тут необходимо действовать исходя из требований цена/качество;
- настройка мониторинга. Монолит проще отслеживать, чем огромное количество различных микросервисов. Необходимо выделить ключевые показатели каждого сервиса в отдельности и грамотно "вывести" их в удобно читаемые дашборды, также необходимо настроить четкое чтение логов, чтобы минимизировать время определения неисправностей (в каком сервисе что сломалось);
- взаимосвязанность технологий. Каждый микросервис может использовать тот язык программирования и те технологии, что удобнее конкретно для задачи, выполняемой микросервисом и команде, которая его разрабатывает. Это значит, что необходимо конфигурировать все микросервисы между собой и поддерживать «зоопарк» технологий;
- возможное снижение производительности. Из-за сложности архитектуры и "минусов", указанных выше, вероятность снижения производительности (особенно на этапе внедрения) крайне высока;
- рост затрат на персонал. Имеется ввиду не только (и не сколько) увеличение численности и корличества команд, а затраты на повышение компетенций (обучение, внедрение DevOps практик).

#### Плюсы перехода на микросервисную архитектуру:
- огромные возможности для масштабирования. Можно с легкостью добавлять новый функционал (а также убирать его путем отключения данного микросервиса, если он себя не оправдал), также можно увеличиать базы данных каталогов товаров (под каждую категорию товаров например выполнить отдельный микросервис), повышать количество одновременных транзакций, вводить новые способы оплаты товара (бонусы, промокоды, сторонние программы, например "Яндекс плюс" или "Сбер спасибо") и так далее. Также необходимо отметить, что масштабирование различных компоненов может выполняться независимо друг от друга;
- возможность интеграции нашего интернет-магазина с другими сервисами (например маркетплейсами);
- повышение доступности и отказоустойчивости. Когда монолит падает, он полностью перестаёт работать. В приложении с микросервисной архитектурой перестаёт работать только какая-то часть. Например, в интернет-магазине может сломаться корзина, но клиенты без проблем продолжат пользоваться каталогом, добавлять товары в избранное;
- возможность справится с неоднородностью нагрузки. Благодаря возможностям масштабирования, описаным в первом буллите появляется возможность реплицирования наиболее востребованных сервисов в пиковые периоды (сезонные распродажи, "Чёрная пятница" и так далее).
    
