# Создание иерархических категорий с django-mptt

Чтобы проиллюстрировать, как работать с **MPTT**, мы будем использовать приложение **ideas** из главы 3 _«Формы и представления»_. В наших изменениях мы заменим категории иерархической моделью **Category** и обновим модель **Idea**, чтобы иметь отношения «многие ко многим» с категориями. Кроме того, вы можете создать новое приложение, используя только показанный здесь контент, чтобы реализовать очень простую версию модели **Idea** с нуля.

## Подготовка

Для начала выполните следующие шаги:

1. Установите **django-mptt** в вашей виртуальной среде с помощью следующей команды:

```bash
(env)$ pip install django-mptt==0.10.0
```

2\. Создайте приложения **categories** и **ideas**, если вы еще этого не сделали. Добавьте эти приложения, а также **mptt** в **INSTALLED\_APPS** в настройках следующим образом:

```python
# myproject/settings/_base.py
INSTALLED_APPS = [
    # …
    "mptt",
    # …
    "myproject.apps.categories",
    "myproject.apps.ideas",
]
```

## Как это сделать...

Мы создадим иерархическую модель **Category** и свяжем ее с моделью **Idea**, которая будет иметь отношения «многие ко многим» с категориями следующим образом:

1. Откройте файл **models.py** в приложении **categories** и добавьте модель **Category**, которая расширяет `mptt.models.MPTTModel` и **CreationModificationDateBase**, определенную в главе 2 «_Модели и структура базы данных_». В дополнение к полям, поступающим из миксинов, модель **Category** должна иметь родительское поле типа **TreeForeignKey** и поле заголовка **title**:

<pre class="language-python"><code class="lang-python"># myproject/apps/ideas/models.py
from django.db import models
from django.utils.translation import ugettext_lazy as _
from mptt.models import MPTTModel
from mptt.fields import TreeForeignKey

from myproject.apps.core.models import CreationModificationDateBase

class Category(MPTTModel, CreationModificationDateBase):
    parent = TreeForeignKey(
        "self", on_delete=models.CASCADE,
<strong>        blank=True, null=True, related_name="children"
</strong>    )
    title = models.CharField(_("Title"), max_length=200)
    
    class Meta:
        ordering = ["tree_id", "lft"]
        verbose_name = _("Category")
        verbose_name_plural = _("Categories")

    class MPTTMeta:
        order_insertion_by = ["title"]

    def __str__(self):
        return self.title</code></pre>

2\. Обновите модель **Idea**, включив в нее поле категорий типа **TreeManyToManyField**:

```python
# myproject/apps/ideas/models.py
from django.utils.translation import gettext_lazy as _

from mptt.fields import TreeManyToManyField

from myproject.apps.core.models import CreationModificationDateBase, UrlBase

class Idea(CreationModificationDateBase, UrlBase):
    # …
    categories = TreeManyToManyField(
        "categories.Category",
        verbose_name=_("Categories"),
        related_name="category_ideas",
    )
```

3\. Обновите свою базу данных, выполнив миграции и запустив их:

```bash
(env)$ python manage.py makemigrations
(env)$ python manage.py migrate
```

## Как это работает...

Миксин **MPTTModel** добавит поля **tree\_id**, **lft**, **rght** и **level** в модель **Category**:

* Поле **tree\_id** используется, поскольку в таблице базы данных может быть несколько деревьев. Фактически каждая корневая категория сохраняется в отдельном дереве.
* В полях **lft** и **rght** хранятся левые и правые значения, используемые в алгоритмах **MPTT**.
* Поле **level** хранит глубину узла в дереве. Корневой узел будет иметь уровень **0**.

С помощью мета-параметра **order\_insertion\_by**, специфичного для **MPTT**, мы гарантируем, что при добавлении новых категорий они останутся в алфавитном порядке по названию.

Помимо новых полей, миксин **MPTTModel** добавляет методы для навигации по древовидной структуре, аналогичные тому, как вы перемещаетесь по элементам DOM с помощью JavaScript. Эти методы следующие:

* Если вы хотите получить доступ к предкам категории, используйте следующий код. Здесь параметр **ascending** определяет, с какого направления читать узлы (по умолчанию — `False`), а параметр **include\_self** определяет, включать ли саму категорию в **QuerySet** (по умолчанию — `False`):

```python
ancestor_categories = category.get_ancestors(
    ascending=False,
    include_self=False,
)
```

* Чтобы просто получить корневую категорию, используйте следующий код:

```python
root = category.get_root()
```

* Если вы хотите получить прямые дочерние элементы категории, используйте следующий код:

```python
children = category.get_children()
```

* Чтобы получить всех потомков категории, используйте следующий код. Здесь параметр **include\_self** снова определяет, включать ли саму категорию в **QuerySet**:

```python
descendants = category.get_descendants(include_self=False)
```

* Если вы хотите получить количество потомков, не запрашивая базу данных, используйте следующий код:

```python
descendants_count = category.get_descendant_count()
```

* Чтобы получить всех братьев и сестер, вызовите следующий метод:

```python
siblings = category.get_siblings(include_self=False)
```

{% hint style="info" %}
Корневые категории считаются родственными другим корневым категориям.
{% endhint %}

* Чтобы просто получить предыдущего и следующего братьев и сестер, вызовите следующие методы:

```python
previous_sibling = category.get_previous_sibling()
next_sibling = category.get_next_sibling()
```

* Кроме того, существуют следующие методы проверки того, является ли категория корневой, дочерней или листовой:

```python
category.is_root_node()
category.is_child_node()
category.is_leaf_node()
```

Все эти методы можно использовать либо в представлениях, либо в шаблонах, либо в командах управления. Если вы хотите манипулировать древовидной структурой, вы также можете использовать методы `insert_at()` и `move_to()`. В этом случае вы можете прочитать о них и методах менеджера дерева по [адресу django-mptt](https://django-mptt.readthedocs.io/en/stable/models.html).

В предыдущих моделях мы использовали **TreeForeignKey** и **TreeManyToManyField**. Они аналогичны **ForeignKey** и **ManyToManyField**, за исключением того, что они отображают варианты выбора с отступом в иерархии в административном интерфейсе.

Также обратите внимание, что в классе **Meta** модели **Category** мы упорядочиваем категории по **tree\_id**, а затем по значению **lft**, чтобы категории естественным образом отображались в древовидной структуре.

## Смотрите также

* Рецепт «_Создание примеси модели для обработки дат создания и модификации_» в главе 2 «_Модели и структура базы данных_».
* Рецепт _Создание интерфейса управления категориями с помощью django-mptt-admin_
