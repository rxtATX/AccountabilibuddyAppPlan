# Accountabilibuddy App Plan

## Table of Contents

- [Initialize App](#initialize-app)
- [Root Level](#root-level)
- [Controllers](#controllers)
- [Routes](#routes)
- [Views](#views)
- [Models](#models)
- [Public](#public)
- [Utils](#utils)
- [Tests](#tests)
- [Deployment](#deployment)

## Initialize App

Create a new GitHub repo, invite your collaborators, upload the initial files to the main branch, and afterwards, protect the main branch.

When you create your GitHub repo, initialize with a .gitignore file. Select the Node option.

### Initial files should include the following structure:

```
Application
├── controllers
│ ├── handlebarsControllers.js
│ ├── taskControllers.js
│ ├── userControllers.js
├── routes
│ ├── index.js
│ ├── handlebarsRoutes.js
│ ├── apiRoutes
│ │ ├── index.js
│ │ ├── taskRoutes.js
│ │ ├── userRoutes.js
├── views
| ├── home.handlebars
| ├── login.handlebars
| ├── signup.handlebars
| ├── layouts
│ │ ├── main.handlebars
│ │ ├── login.handlebars
| ├── partials
│ │ ├── taskForm.handlebars
│ │ ├── buddyForm.handlebars
│ │ ├── taskCard.handlebars
├── models
| ├── schema.sql
| ├── index.js
| ├── User.js
| ├── Task.js
├── public
│ ├── css
│ │ ├── styles.css
│ ├── js
│ │ ├── login.js
│ │ ├── home.js
│ │ ├── addBuddy.js
│ │ ├── addTask.js
├── node_modules
├── utils
│ ├── connection.js
│ ├── seeds.js
│ ├── helpers.js
│ ├── withAuth.js / middleware.js
├── .env
├── server.js
├── package.json
├── package-lock.json
└── .gitignore
```

## Root Level

  * <b>.gitignore</b> - Can be initialized automatically when creating the repository. If you missed that step when creating the repo, you may copy and use the content of this file: [Click Here](https://github.com/github/gitignore/blob/main/Node.gitignore "A collection of useful .gitignore templates (NodeJS).").

    Add .DS_Store for convenience.

  * <b>.env</b> - Will hold variables representing your DB_NAME, DB_USER, and DB_PASS.

  * <b>package.json</b> - Use `npm init` to generate, with an option `-y` flag. 
    
    Install your dependencies:
    ```
    express
    express-handlebars
    sequelize
    dotenv
    connect-session-sequelize
    bcrypt
    mysql2
    express-session
    node-cron
    nodemon (dev-only)
    ```

    Decide with the team between:
    
      * [twilio SMS](https://www.twilio.com/docs/messaging/quickstart/node)
      * [twilio sendGrid](https://docs.sendgrid.com/for-developers/sending-email/api-getting-started)
      * [Nodemailer](https://nodemailer.com/about/)
      * [emailJS](https://nodemailer.com/about/)
      * Or research your own options for contacting the Accountabilibuddy.
          
          * Just make sure your selection allows for the Buddy to opt out of communications with your app by responding to your notification.
          * Their response will make a DELETE to '/api/user/buddy'
    
    For convinience, modify the package.json file to include useful scripts:
    ```
    "start": "node server.js",
    "watch": "nodemon server.js",
    "seed": "node utils/seeds.js"
    ```

  * <b>server.js</b> - Use the server.js boilerplate structure which will allow for authentication sessions with `express-session-sequelize` package.

## Controllers

  * <b>taskControllers</b>

    * <b>postTask</b> - Has access to request body. Use Sequelize to query to the Task model to add a new Task using all the request body data, and the TastKeeper is the authenticated user. Response should redirect to '/'.

    * <b>putTask</b> - Has access to request body and parameter `id`. Use Sequelize to query to the Task model to modify the Task data related to the id, to reflect the changes indicted by the request body. If Task isTodo property is changing from true to false and the alertEach property is true, this controller should dispatch communication to alert the Buddy. At this point, check if every Task's isTodo property is false, and if alert_all User property is true, this controller should dispatch communication to alert the Buddy. Response should redirect to '/'.

    * <b>deleteTask</b> - Has access to request parameter `id`. Use Sequelize to query to the Task model to delete the Task data related to the id. Response should redirect to '/'.

  * <b>userControllers</b>

    * <b>postUserLogin</b> - Has access to request body. Use Sequelize to query to the User model by email value. If no email is found, do not allow the user to authenticate. If the email is found, verify the password supplied is valid. If not, do not allow the user to authenticate. Once email and password is confirmed, authenticate the user by adding to the session object. Response should redirect to home page.

    * <b>postUserSignUp</b> - Has access to request body. Use Sequelize to query to the User model to add a new User using all the request body data; ensure to hash the password before storage. Response should redirect to home page.

    * <b>putUserAddBuddy</b> - Has access to request body. Use Sequelize to query to the User model to modify the User data related to the currently authenticated user, to reflect the changes indicated by the request body. Use the new Buddy's contact information and the chosen communication API to inform the Buddy they've been added to your contact list. Response should redirect to '/'.

    * <b>putUserRemoveBuddy</b> - Has access to request body. Use Sequelize to query to the User model to modify the User data related to the request body properties, to remove Accountabilibuddy's contact details. Response should redirect to '/'.

  * <b>handlebarsControllers</b>

    * <b>homeController</b> - Use Sequelize to query to the User model to read the User data related to the currently authenticated user, include their associated Tasks. Response should render home.handlebars. This controller should also check for query parameters matching `buddyModal=true` or `taskModal=true`, where either property will be added to the context object.

    * <b>loginController</b> - Response should render login.handlebars.

    * <b>signupController</b> - Response should render signup.handlebars.

## Routes

  #### API Routes

  * <b>taskRoutes</b>

    * <b>POST /api/task</b> - Has a request body representing Task-shaped form data sent by client. Should return redirect to '/'.

    * <b>PUT /api/task/:id</b> - Has a request body representing Task-shaped form data sent by client. Has path parameter `id` representing which task to modify. Should return redirect to '/'.

    * <b>DELETE /api/task/:id</b> - Has path parameter `id` representing which task to delete. Should return redirect to '/'.

  * <b>userRoutes</b>

    * <b>POST /api/user/login</b> - Has a request body representing existing User-shaped form data sent by client. Should return redirect to home page.

    * <b>POST /api/user/signup</b> - Has a request body representing new User-shaped form data sent by client. Should return redirect to home page.

    * <b>PUT /api/user/buddy</b> - Has a request body representing User-shaped form data sent by client. Should use the session object to identify which User to modify. Response should redirect to '/'.

    * <b>DELETE /api/user/buddy</b> - Should use the session object to identify which User to modify. Response should redirect to '/'.

  #### Handlebars Routes

  * <b>main.handlebars Layout</b>
 
    * <b>GET /</b> - Should respond with home.handlebars embedded into the main.handlebars layout. Query parameters of `buddyModal=true` or `taskModal=true` will be accepted by this route. 

  * <b>login.handlebars Layout</b>

    * <b>GET /login</b> - Should respond with login.handlebars embedded into the login.handlebars layout.

    * <b>GET /signup</b> - Should respond with signup.handlebars embedded into the login.handlebars layout.

## Views

  ### Layouts

  * <b>main.handlebars</b> - Should link to script.js and style.css. Render the body as a variable.

  * <b>login.handlebars</b> - Should link to login.css. Render the body as a variable.
  
  ### Views

  * <b>home.handlebars</b> - This view will be rendered with the User and related Tasks context. The view will need to conditionally render several elements on the page.

    1. There should always be a header element greeting the User by name.
    
    2. When there is no Accountabilibudddy contact information on the User object, the view should display the addBuddy partial.
   
    3. When the contact information exists, the header element should indicate to the User that the Buddy, by name, believes in them.
   
    4. If the route matches query parameters of `buddyModal=true` or `taskModal=true`, should conditionally render the matching partial.
  
    5. When a Buddy has been added, but no Tasks are sent along in the context object, the view should display the addTask partial.
  
    6. When Tasks exist, if the isTodo property is true, the Task should be displayed in the To Do Tasks section of the page. Otherwise, it should be presented in the Completed Tasks section at the bottom of the page.
 
    7. When Tasks exist, the Add Task button will be removed from the center of the page, but displayed floating in the bottom right corner instead.

  * <b>login.handlebars</b> - Should link to login.js. This view will be rendered with no special context object. The view will render a form element on the page. The form will include inputs for User email and User password, as well as a submit button.

  * <b>signup.handlebars</b> - Should link to signup.js. This view will be rendered with no special context object. The view will render a form element on the page. The form will include inputs for User name, User email and User password, as well as a submit button.

  ### Partials

  * <b>addBuddy.handlebars</b> - Should link to addBuddy.js. This partial will be rendered with { buddyModal: true/false } context object. The view will render a prompt to either add a Buddy or hold themselves accountable. These buttons should change the URL query parameter to `buddyModal=true`. The partial should then hide the button prompt, and instead display a form element on the page. The form will include inputs for Buddy name, Buddy email and Buddy phone number, as well as a submit button.

  * <b>addTask.handlebars</b> - Should link to addTask.js. This partial will be rendered with no special context object. The view will render a prompt to Add Tasks. This button should change the URL query parameter to `taskModal=true`. The partial should then hide the button prompt, and instead display a form element on the page. The form will include inputs for Task name, Task category, Task repetition, and Task points value, as well as a submit button.

  Consider creating additional partials. The application may benefit from breaking down the larger views into smaller, maintainable parts.

## Models

  * <b>index.js</b> - This file should import User and Task models, define their relationships, and export them. User should have many Task, Task should belong to User.

  * <b>User</b> - Refer to ERD to view type definitions for id, email, password, name, buddy_name, buddy_email, and buddy_phone. This model should use beforeCreate and beforeUpdate hooks to encrypt User password before save. This model should have an instance method used to verifyPassword.

  * <b>Task</b> - Refer to ERD to view type definitions for id, content, isTodo, points, repetition, category, and task_keeper. This model should use afterCreate hook to set up a Cron Job to toggle isTodo=false property back to isTodo=true when the repetition time has expired.

## Public

  ### CSS

  * <b>style.css</b> - This will be the stylesheet for the home page. 
  
    * Consider breaking into multiple stylesheets for each partial.

  * <b>login.css</b> - This will be the stylesheet for the login and signup pages.

  ### JavaScript

  * <b>script.js</b> - This script should capture clicks on each Task so that when the user toggles a task from To Do to Done, put to `/api/task/${taskId}` with body content `{ isTodo: false }`.

  * <b>login.js</b> - This script should capture login form data and post it to `/api/user/login` endpoint for authentication. On successful server response, redirect to home page.

  * <b>signup.js</b> - This script should capture signup form data and post it to `/api/user/signup` endpoint for authentication. On successful server response, redirect to home page.

  * <b>addBuddy.js</b> - This script should capture Add Buddy form data and put it to `/api/user/buddy` endpoint to add buddy properties on User object. On successful server response, redirect to same page without query parameter string.

  * <b>addTask.js</b> - This script should capture Add Task form data and post it to `/api/task`. On successful server response, redirect to same page without query parameter string.

## Utils

  * <b>Cron Job</b> - This helper function will be attached to each Task based on its repetition property. When this function's timing has been met, it should modify its associated Task as `{isTodo: true}`.

## Tests

  Consider adding tests.

## Deployment

  Provision a JawsDB Resource and deploy to Heroku.