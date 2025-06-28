# 1. Описание таблиц

| Таблица             | Описание                                                                  | Поля                                                                                                                                                                                                 |
|---------------------|---------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Categories**      | Категории товаров                                                         | `id (PK)`, `name`                                                                                                                                                                                   |
| **Manufacturers**   | Производители товаров                                                     | `id (PK)`, `name`, `contact_info`                                                                                                                                                                   |
| **Suppliers**       | Поставщики товаров                                                        | `id (PK)`, `name`, `contact_info`                                                                                                                                                                   |
| **Products**        | Товары                                                                    | `id (PK)`, `name`, `description`, `category_id (FK)`, `manufacturer_id (FK)`, `price`, `cost_price`                                                                                                  |
| **Warehouses**      | Склады для хранения товаров                                               | `id (PK)`, `name`, `location`                                                                                                                                                                       |
| **Inventory**       | Инвентарные записи товаров на складах                                     | `id (PK)`, `product_id (FK)`, `warehouse_id (FK)`, `quantity`                                                                                                                                        |
| **VendingMachines** | Автоматы                                                                  | `id (PK)`, `model`, `location`, `status`                                                                                                                                                            |
| **MachineSlots**    | Отсеки в автоматах                                                        | `id (PK)`, `machine_id (FK)`, `product_id (FK)`, `max_capacity`, `current_quantity`                                                                                                                 |
| **Staff**           | Сотрудники                                                                | `id (PK)`, `name`, `position`, `contact_info`                                                                                                                                                       |
| **Maintenance**     | Записи техобслуживания автоматов                                          | `id (PK)`, `machine_id (FK)`, `staff_id (FK)`, `maintenance_date`, `description`, `status`                                                                                                          |
| **Users**           | Пользователи системы                                                      | `id (PK)`, `username`, `password`, `role`                                                                                                                                                           |
| **Payments**        | Платежи                                                                   | `id (PK)`, `amount`, `payment_method`, `payment_date`, `status`                                                                                                                                     |
| **Sales**           | Продажи                                                                   | `id (PK)`, `machine_id (FK)`, `user_id (FK)`, `payment_id (FK)`, `sale_date`                                                                                                                        |
| **SaleDetails**     | Детали продаж                                                             | `id (PK)`, `sale_id (FK)`, `slot_id (FK)`, `product_id (FK)`, `quantity`, `price`                                                                                                                   |
| **Purchases**       | Закупки товаров                                                           | `id (PK)`, `supplier_id (FK)`, `product_id (FK)`, `quantity`, `purchase_date`, `cost`                                                                                                               |

---

# 2. Внешние ключи и связи

| Таблица             | Внешний ключ                    | Ссылается на               | Описание связи                                                                 |
|---------------------|---------------------------------|----------------------------|-------------------------------------------------------------------------------|
| **Products**        | `category_id`                   | `Categories.id`            | Товар принадлежит категории                                                   |
|                     | `manufacturer_id`               | `Manufacturers.id`         | Товар произведен производителем                                               |
| **Inventory**       | `product_id`                    | `Products.id`              | Инвентарная запись для товара                                                 |
|                     | `warehouse_id`                  | `Warehouses.id`            | Инвентарная запись на складе                                                  |
| **MachineSlots**    | `machine_id`                    | `VendingMachines.id`       | Отсек принадлежит автомату                                                    |
|                     | `product_id`                    | `Products.id`              | В отсеке находится товар                                                      |
| **Maintenance**     | `machine_id`                    | `VendingMachines.id`       | Обслуживание проводится на автомате                                           |
|                     | `staff_id`                      | `Staff.id`                 | Обслуживание выполняет сотрудник                                              |
| **Sales**           | `machine_id`                    | `VendingMachines.id`       | Продажа совершена в автомате                                                  |
|                     | `user_id`                       | `Users.id`                 | Продажу совершил пользователь                                                 |
|                     | `payment_id`                    | `Payments.id`              | Продажа связана с платежом                                                    |
| **SaleDetails**     | `sale_id`                       | `Sales.id`                 | Детали принадлежат продаже                                                    |
|                     | `slot_id`                       | `MachineSlots.id`          | Товар взят из отсека                                                          |
|                     | `product_id`                    | `Products.id`              | Проданный товар                                                               |
| **Purchases**       | `supplier_id`                   | `Suppliers.id`             | Закупка у поставщика                                                          |
|                     | `product_id`                    | `Products.id`              | Закупленный товар                                                             |

