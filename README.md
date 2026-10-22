# django-orm-sql-cheatsheet
🐍 Django ORM ↔ SQL cheat sheet — a bilingual (English/فارسی) quick reference mapping common Django ORM queries to their SQL equivalents.
# 🐍 Django ORM ↔ SQL Cheat Sheet

> A quick reference for translating common Django ORM queries to their SQL equivalents.
>
> مرجع سریع برای تبدیل دستورات پرکاربرد Django ORM به معادل SQL آن‌ها.

[![Django ORM ↔ SQL](https://img.shields.io/badge/Django%20ORM-%E2%86%94%20SQL-092E20?logo=django&logoColor=white)](https://github.com/mehdiseyfie/django-orm-sql-cheatsheet)


**🌐 Language / زبان:** [🇬🇧 English](#-english) · [🇮🇷 فارسی](#-persian)

---

<a id="-english"></a>
## 🇬🇧 English

### Table of Contents

- [Basic queries and filtering](#basic-queries-and-filtering-select--where)
- [Complex queries: Q, F, and Subquery](#complex-queries-q-f-and-subquery)
- [Selecting columns and optimization](#selecting-columns-and-optimization-select--limit)
- [Aggregation and grouping](#aggregation-and-grouping-group-by--having)
- [Optimizing relations](#optimizing-relations-join--prefetch)
- [Other common commands](#other-common-commands)
- [Key takeaways](#key-takeaways)

### Basic queries and filtering (`SELECT` / `WHERE`)

| Django ORM | SQL equivalent | Notes |
| :--- | :--- | :--- |
| `Model.objects.all()` | `SELECT * FROM app_model;` | Fetch every row |
| `Model.objects.get(pk=1)` | `SELECT * FROM app_model WHERE id = 1;` | Fetch a single row by primary key |
| `Model.objects.filter(price__lt=50)` | `SELECT * FROM app_model WHERE price < 50;` | Comparison lookups (`lt`, `lte`, `gt`, `gte`, `exact`) |
| `Model.objects.filter(name__icontains='test')` | `SELECT * FROM app_model WHERE name ILIKE '%test%';` | Case-insensitive search (`contains`, `startswith`, `endswith` are similar) |
| `Model.objects.filter(id__in=[1, 2, 3])` | `SELECT * FROM app_model WHERE id IN (1, 2, 3);` | Filter against a list of values |
| `Model.objects.filter(field__isnull=True)` | `SELECT * FROM app_model WHERE field IS NULL;` | Check for NULL |
| `Model.objects.exclude(status='draft')` | `SELECT * FROM app_model WHERE NOT (status = 'draft');` | Exclude matching rows |
| `Model.objects.filter(a=1, b=2)` | `SELECT * FROM app_model WHERE a = 1 AND b = 2;` | Multiple filters are `AND`-ed by default |

### Complex queries: `Q`, `F`, and `Subquery`

| Django ORM | SQL equivalent | Notes |
| :--- | :--- | :--- |
| `Model.objects.filter(Q(a=1) \| Q(b=2))` | `SELECT * FROM app_model WHERE a = 1 OR b = 2;` | Use `Q` to build `OR` conditions |
| `Model.objects.filter(~Q(id=3))` | `SELECT * FROM app_model WHERE id != 3;` | Use `~` to negate (`NOT`) |
| `Model.objects.filter(price__gt=F('cost'))` | `SELECT * FROM app_model WHERE price > cost;` | Use `F` to compare two columns within a row |
| `Product.objects.annotate(sales=Subquery(...))` | `SELECT ..., (SELECT SUM(...) FROM ... WHERE ...) AS sales` | `Subquery` emits a correlated subquery in `SELECT` |
| `...filter(product=OuterRef('pk'))` | `WHERE ...product_id = outer_table.id` | `OuterRef` lets the subquery reference the outer query |
| `Coalesce(Subquery(...), Value(0))` | `COALESCE((SELECT ...), 0)` | Replace `NULL` with a default value |
| `Case(When(condition, then=Value(...)), default=...)` | `CASE WHEN condition THEN ... ELSE ... END` | Conditional logic in SQL |

### Selecting columns and optimization (`SELECT` / `LIMIT`)

| Django ORM | SQL equivalent | Notes |
| :--- | :--- | :--- |
| `Model.objects.values('id', 'name')` | `SELECT id, name FROM app_model;` | Narrow `SELECT` columns; returns dicts |
| `Model.objects.values_list('id', flat=True)` | `SELECT id FROM app_model;` | Like `values`, but returns tuples (or a flat list) |
| `Model.objects.only('id', 'name')` | `SELECT id, name FROM app_model;` | Load only the listed fields (rest are deferred) |
| `Model.objects.defer('description')` | `SELECT id, name, ... FROM app_model;` | Remove the listed fields from `SELECT` |
| `Model.objects.distinct()` | `SELECT DISTINCT ... FROM app_model;` | Deduplicate rows |
| `Model.objects.all()[:10]` | `SELECT ... FROM app_model LIMIT 10;` | Slicing limits the result set |
| `Model.objects.all()[5:10]` | `SELECT ... FROM app_model LIMIT 5 OFFSET 5;` | Slicing with `OFFSET` (pagination) |

### Aggregation and grouping (`GROUP BY` / `HAVING`)

| Django ORM | SQL equivalent | Notes |
| :--- | :--- | :--- |
| `Model.objects.aggregate(total=Sum('price'))` | `SELECT SUM(price) AS total FROM app_model;` | Aggregate over the whole queryset (returns a dict) |
| `Model.objects.values('category').annotate(count=Count('id'))` | `SELECT category_id, COUNT(id) AS count FROM app_model GROUP BY category_id;` | `values` + `annotate` = `GROUP BY` |
| `Model.objects.annotate(count=Count('reviews'))` | `SELECT ..., COUNT(review.id) AS count FROM model LEFT JOIN review ... GROUP BY model.id;` | Adds a computed column per row (requires `GROUP BY`) |
| `Model.objects.annotate(total=Sum('price')).filter(total__gt=100)` | `... GROUP BY ... HAVING SUM(price) > 100;` | Filtering on an aggregate = `HAVING` |

### Optimizing relations (`JOIN` / `Prefetch`)

| Django ORM | SQL equivalent | Notes |
| :--- | :--- | :--- |
| `Book.objects.select_related('author')` | `SELECT ... FROM book INNER JOIN author ON book.author_id = author.id;` | For `ForeignKey` / `OneToOne`; one query with `JOIN` |
| `Author.objects.prefetch_related('books')` | **Two queries:**<br>1. `SELECT ... FROM author;`<br>2. `SELECT ... FROM book WHERE author_id IN (...);` | For `ManyToMany` / reverse relations; joined in Python |
| `Prefetch('books', queryset=Book.objects.filter(...))` | Second query with custom `WHERE` | Filter the inner `prefetch_related` queryset |

### Other common commands

| Django ORM | SQL equivalent | Notes |
| :--- | :--- | :--- |
| `Model.objects.order_by('-created_at')` | `SELECT ... FROM app_model ORDER BY created_at DESC;` | Descending (`-` means DESC; no prefix = ASC) |
| `Model.objects.count()` | `SELECT COUNT(*) FROM app_model;` | Count rows |
| `Model.objects.exists()` | `SELECT 1 FROM app_model LIMIT 1;` | Check existence (cheaper than `count()`) |
| `Model.objects.first()` | `SELECT ... FROM app_model ORDER BY pk ASC LIMIT 1;` | First row (requires `ORDER BY`) |
| `Model.objects.last()` | `SELECT ... FROM app_model ORDER BY pk DESC LIMIT 1;` | Last row |
| `Model.objects.none()` | `SELECT ... FROM app_model WHERE FALSE;` | An empty queryset |

### Key takeaways

1. **Lazy evaluation**: querysets don't hit the database until they're evaluated (`list()`, iteration, `len()`). Chaining methods only builds the query.
2. **`select_related` vs `prefetch_related`**: the first is for single-valued relations (`ForeignKey`/`OneToOne`) via `JOIN`; the second is for multi-valued relations (`ManyToMany`/reverse) via a separate query.
3. **`values` + `annotate` = `GROUP BY`**: that exact combination is equivalent to SQL `GROUP BY`. Without `values`, `annotate` aggregates over the whole table (not per group).
4. **`Subquery` + `OuterRef`**: build correlated subqueries and avoid the Cartesian blow-up caused by `JOIN`ing multiple relations at once.
5. **`Coalesce`**: turn `NULL` into a default value for aggregate results.

---

<a id="-persian"></a>
## 🇮🇷 فارسی

### فهرست مطالب

- [کوئری‌های پایه و فیلتر کردن](#کوئری‌های-پایه-و-فیلتر-کردن-select--where)
- [کوئری‌های پیچیده‌تر: Q، F و Subquery](#کوئری‌های-پیچیدهتر-q-f-و-subquery)
- [انتخاب ستون‌ها و بهینه‌سازی](#انتخاب-ستون‌ها-و-بهینه‌سازی-select--limit)
- [تجمیع و گروه‌بندی](#تجمیع-و-گروه‌بندی-group-by--having)
- [بهینه‌سازی روابط](#بهینه‌سازی-روابط-join--prefetch)
- [سایر دستورات پرکاربرد](#سایر-دستورات-پرکاربرد)
- [نکات کلیدی](#نکات-کلیدی-برای-به-خاطر-سپردن)

### کوئری‌های پایه و فیلتر کردن (`SELECT` / `WHERE`)

| کد جنگو (Django ORM) | معادل SQL | توضیح |
| :--- | :--- | :--- |
| `Model.objects.all()` | `SELECT * FROM app_model;` | دریافت تمام ردیف‌ها |
| `Model.objects.get(pk=1)` | `SELECT * FROM app_model WHERE id = 1;` | دریافت یک ردیف بر اساس کلید اصلی |
| `Model.objects.filter(price__lt=50)` | `SELECT * FROM app_model WHERE price < 50;` | فیلتر با عملگرهای مقایسه‌ای (`lt`, `lte`, `gt`, `gte`, `exact`) |
| `Model.objects.filter(name__icontains='test')` | `SELECT * FROM app_model WHERE name ILIKE '%test%';` | جستجوی حساسیت‌ناپذیر به حروف (`contains`, `startswith`, `endswith` هم مشابه‌اند) |
| `Model.objects.filter(id__in=[1, 2, 3])` | `SELECT * FROM app_model WHERE id IN (1, 2, 3);` | فیلتر بر اساس لیست مقادیر |
| `Model.objects.filter(field__isnull=True)` | `SELECT * FROM app_model WHERE field IS NULL;` | بررسی NULL بودن فیلد |
| `Model.objects.exclude(status='draft')` | `SELECT * FROM app_model WHERE NOT (status = 'draft');` | حذف ردیف‌های منطبق با شرط |
| `Model.objects.filter(a=1, b=2)` | `SELECT * FROM app_model WHERE a = 1 AND b = 2;` | فیلترهای چندگانه به‌صورت پیش‌فرض با `AND` ترکیب می‌شوند |

### کوئری‌های پیچیده‌تر: `Q`، `F` و `Subquery`

| کد جنگو (Django ORM) | معادل SQL | توضیح |
| :--- | :--- | :--- |
| `Model.objects.filter(Q(a=1) \| Q(b=2))` | `SELECT * FROM app_model WHERE a = 1 OR b = 2;` | استفاده از `Q` برای ترکیب با `OR` |
| `Model.objects.filter(~Q(id=3))` | `SELECT * FROM app_model WHERE id != 3;` | استفاده از `~` برای نفی (`NOT`) |
| `Model.objects.filter(price__gt=F('cost'))` | `SELECT * FROM app_model WHERE price > cost;` | استفاده از `F` برای مقایسه دو فیلد در یک ردیف |
| `Product.objects.annotate(sales=Subquery(...))` | `SELECT ..., (SELECT SUM(...) FROM ... WHERE ...) AS sales` | `Subquery` یک کوئری همبسته در `SELECT` تولید می‌کند |
| `...filter(product=OuterRef('pk'))` | `WHERE ...product_id = outer_table.id` | `OuterRef` ارجاع به کوئری بیرونی را ممکن می‌سازد |
| `Coalesce(Subquery(...), Value(0))` | `COALESCE((SELECT ...), 0)` | جایگزینی `NULL` با مقدار پیش‌فرض |
| `Case(When(condition, then=Value(...)), default=...)` | `CASE WHEN condition THEN ... ELSE ... END` | پیاده‌سازی منطق شرطی در SQL |

### انتخاب ستون‌ها و بهینه‌سازی (`SELECT` / `LIMIT`)

| کد جنگو (Django ORM) | معادل SQL | توضیح |
| :--- | :--- | :--- |
| `Model.objects.values('id', 'name')` | `SELECT id, name FROM app_model;` | محدود کردن ستون‌های `SELECT`؛ نتیجه دیکشنری است |
| `Model.objects.values_list('id', flat=True)` | `SELECT id FROM app_model;` | مشابه `values` اما نتیجه تاپل (یا لیست مسطح) است |
| `Model.objects.only('id', 'name')` | `SELECT id, name FROM app_model;` | فقط فیلدهای مشخص بارگذاری می‌شوند (بقیه به تعویق) |
| `Model.objects.defer('description')` | `SELECT id, name, ... FROM app_model;` | فیلدهای مشخص از `SELECT` حذف می‌شوند |
| `Model.objects.distinct()` | `SELECT DISTINCT ... FROM app_model;` | حذف ردیف‌های تکراری |
| `Model.objects.all()[:10]` | `SELECT ... FROM app_model LIMIT 10;` | برش برای محدود کردن نتایج |
| `Model.objects.all()[5:10]` | `SELECT ... FROM app_model LIMIT 5 OFFSET 5;` | برش با `OFFSET` برای صفحه‌بندی |

### تجمیع و گروه‌بندی (`GROUP BY` / `HAVING`)

| کد جنگو (Django ORM) | معادل SQL | توضیح |
| :--- | :--- | :--- |
| `Model.objects.aggregate(total=Sum('price'))` | `SELECT SUM(price) AS total FROM app_model;` | تجمیع روی کل کوئری (بدون گروه‌بندی)؛ خروجی دیکشنری |
| `Model.objects.values('category').annotate(count=Count('id'))` | `SELECT category_id, COUNT(id) AS count FROM app_model GROUP BY category_id;` | `values` + `annotate` معادل `GROUP BY` است |
| `Model.objects.annotate(count=Count('reviews'))` | `SELECT ..., COUNT(review.id) AS count FROM model LEFT JOIN review ... GROUP BY model.id;` | `annotate` یک ستون تجمیعی به هر ردیف اضافه می‌کند (نیازمند `GROUP BY`) |
| `Model.objects.annotate(total=Sum('price')).filter(total__gt=100)` | `... GROUP BY ... HAVING SUM(price) > 100;` | فیلتر روی نتیجه تجمیع معادل `HAVING` است |

### بهینه‌سازی روابط (`JOIN` / `Prefetch`)

| کد جنگو (Django ORM) | معادل SQL | توضیح |
| :--- | :--- | :--- |
| `Book.objects.select_related('author')` | `SELECT ... FROM book INNER JOIN author ON book.author_id = author.id;` | برای `ForeignKey` / `OneToOne`؛ یک کوئری با `JOIN` |
| `Author.objects.prefetch_related('books')` | **دو کوئری:**<br>1. `SELECT ... FROM author;`<br>2. `SELECT ... FROM book WHERE author_id IN (...);` | برای `ManyToMany` / روابط معکوس؛ در پایتون ترکیب می‌شوند |
| `Prefetch('books', queryset=Book.objects.filter(...))` | کوئری دوم با `WHERE` سفارشی | امکان فیلتر کردن کوئری داخلی `prefetch_related` |

### سایر دستورات پرکاربرد

| کد جنگو (Django ORM) | معادل SQL | توضیح |
| :--- | :--- | :--- |
| `Model.objects.order_by('-created_at')` | `SELECT ... FROM app_model ORDER BY created_at DESC;` | مرتب‌سازی نزولی (`-` یعنی DESC؛ بدون پیشوند = ASC) |
| `Model.objects.count()` | `SELECT COUNT(*) FROM app_model;` | شمارش تعداد ردیف‌ها |
| `Model.objects.exists()` | `SELECT 1 FROM app_model LIMIT 1;` | بررسی وجود حداقل یک ردیف (بهینه‌تر از `count()`) |
| `Model.objects.first()` | `SELECT ... FROM app_model ORDER BY pk ASC LIMIT 1;` | اولین ردیف (نیازمند `ORDER BY`) |
| `Model.objects.last()` | `SELECT ... FROM app_model ORDER BY pk DESC LIMIT 1;` | آخرین ردیف |
| `Model.objects.none()` | `SELECT ... FROM app_model WHERE FALSE;` | یک کوئری‌ست خالی |

### نکات کلیدی برای به خاطر سپردن

1. **Lazy بودن**: کوئری‌ست‌ها تا زمانی که ارزیابی نشوند (با `list()`، `for`، یا `len()`) هیچ SQL‌ای اجرا نمی‌کنند. زنجیره‌سازی متدها فقط کوئری را می‌سازد.
2. **`select_related` در مقابل `prefetch_related`**: اولی برای روابط تک‌مقداری (`ForeignKey`/`OneToOne`) و با `JOIN`؛ دومی برای روابط چندمقداری (`ManyToMany`/معکوس) و با کوئری جداگانه.
3. **`values` + `annotate` = `GROUP BY`**: این ترکیب دقیقاً معادل `GROUP BY` است. اگر `values` حذف شود، `annotate` روی کل جدول تجمیع می‌کند (نه per group).
4. **`Subquery` + `OuterRef`**: برای ساخت زیرکوئری‌های همبسته و جلوگیری از ضرب دکارتی ناشی از `JOIN` هم‌زمان چند رابطه.
5. **`Coalesce`**: برای تبدیل `NULL` به مقدار پیش‌فرض در نتایج تجمیعی ضروری است.

---

## 🛠️ Debugging tools / ابزارهای دیباگ

```python
# Show the generated SQL without executing it
# نمایش SQL تولیدشده بدون اجرا
print(queryset.query)

# Count queries executed inside a block
# شمارش کوئری‌های اجراشده در یک بلوک
from django.db import connection, reset_queries
reset_queries()
# ... database operations ...
print(len(connection.queries))
for q in connection.queries:
    print(q["sql"])
```

> ⚠️ **Note / توجه**: `DEBUG = True` must be enabled for `connection.queries` to work. In production, use `django-debug-toolbar`.
>
> برای کار کردن `connection.queries` باید `DEBUG = True` باشد. در production از `django-debug-toolbar` استفاده کنید.

---

## 📚 Resources / منابع

- [Django QuerySet API Reference](https://docs.djangoproject.com/en/stable/ref/models/querysets/)
- [Django Aggregation Cheat Sheet](https://docs.djangoproject.com/en/stable/topics/db/aggregation/)
- [Django `select_related` vs `prefetch_related`](https://docs.djangoproject.com/en/stable/ref/models/querysets/#select-related)

---

## 📄 License / لایسنس

MIT © [Your Name / نام شما](https://github.com/mehdiseyfie)
