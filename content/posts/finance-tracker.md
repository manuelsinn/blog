---
title: "How I track my finances with Python (as a minimalist)"
date: 2020-03-01T09:35:01+01:00
authors:
    - Manuel Sinn
tags:
    - python
    - software-development
draft: false
---

## Motivation
To track my monthly expenses, I wanted to create a minimalist solution that would deliver a comprehensive overview of my spending, without needing a lot of work put in. As I was used to writing down every expense in Google Keep anyway, I thought I might as well just keep using Google Keep as the client. 

After a bit of research, I discovered that there was an unofficial  [Python client for the Google Keep API](https://github.com/kiwiz/gkeepapi "Check it out on Github"). As I had previously been pretty much exclusively using Java, I thought this was the perfect opportunity to dive into a new language and learn something new.

## How it works

### Client side
The way I was used to writing down my expenses was simply to note the amount of money spent, and after that a letter to serve as a mnemonic for the corresponding category. When I wanted to remember what the exact reason for that expense was, I would simply write the cause after the mnemonic. 

For example, when I spent 2.50 € on printing papers for university, and another 5 € on presents I would simply write:

[2,50](# "The amount of money I spent") [u](# "The mnemonic for university") [printing my cv for applications](# "Cause for expense") 

[5](# "The amount of money I spent") [p](# "The mnemonic for presents") [present for parent's birthday](# "Cause for expense") 

As most of my expenses fall into the food category (and I do not need to remember much about those) I decided to leave this mnemonic blank, so that all I had to do was write down the price of the purchase.

Each month, I would create a note, pin it to the top and record my expenses in this fashion. When the month was over, I would unpin the note and archive it, since I would only need to keep the data in Keep and not see the note itself any longer.

### Tech Stack 
The Python script I created uses the above mentioned API to gather the data from my notes on Google Keep. I then used pandas dataframes to organize the data into spending categories. For each month the sum of expenses is calculated for each category, so that I can see how much money I spent in each category for a given month in the current year.

Obviously I also wanted to somehow display this overview in a way that was useful to me in my everyday life, so I used the openpyxl library to write the data into a custom made Excel sheet. This was the easiest and most cost-effective way I could enjoy a nice looking, practical and scalable front-end. Of course I could have opted to specifically design and implement one myself, but this would have taken away the focus from what I wanted to work on and likely would have introduced a lot of unnecessary work.

To make the application easily accessable I wrote a tiny little bash script to call from anywhere. All this would do is start the Python script and when finished, open the Excel file where the newly updated information had just been written to.

### Code and GUI
#### The Python script:
```python
import io
import sys
import gkeepapi
import calendar
import pandas as pd
from pandas import ExcelWriter
import numpy as np
import datetime
import openpyxl
from openpyxl.utils.dataframe import dataframe_to_rows


# global variables
current_month = calendar.month_name[datetime.datetime.now().month]
categories = ('Food','Eating Out', 'Leisure', 'Home', 'Uni', 'Clothing', 'Presents')
months = [calendar.month_name[i] for i in range(1, 13)]
notes = gkeepapi.Keep()
now = datetime.datetime.now()
username = "my@email.com"
google_pass = "password"



def log_in(self):
    print("Logging in...")
    notes.login(username, google_pass)


# Returns a dataframe with the sum of spent money, for each category and each month of the year
def get_overview(self):
    print("Getting data from Google Keep...")
    summary = pd.DataFrame(columns=categories, index=months)
    for month in range(1,13):
        for category in categories:
            summary.at[calendar.month_name[month], category] = self.get_category_sum(category, month)
    return summary


# Returns the sum of spent money for a given category in a given moth
def get_category_sum(self, category, month):
    data = self.get_data_of_month(month)
    data = data[data['Category'] == category]
    pd.to_numeric(data['Amount'], errors='coerce')
    return data['Amount'].sum().round(2)


# Returns a dataframe with all expenses from a given month
def get_data_of_month(self, month):
    # find expenses note from month
    month_note = notes.createNote()
    month_note_title = calendar.month_name[month] + " Expenses"
    for note in notes.find(query=month_note_title):
        month_note = note

    # save note content as StringIO
    month_data = io.StringIO(month_note.text)

    # create pandas dataframe from month data
    dataframe = pd.read_csv(month_data, sep=' ', decimal=',', header=None, names=['Amount', 'Category', 'Info'], encoding='ISO-8859-1')             
    return self.clear_data(dataframe)


# Replaces all mnemonics with their respective category in a dataframe and turn numbers into floats
def clear_data(self, df):
    df['Category'] = df['Category'].replace(np.nan, 'Food')
    df['Category'] = df['Category'].replace('o', 'Eating out')
    df['Category'] = df['Category'].replace('l', 'Leisure')
    df['Category'] = df['Category'].replace('h', 'Home')
    df['Category'] = df['Category'].replace('u', 'Uni')
    df['Category'] = df['Category'].replace('c', 'Clothing')
    df['Category'] = df['Category'].replace('p', 'Presents')
    # turn all numbers into floats:
    df['Amount'] = df['Amount'].astype(float)    
    return df
    
# Writes an overview of the yearly spending to the corresponding cells of the Excel sheet
def write_to_sheet(self):
    path = "path/to/ExcelSheet.xlsx"
    wb = openpyxl.load_workbook(path)
    ws = wb["2019"]

    rows = dataframe_to_rows(gget_overview(), index=False, header=False)
    for r_idx, row in enumerate(rows, 7):
        for c_idx, value in enumerate(row, 3):
            ws.cell(row=r_idx, column=c_idx, value=value)        
    wb.save(path)
    print("Excel sheet has been updated.")


log_in()
write_to_sheet()
```


#### My graphical user interface using Excel:
On the left hand side you can see the categories split by months: This (the white area) is where the Python script inserts the calculated sums. For this screenshot I stubbed out the expenses for data protection. The rows below include ones in which I can set a budget, as well as calculate the total and the average for each category. On the right hand side I added a pie chart that displays how big each category's portion of the whole is.

![Here should be an image of the Excel UI...](/excelview.png)


#### The bash script:
```bash
#!/bin/bash
python /path/to/Finances.py
echo 'Opening Excel file...'
xdg-open /path/to/ExcelFile.xlsx 
```


## Lessons learned
### Code
I learned a good deal about Python, the way it is written and the way it works in general. While both are platform independent, Java is statically typed where Python is not (which did leave me with a bunch of errors that were hard to find and grasp at first), but the syntax learning curve is substantially lower with Python, which made it the perfect candidate for a small getting-to-know project like this.

Something I really enjoyed was the power and simplicity of high-level functionalities and datastructures like the ones that come with popular libraries like pandas and numpy. The dataframe structure allowed me to perform my calculations quickly and easily without having to resort to creating my own resource and line heavy solution.

### Usability
It was very rewarding to be able to discover and use a well-documented API, especially since it allowed me to connect front and back end in an extremely low-cost fashion. 

I managed to achieve my objective of creating a custom made solution for myself - surely one that does not work well for a lot of people, as its user interface (especially regarding input) requires a lot of knowledge in the head rather than providing knowledge in the world. While this is usually a bad idea when it comes to Usability and UX Design (since it violates Nielsen's heuristic of [recognition over recall](https://www.nngroup.com/articles/recognition-and-recall/ "Nielsen article on the topic")), it was ideal for the expert-system that I was trying to create for myself.

Over time, I noticed that the approach I chose was too rigid, requiring too much change on the code base itself. In the mean time, I've switched to a different solution that leaves me more room for customization and offers more functionality, without introducing too much clutter. 


## Conclusion
I am glad I took the opportunity of creating this side project, it allowed me to learn a lot and create a product fast without reinventing the wheel. The fact that I am now using a different solution is only natural and in my opinion does not diminish the project's value. 

However, if I were to continue the project, I could imagine building a web client with a simple GUI to record my expenses with a few taps. Perhaps a progressive web app solution could look something like this [interactive Figma prototype I created](https://www.figma.com/proto/hv01ZkOxDHocoVzq5mwnQ7/ExpensesApp?node-id=1%3A549&scaling=scale-down):

![Image of the prototype](/figmaexpenseprotoall.png)