---

# 3. Запросы

**Вложенные:**
```sql
-- 1. Товары в категории "Напитки"
SELECT * FROM Products 
WHERE category_id = (SELECT id FROM Categories WHERE name = 'Напитки');

-- 2. Продажи, оплаченные картой
SELECT * FROM Sales 
WHERE payment_id IN (SELECT id FROM Payments WHERE payment_method = 'карта');

-- 3. Автоматы в статусе "Требует обслуживания"
SELECT * FROM VendingMachines 
WHERE id IN (SELECT machine_id FROM Maintenance WHERE status = 'Требует ремонта');

-- 4. Товары с ценой выше средней
SELECT * FROM Products 
WHERE price > (SELECT AVG(price) FROM Products);

-- 5. Сотрудники, обслуживавшие автоматы в 2023 году
SELECT * FROM Staff 
WHERE id IN (SELECT staff_id FROM Maintenance WHERE EXTRACT(YEAR FROM maintenance_date) = 2023);

-- 6. Закупки у поставщика "ООО Поставщик Плюс"
SELECT * FROM Purchases 
WHERE supplier_id = (SELECT id FROM Suppliers WHERE name = 'ООО Поставщик Плюс');

-- 7. Отсеки с количеством товара < 10% от вместимости
SELECT * FROM MachineSlots 
WHERE current_quantity < 0.1 * max_capacity;

-- 8. Продажи, где сумма платежа превышает 1000 руб
SELECT * FROM Sales 
WHERE payment_id IN (SELECT id FROM Payments WHERE amount > 1000);

-- 9. Товары, отсутствующие на складе
SELECT * FROM Products 
WHERE id NOT IN (SELECT product_id FROM Inventory WHERE quantity > 0);

-- 10. Автоматы без активных отсеков
SELECT * FROM VendingMachines 
WHERE id NOT IN (SELECT machine_id FROM MachineSlots WHERE current_quantity > 0);

-- 11. Поставщики с закупками за последний месяц
SELECT * FROM Suppliers 
WHERE id IN (SELECT supplier_id FROM Purchases WHERE purchase_date >= NOW() - INTERVAL '1 month');

-- 12. Категории без товаров
SELECT * FROM Categories 
WHERE id NOT IN (SELECT category_id FROM Products);

-- 13. Пользователи с ролью "Администратор"
SELECT * FROM Users 
WHERE role = 'Администратор';

-- 14. Продажи с ошибками в деталях (отрицательное количество)
SELECT * FROM Sales 
WHERE id IN (SELECT sale_id FROM SaleDetails WHERE quantity <= 0);

-- 15. Техобслуживание с незакрытым статусом
SELECT * FROM Maintenance 
WHERE status NOT IN ('Завершено', 'Отменено');
```

