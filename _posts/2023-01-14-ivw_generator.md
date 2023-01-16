# How I made practice sheets for English irregular verbs

1. TOC
{:toc}

In autumn 2020, I just started working as an English and Math home tutor. Although I had some
materials provided by my company, I had to find other materials on the web.

During my English teaching sessions, I found out my students had troubles with irregular verbs and
couldn't remember them well. I managed to find some practice sheets on the web but there weren't
never enough.

So I set out to make my own, using Python, because making them by hand on Word would've taken too
long. I created a _generator_ of practice sheets.

## How does my generator work?

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

### The list of irregular verbs

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

### Random selection of 20 verbs

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

### Create content for exercise and answer sheets

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

### Creating the PDF and exporting it

**Exercise sheet**

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

Then, from this list, generate the content for both the exercise and answer sheets:
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

The outer loop simply select each sub-array from the exercise list e.g. `['', 'wrote', '', '']`.

The inner loop will draw a single table row with 4 cells, one containing the clue for the verb, and
leaving the other obviously empty. And formats it nicely, centers it horizontally on the page. And
it will run 20 times, until the table is complete.

**Answer sheet**

For the answer sheet, I simply wanted it right after its corresponding exercise sheet, so I wrote
`pdf.add_page()`. 

Then, I wanted to have the clue in bold font, and leave the other cells as is. So I had to be a bit
more creative by adding counters:

```python
# Generate the table containing the answers
j = 0
for line in answers:
    i = 0
    for value in line:
        if i == form_indices[j]:
            pdf.set_font('Arial', 'IB', size=11)
        else:
            pdf.set_font('Arial', size=11)
        pdf.cell(48, 12, txt=value, ln=0, border=1, align='C')
        i += 1
    j += 1
    pdf.ln()
```

There I had a combination of an exercise sheet and its corresponding answer sheet.

**Exporting the document**

All of this is great, but there's one more step, which is to export the document into a tangible PDF
file. Right now, all of what's been generated is sitting in the RAM, not persistent. Fortunately,
everything can be done through a single command thanks to `FPDF` which is `pdf.output(<filename>)`.

### What's next?

Well, if I left the code as is, I'd be able to generate only one exercise and one answer sheet at
each run. My goal was to make as many randomized sheets as possible in one go. Also, I wanted to
make them a little more professional, with a proper heading and some instructions.

**Make the headings**

In the `irregular_verbs_create` module, I created a simple function that would create the header,
indicating the type of sheet (exercise or answer), as well as the worksheet number and a field for
the student to write the date. Depending on the type of sheet, it will also write (or not)
instructions.

```python
def set_header(pdf, title, version):
    """Set the header for the PDF file.

    :pdf: PDF object generated by PyFPDF.
    :title: Main title of the sheet.
    :version: test OR correction sheet.

    """
    pdf.set_font('Arial', 'B', size=16)
    pdf.cell(0, 15, txt=title, ln=1, align='C')

    if version == 'test':
        pdf.set_font('Arial', 'I', size=11)
        pdf.cell(0, 5, txt='Remplir les champs manquants et donner'
                 ' la traduction des verbes.', ln=1, align='C')
        pdf.cell(0, 6, txt='De gauche à droite : base verbale'
                 ' - prétérit - participe passé - traduction',
                 ln=1, align='C')
```

When this function is called from `main.py`, I'm passing the `title` which contains the worksheet
number at each iteration.

```python
title_student = 'Irregular Verbs Worksheet ' + \
    str(worksheet_number) + " - Date :"
title_teacher = 'Irregular Verbs Worksheet ' + \
    str(worksheet_number) + " - Correction"

[...]


# Set the header for the test sheet
irregular_verbs_create.set_header(pdf, title_student, 'test')

[...]

# Set the header for the correction sheet
irregular_verbs_create.set_header(pdf, title_teacher, 'correction')

[...]
```

Therefore, for each sheet, the heading will be at the same place, in identical fonts.

**Make as many worksheets as desired**

Now my code can create beautiful irregular verbs worksheets (exercise and answer), but only one set
at each run. My desire is to run the script once and have as many worksheets as I wanted (that's the
initial goal!). A loop is in order.

I encapsulated the entire PDF generation code into a `while` loop that'll stop after a decided
number of worksheets to create:

```python
while worksheet_number <= number_of_worksheets:
    title_student = 'Irregular Verbs Worksheet ' + \
        str(worksheet_number) + " - Date :"
    title_teacher = 'Irregular Verbs Worksheet ' + \
        str(worksheet_number) + " - Correction"

    # Generate the table for test/answers
    test, answers, form_indices = irregular_verbs_create.generate_tables(verbs)
    #------------------------------------------
    # 1. TEST SHEET
    #------------------------------------------

    # Add page to the PDF that is the test page
    pdf.add_page()

    # Set the header for the test sheet
    irregular_verbs_create.set_header(pdf, title_student, 'test')

    # Generate the PDF page containing the exercise
    pdf.set_font('Arial', size=11)
    for line in test:
        for value in line:
            pdf.cell(48, 12, txt=value, ln=0, border=1, align='C')
        pdf.ln()

    #------------------------------------------
    # 2. ANSWER SHEET
    #------------------------------------------

    # Add page to the PDF that is the correction page
    pdf.add_page()

    # Set the header for the correction sheet
    irregular_verbs_create.set_header(pdf, title_teacher, 'correction')

    # Generate the table containing the answers
    j = 0
    for line in answers:
        i = 0
        for value in line:
            if i == form_indices[j]:
                pdf.set_font('Arial', 'IB', size=11)
            else:
                pdf.set_font('Arial', size=11)
            pdf.cell(48, 12, txt=value, ln=0, border=1, align='C')
            i += 1
        j += 1
        pdf.ln()

    # Increment the worksheet number
    worksheet_number += 1
```

**Make the script flexible**

With my students, I had some who were in junior high school, and some in high school. So the lists
for the irregular verbs are different. For the latter, the list may be a little more extensive. So I
wanted to add the possibility to source from different CSV files.

As a CLI-run script, I added this possibility by importing the `sys` module to be able to read
arguments from the command line.

```python
if len(sys.argv) < 2:
    print('Usage: python3 main.py [verb list file] '
          '[start worksheet no.] [no. of worksheets]'
          ' [output PDF file]')
    sys.exit()

list_file = sys.argv[1]
worksheet_number = int(sys.argv[2])
number_of_worksheets = int(sys.argv[3])
output_pdf_file = sys.argv[4]
```

## Conclusion

Finally, the script is complete, and I can run it and generate tens of randomized worksheets via a single run.
How efficient is that?

If you want to have a look at what the generated PDF looks like, [click
here](https://github.com/hobyr/irregular_verbs_generator) to get to my repository, and click on the
`IVW.pdf` file.

You'll have access to the repository to check my code in its entirety, and even download it if you
want to use the script.

If you've found this post useful (for you or someone else) and learned something, don't hesitate to
share it. Thank you for reading!
