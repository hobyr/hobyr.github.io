# How I made practice sheets for English irregular verbs

1. TOC
{:toc}

In autumn 2020, I just started working as an English and Math home tutor.
Although I had some materials provided by my company, I had to find other
materials on the web.

When it came to teaching English, I found out my students had troubles
with irregular verbs and couldn't remember them well. I managed to find
some practice sheets on the web but there weren't never enough.

So I set out to make my own, using Python, because making them by hand on
Word would've taken too long.

## What does my generator do?

As I didn't want to have a complicated piece of software with a GUI,
I chose to stay with the command-line. It's just quicker.

My goal was to generate as many practice sheets and their corresponding
answer sheets as desired, and have each one of them randomized. Just with
one command.

The way my generator works is this:
1. Import the list of irregular verbs from a CSV file into a Python list.
2. Select 20 verbs at random.
3. For each of these 20 verbs, select a random form to show (present, preterit, past
   perfect or French meaning) on a table row.
4. Create a PDF object to draw a table and insert the generated table row
   from 3.
5. Export PDF object into a PDF file.

## The list of irregular verbs

First of all, I had to gather the list of common irregular verbs my
students were learning. Conveniently, I found a CSV file containing all of
them, alongside their French meaning.