**JOIN:**
```sql
-- 1. Детали продаж с информацией о товаре и автомате
SELECT 
    sd.id,
    p.name AS product_name,
    vm.location AS machine_location,
    s.sale_date
FROM SaleDetails sd
JOIN Products p ON sd.product_id = p.id
JOIN Sales s ON sd.sale_id = s.id
JOIN VendingMachines vm ON s.machine_id = vm.id;

-- 2. Закупки с названием товара и поставщика
SELECT 
    pur.id,
    p.name AS product,
    s.name AS supplier,
    pur.quantity,
    pur.purchase_date
FROM Purchases pur
JOIN Products p ON pur.product_id = p.id
JOIN Suppliers s ON pur.supplier_id = s.id;

-- 3. Инвентаризация со складом и категорией товара
SELECT 
    i.id,
    p.name AS product,
    w.name AS warehouse,
    c.name AS category,
    i.quantity
FROM Inventory i
JOIN Products p ON i.product_id = p.id
JOIN Warehouses w ON i.warehouse_id = w.id
JOIN Categories c ON p.category_id = c.id;

-- 4. Техобслуживание с данными сотрудника и автомата
SELECT 
    m.id,
    vm.model,
    vm.location,
    s.name AS staff_name,
    m.maintenance_date,
    m.status
FROM Maintenance m
JOIN VendingMachines vm ON m.machine_id = vm.id
JOIN Staff s ON m.staff_id = s.id;

-- 5. Продажи с детализацией платежа и пользователя
SELECT 
    s.id AS sale_id,
    u.username,
    p.amount,
    p.payment_method,
    s.sale_date
FROM Sales s
JOIN Users u ON s.user_id = u.id
JOIN Payments p ON s.payment_id = p.id;

-- 6. Отсеки автоматов с товарами и их категориями
SELECT 
    ms.id AS slot_id,
    vm.model,
    p.name AS product,
    c.name AS category,
    ms.current_quantity,
    ms.max_capacity
FROM MachineSlots ms
JOIN VendingMachines vm ON ms.machine_id = vm.id
JOIN Products p ON ms.product_id = p.id
JOIN Categories c ON p.category_id = c.id;

-- 7. Закупки с информацией о поставщике и производителе
SELECT 
    pur.id,
    p.name AS product,
    sup.name AS supplier,
    m.name AS manufacturer,
    pur.quantity,
    pur.cost
FROM Purchases pur
JOIN Products p ON pur.product_id = p.id
JOIN Suppliers sup ON pur.supplier_id = sup.id
JOIN Manufacturers m ON p.manufacturer_id = m.id;

-- 8. Продажи с выручкой по категориям
SELECT 
    c.name AS category,
    SUM(sd.quantity * sd.price) AS total_revenue
FROM SaleDetails sd
JOIN Products p ON sd.product_id = p.id
JOIN Categories c ON p.category_id = c.id
GROUP BY c.name;

-- 9. Автоматы с количеством товаров в отсеках
SELECT 
    vm.id,
    vm.location,
    COUNT(ms.id) AS slots_count,
    SUM(ms.current_quantity) AS total_items
FROM VendingMachines vm
LEFT JOIN MachineSlots ms ON vm.id = ms.machine_id
GROUP BY vm.id;

-- 10. Товары с остатками на складах и в автоматах
SELECT 
    p.id,
    p.name,
    COALESCE(SUM(i.quantity), 0) AS warehouse_qty,
    COALESCE(SUM(ms.current_quantity), 0) AS vending_qty
FROM Products p
LEFT JOIN Inventory i ON p.id = i.product_id
LEFT JOIN MachineSlots ms ON p.id = ms.product_id
GROUP BY p.id;

-- 11. Техобслуживание с контактами ответственных
SELECT 
    m.id,
    vm.model,
    s.name AS staff_name,
    s.contact_info,
    m.maintenance_date
FROM Maintenance m
JOIN VendingMachines vm ON m.machine_id = vm.id
JOIN Staff s ON m.staff_id = s.id;

-- 12. Детализация продаж с прибылью
SELECT 
    sd.id,
    p.name AS product,
    sd.quantity,
    sd.price AS sale_price,
    p.cost_price,
    (sd.price - p.cost_price) * sd.quantity AS profit
FROM SaleDetails sd
JOIN Products p ON sd.product_id = p.id;

-- 13. Поставщики и их последняя закупка
SELECT 
    s.id,
    s.name,
    MAX(pur.purchase_date) AS last_purchase_date
FROM Suppliers s
LEFT JOIN Purchases pur ON s.id = pur.supplier_id
GROUP BY s.id;

-- 14. Автоматы с проблемными отсеками (<5% заполненности)
SELECT 
    vm.id,
    vm.location,
    ms.id AS slot_id,
    p.name AS product,
    ms.current_quantity,
    ms.max_capacity
FROM VendingMachines vm
JOIN MachineSlots ms ON vm.id = ms.machine_id
JOIN Products p ON ms.product_id = p.id
WHERE ms.current_quantity < 0.05 * ms.max_capacity;

-- 15. Ежемесячная выручка по автоматам
SELECT 
    vm.id,
    vm.location,
    EXTRACT(YEAR FROM s.sale_date) AS year,
    EXTRACT(MONTH FROM s.sale_date) AS month,
    SUM(p.amount) AS total_revenue
FROM Sales s
JOIN VendingMachines vm ON s.machine_id = vm.id
JOIN Payments p ON s.payment_id = p.id
GROUP BY vm.id, year, month;
```

# **Агрегатные(10 на каждую функцию):**

# COUNT

