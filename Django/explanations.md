> Sometimes it is better to know more about how the something actually works than just know how to use it.

- [save()](#save)
  - [*What happens after calling save()?*](#what-happens-after-calling-save)
  - [*UPDATE or INSERT?*](#update-or-insert)
  - [***Update fields using F expressions***](#update-fields-using-f-expressions)
  - [*Arguments*](#arguments)


## save()

### *What happens after calling save()?*

1. **Emit pre-save signal**. Let them know smt is happening.
2. **Adjust data**. Some data (for example datetime) needs extra modification because of their nature.
3. **Prepare data**. Database have can have diffrent datatypes than Python (datatime).
4. **INSERT data into database**. SQL INSERT statement is used.
5. **Emit post-save signal**. Someone is waiting for it...

### *UPDATE or INSERT?*
The `save()` method is also used for updating (SQL UPDATE statement). To check which statement use, Django does following:

- Look at Field's default attribute. Does primary key attribute define it?
    - if no then:
        - UPDATE if primary key attribute is set to True,
        - INSERT if to False or update doesn't change anything.

### ***Update fields using F expressions***
If you want to update some field, for example increment it, **it is better to use ***F expression***, because the work is done by database**, not Python. That means:

- things go faster,
- we avoid race conditions:
    - if 2 Python threads will try to do something with the same field, one's result could be lost because of 2nd threat.
    - Database always perfom changes based on actual values of fields (not of values taken a moment ago like Python thread)

For more details aboud race conditions check [Django docs' entry](https://docs.djangoproject.com/en/4.1/ref/models/expressions/#avoiding-race-conditions-using-f).

**Practical use of F expression:**
```
from django.db.models import F
reporter = Reporters.objects.get(name='Tintin')
reporter.stories_filed = F('stories_filed') + 1
reporter.save()

reporter.name = 'Tintin Jr.'
reporter.save()
```
The nice thing about the F expression is that you can use it one time and after that you can do things normally like in Python, but they will be interpretted like you would continue to use F expression!

### *Arguments*
**force_insert=False** --> force SQL INSERT statement. Can't be used in the same time with force_update arg.

**force_update=False** --> force SQL UPDATE statement. Can't be used in the same time with force_insert arg.

**using=DEFAULT_DB_ALIAS** --> decide to which db save data. 

**update_field=None** --> pass the list of fields' names to choose which fields update. Forces update and makes slightly better perfomances.