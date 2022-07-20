# Решение технического задания из пункта 3

# Ответы на вопросы

## Вычислите общую выручку за июль 2021 по тем сделкам, приход денежных средств которых не просрочен.

#### Подготовка данных к анализу

    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    %matplotlib inline
    import seaborn as sns
    from datetime import datetime

    data_file = pd.read_excel('./data.xlsx', engine='openpyxl', index_col='client_id')
    data_file.head()

![Вывод начала таблицы](/images/pic1.png)

    data_file = data_file.replace('-', )

#### Вычисление общей выручки

    start_july = datetime(2021, 7, 1)
    end_july = datetime(2021, 7, 31)
    pd.to_datetime(data_file['receiving_date'], format='%b %d, %Y')
    data_file_sum_in_july = data_file[data_file['receiving_date'] >= start_july]
    data_file_sum_in_july = data_file_sum_in_july[data_file_sum_in_july['receiving_date'] <= end_july]
    data_file_sum_in_july = data_file_sum_in_july[data_file_sum_in_july['status'] != 'ПРОСРОЧЕНО']
    july_sum = data_file_sum_in_july['sum'].sum()
    print("Общая выручка за июль 2021 года по непросроченным сделкам:", july_sum.round(2))

#### Результат

![Результат вычислений](/images/pic2.png)

## Как изменялась выручка компании за рассматриваемый период? Проиллюстрируйте графиком

#### Подготовка данных

    group_sum_data = data_file.groupby(['receiving_date'])['sum'].agg('sum')

#### Рисование графика

    plt.figure(figsize=(18,3), dpi=300)
    plt.xlabel('Период')
    plt.ylabel('Выручка')
    plt.title('Выручка компании за период Май 2021 - Ноябрь 2021')
    plt.plot(group_sum_data)
    plt.show();

#### Результат
![График](/images/pic3.png)

## Кто из менеджеров привлек для компании больше всего денежных средств в сентябре 2021?

#### Вычисление

    best_sellers = data_file.groupby(['sale'])['sum'].sum()
    best_sellers.sort_values(ascending=False)[:1]

#### Результат
![Вывод лучшего менеджера](/images/pic4.png)

#### Вывод

Вывод: менеджер, привлекший больше всего денежных средств за указанный период: Петрова

## Какой тип сделок (новая/текущая) был преобладающим в октябре 2021?

#### Вычисление

    start_october = datetime(2021, 10, 1)
    end_october = datetime(2021, 10, 31)
    data_file_sum_in_october = data_file[data_file['receiving_date'] >= start_october]
    data_file_sum_in_october = data_file_sum_in_october[data_file_sum_in_october['receiving_date'] <= end_october]
    data_file_sum_in_october.groupby('new/current')['sum'].count()

#### Результат

![Результат по сделкам](/images/pic5.png)
#### Вывод

Вывод: в октябре преобладающими были текущие сделки (105), а не новые (17)

## Сколько оригиналов договора по майским сделкам было получено в июне 2021?

#### Вычисление

    start_may = datetime(2021, 5, 1)
    end_may = datetime(2021, 5, 31)
    data_file_sum_in_may = data_file[data_file['receiving_date'] >= start_may]
    data_file_sum_in_may = data_file_sum_in_may[data_file_sum_in_may['receiving_date'] <= end_may]
    data_file_sum_in_may = data_file_sum_in_may[data_file_sum_in_may['document'] == 'оригинал']
    print("Количество оригиналов договоров по сделкам в мае:", len(data_file_sum_in_may))

#### Результат

![Результат по оригиналам договоров](/images/pic6.png)

# Задание

#### Подготовка датафрейма со всеми менеджерами

    sellers = data_file['sale'].unique()
    sellers_data = pd.DataFrame(sellers, columns = ['sale'])
    sellers_data = sellers_data.dropna()
    sellers_data.insert(1, 'bonus', 0)
    sellers_data

![Датафрейм с менеджерами](/images/pic7.png)

#### Вычисление

    sellers_new = data_file[data_file['new/current'] == 'новая']
    sellers_new.insert(7, 'bonus', 0)
    sellers_cur = data_file[data_file['new/current'] == 'текущая']
    sellers_cur.insert(7, 'bonus', 0)
    first_of_july = datetime(2021, 7, 1)
    sellers_new = sellers_new[sellers_new['receiving_date'] < first_of_july]
    sellers_cur = sellers_cur[sellers_cur['receiving_date'] < first_of_july]
    for index, row in sellers_new.iterrows():
        if(row['status'] == 'ОПЛАЧЕНО' and row['document'] == 'оригинал'):
            sellers_new.at[index, 'bonus'] = float(row['sum'] * 0.07)
    for index, row in sellers_cur.iterrows():
        if(row['status'] != 'ПРОСРОЧЕНО' and row['document'] == 'оригинал'):
            if(row['sum'] <= 10000.00):
                sellers_cur.at[index, 'bonus'] = float(row['sum'] * 0.03)
            else:
                sellers_cur.at[index, 'bonus'] = float(row['sum'] * 0.05)
    sellers_new_bonuses = sellers_new.groupby('sale')['bonus'].sum()
    sellers_cur_bonuses = sellers_cur.groupby('sale')['bonus'].sum().astype(int)
    res = sellers_new_bonuses.append(sellers_cur_bonuses)
    res = res.append(sellers_new_bonuses)
    result = res.groupby('sale')['bonus'].sum().to_frame()

#### Результат
![Результат](/images/pic8.png)