```sql
-- 1. Количество товаров в категориях
SELECT c.name, COUNT(p.id) AS product_count
FROM Categories c
LEFT JOIN Products p ON c.id = p.category_id 
GROUP BY c.name;

-- 2. Количество отсеков в каждом автомате
SELECT vm.model, vm.location, COUNT(ms.id) AS slot_count
FROM VendingMachines vm
LEFT JOIN MachineSlots ms ON vm.id = ms.machine_id
GROUP BY vm.id;

-- 3. Количество продаж по пользователям
SELECT u.username, COUNT(s.id) AS sale_count
FROM Users u
LEFT JOIN Sales s ON u.id = s.user_id
GROUP BY u.id;

-- 4. Количество закупок по поставщикам
SELECT s.name, COUNT(pur.id) AS purchase_count
FROM Suppliers s
LEFT JOIN Purchases pur ON s.id = pur.supplier_id
GROUP BY s.id;

-- 5. Количество техобслуживаний по сотрудникам
SELECT st.name, COUNT(m.id) AS maintenance_count
FROM Staff st
LEFT JOIN Maintenance m ON st.id = m.staff_id
GROUP BY st.id;

-- 6. Количество товаров на каждом складе
SELECT w.name, SUM(i.quantity) AS total_items
FROM Warehouses w
LEFT JOIN Inventory i ON w.id = i.warehouse_id
GROUP BY w.id;

-- 7. Количество платежей по методам оплаты
SELECT payment_method, COUNT(id) AS payment_count
FROM Payments
GROUP BY payment_method;

-- 8. Количество активных автоматов по статусам
SELECT status, COUNT(id) AS machine_count
FROM VendingMachines
GROUP BY status;

-- 9. Количество товаров в отсеках по автоматам
SELECT vm.location, SUM(ms.current_quantity) AS total_items
FROM VendingMachines vm
JOIN MachineSlots ms ON vm.id = ms.machine_id
GROUP BY vm.id;

-- 10. Количество уникальных товаров в продажах
SELECT s.id AS sale_id, COUNT(DISTINCT sd.product_id) AS unique_products
FROM Sales s
JOIN SaleDetails sd ON s.id = sd.sale_id
GROUP BY s.id;
```

# SUM

```sql
-- 1. Общая выручка по автоматам
SELECT vm.id, vm.location, SUM(p.amount) AS total_revenue
FROM Sales s
JOIN Payments p ON s.payment_id = p.id
JOIN VendingMachines vm ON s.machine_id = vm.id 
GROUP BY vm.id;

-- 2. Суммарная закупочная стоимость по категориям
SELECT c.name, SUM(pur.cost * pur.quantity) AS total_cost
FROM Purchases pur
JOIN Products p ON pur.product_id = p.id
JOIN Categories c ON p.category_id = c.id
GROUP BY c.id;

-- 3. Общее количество проданных товаров
SELECT p.name, SUM(sd.quantity) AS total_sold
FROM SaleDetails sd
JOIN Products p ON sd.product_id = p.id
GROUP BY p.id;

-- 4. Суммарная вместимость отсеков по автоматам
SELECT vm.id, SUM(ms.max_capacity) AS total_capacity
FROM VendingMachines vm
JOIN MachineSlots ms ON vm.id = ms.machine_id
GROUP BY vm.id;

-- 5. Общая стоимость инвентаря на складах
SELECT w.name, SUM(i.quantity * p.cost_price) AS total_value
FROM Inventory i
JOIN Products p ON i.product_id = p.id
JOIN Warehouses w ON i.warehouse_id = w.id
GROUP BY w.id;

-- 6. Сумма платежей по дням
SELECT payment_date::DATE, SUM(amount) AS daily_income
FROM Payments
GROUP BY payment_date::DATE;

-- 7. Общая прибыль по товарам (продажи - себестоимость)
SELECT 
    p.name,
    SUM(sd.quantity * sd.price) AS revenue,
    SUM(sd.quantity * p.cost_price) AS cost,
    SUM(sd.quantity * (sd.price - p.cost_price)) AS profit
FROM SaleDetails sd
JOIN Products p ON sd.product_id = p.id
GROUP BY p.id;

-- 8. Суммарное количество закупленных товаров
SELECT p.name, SUM(pur.quantity) AS total_purchased
FROM Purchases pur
JOIN Products p ON pur.product_id = p.id
GROUP BY p.id;

-- 9. Общая выручка по месяцам
SELECT 
    EXTRACT(YEAR FROM sale_date) AS year,
    EXTRACT(MONTH FROM sale_date) AS month,
    SUM(p.amount) AS monthly_revenue
FROM Sales s
JOIN Payments p ON s.payment_id = p.id
GROUP BY year, month;

-- 10. Суммарное количество товаров в автоматах по категориям
SELECT c.name, SUM(ms.current_quantity) AS total_in_machines
FROM MachineSlots ms
JOIN Products p ON ms.product_id = p.id
JOIN Categories c ON p.category_id = c.id
GROUP BY c.id;
```

