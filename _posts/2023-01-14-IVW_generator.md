# How I made practice sheets for English irregular verbs #

1. TOC {:toc}

In autumn 2020, I just started working as an English and Math home tutor. Although I had some
materials provided by my company, I had to find other materials on the web.

During my English teaching sessions, I found out my students had troubles with irregular verbs and
couldn't remember them well. I managed to find some practice sheets on the web but there weren't
never enough.

So I set out to make my own, using Python, because making them by hand on Word would've taken too
long. I created a _generator_ of practice sheets.

## How does my generator work? ##

As I didn't want to have a complicated piece of software with a GUI, I chose to stay with the
command-line. It's just quicker.

My goal was to generate as many practice sheets and their corresponding answer sheets as desired,
and have each one of them randomized. Just with one command.

The way my generator works is this:

1. Import the list of irregular verbs from a CSV file into a regular Python list
2. Select 20 verbs at random
3. For each of these 20 verbs, select one random form (present, preterit, present perfect or meaning)
to show on a table row, hiding the other. **This is the exercise**, the student has to write the
remaining forms correctly.
4. Create a PDF object to draw a table from the 20 rows generator in the previous step
5. Export the PDF object into a PDF file

I chose to create a separate module from the `main.py` file, it makes for a tidier code. I named it
`irregular_verbs_create.py` where I wrote all the functions that generate content for the PDF.

In the `main.py`, the code takes all the content and formats it into a nice PDF file.

### The list of irregular verbs ###

First of all, I had to gather the list of common irregular verbs my students were learning.
Conveniently, I found a CSV file containing all of them, alongside their French meaning.

I created a function to import the verbs, conveniently named `import_verbs()`:

```python
def import_verbs(list_file):
    """Import the verbs list into a suitable array.

    :list_file: CSV file containing the irregular verbs.
    """
    with open(list_file, 'r') as file:
        file.seek(53) # Ignore the column headings
        verbs = []
        for line in file:
            verbs.append(line[:-2].split(' ,'))
        return verbs
```

As a result, I get a list of lists: each sublist contains the 4 forms (present, preterit, present
perfect and its French meaning) of each irregular verb.

```bash
[['abide', 'abode', 'abode', 'respecter/se conformer à'],
 ['arise', 'arose', 'arisen', 'survenir'],
 ['awake', 'awoke', 'awoken', 'se réveiller'],
 ...,
 ['wring', 'wrang', 'wrung', 'tordre'],
 ['write', 'wrote', 'written', 'écrire']
]
```

### Random selection of 20 verbs ###

In this list, the verbs are in alphabetical order. However, having them in a random order makes for
a more effective practice session. 

First, I'll get a list of indices, each index refering to a sublist from the `verbs` list. I'll then
shuffle and select the first 20.

```python
indices = list(range(len(verbs)))
shuffle(indices)
verb_indices = indices[0:20]
```

Then I'll have to choose which form of the each verb to show on the table row. They also have to be
chosen at random. So I'll create a list that contains 5 of each possible form, then shuffle it.

```python
form_indices = 5*[0, 1, 2, 3]
shuffle(form_indices)
```

### Create content for exercise and answer sheets ###

From these two lists `verb_indices` and `form_indices`, I can now create the exercise table. I'll
first create an empty row, by making a list of 4 empty strings. Then in this list, I'll put the
selected verb form in its corresponding index. So for example I'd get: `['', 'wrote', '', '']`.
Finally, I'll append that list into a global list I call `data_test`, from which I'll get content
when drawing the PDF file. And do this 20 times.

Regarding the answer sheet, I'll just copy the unaltered list. Following the above example, I'll
append `['write', 'wrote', 'written', 'écrire']` to the list aptly named `data_correction`.

```python
data_test = []
data_correction = []
for i in range(20):
    line = 4*['']
    line[form_indices[i]] = verbs[verb_indices[i]][form_indices[i]]
    data_test.append(line)
    data_correction.append(verbs[verb_indices[i]])
```

I finally combined all of this steps into a single function called `generate_tables()`:

```python
def generate_tables(verbs):
    """Generate the answer rows with only one clue chosen at random.

    :verbs: contains the array of irregular verbs and the 4 forms.
    :returns: test and answer tables with indices of the forms.
    """
    indices = list(range(len(verbs)))
    shuffle(indices)
    verb_indices = indices[0:20]

    form_indices = 5*[0, 1, 2, 3]
    shuffle(form_indices)

    data_test = []
    data_correction = []
    for i in range(20):
        line = 4*['']
        line[form_indices[i]] = verbs[verb_indices[i]][form_indices[i]]
        data_test.append(line)
        data_correction.append(verbs[verb_indices[i]])

    return data_test, data_correction, form_indices
```

### Creating the PDF and exporting it ###

To create the PDF, I used an external module named `FPDF` which is quite popular for making simple
PDFs.

Since the external module I created `irregular_verbs_create` is only for generating content, I wrote
all the PDF related functions in the `main.py` file.

Also, I wanted to create several practice sheets in one go. For example, create 100 different
practice sheets would be great!

First, I had to import the irregular verbs into a Python list:
```python
# Import the verbs list into a suitable array
verbs = irregular_verbs_create.import_verbs(list_file)
```

Then, from this list, generate the content:
```python
# Generate the table for test/answers
test, answers, form_indices = irregular_verbs_create.generate_tables(verbs)
```

I could then create the PDF object to draw on it. Luckily, with `FPDF` it's super easy: `pdf
= FPDF()` and it generates what I call the master object. Before drawing, I'll have to add a page to
the object by calling `pdf.add_page()`.

Now I can draw the table row by row. I must choose a font and font size too before drawing any text.
```python
# Generate the PDF page containing the exercise
pdf.set_font('Arial', size=11)
for line in test:
    for value in line:
        pdf.cell(48, 12, txt=value, ln=0, border=1, align='C')
    pdf.ln()
```

The inner loop will draw a single table row with 4 cells, one containing the clue for the verb, and
leaving the other obviously empty.
