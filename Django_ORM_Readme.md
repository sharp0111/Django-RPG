What is the Django ORM?

Object Relational Mapper. Magic glue between Python model classes, and it being translated into SQL.

 Check out the django RPG. https://github.com/LambdaSchool/Django-RPG
pipenv install (like yarn or npm install)
pipenv shell

Look at charactercreator/models.py, examine classes.

Look at armory/models.py, same.

Look at charactercreator/models.py, examine Character. There's a many-to-many relationship. A character can have many items, and items can be had by many characters. E.g. many characters might have a bronze sword.

Try to run the server. Problems.

Look in settings.py. Expecting env variables.
Be lazy and copy .env from another project.

Run server. Note about migrations.

manage.py showmigrations
manage.py migrate

Check out /api

But we won't use that today--we'll just use the ORM in the django shell.

manage.py to get help

manage.py help loaddata

Installs named fixtures. Fixture is a JSON file. 

Look at testdata.json in the root.

manage.py loaddata testdata.json

manage.py shell

What do we need to do to query the data we just imported?

Import models. Look at charactercreator/models.py.

from charactercreator.models import Character, Fighter, Mage, Cleric, Thief, Necromancer
from amory.models import Item, Weapon
dir()

How do we get all the weapons?

Weapon.objects.all()

weapons = Weapon.objects.all()
weapons[0].name
weapons[1].power
etc.

dir(Weapon.objects)

We've seen create, filter, all, none, exclude, values_list.

How about raw? It allows you to do direct SQL queries.

What tables do we have? Go to the db shell

manage.py dbshell
.tables

We see armory_weapon.

Back to shell

manage.py shell
from armory.models import Weapon, Item
raw_test = Weapon.objects.raw('SELECT * FROM armory_weapon;')
raw_test[0]
raw_test[1]
raw_test[2]

Basically the same as Weapon.objects.all().

It can be dangerous to build queries for .raw(). If you allow user input in the query, what kind of attacks are you opening yourself up to? [SQL Injection]

https://www.xkcd.com/327/ Bobby Tables

Example:

query = "select * from characters where name = '%s';"

user_input = "Thor"
query % user_input

user_input = "Thor')re; DROP TABLE characters;--"
User_input = "Thor'; DELETE FROM characters;"
query % user_input

So be very, very careful with raw(). With ORM, there's no problem because we're not constructing the SQL by hand.

Back on track.

from charactercreator.models import Character, Fighter, Mage, Cleric, Thief, Necromancer

dir(Character.objects)

How to get the inventory of the character.

character = Character.objects.first()

character.name
dir(character)


character.strength
character.hp

Inventory is a m:m relationship.

character.inventory
dir(character.inventory)

Looks like a lot of ORM methods again.

character.inventory.all()

Back to dbshell

manage.py dbshell

How is this working under the hood?

.mode column
.headers on

What tables do we need to have a m:m relation between characters and items?

How do you represent 1:1 or 1:m? A foreign key.

Let's look for inherited classes.

select * from charactercreator_character limit 5;

See all the fields in Character.

select * from charactercreator_cleric limit 5;

There's just a pointer to a Character, and then the additional fields for Cleric.

That's for 1:1 and 1:m relationships. How about m:m?

We want something that somehow relates character and inventory.

.tables

See charactercreator_character_inventory table.

select * from charactercreator_character_inventory limit 10;

Just foreign keys into two other tables to relate them. You need a third join table for a m:m relationship. It connects data in two other tables.

So let's get all the inventory items for a single character with a join.

select c.name, i.name from charactercreator_character as c, armory_item as i, charactercreator_character_inventory as ci where c.character_id = ci.character_id and i.item_id = ci.item_id and c.character_id = 1;

Of course, it was easier just to say

character = Character.objects.get(character_id=1)
character = Character.objects.get(pk=1)  # same, alias for primary key
character.inventory.all()


Let's make a new item and give it to a character.

from armory.models import Item
new_item = Item.objects.create(name='Sword of Awesomeness', value=1000000, weight=25)
new_item.name

How do we give it to the character?

character.name
character.inventory
dir(character.inventory)

see add()

character.inventory.add(new_item)
character.inventory.all()

See the new item in there.

Multiple SQL INSERTs to make it happen. We don't have to worry about that.

Sprint Challenge will be to set up the project just like this, then run some ORM queries against it.

Get a list of all information in the Character table:

Character.objects.values_list()

contrast to

Character.objects.all()

Character.objects.values_list('name')  # lots of tuples

Character.objects.values_list('name', flat=True)  # Just make a list