# AVG

```sql
-- 1. Средняя цена товара по категориям
SELECT c.name, AVG(p.price) AS avg_price
FROM Products p
JOIN Categories c ON p.category_id = c.id
GROUP BY c.id;

-- 2. Среднее количество товара в отсеках
SELECT vm.model, AVG(ms.current_quantity) AS avg_items
FROM VendingMachines vm
JOIN MachineSlots ms ON vm.id = ms.machine_id
GROUP BY vm.id;

-- 3. Средний чек продажи
SELECT AVG(p.amount) AS avg_sale_amount
FROM Payments p;

-- 4. Средняя себестоимость товаров по производителям
SELECT m.name, AVG(p.cost_price) AS avg_cost
FROM Products p
JOIN Manufacturers m ON p.manufacturer_id = m.id
GROUP BY m.id;

-- 5. Среднее время между обслуживаниями автомата
SELECT 
    vm.id,
    AVG(EXTRACT(EPOCH FROM (m2.maintenance_date - m1.maintenance_date))) / 86400 AS avg_days
FROM Maintenance m1
JOIN Maintenance m2 ON m1.machine_id = m2.machine_id AND m2.maintenance_date > m1.maintenance_date
JOIN VendingMachines vm ON m1.machine_id = vm.id
GROUP BY vm.id;

-- 6. Среднее количество проданных товаров за транзакцию
SELECT AVG(sd.quantity) AS avg_items_per_sale
FROM SaleDetails sd;

-- 7. Средняя заполненность отсеков в %
SELECT 
    vm.id,
    AVG((ms.current_quantity::FLOAT / ms.max_capacity) * 100) AS avg_fill_percent
FROM MachineSlots ms
JOIN VendingMachines vm ON ms.machine_id = vm.id
GROUP BY vm.id;

-- 8. Средняя стоимость закупки по поставщикам
SELECT s.name, AVG(pur.cost * pur.quantity) AS avg_purchase_cost
FROM Purchases pur
JOIN Suppliers s ON pur.supplier_id = s.id
GROUP BY s.id;

-- 9. Средняя выручка автомата в день
SELECT 
    vm.id,
    AVG(daily_revenue) AS avg_daily_revenue
FROM (
    SELECT s.machine_id, s.sale_date::DATE, SUM(p.amount) AS daily_revenue
    FROM Sales s
    JOIN Payments p ON s.payment_id = p.id
    GROUP BY s.machine_id, s.sale_date::DATE
) AS daily
JOIN VendingMachines vm ON daily.machine_id = vm.id
GROUP BY vm.id;

-- 10. Среднее количество техобслуживаний в месяц
SELECT 
    EXTRACT(YEAR FROM maintenance_date) AS year,
    EXTRACT(MONTH FROM maintenance_date) AS month,
    COUNT(id) AS maintenance_count
FROM Maintenance
GROUP BY year, month;
```

# MIN/MAX

