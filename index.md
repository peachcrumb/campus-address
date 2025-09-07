## Overview

BowFolios is an example web application that provides pages to view and (in some cases) modify profiles, projects, and interests. It illustrates various technologies useful to ICS software engineering students, including:

- [Next.js](https://nextjs.org/) for Typescript-based implementation of client and server code.


It also provides code that implements a variety of useful design concepts, including:

- Three primary tables (Profiles, Projects, Interests) as well as three "join" tables (ProfilesInterests, ProfilesProjects, and ProjectsInterests) that implement many-to-many relationships between them.
- Top-level index pages (Profiles, Interests, and Projects) that show how to manipulate these six tables in various ways.
- Initialization code to define default Profiles, Interests, and Projects and relations between them.
- A simple Filter page to illustrate how to perform simple queries on the database and display the results.
- Use of Prisma to illustrate how to simplify implementation of multiple table updates.
- Use of indexes to enforce uniqueness of certain fields in the tables, enabling them to serve as primary keys.
- Authentication using the built-in [NextAuth.js](https://next-auth.js.org/) along with Sign Up and Sign In pages.
- Authorization examples: certain pages are public (Profiles, Projects, Interests), while other pages require login (AddProject, Filter).

## User Guide

This section provides a walkthrough of the Bowfolios user interface and its capabilities.

![](doc/landing-page.png)


## Community Feedback

Bowfolios is based upon [Next.js-application-template-react](https://ics-software-engineering.github.io/Next.js-application-template-react/) and [Next.js-example-form-react](https://ics-software-engineering.github.io/Next.js-example-form-react/). Please use the videos and documentation at those sites to better acquaint yourself with the basic application design and form processing in Bowfolios.

### Data model

As noted above, the Bowfolios data model consists of three "primary" tables (Projects, Profiles, and Interests), as well as three "join" tables (ProfilesProjects, ProfilesInterests, and ProjectsInterests). To understand this design choice, consider the situation where you want to specify the projects associated with a Profile.

Design choice #1: Provide a field in Profile table called "Projects", and fill it with an array of project names. This choice works great when you want to display a Profile and indicate the Projects it's associated with. But what if you want to go the other direction: display a Project and all of the Profiles associated with it? Then you have to do a sequential search through all of the Profiles, then do a sequential search through that array field looking for a match. That's computationally expensive and also just silly.

Design choice #2: Provide a "join" table where each document contains two fields: Profile name and Project name. Each entry indicates that there is a relationship between those two entities. Now, to find all the Projects associated with a Profile, just search this table for all the documents that match the Profile, then extract the Project field. Going the other way is just as easy: to find all the Profiles associated with a Project, just search the table for all documents matching the Project, then extract the Profile field.

Bowfolios implements Design choice #2 to provide pair-wise relations between all three of its primary tables:

![](doc/data-model.png)

The fields in boldface (Email for Profiles, and Name for Projects and Interests) indicate that those fields must have unique values so that they can be used as a primary key for that table. This constraint is enforced in the schema definition associated with that table.

## Initialization

The [config](https://github.com/bowfolios/bowfolios/tree/master/config) directory is intended to hold settings files. The repository contains one file: [config/settings.development.json](https://github.com/bowfolios/bowfolios/blob/master/config/settings.development.json).

This file contains default definitions for Profiles, Projects, and Interests and the relationships between them. Consult the walkthrough video for more details.

The settings.development.json file contains a field called "loadAssetsFile". It is set to false, but if you change it to true, then the data in the file app/private/data.json will also be loaded. The code to do this illustrates how to initialize a system when the initial data exceeds the size limitations for the settings file.

### Quality Assurance

#### ESLint

BowFolios includes a [.eslintrc](https://github.com/bowfolios/bowfolios/blob/master/app/.eslintrc) file to define the coding style adhered to in this application. You can invoke ESLint from the command line as follows:

ESLint should run without generating any errors.

It's significantly easier to do development with ESLint integrated directly into your IDE (such as IntelliJ).

#### End to End Testing

BowFolios uses [TestCafe](https://devexpress.github.io/testcafe/) to provide automated end-to-end testing.

The BowFolios end-to-end test code employs the page object model design pattern. In the [bowfolios tests/ directory](https://github.com/bowfolios/bowfolios/tree/master/app/tests), the file [tests.testcafe.js](https://github.com/bowfolios/bowfolios/blob/master/app/tests/tests.testcafe.js) contains the TestCafe test definitions. The remaining files in the directory contain "page object models" for the various pages in the system (i.e. Home, Landing, Interests, etc.) as well as one component (navbar). This organization makes the test code shorter, easier to understand, and easier to debug.

To run the end-to-end tests in development mode, you must first start up a BowFolios instance by invoking `Next.js npm run start` in one console window.

Then, in another console window, start up the end-to-end tests with:


You will see browser windows appear and disappear as the tests run. If the tests finish successfully, you should see the following in your second console window:


You can also run the testcafe tests in "continuous integration mode". This mode is appropriate when you want to run the tests using a continuous integration service like Jenkins, Semaphore, CircleCI, etc. In this case, it is problematic to already have the server running in a separate console, and you cannot have the browser window appear and disappear.

To run the testcafe tests in continuous integration mode, first ensure that BowFolios is not running in any console.

Then, invoke `Next.js npm run testcafe-ci`. You will not see any windows appear. When the tests finish, the console should look like this:


All the tests pass, but the first test is marked as "unstable". At the time of writing, TestCafe fails the first time it tries to run a test in this mode, but subsequent attempts run normally. To prevent the test run from failing due to this problem with TestCafe, we enable [testcafe quarantine mode](https://devexpress.github.io/testcafe/documentation/guides/basic-guides/run-tests.html#quarantine-mode).

The only impact of quarantine mode should be that the first test is marked as "unstable".

## From mockup to production

Bowfolios is meant to illustrate the use of Next.js for developing an initial proof-of-concept prototype. For a production application, several additional security-related changes must be implemented:

- Use of email-based password specification for users, and/or use of an alternative authentication mechanism.
- Use of https so that passwords are sent in encrypted format.
- Removal of the insecure package, and the addition of Next.js Methods to replace client-side DB updates.

(Note that these changes do not need to be implemented for ICS 314, although they are relatively straightforward to accomplish.)

<!-- ## Walkthrough videos

BowFolios is intended as a model of how an ICS 314 project could be organized and executed. Here are videos that walk you through various aspects of the system:

* [BowFolios Part 1: Application Overview (5 min)](https://www.youtube.com/watch?v=gr55MMWD8ok)
* [BowFolios Part 2: Application Structure and Control Flow (14 min)](https://www.youtube.com/watch?v=LYh06HSYv54)
* [BowFolios Part 3: Data Model, Data Initialization, Publications and Subscriptions (22 min)](https://www.youtube.com/watch?v=2F2Cw5Ipubc)
* [BowFolios Part 4: Forms and Next.js Methods (20 min)](https://www.youtube.com/watch?v=5qim9mXpbTM)
* [BowFolios Part 5: Loading data using Assets (8 min)](https://www.youtube.com/watch?v=NzrTzBPCJPo)
* [BowFolios Part 6: Design Patterns in BowFolios (22 min)](https://www.youtube.com/watch?v=yP-t44HBCPQ)
* [BowFolios Part 7: End-to-End testing in BowFolios (24 min)](https://www.youtube.com/watch?v=B8TSiCLBeaA)
-->

## Example enhancements

There are a number of simple enhancements you can make to the system to become better acquainted with the codebase:

- Display an email icon that links to a mailto: for each user in the profile page.
- Display the home page for each project as a home icon. Click on it to visit the Project's home page.
- Add social media accounts to the profile (facebook, twitter, instagram) and show the associated icon in the Profile.
- The system supports the definition of users with an Admin role, but there are no Admin-specific capabilities. Implement some Admin-specific functions, such as the ability to delete users or add/modify/delete Interests.
- There is no way to edit or delete a project definition. Add this ability.


## Credits

BowFolios template is designed, implemented, and maintained by [Philip Johnson](https://philipmjohnson.org) and [Cam Moore](https://cammoore.github.io/).
