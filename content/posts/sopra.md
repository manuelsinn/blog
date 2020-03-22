---
title: "Creating a serious game that teaches sustainable shopping"
date: 2020-03-22T12:07:28+01:00
draft: false
authors:
    - Manuel Sinn
---

## Idea and Vision
A few brainstorming sessions into this semester's big software team project, the basic idea for **Green Groceries** was created.
As topics like sustainable living and eco-consciousness gain relevance every day, consumers can sometimes be left with the feeling of being lost, especially when it comes to making decisions in their everyday lives that follow their ideals.

As stated in our *vision and scope* document by one of our team mates:
> Often never having been properly taught to shop for “green” groceries, many people struggle with switching to this lifestyle and feel like it’s impossible to eat conscious without restricting themselves to just a few items or spending a fortune.   
> In the age of digitalization and “having an app for everything”, we present our web application **Green Groceries**, which makes learning how to grocery-shop more eco-friendly, fun and easy.  
> Our application features explicit attributes like country of origin and packaging in order to provide exact and directly applicable knowledge for the player.    
**Green Groceries** features a meal-bound shopping trip through a virtual supermarket that provides multiple choices for each shopping item. In a final billing, the player will be shown how his choices in price, country of origin and packaging would influence the environment, earning them a choice-dependent amount of “ecoPoints”, the in-game point system.

Our serious game seeks to educate people about the simplicity of eating sustainably and the general accessibility of eco-friendly grocery choices. In order to do so, it uses concepts like gamification to provide meaningful information in a fun and explorative way.




## Scope and Features

### The web application
This fully fletched out software project includes various documents to accompany the raw code. These include a vision and scope document, a documentation of the architecture and interface design process (featuring various mock ups and schemas), an application documentation for users as well as developers (complete with additional JavaDoc documentation) and a documentation of the software's test process. We even created a video to showcase the finished product.

With dedicated heuristic evaluations performed on both the web application as well as the authoring tool, both are designed to be highly usable and follow Nielsen's ten heuristics for good user interface design.

The game offers meaningful rewards mainly through the use of *ecoPoints* and the *trophy* system. At the beginning of a new run, the player can decide if they want to play to earn a trophy. Different trophies represent various real life circumstances and therefore require different gameplay.  
To give an example: The “Speedy Greenzales” trophy requires the player to get to checkout with all ingredients in a specific amount of time. All trophy achievements are then displayed on the profile page and "unlocked" when the player managed to earn a trophy five times.

Apart from that, the application also enables social interaction: Users can become friends and view each others' progress, as well as send messages back and forth.


### The authoring tool
To provide an easy way to edit the game's contents as well as administer its users, a dedicated *authoring tool* is provided. It is a standalone application that does not require a browser. After logging into the database, authors can choose to change user attributes or alter the game's contents, e.g. through adding new products or dishes to the game.

In order to be able to create new products without having to search for their CO2 values and finding sometimes confusing values, the user can also choose to retrieve the value from the [Klimatarier website](https://www.klimatarier.com/de/CO2_Rechner) with the click of a button.



## Tech Stack & Process
The web application's base follows the model-view-controller pattern and is built with the help of the [Play Framework](https://www.playframework.com). This enabled us to use Scala templates to simplify our HTML files and write our model and controller classes in Java.  
While the server side was basically all Java and SQL (using a MySQL database in the background), the web client's features were implemented with Javascript, while the authoring tool was created with the help of the JavaFX framework.

The authoring tool extension that enables the automatic calculation of CO2 values uses the Java library [Jsoup](https://jsoup.org/) to parse the HTML and get the relevant data from the site.

Using the collaborative web based design tool [Figma](https://www.figma.com/), I created a few of the in game graphics like the trophies or the logo (which was not an original design though).

![Login image](/login.png)



## First drafts to fully working
The initial database architecture:
![Image of old db](/dbOld.png)

DB structure after lots of tweaking and refining:
![Image of new db](/dbNew.png)

An initial mock-up for the checkout page:
![Initial checkout mock up](/checkoutOld.png)

The fully working checkout page:
![Implemented checkout](/checkoutNew.png)





## Lessons learned
### Team Work
This being the first big collaborative software project I was involved in, I am happy about the way the team work played out with good communication on all sides and hardly any issues.   
We used GitLab for version control and its issue boards for efficient task management. Although I was familiar with the basics before the project, now (after over 200 commits for the web application alone) I can say much more confidently that I know my way around git.

### Using different frameworks
A vital part to completing this project were the various frameworks and libraries we used.  
Among others, I learned to use jQuery and jQuery UI to enhance and simplify DOM manipulation, animations and drag and drop functionality, Bootstrap for efficient and good looking styling, JUnit for Java testing and even jasmine to write tests for the javascript code.  
Before, I had only used JUnit and so jasmine's BDD (behaviour driven development) approach was new but interesting to get to know. What I found to be difficult was testing the DOM manipulation efficiently, since the creation of HTML fixtures for every function to be tested was hardly the optimal path. Whenever possible I tried to adhere to coding standards like the AAAA (arrange, act, assert, annihilate - very easy to follow with BDD) or the given-when-then principle for testing.

### Web technologies
Through this project I learned a lot about the way websites work. I learned how to structure, layout and design a page with HTML and CSS and how to make the game we came up with come to life using vanilla Javascript, jQuery and jQuery UI.

I learned how to manage the communication between client and server, defining lots of manual routes from client-side javascript to Java controller classes, which then did the the database accessing through prepared SQL statements. Having to constantly check back with the contents in the database, I got used to writing quick SQL queries to get what I wanted.  
Using the MySQL workbench tool we transferred our data architecture from paper to SQL schemas. Interestingly, after working on all aspects of the project iteratively, we eventually ended up with a database structure that was immensely less complex than our initial prototype.

Perhaps most importantly, I was part of the process of creating a product from start to finish, from rough concept drafts and paper mock ups to a fully working web application with an attractive front end and a fleshed out back end. This realization of an idea we created ourselves was a very fulfilling experience.