```sql
-- 1. Самый дорогой товар в каждой категории
SELECT c.name, MAX(p.price) AS max_price
FROM Products p
JOIN Categories c ON p.category_id = c.id
GROUP BY c.id;

-- 2. Автомат с минимальным количеством продаж
SELECT vm.id, vm.location, COUNT(s.id) AS sale_count
FROM VendingMachines vm
LEFT JOIN Sales s ON vm.id = s.machine_id
GROUP BY vm.id
ORDER BY sale_count ASC
LIMIT 1;

-- 3. Самая ранняя и поздняя дата техобслуживания
SELECT MIN(maintenance_date) AS first_maintenance, MAX(maintenance_date) AS last_maintenance
FROM Maintenance;

-- 4. Товар с минимальным остатком на складе
SELECT p.name, SUM(i.quantity) AS total_stock
FROM Products p
JOIN Inventory i ON p.id = i.product_id
GROUP BY p.id
ORDER BY total_stock ASC
LIMIT 1;

-- 5. Максимальное количество товара в отсеке
SELECT MAX(current_quantity) AS max_items_in_slot
FROM MachineSlots;

-- 6. Минимальная и максимальная стоимость закупки
SELECT MIN(cost * quantity) AS min_purchase, MAX(cost * quantity) AS max_purchase
FROM Purchases;

-- 7. Самый популярный товар (по количеству продаж)
SELECT p.name, SUM(sd.quantity) AS total_sold
FROM SaleDetails sd
JOIN Products p ON sd.product_id = p.id
GROUP BY p.id
ORDER BY total_sold DESC
LIMIT 1;

-- 8. Автомат с максимальной выручкой
SELECT vm.id, vm.location, SUM(p.amount) AS total_revenue
FROM Sales s
JOIN Payments p ON s.payment_id = p.id
JOIN VendingMachines vm ON s.machine_id = vm.id
GROUP BY vm.id
ORDER BY total_revenue DESC
LIMIT 1;

-- 9. Самая длительная задержка между обслуживаниями
SELECT 
    machine_id,
    MAX(days_between) AS max_days_between_maintenance
FROM (
    SELECT 
        m1.machine_id,
        EXTRACT(EPOCH FROM (m2.maintenance_date - m1.maintenance_date)) / 86400 AS days_between
    FROM Maintenance m1
    JOIN Maintenance m2 ON m1.machine_id = m2.machine_id AND m2.maintenance_date > m1.maintenance_date
) AS intervals
GROUP BY machine_id;

-- 10. Товар с максимальной прибылью
SELECT 
    p.name,
    SUM(sd.quantity * (sd.price - p.cost_price)) AS total_profit
FROM SaleDetails sd
JOIN Products p ON sd.product_id = p.id
GROUP BY p.id
ORDER BY total_profit DESC
LIMIT 1;
```
---

#### 4. Хранимые процедуры

```sql
-- 1. Добавление товара в отсек
CREATE OR REPLACE PROCEDURE add_product_to_slot(
    slot_id INT, 
    product_id INT, 
    quantity INT
)
LANGUAGE plpgsql AS $$
BEGIN
    UPDATE MachineSlots 
    SET 
        current_quantity = current_quantity + quantity, 
        product_id = product_id
    WHERE id = slot_id;
    
    -- Уменьшаем инвентарь на складе
    UPDATE Inventory
    SET quantity = quantity - quantity
    WHERE product_id = product_id
    AND warehouse_id = (SELECT warehouse_id FROM MachineRestock WHERE slot_id = slot_id)
    LIMIT 1;
END;
$$;

-- 2. Регистрация продажи
CREATE OR REPLACE PROCEDURE register_sale(
    machine_id INT, 
    user_id INT, 
    payment_method TEXT, 
    amount DECIMAL, 
    product_id INT, 
    quantity INT, 
    slot_id INT
) 
LANGUAGE plpgsql AS $$
DECLARE 
    payment_id INT;
    sale_id INT;
BEGIN
    -- Создаем платеж
    INSERT INTO Payments (amount, payment_method, payment_date, status) 
    VALUES (amount, payment_method, NOW(), 'Завершен')
    RETURNING id INTO payment_id;
    
    -- Создаем продажу
    INSERT INTO Sales (machine_id, user_id, payment_id, sale_date)
    VALUES (machine_id, user_id, payment_id, NOW())
    RETURNING id INTO sale_id;
    
    -- Добавляем детали продажи
    INSERT INTO SaleDetails (sale_id, slot_id, product_id, quantity, price)
    VALUES (
        sale_id, 
        slot_id, 
        product_id, 
        quantity,
        (SELECT price FROM Products WHERE id = product_id)
    );
    
    -- Обновляем количество в отсеке
    UPDATE MachineSlots 
    SET current_quantity = current_quantity - quantity 
    WHERE id = slot_id;
END;
$$;

-- 3. Перемещение товара между складами
CREATE OR REPLACE PROCEDURE move_inventory(
    product_id INT,
    from_warehouse_id INT,
    to_warehouse_id INT,
    quantity INT
)
LANGUAGE plpgsql AS $$
BEGIN
    -- Уменьшаем количество на исходном складе
    UPDATE Inventory
    SET quantity = quantity - quantity
    WHERE warehouse_id = from_warehouse_id 
    AND product_id = product_id;
    
    -- Увеличиваем количество на целевом складе
    INSERT INTO Inventory (product_id, warehouse_id, quantity)
    VALUES (product_id, to_warehouse_id, quantity)
    ON CONFLICT (product_id, warehouse_id)
    DO UPDATE SET quantity = Inventory.quantity + quantity;
END;
$$;

-- 4. Плановое техобслуживание
CREATE OR REPLACE PROCEDURE schedule_maintenance(
    machine_id INT,
    staff_id INT,
    maintenance_date DATE
)
LANGUAGE plpgsql AS $$
BEGIN
    INSERT INTO Maintenance (machine_id, staff_id, maintenance_date, status)
    VALUES (machine_id, staff_id, maintenance_date, 'Запланировано');
    
    -- Обновляем статус автомата
    UPDATE VendingMachines
    SET status = 'В обслуживании'
    WHERE id = machine_id;
END;
$$;

-- 5. Регистрация закупки
CREATE OR REPLACE PROCEDURE register_purchase(
    supplier_id INT,
    product_id INT,
    quantity INT,
    cost DECIMAL
)
LANGUAGE plpgsql AS $$
BEGIN
    -- Добавляем закупку
    INSERT INTO Purchases (supplier_id, product_id, quantity, purchase_date, cost)
    VALUES (supplier_id, product_id, quantity, NOW(), cost);
    
    -- Обновляем инвентарь на основном складе
    UPDATE Inventory
    SET quantity = quantity + quantity
    WHERE product_id = product_id
    AND warehouse_id = (SELECT id FROM Warehouses WHERE name = 'Основной склад');
END;
$$;
```

