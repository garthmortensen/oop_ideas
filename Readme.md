# ORM

> ORMs let you interact with a database using OOP, instead of SQL. For whatever language you're using, you can find an ORM library. That means you don't need SQL, and can interact directly with the database. This forces you to use MVC. ORMs are to setup and learn though. 

[Source](https://stackoverflow.com/questions/1279613/what-is-an-orm-how-does-it-work-and-how-should-i-use-one)

Bootcamp Data 10:

Object means the objects you're using in your python program. Relational refers to the RDBMS. Mapping refers to bridging between the two.

ORM advantages:

- A single python query can work on Postgres, MySQL, SQLite, etc.
- You don't need to learn SQL.

SQLite syntax is similar to postgres, but write to disk. Install via `conda install -c anaconda sqlite`.

With ORM, you use classes to create table blueprints and update the schema. SQLAlchemy uses classes make changes to the databases. That is, it uses objects to map changes to SQL tables.

## Create a class

`def __init__(self):` is a "constructor". Python calls it whenever a new class instance is created.

Except for `self`, any params declared in `__init__` need to be passed if you want to create a new class instance.

## SQLAlchemy

`create_engine` creates database connections.

`declarative_base` converts Python classes into tables.

You need to import SQL datatypes into Python. You use them to create class fields, or table column fields.

```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base  # methods to abstract classes into tables
from sqlalchemy import Column, Integer, String, Float  # declare column types
```

After you create SQLAlchemy classes, create them on the database by first creating a connection engine, then calling `Base.metadata.create_all(engine)`.

`Base.metadata.create_all(engine)` scans through the script to see if declared classes are tables in the database. If not, they're created.

SQLAlchemy is like Git. You create a session, and then add and commit changes.

```python
from sqlalchemy.orm import Session
session = Session(bind=engine)
session.add(class1)
session.add(class2)
session.commit()
```

Want data from the database? Ez.

```sql
catS = session.query(Cat)
for each_cat in cats:
    print(each_cat.name)
```

`session.query()` is for getting all table rows.

You can also do this:

```python
engine.execute('select * from pet').fetchall()
```

To select a specific column with a WHERE clause, use `session.query(<SQL Class>).filter()` method.

```python
result = session.query(Cat).\
    filter(Cat.state == 'MN').count()
print(f"There are {usa} cats from MN")
```

Select several columns using AND/OR:

```python
born_after_2018 = session.query(Cat).\
    filter(Cat.birth_year > 2018).filter(Cat.state == "MN").\
    count()
print(f"{born_after_2018} MN cats were born after 2018")
```

Create a list from the results of a filter.

```python
post_2018_weight_list = []
for cat in born_after_2018_weight:
    if type(cat.weight) == int:
        post_2018_weight_list.append(cat.weight)
```

### Update

Want to update records? Because the records already exists in the database, you don't need to use `session.add()`. You only need `session.commit()` to update rows.

```python
cat = session.query(Cat).filter_by(name="Pony").first()  # query
cat.age += 2  # update data
# for modifying, you can use dirty attribute
session.dirty  # check status
session.commit()
session.dirty  # check status
```

### Delete

```python
cat = session.query(Cat).filter_by(id=2).one()
session.delete(cat)
session.deleted
session.commit()
session.query(Cat.id, Cat.name, Cat.type, Cat.age).all()
```

Stopped at 10.2 half 2...

### Reflect

Reflection automatically creates ORM classes from an existing database. Without it, you'd need to write SQLAlchemy classes for every table and column by hand. 

Reflection loads data from a database, and uses it to infer how to write ORM classes automatically. This is the process of reflection. It's pretty much like opening a csv in Excel and it automatically sets column types (string, date, number).

`automap_base` is like the `Base` class in `declarative_base`,  it has additional methods such as prepare, which will automatically reflects the data in a database. View these generated ORM classes via `Base.classes.keys()`. Said another way, this shows you a list of all the reflected tables. To access these classes, you can also use dot notation: `<ExampleClassName> = Base.classes.<ExampleClassName>`

After reflection, the autogenerated ORM classes can be used just like regular classes.

In a nutshell, you setup the database connection and reflect. Then you query.

### Inspect

If you need to examine the connected database and contents, use the inspector tool. You can look up tables, columns, and datatypes. I guess schema/meta data. For specific records in a table, use queries.

Import `inspect` module alongside the `create_engine` module.

Create an inspector by instantiating `inspect(engine)`. Used it to inspect various database elements.

To get table names, use `inspector.get_table_names()`.

To get table columns use `inspector.get_columns(<Table Name>)`.

### Joins

Before joining, identify the tables and columns you want, and what columns to join on. Use `inspect` to get table and column names. After that, use `.filter()` to merge the tables and get results.

### Inheritance

The python inheritance approach:

```python
# parent class
class Character(object):
    def __init__(self, species, name, size):
        self.species = species
        self.name = name
        self.size = size
        print(f"species: {self.species}, name: {self.name}, size: {self.size}.")

# child class
class Monster(Character):
    def __init__(self, species, name, size, hp, gold, hit_dice):
        super().__init__(species, name, size)
        self.hp = hp
        self.gold = gold
        self.hit_dice = hit_dice
```

Under the hood, inheritance and composition can be handled in many ways. According to [this](https://stackoverflow.com/a/25433198/5825523) SO post, some implementations spread data across tables and perform joins, others use single tables and perform unions. The user says that these details shouldn't be thought about, since we are after all abstracting the database work away! We are using ORM to get away from the database work. Very true...but not informative.

According to [SO](https://stackoverflow.com/questions/1337095/sqlalchemy-inheritance), we implement inheritance using different [techniques](https://docs.sqlalchemy.org/en/20/orm/inheritance.html), with single table inheritance and [joined table inheritance](https://stackoverflow.com/questions/6146795/how-do-i-correctly-model-joined-table-inheritance-with-discriminator-based-on-ro) being the most common.

Single table inheritance = multiple classes are represented by a single table. Better performance.

Joined table inheritance = class hierarchy is broken across multiple tables, where each class has its own table with its child attributes. Better design, with pk-fk relationships between parent-child, enforced by the database.

```python
# define inheritance in table class
class Hero(Character):
    __mapper_args__ = {'polymorphic_identity': 'hero'}
    attribute_local = Column(String(50))

# add to hero childclass a joined table inheritance   
__tablename__ = 'hero'
id = Column(None, ForeignKey('character.id'), primary_key=True)
```

Querying is mostly the same.

The big thing though is that Inheritance is Evil. If you can make do with Association (Aggregation), you dodge any fragile base class problems further down the road.

Association = relationship between 2 classes, joins two separate entities. It only means two objects "know" each other. Mother -> child. In a database, a junction tables join two tables (aka entities) together. As seen in Head First SQL pg 315. But, in this sense, we don't necessarily have a many-to-many entity relationship requiring a junction m-to-m table. Anyways. Implemented in OOP code, it means having an instance field.

```java
class Child {
    Mother mother;
}

class Mother {
    List<Child> children;
}
```

Aggregation = has-a. It's a special form of association which unidirectional. Wallet has money. Money doesn't have a wallet.

```java
class Wheel {}
class Car {
    List<Wheel> wheels;
}
```

So going back to the matter of Composition over Inheritance, avoid inheritance using this aggregation approach. One SO post wrote they go with association 90% of the time.

### Composition

If you need to use more than 2 levels of inheritance, it's probably a good idea to switch to composition. Deep inheritance is a bit of a "im smart" coding technique.

```python
# parent class, if this were inheritance
class Character(object):
    def __init__(self, species, name, size):
        self.species = species
        self.name = name
        self.size = size
        print(f"species: {self.species}, name: {self.name}, size: {self.size}.")

# child class, if this were inheritance
class Monster(object):
    def __init__(self, species, name, size, hp, gold, hit_dice):
        self.hp = hp
        self.gold = gold
        self.hit_dice = hit_dice
        self.character = Character(species, name, size)  # composition!
```

### More

You can also keep your class definitions independent from your table definitions. If you do this, instead of using `declarative_base`, you need to map your classes to your tables using the mapper function. E.g.

```python
mapper(Character, character)  # Classname, tablename
mapper(Hero, hero, inherits=Character, polymorphic_identity='hero')
mapper(Monster, monster, inherits=Character, polymorphic_identity='monster')
```

Read about the Repository pattern (a design pattern) using classical mapping here:

https://www.cosmicpython.com/book/chapter_02_repository.html
