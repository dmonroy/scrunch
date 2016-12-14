Introduction
============

The following document describes the basic usage of scrunch.

Step 1: Connect and get a dataset
---------------------------------

Lets start a simple session and get a Crunch dataset:

```python
from scrunch import connect, get_dataset
site = connect(user="me@mycompany.com", pw="yourpassword")
ds = get_dataset("Example Dataset", site=site)
```

There is an optional argument `project` that indicates which 
Crunch project should the scope of the session be based on.

A session can also be created with a token:

```python
from scrunch import connect_with_token, get_dataset
site = connect_with_token(token="token")
ds = get_dataset("Example Dataset", site=site)
```

Step 2: Set yourself as the current editor
------------------------------------------

One crucial step when manipulating dataset is that you need to be the current
editor. Set yourself as the editor by calling:

```python
ds.change_editor(user='me@mycompany.com')
```


Step 3: Access and mutate variables
-----------------------------------

Variables are available as members of the dataset object, like a dictionary. You can change any of the variable's attributes by providing them as keyword
arguments to the *edit* method. You can also add a description, and hide a variable for example:
:

```python
var = ds['gender']
# Edit existing attributes:
var.edit(name='my new name', alias='gender_ng')
# Add a description
var.edit(description='My awesome description')
# Hide the variable
var.edit(discarded=True)
```

You can also access and edit categories for categorical variables. Either 
provide a complete list of new categories or if you want to change only the name 
of a category you can achieve that with:

[comment]: TODO are we going to access categories as var.categories instead?
 

```python
var.body.categories[0]['name'] = 'My new category'
var.edit(categories=var.body.categories)
```

Step 4: Ordering and organizing variables
-----------------------------------------
 
[comment]: TODO: To be rewritten by Alejandro.


Step 5: Standard Cleaning Features
----------------------------------

### Exclusion Filters

Drop rules are used to delete invalid cases -- respondents who spent too little 
time answering the survey ("speeders"), cases with inconsistent data, etc. 
In Crunch, these are supported using *exclusion filters*, which are 
specified using a logical expression.

For example, assume that we have `disposition` as the alias of a variable 
in the dataset. Then apply the exclusion filter:

```python
ds.exclude("disposition != 0")
```

(Here, zero is the id (or code) assigned to completed interviews.)

We can also exclude a list of ids using:

```python
ds.exclude("disposition in [0, 1]")
```

Date variables are also supported in exclusion filters, like shown in the
following examples:

```python
ds.exclude('registration_time >= "2015-01-01T00:00:00+00:00"')
ds.exclude('registration_time < "2015-01-01T00:00:00+00:00"')
ds.exclude('registration_time == "2015-01-01T00:00:00+00:00"')
ds.exclude('not(registration_time == "2015-01-01T00:00:00+00:00")')
```

#### Filter expressions

At the moment *filter expressions* can be composed using the following logical expressions:

| operator | meaning               |
|:--------:|-----------------------|
| ==       | equal                 |
| !=       | unequal               |
| >        | greater than          |
| >=       | greater or equal      |
| <        | less than             |
| <=       | less or equal         |
| and      | logical *and*         |
| or       | logical *or*          |
| in       | in *list/tuple*       |
| not in   | not in *list/tuple*   |
| has_any  | has_any(*list/tuple*) |
| has_all  | has_all(*list/tuple*) |

`Note:` The expression needs to contain the **alias** and the **value**.

### Combine categories

[comment]: TODO: rename recode -> combine

scrunch's variable objects provide a `recode` method that tries to mimic
the SPSS recode syntax as much as possible.

For example, if `brandrating` is a variable with categories: `Very favorable`,
`Somewhat favorable`, `Neutral`, `Somewhat unfavorable`, `Very unfavorable`,
`Don't know` (with codes 1,2,3,4,5,9 respectively), we can create a new
re-coded variable `brandrating2` using the following code:

```python
var = ds.brandrating
new_var = var.recode(
    alias='brandrating2',
    map={1: (1, 2), 2: 3, 3: (4, 5)},
    names=('Favorable', 'Neutral', 'Unfavorable'),
    name='Brandrating 2',
    description='Recoding brandrating'
)
```

Sensible defaults are used whenever possible. For example, we can omit the
`name` and `description` arguments:

```python
var = ds.brandrating
new_var = var.recode(
    alias='brandrating2',
    map={1: (1, 2), 2: 3, 3: (4, 5)},
    names=('Favorable', 'Neutral', 'Unfavorable')
)
```

in which case the name of the new variable would be the original name plus the
" (recoded)" suffix and the description would be same as the original variable.

The `recode` method also works on `multiple_response` Variables. For example:

```python
var = ds.mult_resp
new_var = var.recode(
    alias='recoded_mult_resp',
    map={
        'new_subvar_alias1': ('orig_subvar_alias1', 'orig_subvar_alias2'),
        'new_subvar_alias2': ('orig_subvar_alias3', 'orig_subvar_alias4')
    },
    name='Re-coded multiple response',
    description='Combining responses for mult_resp'
)
```

Note: the SPSS-like `recode` method only works on `categorical`,
`categorical_array` and `multiple_response` Variable entities.

#### Creating a categorical variable

Transformations create new variables based upon the values of one or more input variables. 

```python
categories = [
    {"id": 1, "name": "Hipsters", "numeric_value": None, "missing": False},
    {"id": 2, "name": "Techies", "numeric_value": None, "missing": False},
    {"id": 3, "name": "Others", "numeric_value": None, "missing": False}
]

rules = ['var1 == 1 and var2 == 1', 'var1 == 2 and var2 == 2']

categorical_var = ds.create_categorical(
    categories=categories,
    rules=rules,
    name='New variable',
    alias='alias', 
    description='description')
```

*Rules* are evaluated in order (as if this were a sequence of if/elif/else statements. The values for the rule expressions must be valid values on the source variables.


Step 6: Downloading Datasets
----------------------------

Downloading datasets is as easy as:

```python
ds.download(
    path='/Dowloads/ds.csv', 
    filter='var1 == 4',
    variables=['var1, var2'],
    include_hidden=True
)
```

This download will be a CSV file with categories as numeric id's. The `filter` 
argument allows the filtering of rows based on expressions and defaults to all 
rows. `variables` is an optional argument which allows to define which variables 
to include  in the download. The default behavior is to download all variables 
including hidden ones. `variables` is a list of variables `alias`, `id` or `objects`. 
You can pass  `include_hidden=False` which will avoid including hidden variables 
in the downloaded file. 

When `variables` is specified the `include_hidden` argument is ignored for obvious reasons.