---

#### 5. Триггеры для таблиц `Products` и `Inventory`

**Триггеры `Products`:**
```sql
-- BEFORE INSERT: Проверка отрицательной цены
CREATE OR REPLACE FUNCTION check_product_price()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.price <= 0 OR NEW.cost_price <= 0 THEN
        RAISE EXCEPTION 'Цена и себестоимость должны быть положительными';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER product_price_check
BEFORE INSERT OR UPDATE ON Products
FOR EACH ROW EXECUTE FUNCTION check_product_price();

-- BEFORE DELETE: Запрет удаления при наличии в инвентаре
CREATE OR REPLACE FUNCTION prevent_product_delete()
RETURNS TRIGGER AS $$
BEGIN
    IF EXISTS (SELECT 1 FROM Inventory WHERE product_id = OLD.id AND quantity > 0) THEN
        RAISE EXCEPTION 'Невозможно удалить товар с остатками на складе';
    END IF;
    RETURN OLD;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER product_delete_protection
BEFORE DELETE ON Products
FOR EACH ROW EXECUTE FUNCTION prevent_product_delete();

-- AFTER UPDATE: Логирование изменений цен
CREATE OR REPLACE FUNCTION log_price_changes()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.price <> OLD.price THEN
        INSERT INTO PriceChangeLog (product_id, old_price, new_price, change_date)
        VALUES (OLD.id, OLD.price, NEW.price, NOW());
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER product_price_logger
AFTER UPDATE ON Products
FOR EACH ROW EXECUTE FUNCTION log_price_changes();
```

**Триггеры `Inventory`:**
```sql
-- BEFORE INSERT: Проверка отрицательного количества
CREATE OR REPLACE FUNCTION check_inventory_quantity()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.quantity < 0 THEN
        RAISE EXCEPTION 'Количество товара не может быть отрицательным';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER inventory_quantity_check
BEFORE INSERT OR UPDATE ON Inventory
FOR EACH ROW EXECUTE FUNCTION check_inventory_quantity();

-- AFTER UPDATE: Уведомление о низком запасе
CREATE OR REPLACE FUNCTION notify_low_stock()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.quantity < 10 THEN
        INSERT INTO Notifications (product_id, warehouse_id, message, created_at)
        VALUES (
            NEW.product_id, 
            NEW.warehouse_id, 
            'Критически низкий запас товара', 
            NOW()
        );
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER low_stock_alert
AFTER UPDATE ON Inventory
FOR EACH ROW EXECUTE FUNCTION notify_low_stock();

-- AFTER INSERT/UPDATE: Синхронизация с MachineSlots
CREATE OR REPLACE FUNCTION sync_restock_requests()
RETURNS TRIGGER AS $$
BEGIN
    -- Если запас пополнен, сбрасываем флаг пополнения
    UPDATE MachineSlots
    SET restock_requested = false
    WHERE product_id = NEW.product_id
    AND restock_requested = true;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER restock_sync
AFTER INSERT OR UPDATE ON Inventory
FOR EACH ROW EXECUTE FUNCTION sync_restock_requests();
```
