---
layout: post
title:  "Creating Genrello part 1"
date:   2016-12-05 08:00:00 +0100
categories: tutorial genrello genropy
description: How to create genrello...
comments: true
---

In this tutorial/course i would like show how i made Genrello (A Trello in Genropy).
Usally an application in genropy does not look like Genrello at all,
the basic application created in genropy are usally stuff
like ERP and CRM something like this:

![Sandbox Fatturazione](
{{site.baseurl}}/assets/images/creating_genrello/part_1/sandbox_fatt.png
)

But i want to show that is possible to create an application like
[Genrello](http://genrello.mkshid.me) with Genropy.

![Genrello]({{site.baseurl}}/assets/images/creating_genrello/part_1/genrello_prev.png)

I am going to skip on the installation part for local development
(checkout the guide on Genropy website [www.genropy.org](www.genropy.org))
but i will show you the one in production.

Well once you have got a starting project in genropy it looks like this:

![Genropy Starting Directory](
{{site.baseurl}}/assets/images/creating_genrello/part_1/starting_dir.png
)

*(app is the project name and base the main package.)*

After that you have create to the models,
it pretty the first step of almost all web application.


I think it is best to start simple with some core models like:

- team
- board
- list
- card


these should be the initial models.


All models are defined under `/packages/base/model`

Lets look at one of them for example 'board.py'

```python

class Table(object):

    def config_db(self, pkg):
        tbl = pkg.table('board', pkey='id', name_long='!!Board',
                        name_plural='!!Boards', caption_field='name')
        self.sysFields(tbl)

        tbl.column('name', name_short='Name',
                   validate_notnull=True, validate_case='c',
                   validate_notnull_error='!!Mandatory field')

        tbl.column('description', name_long='!!Description')
        tbl.column('position', dtype='I', name_long='!!Position')

        tbl.column(
            'team_id', size='22', group='_',
            name_long='!!Team id'
            ).relation(
                'team.id', relation_name='board_team',
                mode='foreignkey', onDelete='cascade'
            )

        tbl.column(
            'owner_user_id', size='22', group='_',
            name_long='!!Owner'
            ).relation(
                'adm.user.id', relation_name='board_user',
                mode='foreignkey', onDelete='cascade'
            )
            tbl.aliasColumn(
                'username',
                relation_path='@owner_user_id.username',
                name_long='!!Username'
            )

        tbl.aliasColumn('team_name',
                        relation_path='@team_id.name',
                        name_long='!!Team name')

```
First thing to beware of is to remember that model name and file name **MUST** be the same.
If not it will raise an Exception when you will actually create the db.

This model is pretty interesting it almost has everything except triggers and
some other fancy stuff.

If you notice the `class Table(object)` is only inheriting from `object`, that's because
Genropy use mixin for models, so you can override everything like the function
`config_db(self, pkg)`.

The first line of model:

```python
    tbl = pkg.table('board', pkey='id', name_long='!!Board',
                    name_plural='!!Boards', caption_field='name')
```
define the name of table and other stuff like `caption_field` which shows the referring
field instead of pkey.

the second line:

```python
    self.sysFields(tbl)
```

Needs to genropy to add default columns:

    "__ins_ts" -> insertion timestamp
    "__mod_ts" -> modification timestamp
    "__del_ts" -> deletion timestamp


then the most used one statement the one to create the column of the model:

```python
    tbl.column('name', name_short='Name',
               validate_notnull=True, validate_case='c',
               validate_notnull_error='!!Mandatory field')
```
pretty straight forward isn't it? it ask the name as first param (that could be enough).
`name_short` and `name_long` are used from genropy for the frametableh. This column
has some validation in this case,
one check that this column can not be null and the other one that is case sensitive.

Other important param is `dtype` which define the type of column if not specified
it well set as string type others types could be:

- 'A': 'character varying'
- 'C': 'character'
- 'T': 'text'
- 'X': 'text'
- 'N': 'numeric'
- 'B': 'boolean'
- 'D': 'date'
- 'H': 'time without time zone'
- 'HZ': 'time without time zone'
- 'DH': 'timestamp without time zone'
- 'DHZ': 'timestamp with time zone'
- 'I': 'integer'
- 'L': 'bigint'
- 'R': 'real'

Almost all models/table has foreign keys and in genropy they are defined like this:

```python
    tbl.column(
    'team_id', size='22', group='_',
    name_long='!!Team id'
    ).relation(
    'team.id', relation_name='board_team',
    mode='foreignkey', onDelete='cascade'
    )
```

this create an 1 to N relationship with the team table, basically you define a column
in this case `team_id` and on that you define a relation in this case with `id` of the
`team table` as you can see the first argument of relation
function is string like `tablename.field_to_realed_with`.

Last but not the least you can define `aliasColumn` which are columns not stored on the
db but gathered once request through a query
**(They must me explicitly specified in the queries)** pretty handy!.

Once done to create it actually into the db like (postgres).
you have to lunch the command `gnrdbsetup instanceName`,
in this case `gnrdbsetup genrello`.

As first part i think it is enough.

The code of the others models can be found [here](
https://github.com/mkshid/genrello/tree/master/packages/base/model
)

The next part will on frametable of genropy which is the real strength of genropy.

See you soon!!
