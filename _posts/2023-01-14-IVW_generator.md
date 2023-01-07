# How I made practice sheets for English irregular verbs #

1. TOC {:toc}

In autumn 2020, I just started working as an English and Math home tutor. Although I had some
materials provided by my company, I had to find other materials on the web.

When it came to teaching English, I found out my students had troubles with irregular verbs and
couldn't remember them well. I managed to find some practice sheets on the web but there weren't
never enough.

So I set out to make my own, using Python, because making them by hand on Word would've taken too
long. I created a _generator_ of practice sheets.

## What does my generator do? ##

As I didn't want to have a complicated piece of software with a GUI, I chose to stay with the
command-line. It's just quicker.

My goal was to generate as many practice sheets and their corresponding answer sheets as desired,
and have each one of them randomized. Just with one command.

The way my generator works is this:

1. Import the list of irregular verbs from a CSV file into a regular Python list
2. Select 20 verbs at random
3. For each of these 20 verbs, select a random form (present, preterit, present perfect or meaning)
to show on a table row
4. Create a PDF object to draw a table from the 20 rows generator in the previous step
5. Export the PDF object into a PDF file

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
        file.seek(53)
        verbs = []
        for line in file:
            verbs.append(line[:-2].split(' ,'))
        return verbs

```

### Random selection of 20 verbs ###

I chose to create a separate module from the `main.py` file, it makes for a tidier code.


