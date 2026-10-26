# Pset 3: Web Application (Version 1)

### Due Friday Nov 6 10:59 PM NHT (New Haven Time)

## Table of Contents

- [Pset 3: Web Application (Version 1)](#pset-3-web-application-version-1)
  - [Due Friday Nov 6 10:59 PM NHT (New Haven Time)](#due-friday-nov-6-1059-pm-nht-new-haven-time)
  - [Table of Contents](#table-of-contents)
  - [Purpose](#purpose)
  - [Rules](#rules)
  - [Getting Started](#getting-started)
  - [Your Task](#your-task)
  - [The Database](#the-database)
  - [The Application](#the-application)
    - [Output Requirements](#output-requirements)
    - [Filtering Requirements](#filtering-requirements)
    - [The Secondary Page](#the-secondary-page)
  - [Additional Requirements for 519 Students](#additional-requirements-for-519-students)
    - [Required Additional Feature](#required-additional-feature)
    - [Additional Additional Features](#additional-additional-features)
  - [Specific Endpoints](#specific-endpoints)
  - [Error Handling: Bad Server](#error-handling-bad-server)
  - [Error Handling: Bad Client](#error-handling-bad-client)
    - [Invalid search parameters](#invalid-search-parameters)
    - [Invalid `crn`](#invalid-crn)
    - [Missing `crn`](#missing-crn)
    - [Invalid path](#invalid-path)
    - [Aside: Error Pages](#aside-error-pages)
    - [Other invalid requests](#other-invalid-requests)
  - [Source Code Guide](#source-code-guide)
  - [Submission](#submission)
    - [Late Submissions](#late-submissions)
    - [Grading](#grading)

## Purpose

The purpose of this assignment is to help you learn or review web programming in Flask, including HTML.

---

## Rules

You may work with one teammate on this assignment, and we prefer that you do so.

It must be the case that either you submit all of your team’s files or your teammate submits all of your team’s files.
(It must not be the case that you submit some of your team’s files and your teammate submits some of your team’s files.)
Your `README` file and your source code files must contain your name and your teammate’s name.

---

## Getting Started

> **Note**: this section contains exactly the text on the Canvas assignment, reproduced here only for completeness of this document.
> Since you made it here, you can safely ignore this section.

1. Accept the GitHub Classroom assignment.
   * Select your team.
     * This step creates a GitHub repository for your team and links your team members' GitHub ids.
   * If you do not have a GitHub account, you are required to create one for this course
   * Use this git repository to track your assignment development.

2. The `reg.sqlite` database file is included in your GitHub repository, so no separate download or setup is required.

---

## Your Task

As with Psets 1 and 2, assume you are working for Yale's Registrar's Office.
You are given a database containing data about courses and classes offered during this semester at Yale.
Your task is to compose an application that allows Yale students and other interested parties to query the database.

---

## The Database

The database is identical to the one from Psets 1 and 2.
Refer to the Pset 1 specification for the list of tables and fields in the database, and for the relationships among those tables.
The database is a SQLite database in a file named `reg.sqlite`, included in this template repository.

Department names in this database include the department code, for example `Computer Science (CPSC)`.

---

## The Application

Compose a program named `runserver.py`.
When executed with `-h` as a command-line argument, the program must display the following help message describing the program's behavior:

```text
$ python runserver.py -h
usage: runserver.py [-h] port

The registrar application

positional arguments:
  port        the port at which the server should listen

options:
  -h, --help  show this help message and exit
```

> **Note**: The specific verbiage of this help message should be the default for your version of the `argparse` module, which differs slightly between Python versions.

Your `runserver.py` must run an instance of the Flask test server on the specified port, which must in turn run your application.

> **Note**: Your application code should *not* be in your `runserver.py` program, which should do nothing but start a Flask server on the provided port number.
> The file containing your application code may be named whatever you want, but we suggest something simple such as `regapp.py`.

When a client makes an HTTP request to the server, your application must return an HTML webpage appropriate to the request.
Beyond some [specific endpoints](#specific-endpoints) mentioned below, there are several requirements your application must satisfy:

* Your application's **primary web page** (*i.e.*, the webpage returned by a request to the server's root&mdash;*e.g.* `http://yourserver:80/` if your server is listening on port 80) must contain an **HTML form**.
  * The form must include four text input fields labeled "Department", "Course Number", "Subject", and "Title".
    These should, respectively, allow the user to specify a department name, a course number, a subject, and a course title.

    > **Note**: The value for "Department" must be used to filter courses by **department name** (*i.e.*, the `deptname` column in the `departments` table).

  * In addition to the text input fields, your form must include a `submit` input field.
    For this assignment, the submit field must be a button labeled "Search".
  * The form may contain other input elements, but any additional input elements you include in the form must be of the `hidden` type (that is, they must not be visible to the user when they view the webpage).
    * Any non-input elements that enhance the look and feel of the webpage are encouraged and of course do not have to be hidden
  * Whenever this page is loaded, the form input fields must be filled in with the parameters used to make the most recent query, that is, the values of the input fields as they were upon the most recent submission of the form (if the form has never been submitted, those fields should be empty)
* After the user clicks the "Search" button, the webpage must display an HTML `table` containing no more than the first 1000 results of the query that was triggered by the most recent submission of the form (or nothing at all if there is no most recent submission).
  * If the user entered no text in the form elements, the webpage should display a message such as "No search terms provided. Please enter some search terms."

    > **Note**: This is a change in requirements from previous psets, in which an empty search returned all classes in the database.

### Output Requirements

The columns displayed in the table must be, in order, the `crn`, `deptname`, `subjectcode`, `coursenum`, and `title` of each **section** that matches the specified criteria.
The columns must be labeled "CRN", "Department Name", "Subject Code", "Course Number", and "Title", respectively.

There is one table row per section. Repeat `deptname`, `subjectcode`, `coursenum`, and `title` on each row for that section.

The table rows must be sorted. The primary sort must be by `deptname` in ascending order, the secondary sort must be by `coursenum` in ascending order, and the tertiary sort must be by `crn` in ascending order.

A user must be able to click on the `crn` for any displayed section to request more information about that section on a different webpage (the secondary page) at `/details?crn={crn}`.
The requirements for the secondary webpage are below.

> **Note**: The course's internal `courseid` must not be displayed in the table.

For example, a search with Department `Computer Science (CPSC)` and Course Number `4190` includes a row such as:

```text
10859    Computer Science (CPSC)    CPSC    4190    Full Stack Web Programming
```

A search that matches every section of CHEM 6000 includes one row per section:

```text
13753    Chemistry (CHEM)    CHEM    6000    Research Seminar
13754    Chemistry (CHEM)    CHEM    6000    Research Seminar
```

### Filtering Requirements

| Field | Meaning |
| --- | --- |
| Department | Include sections whose `deptname` contains the supplied value. |
| Course Number | Include sections whose `coursenum` contains the supplied value. |
| Subject | Include sections whose `subjectcode` contains the supplied value. |
| Title | Include sections whose `title` contains the supplied value. |

If several fields are non-empty, combine the filters using `AND`.
Filters must be case-insensitive: a Title field containing `web` must match a title such as "Full Stack Web Programming".
Filters must preserve leading and trailing whitespace.

### The Secondary Page

Your application must accept requests to `/details?crn={crn}`.
The webpage returned by this endpoint must satisfy the following requirements:

* It must display at least the following information about the selected section:
  * `deptname`, `subjectcode`, and `coursenum` in a single-row HTML table.
    These columns must be labeled "Department Name", "Subject Code", and "Course Number", respectively.
  * `title`, `descrip`, and `prereqs`, each in its own section.
    These sections must be labeled "Title", "Description", and "Prerequisites", respectively.
    Each section may be formatted as an HTML table, but does not have to be.
    If there is no description or no prerequisites, display the string `None`.
  * `sectionnumber`, `crn`, and `meetinginfo` in a single-row HTML table for the selected section.
    These columns must be labeled "Section Number", "CRN", and "Meetings", respectively.
    Each meeting must be formatted as the meeting time, followed by ` @ `, followed by the meeting location, for example `MW 9.00-10.15 @ WTS A74`.
    Put one meeting on each line.
  * Crosslistings (`subjectcode` and `coursenum`) in an HTML table.
    These columns must be labeled "Subject Code" and "Course Number", respectively.
  * Professors (`profname`s) in a single-column HTML table.
    The column must be labeled "Professors".

> **Note**: The description and prerequisites strings for some courses contain HTML.
> That HTML must be rendered according to the included elements.

* Its information must be well-formatted (*e.g.*, the headers must be within semantic HTML `<h`*`N`*`>` header elements, lists must be within HTML `<ul>` elements, and tables should use `<thead>` and `<tbody>` elements to distinguish between header rows and body rows)
* It must provide a link back to the primary webpage to allow the user to perform another search.
  The resulting instance of the primary webpage must contain exactly the same content as was most recently displayed on the page before the user clicked a `crn` to retrieve the secondary webpage, including both the results table and filled-in input fields.

  > **Note**: Your application must be *stateful* to accomplish this, and in particular this pset requires the use of **cookies** for state-tracking.
  > Review the lecture slides and demo code on how to create a stateful web application using cookies.

---

## Additional Requirements for 519 Students

If you *or your partner* are enrolled in CPSC 519, your application must satisfy all of the above requirements and you must implement some of the following features.

If at least one member of your team is enrolled in CPSC 519, you are required to implement editing functionality (the [Required Additional Feature](#required-additional-feature)).
If both members of your team are enrolled in CPSC 519, you are *also* required to implement at least one of the [Additional Additional Features](#additional-additional-features).

Even if you and your partner are in 419 and not 519, you are more than welcome to attempt these activities.
They will not have any bearing on your grade for this assignment, but they will provide valuable experience in adding features to a webpage that may prove useful in developing your final project.
(And at least one of them is a required feature for Pset 4!)

### Required Additional Feature

If you *or your partner* are enrolled in CPSC 519, you must implement the following additional feature.

1. On the secondary webpage, add a button or a link labeled "Edit" associated with each of the "Title", "Description", and "Prerequisites" sections.
   Upon clicking this button, the user must be presented with a webpage containing a form with a `textarea` element (see [this webpage](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea) for details) initially populated with the current value of that field for the course, and a submit button labeled "Submit".
   When the user submits this form, it must send an HTTP `POST` request to your server that will change the value of that field for this course in the database to match the new text that the user entered in the text area, and the user should be presented with the secondary webpage showing the course's new value (and, of course, the other required information about the section).
   The server must accept this `POST` request at the endpoint `/details`.

> **Note**: This is the same endpoint as the secondary webpage; the only difference is the HTTP method associated with the request: in this case, that method is `POST`; in the other, it is `GET`.

### Additional Additional Features

If you *and your partner* are both enrolled in CPSC 519, you must implement at least one of the following additional features.

1. The default styling of an HTML table is quite ugly in most browsers.
   Use CSS to spruce up your tables:
   * Add padding of `5px` to all sides of every cell in each table
   * Add a `1px` wide `gray` border between rows of each table
   * When the mouse hovers over a particular row of each table, change that row's background color to `lightgray`
   * You are free to style your application in any other manner that looks good to you (but nothing else is required)
   * Place your CSS into a file named `styles.css` that is loaded by your primary and secondary webpages, but not included directly in those pages
2. On the primary page, add a button or link labeled "Clear Previous Search" that, when clicked, displays the primary page with empty input fields and no table of results.
   It must be the case that clicking the button or link causes any stored state to be reset to an initial or empty state, including any state local to the browser (such as cookies) and any state stored on the server.
   The endpoint for this feature must be `/reset`.

---

## Specific Endpoints

There are three endpoints to which your application must respond:

1. `/` must return the primary page, with the input fields filled in having the values of the most recent search query
   * If the user has previously made a search query, the table containing the results of the most recent query must be displayed on this page
2. `/search?...` must return the results of a search using the parameters in the query string (that is, those that the user entered on the primary page).
   * The parameters accepted by the `/search` endpoint must be `d` (for the department name), `n` (for the course number), `s` (for the subject code), and `t` (for the title)
3. `/details?crn=...` must return the secondary page populated with information about the section with the `crn` provided in the query string.

Students completing the [additional 519 activities](#additional-requirements-for-519-students) must add endpoints to accomplish some of those tasks, and the specifics are indicated above.

---

## Error Handling: Bad Server

Since you control the machine that is running both the server application and the database, there is not much we will force you to worry about for server-side errors.
There are, however, two errors that your program must handle gracefully:

1. The server is started with a port that is not a positive integer.
   If the server is started with a command such as `$ python runserver.py notaport`, it should display a meaningful error message and exit.
2. The database file does not exist.
   If the server attempted to open the `reg.sqlite` file but the file does not exist (or otherwise cannot be opened), your program should display a meaningful error message and exit.

---

## Error Handling: Bad Client

It is an unfortunate truth about web applications of this kind that a user can enter anything they want in their request to your server!
For example, anyone can send a request to a URL such as `http://yourserver:80/details?crn=gobbeldygook`, despite there being no class with crn "gobbeldygook" on which they could have clicked.
This means that starting with this assignment, we will be ramping up the error handling requirements on your software.
Specifically...

### Invalid search parameters

If the user requests a page at a URL such as `http://yourserver:80/search?foo=bar`&mdash;in which some parameters to a search request are not one of `d`, `n`, `s`, or `t`&mdash;your application must treat those parameters as not existing.
For example, in response to a request to the URL `http://yourserver:80/search?foo=bar&d=computer`, your server must produce a webpage containing results of a search for sections whose department name contains `computer`.

### Invalid `crn`

If the user requests a page at a URL such as `http://yourserver:80/details?crn=gobbeldygook`&mdash;in which the crn value does not appear in the database&mdash;the server must return an appropriate error page as HTML that displays, at a minimum, text that reads "Error: no class with crn gobbeldygook exists". (The "gobbeldygook" part should, of course, be replaced with the actual nonexistent crn that was queried.)
This page must be returned with status code 404 (not found).

Your application must respond with this error page for *any* missing crn, including numeric and non-numeric crns.

### Missing `crn`

If the user requests a page at a URL such as `http://yourserver:80/details` (that is, missing a `crn` query parameter), the application must return an appropriate error page as HTML that displays, at a minimum, text that reads "Error: missing crn in details request" and status code 404 (not found).

### Invalid path

If the user requests a page at a URL such as `http://yourserver:80/notapath`&mdash;in which the path is not a path your server is programmed to respond to&mdash;the server must return an error page as HTML that displays, at a minimum, text that reads "Error: invalid URL".
This page must be returned with status code 404 (not found).

### Aside: Error Pages

Your error page must have some special content, but it should not live at a special URL.
That is, your application should behave in a similar manner as Google when an invalid page is requested, such as `google.com/notreal`.
The page displayed should clearly indicate an error occurred, but the URL in the browser should remain `google.com/notreal`: it *should not* be some other URL, such as `google.com/404`.

### Other invalid requests

There are many other requests that the user could send that are "wrong", such as:

* `http://yourserver:notyourport/`
* `http://yourserver:80/search?not/well/formed/url`

In these cases, the request will not even reach your server application, so there is nothing you need to do (or, indeed, even *could* do) in response.

---

## Source Code Guide

Here are the **requirements** for the source code of your solution.

* The `runserver.py` program must start a Flask server for your application on the port provided as a command-line argument, listening on all IP addresses
* Your application program must communicate with a SQLite database in a file named `reg.sqlite`, organized as described above.
* Your application program must use SQL prepared statements for every database query.
  (This protects the database against SQL injection attacks.)
* Use cookies to keep track of the application's state
* Any display of user-entered content (such as search terms or an invalid URL) on your webpage must be **escaped** before display. This prevents XSS (cross-site-scripting) attacks
* Reuse code from your solution to Pset 2 in this assignment.
* Modularize your application program so that database communication code is cleanly separated from response production code.
  * Use HTML templates to keep your response production code as clear as possible
  * Structure your code according to MVC design principles (the HTML templates are your Views)
* You may use external dependencies, but we advise you not to (with the obvious exception of `flask`).
  This assignment is designed such that everything can be accomplished without too much pain using only packages from the Python standard library.

---

## Submission

Rename this file `TEMPLATE_README.md` and replace it with a new `README.md` file.
Your new `README` file must contain:

* Your name and netid and your teammate's name and netid, at the beginning of the file
* A paragraph describing your contribution, and another paragraph describing your teammate's contribution
  * Please be thorough; we are looking for two substantial paragraphs, not a sentence or two
* A description of whatever help (if any) you received from other people while doing the assignment
* A description of the sources of information that you used while doing the assignment, that are not direct help from other people
* An indication of how much time you spent doing the assignment, rounded to the nearest hour
* Your assessment of the assignment:
  * Did it help you to learn?
  * What did it help you to learn?
  * Do you have any suggestions for improvement? *Etc.*
* (Optionally) Any information that will help us to grade your work in the most favorable light
  * In particular, describe all known bugs and explain why any pylint style warnings you received are unavoidable or why you know better than pylint (a convincing argument may negate some pylint style penalties you accrue)

Your README file must be a plain text file: don't create it using Microsoft Word or any other word processor, although you are encouraged to format it using [markdown](https://www.markdownguide.org/) tags.

Package your assignment files by [creating a release](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository#creating-a-release) on GitHub in your assignment repository.
There must be at least the following files with the following (exact) names in that repository when you submit it:

* `README.md`
* `runserver.py`

Ensure that any additional files needed by your program (such as other Python modules or HTML files) are in the repository snapshot captured by the release.

> **Note**: If you have installed external packages beyond flask, you must also include a file named `requirements.txt` containing the dependencies of your project.
> It can be created from your virtual environment by running the following command:
>
> ```text
> $ pip freeze -r requirements.txt
> ```
>
> Failure to include a `requirements.txt` file if you use third-party packages will result in an automatic 5% penalty and a request that you submit an appropriate `requirements.txt` file to the graders.

Submit your solution to Canvas (in the assignment named "Pset 3: Web Version 1") as [a link to that release](https://docs.github.com/en/repositories/releasing-projects-on-github/linking-to-releases).

As noted above in the [Rules](#rules) section, it must be the case that either you submit all of your team’s files or your teammate submits all of your team’s files.
(It must not be the case that you submit some of your team’s files and your teammate submits some of your team’s files.)
You and your team may submit multiple times; we will grade the latest files that you submit before the deadline unless a particular version is requested as the canonical version.

**Please follow the directions on what to submit and how.**
It will be a big help to us if you get the filenames right and submit exactly what’s asked for.
Thanks.

### Late Submissions

The deadline for this assignment is **10:59 PM NHT (New Haven Time) on Friday Nov 6**.
There is a strict 1-hour grace period beyond the deadline, to be used in case of technical or administrative difficulties, and not for putting final touches on your solution.

Late submissions will receive a 5% deduction for every 12-hour period (or part thereof) after the deadline.
After 48 hours, the Canvas assignment will close and submissions after that time will not receive any credit.

Except for submissions after the 48-hour deadline (*which are not accepted*), the timestamp on the commit associated with the linked release will determine what late penalties, if any, are applied.

---

### Grading

Your grade will be based upon:

* Correctness, that is, how closely your programs conform to the specifications in this document.
* Style, that is, the quality of your program style. This includes not only style as qualitatively assessed by the graders (including modularity, cleanliness, and algorithmic efficiency) but also style as reported by the pylint tool, using the default settings, and when executed via the command `python -m pylint **/*.py`.
  * We will ignore any of the starter code that you were given (such as `table.py` from psets 1/2) in this assessment.
  * Any pylint warnings should be written about (briefly) in your submitted README document. If you believe that your solution is the right one despite pylint warning you about it, you should make such an argument and if we accept it the style penalty will be waived for that warning

If your code fails the tests on some particular functionality, your grader will inspect your code manually to try to assign partial credit for that functionality.
Partial credit will be given only if there is an *obvious* "quick fix" (*e.g.*, you have accidentally changed the name of the database file and your solution points to a file with a name that does not match the grader's copy of the database); if no such quick fix exists then no partial credit for that feature will be given.

---

Adapted from Assignment 3 for COS 333 &copy; 2021 by Robert M. Dondero, Jr., Princeton University

This version &copy; 2026 by Alan Weide, Yale University
