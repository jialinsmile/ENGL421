# ENGL421
Research Problem: What is the effects of transparency on innovation productivity in open hacking competition? 
Relevance: The role of transparency in technical communication, and how it affected the quality of the end products came up by developers. 

# Novel Virtual Social Coding Environment developed by Jia 
This document contains technical details about how the new technology, which is the social coding platform developed by Jia works.

### Dependencies

The app is powered by [Bloggify](https://bloggify.org)—a modular and flexible platform for building modular applications. Since Bloggify is a Node.js platform, you have to install Node.js on your machine. The data is stored in a MongoDB database.

### Installation

To run this application, you will have to have Node.js and MongoDB installed on your machine.

 - [Node.js](https://nodejs.org/en/)
 - [MongoDB](https://www.mongodb.com/download-center?jmp=nav#community)

Then you can proceed to the next steps:

 1. Clone the repo either from GitHub or Heroku:

  ```sh
  git clone https://github.com/jialinsmile/ENGL421.git
  ```

  or Heroku (this is not recommended, but it will still work as long you have access to the project):

  ```sh
  git clone https://git.heroku.com/ENGL421.git courseswebpage
  ```

  **Note**: This is not recommended because the code pushed to Heroku is a little
  bit different than the version we push to GitHub. When deploying the app on
  Heroku, we bundle the files **before** pushing them, therefore the Heroku
  version will store the production bundles, while the GitHub version doesn't
  store any bundles at all.

  There are two reasons why we bundle the files before deployment:

   1. Heroku is going to complain if we do it on their servers because it's a
      very resource-consuming process (it's using RAM memory, eventually
      exceeding it).

      Doing it on our machines is much better because we have enough RAM.

   2. The boot time of the app will be much faster (because no bundles have to
      be generated on Heroku).

 2. Open the folder:

  ```sh
  cd ENGL421
  ```

 3. Install the dependencies:

  ```sh
  npm install
  ```

Start MongoDB:

```sh
mongod
# or
sudo mongod
```


#### Set up the enviroment

Before starting the app, you will have to create a file named `.env`, containing:

```env
GITHUB_CLIENT=...
GITHUB_SECRET=...
SENDGRID_KEY=...
MONGODB_URI=mongodb://localhost/ENGL421
```

You can get the GitHub keys after creating a GitHub application. Do not share these with anyone.

#### Starting the application

Start the app in dev mode:

```sh
npm run start:dev
```

Make yourself an admin, by passing YOUR GitHub username (the username of the
account you use to sign in for the first time):

```sh
ADMIN_USERNAME=<your-github-username> npm run start:dev
```

For example:

```sh
ADMIN_USERNAME=jialinsmile npm run start:dev
```

**Note**: The very first start takes up to 30 seconds because there is no
existing cache. After the cache is created, the next application starts will be
much faster (1-3 seconds).


### Deployment

When deployed to Heroku, the application url is `https://<app-name>.herokuapp.com` (unless it's using a custom domain).
**Note:** When using a free dyno, it's working fine, but with some limitations:

 - it's slower
 - it's going to sleep if it's innactive for a certain period of time.
 - it has bandwidth limits, but pretty liberal

The app configuration is stored in the `bloggify.js` file.

 1. Make sure that the `heroku` remote exists (run `git remote -v` for that). If it doesn't exist, run:

    ```sh
    heroku git:remote ironhackplatform
    ```

 2. Commit all the changes and then run the following command:

    ```sh
    npm run deploy
    ```


### Forum structure

There is two groups with 20 students in each group. 

The posts and discussions from one forum are *not* visible to the users from the other forums.


### Workflow and functionality

Being a Bloggify application, the application configuration is kept in a file: `bloggify.js`. This contains (see the inline comments):

```js
"use strict";

const conf = require("bloggify-config");

// Set the right MongoDB URI (depending on the environment).
const DB_URI = process.env.MONGODB_URI
if (!DB_URI) {
    console.error(">>>> Please provide the MongoDB URI. Set the MONGODB_URI environment variable.");
}

module.exports = conf({
    // Application metadata
    title: "IronHacks"
  , description: "Hack for inovation and join the open data movement."

    // The production domain
  , domain: "http://www.ironhacks.com"

    // Core plugins (which are initialized before the others)
  , corePlugins: [
        "bloggify-mongoose"
    ]

    // Application plugins
  , plugins: [
        "bloggify-sendgrid"
      , "bloggify-custom-assets"
      , "bloggify-github-login"
    ]

    // The application router
  , router: "bloggify-flexible-router"

    // We do not have a blog page, so we do not need a Bloggify viewer at all
  , viewer: null

    // Plugins configuration
  , config: {

        // Custom application assets
        "bloggify-custom-assets": {
            styles: [
                "app/assets/stylesheets/index.css"
            ]
          , server: [
                "app/server/index.js"
            ]
        }

        // The application router
      , "bloggify-flexible-router": {
            controllers_dir: "app/controllers"
          , routes_dir: "app/routes"
          , error_pages: {
                404: "404.ajs"
              , 500: "500.ajs"
              , bad_csrf: "422.ajs"
            }
        }

        // Login with GitHub
      , "bloggify-github-login": {
            githubClient: process.env.GITHUB_CLIENT
          , githubSecret: process.env.GITHUB_SECRET
        }

        // Connect to the MongoDB database
      , "bloggify-mongoose": {
            db: DB_URI
          , models_dir: "app/models"
        }

        // Send emails
      , "bloggify-sendgrid": {
            key: process.env.SENDGRID_KEY
        }
    }
}, {
    cms_methods: false
  , server: {
        session: {
            storeOptions: {
                url: DB_URI
            }
        }
    }
});
```

The way how this Bloggify application is structured is explained below.

The `app` directory contains the application files

The application routes (urls) are:

```sh
GET         /
GET         /404
GET         /500
GET/POST    /admin
GET         /countdown
GET/POST    /logout
GET/POST    /new
GET/POST    /register
GET         /login
GET         /scores
GET         /search
GET         /quizzes
GET/POST    /posts/topicId-_slug/
POST        /posts/topicId-_slug/comments
POST        /posts/topicId-_slug/delete
GET/POST    /posts/topicId-_slug/edit
POST        /posts/topicId-_slug/toggle-vote
GET/POST    /users/_user/edit
GET         /users/_user
```

The `GET` method means that we fetch information from the server, while the `POST` means we post information to the server side.

The routes may have associated controllers which are located in the `app/controllers` directory.

#### Qualtrics integration

All the quizzes created on the Qualtrics platform have a snippet of JavaScript which stores the user data in the response (as known as *Embedded data*).

We store the `user_email` and the `user_id` which are sent in the url.


The code in the Qualtrics quizzes is set in the last block, last question (which may happen to be an empty question, used for tracking):

TODO

This approach is being used for all the quizzes: the sign up survey and the other technical quizzes.

#### Login / Register Process

When the user is not authenticated, the main page displays two buttons:
a login button and a <kbd>Sign up</kbd> button.

When the login button is clicked, the `/login` url is opened.
The login url redirects to the GitHub authentication workflow (OAuth2) where the user should accept access of our GitHub app in their GitHub account.


```
IronHack Platform                             GitHub.com
-----------------                             ----------
/login  ----------------------------------------> Do you want to accept access of the app?
                                            (the message won't appear if accept was already granted)
                                                    |
                                                    *------------ GitHub App: IronHack
                                                    |
                                        ___________/ \___________
                                       /                         \
                                     Yes                          No
/login-callback  <-------------------/                            |
/                <------------------------------------------------/
```

The difference between the <kbd>Sign in</kbd> and <kbd>Sign up</kbd> buttons is that for the sign in,
we assume that the user already has an account, therefore we make the redirection
to GitHub APIs and authenticate them with their GitHub account. In case they do
*not* have an account on the site, they will have to choose the hack type *after*
the GitHub authentication.

In <kbd>Sign up</kbd> case we will assume that the user doesn't have an account yet, therefore
we ask them to choose the hack type and only then the GitHub authentication is
made. Still, if an existing user clicks the <kbd>Sign up</kbd> button, they will log in
into their account anyways.

This is the authentication workflow:

![](http://i.imgur.com/b0RRhc6.png)

In the `controllers/register.js` file, we create a new user or authenticate an existing user.
For the first-time users, we display the selection of the hack type: Purdue, Bogota and Platzi, and then redirect to the survey.
At this moment, we start a session on the server side, but we don't write any data in the users' database yet.

After the user completes the survey, they are redirected back, and the user account is created.

To redirect the users to the survey, we send the `Location` header from the server to the client. Additional data is added in the url parameters to track the user:

 - `email`: The email of the user
 - `user_id`: The user id
 - `redirect_to`: The redirection url where they will arrive after answering the survey.

Using a JavaScript snippet in the survey page we store the email and the user id in the survey answer and we detect when the survey is done and redirect the users back to the app.

When they finish the survey, they are redirected to the main app and the account is created.

When creating a user account, we do *not* assign a `hack_id` to the user but we wait until the contest is started.
The `hack_id` is a number between `0` and `2` based on which we create multiple forums inside of the same hack type.
The range can be configured in the admin dashboard.

For Purdue, we have one forum (the `hack_id` will be always `0`) and for `Bogota` and `Platzi`, we have three forums for each (the `hack_id` can be `0`, `1` or `2`).

```
+--------------------+-------------+--------------+
|       Purdue       |   Bogota    |     Plazi    |
+--------------------+-------------+--------------+
|         0          |  0  | 1 | 2 |  0  | 1 | 2  |
+--------------------+-------------+--------------+
|                    |     |   |   |     |   |    |
...
```

If the user is already registered, they are authenticated based on the existing data.

The `hack_id` values are assigned either when the user signs up (if the contest is already started) or when the contest starts.
The default value for `hack_id` is `null`. This is changed automatically when the contest starts.

The algorithm which assigns the hack ids is designed to create groups of an equal number of users.
Specifically, the user will join in the hack id with the fewest users at that moment.

The database query is: find how many users we have in each hack id, for a given hack type.
Then, join the current user in the hack id with the fewest users.

```js
function generateGetHackId(hType, name) {
    return cb => {
        User.model.aggregate([{
            $match: {
                "profile.hack_id": { $ne: null },
                "profile.hack_type": name
            }
        }, {
            $group: {
                _id: "$profile.hack_id",
                total: { $sum: 1 }
            }
        }], (err, docs) => {
            if (err) { return cb(0); }
            const ids = Array(hType.subforums_count + 1).fill(0);
            docs.forEach(c => {
                ids[c._id] = c.total;
            });
            let minId = 0;
            let min = ids[minId];
            ids.forEach((count, index) => {
                if (count < min) {
                    minId = index;
                    min = ids[minId];
                }
            });
            cb(minId);
        });
    };
}

forEach(HACK_TYPES, (c, name) => {
    c.getHackId = generateGetHackId(c, name);
});
```

The function which assigns the hack id values to the users is in the `HackTypes` controller (`app/controllers/HackTypes.js`).

This function receives as input a hack type object and groups the users inside of the hack type.

```js
const assignHackIdsToUsers = hType => {
    const usersCursor = User.model.find({
        "profile.hack_id": null,
        "profile.hack_type": hType.name
    }).cursor();

    usersCursor.on("data", cDoc => {
        usersCursor.pause();
        hType.getHackId(uHackId => {
            User.update({
                _id: cDoc._id
            }, {
                profile: {
                    hack_id: uHackId
                }
            }, (err, data) => {
                if (err) { Bloggify.log(err); }
                usersCursor.resume();
            });
        });
    });

    usersCursor.on("error", err => {
        Bloggify.log(err);
    });

    usersCursor.on("end", cDoc => {
        Bloggify.log(`Grouped the studends from ${hType.name}.`);
    });
};
```

The function above is called when the countdown finishes, being triggered by a
scheduler:

```js
const setScheduleForHackType = name => {
    if (name.name) {
        name = name.name;
    }

    let hackTypeObj = HACK_TYPES[name];
    if (hackTypeObj.startSchedule) {
        hackTypeObj.startSchedule.cancel();
    }

    hackTypeObj.startSchedule = schedule.scheduleJob(hackTypeObj.start_date, () => {
        assignHackIdsToUsers(hackTypeObj);
    });
};
```

Or, it may be triggered when we make changes in the admin dashboard, changing the
start of the contest.

```js
if (new Date() > thisHackType.start_date) {
    if (thisHackType.startSchedule) {
        thisHackType.startSchedule.cancel();
    }
    assignHackIdsToUsers(thisHackType);
} else {
    setScheduleForHackType(thisHackType);
}
```

To catch the `save` event, we add a hook using the `addHook` method defined
by the [`bloggify-mongoose`](https://github.com/Bloggify/bloggify-mongoose) plugin.

```js
Settings.model.addHook("post", "save", update);
```

#### Quizzes

The quizzes page displays the quizzes that can be taken by the user. The user may answer the same quiz multiple times.

In the view file (`app/routes/quizzes.ajs`) we have the part which renders the links to each quiz:

```erb
<% include("../views/header", { title: "Quizzes" }) %>
<% include("../views/container/start") %>
<h1>Quizzes</h1>

<% quizzes.forEach(function (quiz) { %>
    <a class="btn" href="<%= quiz.url %>"><%= quiz.label %></a>
<% }); %>

<% include("../views/container/end") %>
<% include("../views/footer") %>
```

The data associated with this view is storred in the controller (`app/controllers/quizzes.js`)–see below. The user can click the generated link which contains information about the user (the email address and the user id)–which are storred in the Qualtrics quiz responses as embedded data, and also the redirect url.
When the user finishes the quiz, they are redirected back the application, on the `/quizzes` page and the application marks the quiz complete internally. Even the quiz was completed, the user can take it again.

```js
const Bloggify = require("bloggify")
    , Session = require("./Session")
    , User = require("./User")
    , findValue = require("find-value")
    ;

// Define the quizzes list
const quizzes = [
    ["d3.js", "https://purdue.qualtrics.com/jfe/form/SV_71xEzp5vQ7rC817", "d3"]
  , ["HTML & CSS", "https://purdue.qualtrics.com/jfe/form/SV_do6Sc9VJsAMmOih", "html_css"]
  , ["JavaScript & jQuery", "https://purdue.qualtrics.com/jfe/form/SV_b8zyxX8wozQfNul", "javascript_jquery"]
];

// Map the quizzes labels to the data
const validQuizzes = {};
quizzes.forEach(c => {
    validQuizzes[c[2]] = c;
});

module.exports = (lien, cb) => {
    const user = Session.getUser(lien);
    if (!user) { return lien.redirect("/"); }

    // Set the quiz complete
    const completed = lien.query.markComplete;
    if (completed && validQuizzes[completed]) {
        return User.update({
            _id: user._id
        }, {
            profile: {
                surveys: {
                    [completed]: {
                        ended_at: new Date()
                    }
                }
            }
        }, (err, _user) => {
            lien.redirect("/quizzes");
        })
    }

    // Generate the redirect links
    const completedSurveys = findValue(user, "profile.surveys") || {};
    const userQuizzes = quizzes.map(c => {
        const redirectTo =  encodeURIComponent(`${Bloggify.options.metadata.domain}/quizzes?markComplete=${c[2]}`);
        return {
            label: c[0]
          , url: `${c[1]}?redirect_to=${redirectTo}&user_email=${user.email}&user_id=${user._id}`
          , is_complete: !!completedSurveys[c[2]]
        };
    });

    // Send the quizzes array to the view
    cb(null, {
        quizzes: userQuizzes
    });
};
```


#### Search

On the search page (`/search`) we can search for content which appears either in the post data or in the comments.

The view associated with this page is storred in the `app/routes/search.ajs` and it looks like this:

```erb
<% include("../views/header", { title: "Search" }) %>
<% include("../views/container/start") %>
<h1>Search</h1>
<div class="search-form-wrapper">
    <% include("../views/search-form") %>
</div>
<% if (f("results")) { %>
    <p class="search-results-text">Search results for <em>‘<%= lien.query.search %>’</em></p>
    <div class="search-results">
        <% if (results.length) { %>
            <% results.forEach(function (cResult) { %>
                <div class="seach-result-item">
                    <h2>
                        <a href="<%= cResult.url %>">
                            <%= cResult.title %>
                        </a>
                    </h2>
                </div>
            <% }) %>
        <% } else { %>
            <div class="no-search-results">
                There are no results. Maybe try a different query.
            </div>
        <% } %>
    </div>
<% } %>
<% include("../views/container/end") %>
<% include("../views/footer") %>
```

This file requires the `search-form` which appears in the `app/views/search-form.ajs` file, representing the search form itself:

```erb
<form>
    <input type="text" name="search" value="<%= lien.query.search || "" %>" placeholder="Search for something..." />
</form>
```

When the user submits the query, the `?search=<query>` querystring parameter is added in the url, triggering the search functionality in the controller (located in `app/controllers/search.js`). To increase the search results accuracy we used the internal MongoDB text search indexes like this: we created text indexes for the topic title and content and comment content, using the `text: true` in the model configuration:

**`app/models/Topic.js`**:

```js
module.exports = {
    ...
    title: {
        type: String,
        text: true
    },
    ...
    body: {
        type: String,
        text: true
    },
    ...
};
```

**`app/models/Comment.js`**:

```js
module.exports = {
    ...
    body: {
        type: String,
        text: true
    },
    ...
};
```

**Note**: The admin users will see the search results from all the forums, while the simple users will see the search results from the forum they belong to.

The controller which takes care of searching looks like this:

```js
const Session = require("./Session")
    , Topic = require("./Topic")
    , Comment = require("./Comment")
    ;

module.exports = (lien, cb) => {

    const user = Session.getUser(lien);
    if (!user) {
        return lien.redirect("/");
    }

    const isAdmin = Session.isAdmin(user);

    // Perform the search query
    if (lien.query.search) {

        // Use the $text index to search
        const filters = {
            $text: {
                $search: lien.query.search
            }
        };
        let results = {};

        // Search in the topics and comments
        Promise.all([
            Topic.model.find(filters)
          , Comment.model.find(filters)
        ]).then(data => {
            results.topics = data[0];
            results.comments = data[1].map(c => c.toObject());
            return Promise.all(results.comments.map(cComment => {
                return Topic.model.findOne({ _id: cComment.topic });
            }));
        }).then(topics => {
            let uniqueTopics = {};
            results.topics.concat(topics).forEach(c => {
                if (!c) { return; }

                // Let the admin see all the posts/comments in all the forums
                if (!isAdmin) {
                    if (c.metadata.hack_type !== user.profile.hack_type ||
                        c.metadata.hack_id !== user.profile.hack_id) {
                        return;
                    }
                }
                uniqueTopics[c._id] = c;
                c.url = Topic.getUrl(c);
            });

            cb(null, { results: Object.keys(uniqueTopics).map(k => uniqueTopics[k]) });
        }).catch(e => {
            cb(e);
        });
    } else {
        cb();
    }
};
```


#### Posts Page

For authenticated users, we display the posts on the first page, ordered by the date, but the sticky posts are always the first ones. Only the admin users can make create sticky posts (or edit a post and make it sticky).

Here, the users from a specific forum can see and upvote the posts from the same forum. They can click on the post urls and post comments.

#### Single post pages
The single post pages are accessible by authenticated users only. They display the post title, body, votes and comments.

In case somebody comments, the comments are updated in real-time, the votes too.

When a user opens a topic page, we collect stats about that event:

 - `actor`: the user id who clicked the button
 - `topic_id`: the topic id
 - `phase`: the phase of the project
 - `created_at`: the timestamp

#### Posting a new topic

By accessing the `/new` endpoint, one can post a topic in their forum. They have to write the title and the topic content.

The topic content can be styled with Markdown specific styles (bold, italic etc).

#### The scores page

We display the scores of the users, on the `/scores` page. The users see the anonymous name of the users in the table. The displayed items in the table are shuffled each time.

If in the admin interface the scores are not provided, the scores collumns will not appear in the scores. In a similar way it happens for the urls: if we don't enter the urls, the urls collumns will not appear in the scores page.

When the user clicks on the <kbd>View scores</kbd> button, we collect stats:

 - `actor`: the user id who clicked the button
 - `hacker_id`: the user from the table which was clicked
 - `phase`: the phase of the project
 - `created_at`: the timestamp

Similar things happen when one clicks the Project url or the GitHub repository url. We know what was clicked and who did it.

In the scores controller (the `controllers/scores.js` file) a query to fetch the users from a certain forum is made. Then we get the active scores and urls for the current phase of the contest and create an array which is passed to the scores view.
To keep the users semi-anonymous, we change the usernames into `Hacker {1-...}`. Then the users array is shuffled.

This is the code snippet which fetches the users, modifies the usernames and shuffles the array.

```js
User.model.find({
    "profile.hack_type": user.profile.hack_type,
    "profile.hack_id": user.profile.hack_iand usls for the current phase of the contest and create an array which is passed to the scores view.
}, (err, users) => {
    if (err) { return cb(err); }
    Settings.get((err, options) => {
        if (err) { return cb(err); }
        const phase = options.settings.hack_types[user.profile.hack_type].phase;
        users = users.map((u, i) => {
            u = u.toObject();
            u.username = `Hacker ${i + 1}`;
            const phaseObj = Object(u.profile[phase]);
            return {
                _id: u._id,
                username: u.username,
                score_technical: phaseObj.score_technical,
                score_info_viz: phaseObj.score_info_viz,
                score_novelty: phaseObj.score_novelty,
                score_total: phaseObj.score_total,
                project_url: phaseObj.project_url,
                github_repo_url: phaseObj.github_repo_url
            };
        });

        shuffle(users);

        cb(null, {
            users: users,
            phase: phase
        });
    });
});
```

The `shuffle` function is a basic algorithm of shuffling the elements from a given array:

```js
function shuffle(array) {
  var currentIndex = array.length, temporaryValue, randomIndex;

  // While there remain elements to shuffle...
  while (0 !== currentIndex) {

    // Pick a remaining element...
    randomIndex = Math.floor(Math.random() * currentIndex);
    currentIndex -= 1;

    // And swap it with the current element.
    temporaryValue = array[currentIndex];
    array[currentIndex] = array[randomIndex];
    array[randomIndex] = temporaryValue;
  }

  return array;
}
```

#### Admin interface

An admin can access additional functionality (such as deleting and editing any post).
They have access to the dashboard (`/admin`) where they can make other users admins.
If nobody is admin (say there are no users), we can make somebody an admin (even if
they don't exist *yet* in the database) by assiging the GitHub username of an eventual
user to the environment variable called `ADMIN_USERNAME`.

The `ADMIN_USERNAME` environment variable represents the GitHub username of the user
which should be an admin (this user cannot be a simple user anymore, nobody being
able to remove the admin rights from them). When they are going to log in,
they will be authenticated as admin.

To set the `ADMIN_USERNAME` variable, there are multiple ways, but the easiest ones are:

 - If the application runs in a Heroku environment, the variable can be set from the
   browser interface: `https://dashboard.heroku.com/apps/<app-name>/settings`, by
   clicking the `Reveal Config Vars` button or in the command line using:

    ```sh
    heroku config:set ADMIN_USERNAME=hackpurdue
    ```

    Note: after setting an enviroment variable on Heroku (either from the
    command line) or from the browser interface, the application will be
    restarted automatically.

 - When running locally, the environment variable can be set when starting the app:

    ```sh
    ADMIN_USERNAME=hackpurdue npm run start:dev
    ```

In the admin interface, the admin can:

 1. Change the Phase.
 2. Download the CSV stats.
 3. Set the start dates for each hack type.
 4. See all the users and update the scores for each and eventually make them admins.

In case another user is made admin, they should log out (if they are authenticated) and log in back.

###### Custom database Filters for admin

For simple users, the database queries include the `author`, being the current authenticated user.
When the user has admin permissions, we do not append anymore the `author` the queries, therefore making the queries more liberal, giving more power to the admin.

For instance, when deleting a post, the user will create the following query:

*delete the post with `_id`=... and `author=...`*

Therefore, if the user tries to delete another post, having the id, that post will not be found because it is not created by the authenticated user.

Tho, if the user is an admin, the query is simpler, lacking the `author` field (we want to give them the power to delete any post):


*delete the post with `_id=...` *


This is happening in the `app/controllers/posts/_topicId-_slug/delete.js` controller:

```js
const Topic = require("../../Topic")
    , Session = require("../../Session")
    ;

exports.post = (lien, cb) => {
    const user = Session.getUser(lien);
    if (!user) {
        return lien.next();
    }

    const filters = {
        _id: lien.params.topicId
    };

    if (!Session.isAdmin(user)) {
         filters.author = user._id;
    }

    Topic.remove(filters, (err, count) => {
        if (err) { return lien.apiError(err); }
        lien.redirect("/");
    })
};
```

#### Application structure

In the `routes` folder, we have the page templates which are linked to the controllers from the `controllers` folder.

```
routes/
├── 404.ajs
├── 500.ajs
├── admin.ajs
├── countdown.ajs
├── index.ajs
├── logout.ajs
├── new.ajs
├── posts
│   ├── index.ajs
│   └── _topicId-_slug
│       ├── comments.ajs
│       ├── delete.ajs
│       ├── edit.ajs
│       ├── index.ajs
│       └── toggle-vote.ajs
├── quizzes.ajs
├── register.ajs
├── scores.ajs
├── search.ajs
└── users
    └── _user
        ├── edit.ajs
        └── index.ajs
```

The `_` character marks a dynamic route (such as a topic id/slug, or user).

The controllers are:

```
controllers/
├── admin.js
├── Comment.js
├── countdown.js
├── HackTypes.js
├── index.js
├── login.js
├── logout.js
├── new.js
├── posts
│   └── _topicId-_slug
│       ├── comments.js
│       ├── delete.js
│       ├── edit.js
│       ├── index.js
│       └── toggle-vote.js
├── quizzes.js
├── register.js
├── scores.js
├── search.js
├── Session.js
├── Settings.js
├── Stats.js
├── Topic.js
├── User.js
└── users
    └── _user
        ├── edit.js
        └── index.js
```

#### Modules

In this application we use the following main modules:

##### Database Connection
To connect with the database the [`bloggify-mongoose`](https://github.com/Bloggify/bloggify-mongoose) module was used.
The database models are stored in the `app/models` directory:

```
models/
├── Comment.js
├── Settings.js
├── Stats.js
├── Topic.js
└── User.js
```

To see the raw database collections and documents, we can connect directly using the MongoDB CLI:

```sh
mongo ....mlab.com:63758/heroku_... -u <dbuser> -p <dbpassword>
```

or we can see that in the browser:

 1. Open the application *Overview*  page `https://dashboard.heroku.com/apps/ironhackplatform`
 2. Clik the `mLab MongoDB`. This will redirect to an url like this:

    ```
    https://www.mlab.com/databases/heroku_...
    ```
 3. On this page, we can see the collections and eventually the documents and edit them.

The models we interact with are:

###### `Comment`
Stores the comments in the `comments` collection.

```js
module.exports = {
    author: "string",
    body: {
        type: String,
        text: true
    },
    created_at: "date",
    topic: "string",
    votes: ["string"]
};
```

###### `Settings`

Stores the application settings.

```sh
module.exports = {
    settings: "object"
};
```

###### `Stats`
Used to store the stats we collect.

```js
module.exports = {
    actor: "string",
    metadata: "object",
    event: "string",
    created_at: "date"
};
```

In this collection we save the user events. See below what stats we collect.

###### `Topic`
Stores the topics in the `topics` collection.

```js
module.exports = {
    author: "string",
    title: {
        type: String,
        text: true
    },
    slug: "string",
    body: {
        type: String,
        text: true
    },
    created_at: "date",
    votes: ["string"],
    sticky: "boolean",
    metadata: "object"
};
```


###### `User`
Stores the users in the `users` collection.

```js
module.exports = {
    username: "string",
    email: "string",
    password: "string",
    profile: "object",
    role: "string"
};
```


##### What stats we collect

We collect stats when the user:

 - clicks the <kbd>View scores</kbd> button
 - opens a topic page

The stats are collected by making HTTP requests to the server, using the
`fetch` technology (it's a new browser API, similar to `XHRHttpRequest`).

The code snippets that take care of this is located in `app/assets/javascripts/util/index.js`:

```js
...
    /**
     * post
     * Posts the data to the server.
     *
     * @name post
     * @function
     * @param {String} url The endpoint url.
     * @param {Object} data The post data.
     * @returns {Promise} The `fetch` promise.
     */
  , post (url, data) {
        data._csrf = data._csrf || _pageData.csrfToken;
        return fetch(url, {
            method: "POST",
            headers: {
                "Content-Type": "application/json"
            },
            credentials: "same-origin",
            body: JSON.stringify(data)
        });
    }

    /**
     * getJSON
     * Fetches from the server JSON data.
     *
     * @name getJSON
     * @function
     * @param {String} url The endpoint url.
     * @returns {Promise} The `fetch` promise.
     */
  , getJSON (url) {
        return fetch(url, {
            credentials: "same-origin"
        }).then(c => c.json())
    }
...
```

That's the low-level side of sending/receiving any data to/from the server using
`fetch`. Note: some browsers don't have the `fetch` technology, therefore we use
a [polyfill created by GitHub](https://github.com/github/fetch) to ensure the function is there.

We collect three types of stats:

 1. `view-topic`

    Emitted when the user opens a topic.

    Metadata:

     - `topic_id`: The topic id.
     - `topic_author`: The user id of the topic author.

    Code snippet:

    ```js
    util.post("/api/stats", {
        event: "view-topic",
        metadata: {
            topic_id: topic._id,
            topic_author: topic.author._id
        }
    });
    ```

 2. `score-click`

    Emitted when the user clicks the <kbd>View scores</kbd> button.

    Metadata:

      - `hacker_id`: The user that *was clicked*.

    Code snippet:

    ```js
    util.post("/api/stats", {
        event: "score-click",
        metadata: {
            hacker_id: this.props.hacker._id
        }
    });
    ```

 3. Clicks on the urls.

    The following events are emitted:

     - `click-project-url`: When clicking the project url.
     - `click-github-repo-url`: When clicking the GitHub repository url.

    Metadata:

     - `hacker_id`: The hacker id from the table.
     - `url`: The clicked url.

    Code snippet:

    ```js
    util.post("/api/stats", {
        event: e.target.dataset.event,
        metadata: {
            hacker_id: this.props.hacker._id,
            url: e.target.href
        }
    });
    ```

**The stats functionality on the server**:

On the server, we create a custom endpoint at `/api/stats` which expects `POST`
data. We do not collect any stats from unauthenticated users.

Along with the metadata we receive from the client side (see above) we add in the
stat object the following information:

 - `actor`: The current authenticated user id.
 - `event`: The event name.
 - `user_agent`: The user agent: this contains device and browser information.
 - `phase`: The phase of the contest.

The `actor` is the authenticated user id, and it will always be appended in the
event object because we know there is an authenticated user.

After we build the stats object, we call the `Stats.record` which will record
the event in the database. The `record` method is not anything more than just
a create query, after appending the `created_at` field in the event object.

```js
Bloggify.server.addPage("/api/stats", "post", lien => {
    const user = Session.getUser(lien);

    if (!user) {
        return lien.next();
    }

    const ev = {
        actor: user._id,
        event: lien.data.event,
        metadata: lien.data.metadata || {}
    };

    ev.metadata.user_agent = lien.header("user-agent");

    Settings.get((err, settings) => {
        if (settings) {
            ev.metadata.phase = settings.settings.hack_types[user.profile.hack_type].phase;
        }
        Stats.record(ev, (err, data) => {
            if (err) {
                return lien.apiError(err);
            }
            lien.apiMsg("success");
        });
    });
});
```

##### GitHub Login

We used [`bloggify-github-login`](https://github.com/Bloggify/github-login) to handle the GitHub authentication.

By providing the GitHub application credentials, this module handles the OAuth2 workflow.

```js
{
    "githubClient": "45...7",
    "githubSecret": "1f...2"
}
```

##### Email Notifications

To send emails, we use [`bloggify-sendgrid`](https://github.com/Bloggify/bloggify-sendgrid).

```js
{
    "key": "SG.SmlHGA...ylY"
}
```

The `notifications.js` file takes care of sending the emails using this module. We send emails when:

 1. Somebody creates a new topic

    Emails are sent to all the users from the forum the author belongs to, except to the author.

 2. Somebody posts a comment

    Emails are sent to previous people involved in the conversation.
    
# Visual_Similarity
Visual Similarity Program built by Jia 
to get the image similarities between web applications to test the effects of transparency on the end products developed. 

# This work is used in ENGL 421 class in Purdue University for data analysis

## Notes

You'll have to use the `Terminal` application to run the commands.

Open the terminal, by pressing <kbd>Cmd</kbd> + <kbd>Space</kbd>. Then, write `Terminal` and press <kbd>Enter</kbd>.

## Reference
Z. Wang and Q. Li, "Information Content Weighting for Perceptual Image Quality Assessment," in IEEE Transactions on Image Processing, vol. 20, no. 5, pp. 1185-1198, May 2011.

N. Shi and R. A. Olsson, "Reverse Engineering of Design Patterns from Java Source Code," 21st IEEE/ACM International Conference on Automated Software Engineering (ASE'06), Tokyo, 2006, pp. 123-134.

H. Kou, S. Zhang and D. Zhang, "A Segmentation and Extraction Method for Crops of Image in Complex Background," 2008 Workshop on Power Electronics and Intelligent Transportation System, Guangzhou, 2008, pp. 157-160.

P. C. Su and Y. C. Chou, "An image retargeting scheme with content-based cropping and local significance aware seam carving," 2015 Asia-Pacific Signal and Information Processing Association Annual Summit and Conference (APSIPA), Hong Kong, 2015, pp. 643-648.

## Mandatory Software

 - Download and install Node.js
 - Make sure you have installed `git`

## Installation

```sh
# Clone the repository

# Enter in the downloaded directory
cd visual-similarity-ssim

# Install the dependencies (run once)
npm install
```

## Usage

Follow the steps below.

### 1. Download the screenshots

This script will download the screenshots from the input JSON files.

```sh
node nightmare lists/BlackIronHack-Phase1.json
node nightmare lists/BlackIronHack-Phase2.json
node nightmare lists/BlackIronHack-Phase3.json
node nightmare lists/BlackIronHack-Phase4.json

# ...and so on for GoldIronHack and GreenIronHack
```

A more convenient way would be to run the `download-screenshots.sh` script to download the screenshots from *all* the lists:

```sh
sh download-screenshots.sh
```

### 2. Remove the white areas

Some images may have big white areas, therefore we want to remove them. You can do it by running this script:

```sh
node remove-white-areas
```

### 3. Generate the Canny Edge Borders images

This will generate the screenshots without colors, storing them in the `nocolor-screenshots` folder.

```sh
node create-edge-borders
```

### 4. Get the results

To compare the screenshots with the colors:

```sh
node compare BlackIronhack-Phase4
```

To compare the screenshots without colors:

```sh
node compare BlackIronhack-Phase4 --no-colors
```

To compare two lists, do:

```sh
node compare BlackIronhack-Phase3 BlackIronhack-Phase4
```

A convenient way is to run the `compare-all.sh` script to compare all the screenshots:

```sh
sh compare-all.sh
```

### 5. Push the changes (optional) :octocat:

Push on GitHub (where we're going to see the nicely formatted tables):

```sh
git add .
git commit -m 'Update results.'
git push
```

### 6. Check the results

The results will appear in the [`results`](https://github.com/jialincheoh/visual-similarity-ssim/tree/master/results) directory.

### 7. Generate the CSV files

Run:

```sh
node generate-csv
```

### 8. Generate the PDF files

Install `electron-pdf`:

```sh
npm i -g electron-pdf
```

Then, use the `to-pdf.sh` script:

```sh
. to-pdf.sh
```

## Examples

```sh
node nightmare lists/BlackIronHack-Phase4.json

## Optionally, use the --dev option to open the developer tools
node nightmare lists/GreenIronHack-Phase1.json --dev

# Create Canny edge images
node create-edge-borders.js

# Run the compare script which generates the results
node compare BlackIronHack-Phase4
node compare BlackIronHack-Phase4 BlackIronHack-Phase3

# Without colors
node compare BlackIronHack-Phase4 BlackIronHack-Phase3 --no-colors
node compare BlackIronHack-Phase4 --no-colors

node compare BlackIronHack-Phase4
```

Then check the `results` directory.

## How it works?

The code is powered by the Node.js platform and the `npm` package manager (where we download the dependencies from).

### Why Node.js & JavaScript?

These days, with JavaScript we can build pretty much anything: amazing applications in the browser (desktop and mobile), native applications for iOS and Android devices. With the power of Node.js we can handle the server side with JavaScript, and also creating command line applications.

Its flexibility allows us to easily build applications, in our case small scripts––we can call them robots––which will simulate the human behavior on different pages.

Another reason is that we can write the same piece of code that will be executed on the server and client, building izomorphic modules (literaly the same code running on server and client). In our case, most of the websites are powered by JavaScript, so that makes it even easier to work with.

#### Dependencies

## bindy
Combined with `one-by-one` and `same-time` it makes it easier to create flow behaviors by passing an array of elements.

## daty
Work with Date objects.

## cli-table
Creates tables in the command line interface. Used in the test files.

## fast-csv
Generates the CSV files.

## fs-file-tree
Reads the directories and the files and creates an object easy to work with.

## idy
This generates random ids.

## image-parser
Parses the image files into a programmatic format, JavaScript friendly.

## img-ssim
Used to get the SSIM similarity values between two images.

## is-there
Used to check if a directory or a file exists or not on the hard disk.

## iterate-object
Iterates the keys and values in a specified object.

## jquery
Enables a friendly syntax to select DOM elements from the page. We inject `jQuery` in the `nightmare` pages to hide all the elements except one.

## json2md
Converts JSON code into MarkDown.

## mkdirp
Creates directories.

## nightmare
Used to load the pages, using Electron in the background.

## one-by-one
Runs tasks (functions), one after another (when one is finished, the next one is called).

## opencv
Used to generate Canny Edge Borders images and get the Feature Matching similarities.

## parse-url
We parse the urls so we can use the friendly domain names in the file names.

## r-json
Reads JSON files (outputs the JavaScript objects in scripts).

## read-utf8
Read text files from the hard disk.

## same-time
Runs tasks (functions), in parallel (they start in the same moment, but since they finishes in different moments of time, the `same-time` module will catch these events and will call another function when all the tasks are complete).

## same-time-limit
Runs tasks (functions), in parallel, but with a sepcific limit (not more than `<x>` in parallel).

## streamp
Creates readable and writable streams after making sure that the directory structure.

## w-json
Write JSON files (outputs the JavaScript objects into files).

----

The are several scripts doing different tasks: downloading images, removing the white areas, creating Canny edge images, comparing markers, comparing images etc etc.

### `nightmare.js`

It creates screenshots of the pages, after clicking on a potential Google Maps marker (if it exists).
The script requires a mandatory JSON file as input (second argument) containing the list of urls to screenshot.

To run the script you have to:

```sh
node nightmare list.json
```

This will load in the background each URL in the Electron browser (powered by Chromium--the open-source version of Google Chrome) using the Nightmare module. After loading the page, it will intercept the `addListener` method from the Google Maps markers and will start collecting the markers when they are adding the click listeners.

This way, we can simulate the human behavior on the page, by triggering the click on the first marker. This will make the chart visible on the page.

Then the script will save the screenshots:

 - the complete page
 - the map
 - the charts (split in different images)
 - the simple pages (without maps and charts)

The screenshots are saved in the `screenshots` directory under the following structure:

```
-+ screenshots
   + <list-name>
     + maps
     + charts
     + complete
     + simple
```

The screenshots are taken by loading the urls in parallel instances of Nightmare (that means multiple Electron processes in the background) to provide an as fast as possible result. 20 pages are loaded in parallel.
After all the pages are loaded and the screenshots are taken, the script finishes its task and closes the process.

:bulb: **Pro Tip**: For debugging we can use the `--dev` option to open the Electron browser in the GUI (graphical user interface) and also the Developer Tools:

```sh
node nightmare list.json --dev
```

By default the screenshots are cached: if the screenshot already exists, the script will skip that url to be loaded.
We can turn this off by running the script using the `--no-cache` option:

```sh
node nightmare list.json --no-cache
```

Before saving the map screenshot, we check if there is a map on the page. If there is no map, we don't save it (no image will be created in that case).
Something similar happens in case of charts: the chart is not saved if there is no chart available. In case there are more charts on the page, the script screenshots each individual chart into a different image.

This is done in two steps:

 1. Check how many charts we have on the page

    We send the chart selector to the Electron browser and run on the page a piece of JavaScript code which tells us how many charts are in there:

    ```js
    // For example, this will give us the count of <svg> elements on the page
    // (supposing they are charts)
    $("svg").length
    ```

    By using `$(selector).length` we can find how many elements are on the page, matched by a specific selector.

    The charts count is sent back to the main nightmare process.

 2. After knowing how many charts we have on the page, the script will hide all the other elements except the current chart to screenshot (by the index of the element).

The chart index appears in the file name of the screenshot.

---

#### Markers

The script also saves the positions of the markers in the `info/<list-name>/markers` directory by calling a function inject in the page: `getMarkers`.
This function returns back a list of markers in this format:


```js
[
    { "lat": ..., "lng": ... }
]
```

### `create-edge-borders.js`

This script generates the `nocolor-screenshots` directory containing the same images like the `screenshots` directory, but without colors, specifically using the Canny Edge Borders algorithm from OpenCV.

```sh
node create-edge-borders.js
```

The `nocolor-screenshots` directory will have the same structure like the `screenshots` folder.

### `compare-markers.js`

This script compares the markers by using two different algorithms:

 1. The average of the distances between each marker and all the others (from other pages), and

    Example:

    ```
    This happens when both pages have the same number of markers.

    A                              B
    ========                       ========
    Marker 1                       Marker X
    Marker 2                       Marker Y
    Marker 3                       Marker Z

    1 -> X = 1X < The distance between 1 and X
    1 -> Y = 1Y < The distance between 1 and Y
    1 -> Z = 1Z ...

    2 -> X = 2X
    2 -> Y = 2Y
    2 -> Z = 2Z

    3 -> X = 3X
    3 -> Y = 3Y
    3 -> Z = 3Z
             ====
             1X + 1Y + 1Z
           + 2X + 2Y + 2Z
           + 3X + 3Y + 3Z = total

    The final result: total / (number of distances: 9)
                      total / 9
                      average of distances

             * * *

    This happens when both pages have different numbers of markers.

    A                              B
    ========                       ========
    Marker 1                       Marker X
    Marker 2                       Marker Y
    Marker 3

    1 -> X = 1X < The distance between 1 and X
    1 -> Y = 1Y < The distance between 1 and Y

    2 -> X = 2X
    2 -> Y = 2Y

    3 -> X = 3X
    3 -> Y = 3Y
             ====
             1X + 1Y
           + 2X + 2Y
           + 3X + 3Y = total

    The final result: total / (number of distances: 6) <<<< This can be computed by multiplying
                      total / 6                           | the number of markers from the first
                      average of distances                | page with the number of markers from
                                                          | the second page (2 x 3 => 6)
    ```

 2. The distance between the average coordinates of the markers.

    Example:

    ```
    A                              B
    ========                       ========
    Marker 1 (lat, lng)            Marker X (lat, lng)
    Marker 2 (lat, lng)            Marker Y (lat, lng)
    Marker 3 (lat, lng)            Marker Z (lat, lng)
    Marker 4 (lat, lng)
    Marker 5 (lat, lng)
    Marker 6 (lat, lng)

    ---------------------------------------- < Average coordinates
    * = (                          # = (
        average(lat)                   average(lat)
        average(lng)                   average(lng)
    )                              )


    Map A:                         Map B:

                        Final value (distance)
                    ______________________
                   |                      |
    +--------------|----+          +------|------------+
    | 1            V 5   |         |  X   #  Z         |
    |              *    |          |                   |
    | 6               4 |          |      Y            |
    |              3    |          |                   |
    |                   |          |                   |
    |                 2 |          |                   |
    +-------------------+          +-------------------+

    Average A   <--- Distance ---> Average B

    The final result: the distance between the average
    coordinate of point from page A (*) and from page B (#)

             ,__                                                  _,
     \~\|  ~~---___              ,                          | \
      | Wash./ |   ~~~~~~~|~~~~~| ~~---,                VT_/,ME>
     /~-_--__| |  Montana |N Dak\ Minn/ ~\~~/Mich.     /~| ||,'
     |Oregon /  \         |------|   { WI / /~)     __-NY',|_\,NH
    /       |Ida.|~~~~~~~~|S Dak.\    \   | | '~\  |_____,|~,-'Mass.
    |~~--__ |    | Wyoming|____  |~~~~~|--| |__ /_-'Penn.{,~Conn (RI)
    |   |  ~~~|~~|        |    ~~\ Iowa/  `-' |`~ |~_____{/NJ
    |   |     |  '---------, Nebr.\----| IL|IN|OH,' ~/~\,|`MD (DE)
    ',  \ Nev.|Utah| Colo. |~~~~~~~|    \  | ,'~~\WV/ VA | Y <--------------+
    X|Cal\    |    |       | Kansas| MO  \_-~ KY /`~___--\                  |
    ^',   \  ,-----|-------+-------'_____/__----~~/N Car./                  |
    | '_   '\|     |      |~~~|Okla.|    | Tenn._/-,~~-,/                   |
    |   \    |Ariz.| New  |   |_    |Ark./~~|~~\    \,/S Car.               |
    |    ~~~-'     | Mex. |     `~~~\___|MS |AL | GA /                      |
    |        '-,_  | _____|          |  /   | ,-'---~\                      |
    |            `~'~  \    Texas    |LA`--,~~~~-~~,FL\                     |
    |                   \/~\      /~~~`---`         |  \                    |
    |                       \    /                   \  |                   |
    |                        \  |                     '\'                   |
    |                                                                       |
    |                                                                       |
    |                                                                       |
    *                                                                       |
    X (x1, y1): San Francisco                                               |
    Y (x2, y2): Philadelphia *----------------------------------------------+

    Distance between averages:
    ==========================
    M = [(x1 + x2) / 2, (y1 + y2) / 2]

    X (37.766886, -122.439776)
    Y (40.009006, -75.178410)

    M = [(37 + 40) / 2, (-122 + -75) / 2]
    M = [38.5, -98.5]

    Average of distances:
    =======================
    Chicago -> SF => 2000 M
    Chicago -> PH => 759 M
    ---------------------
    Average: 1.3k M
    ```

Executed without any arguments, it will generate the `./results/<list-name>/MARKERS.md`.

To compare the markers across two lists, simply pass two arguments:

```sh
node compare-markers List1 List2
```

### `remove-white-areas.js`

Since we screenshot big pages, sometimes big white areas will appear, because of the page design. To prevent the white areas to affect our statistics, we decided to crop the images using an algorithm which detects the white areas.

The algorithm scans each image and removes potential transparent/white areas from the margins:

 - from the top to the bottom
 - from the bottom to the top
 - from the left to right, and
 - from the right to the left

### `compare.js`

This uses the SSIM and Feature Matching algorithms to find the similarities between the images inside of the same list. It uses the `screenshots/<list-name>` directory, generating `results/<list-name>RESULTS.md`, but by adding the `--no-colors` options, we can generate the results for `nocolor-screenshots` directory (generating `results/<list-name>/NO_COLOR_RESULTS.md`).

This compares each image with all the others, inside of the same directory (for example, it doesn't compare the maps from one list with the maps from another list or a map with a chart).

You can run it like this:

```sh
# Use the screenshots directory
node compare <list-name>

# Use the nocolor-screenshots directory
node compare <list-name> --no-colors

# Compare two lists
node compare <list1> <list2> [--no-colors]
```

It supports comparing two different lists by passing two list names. In that case, we will see in the `results` folder a directory called `<list1>→<list2>`

### How do intercept the markers?

Because Google APIs do not give us access to the map or markers in a friendly way, way need to *override* specific functions from the Google APIs library, redirecting all the calls to these functions through our code. To do that, we should be fast enough to catch the moment when the Google Maps library is loaded on the page, but *before* rendering the maps in the page.

To catch the load event of the Google Maps, we reverse-engineered the library code to find out how they define the public variables on the page. We were lucky because they set a global variable in the `window` object.

We opened the Google Maps library file in the browser and looked at the beautified code (which by default is minified, therefore not human understandable).

We opened the `https://maps.googleapis.com/maps/api/js?key=<key>` url and, beatified the code, and looked where the `google` global variable is defined in the `window` object. By searching for `window.google = ` we found this:

```js
window.google = window.google || {};
google.maps = google.maps || {};
```

**Note** :joy:: The `window` object stores the global variables. Since the `window` object *is* a global variable, there is a reference in the `window` object to itself. That means we can access `window.window` and `window.window.window.window...`.

It's clear that this is setting a global variable, therefore we can intercept it and see when it's actually set.


````
Timeline:
       Google Loaded                       Add the markers
|------------|*---------|-------------------------|---->
Page          ^      Init the map
load          |
started       |
              |
              \ This is the moment of time we
                are interested in: after Google
                Maps library was loaded, but
                before rendering the maps and the
                markers.
````

Using a small trick—specifically using the `Object.defineProperty` method—we can see when this variable is defined, and call a custom function which overrides the functions:

```js
// Catch the google event
var ___google = null;

// Define the "google" property
Object.defineProperty(window, "google", {
  set: function (google) {
    ___google = google;
    // Asynchronously call the `googleLoaded` function
    setTimeout(function() { googleLoaded(); }, 0);
  },
  get: function () {
    return ___google;
  }
});
```

Then, we override the `addListener` methods: in the `prototype` of the `Marker` class and also in the `google.maps.event.addListener`. This way we can run a piece of custom stuff, such as adding the marker objects into a list.

Here is the specific code snippet that does that.

```js
// Save in two variables the original methods
// We're going to call them later
var oldAddEventListener = google.maps.event.addListener;
var oldAddEventListenerProto = google.maps.Marker.prototype.addListener;

// Click the first marker after one second after the
// last marker was added
function prepareToClickMarker() {
    clearTimeout(timeout);
    timeout = setTimeout(function() {
        setTimeout(function() {
            window.__mapMarkerClicked = true;
        }, 10);
        new google.maps.event.trigger(_markers[0], 'click')
    }, 1000);
}

// Here we're going to store all the markers
// And then click on the first of them
var _markers = [];

//// Override the addListener (google.maps)
google.maps.Marker.prototype.addListener = function () {
    _markers.push(this);
    prepareToClickMarker();
    return oldAddEventListenerProto.apply(this, arguments);
};

var timeout = null;
// Some of the urls use the addListener on
// the google.maps.event object, so we should override it as well
google.maps.event.addListener  = function (marker, ev) {
    if (marker.setIcon && ev === "click") {
        _markers.push(marker);
        prepareToClickMarker();
    }
    return oldAddEventListener.apply(google.maps.event, arguments);
};
```
Using the `trigger` method we call the click handler programatically:

```js
new google.maps.event.trigger(_markers[0], 'click')
```

Then, after 10 milliseconds, we tell the nightmare script that we clicked on the marker:

```js
setTimeout(function() {
   window.__mapMarkerClicked = true;
}, 10);
```

### Taking the screenshots

Using the `nightmare` module, we can screenshot the page using the `screenshot` method.

We make these screenshots:

 - the complete page
 - the map
 - the charts
 - the simple pages (no maps and no charts)

Since the Nightmare module does *not* allow us to screenshot a specific element, we had to build that. By looking in the Google Maps code we found out they add the `gm-style` class on the map element. By hidding,  all the elements, except the `.gm-style` elements, we get only the maps on the page, and then make the screenshot.

In an analog way, we do it for the charts: we hide everything except the `svg` elements.

This is the specific code which extends Nightmare with this method:

```js
// This extends Nightmare with a new method which makes all the
// elements selected by a specific jQuery selector visible on the page
Nightmare.action('hideEverythingExcept', function(selector, done) {
    var self = this;

    // Run a script in the page
    self.evaluate_now(function(selector) {
        // Make the background white by default,
        // so we don't get transparent screenshots
        $("head").append($("<style>", { html: "body { background-color: #fff; }" }));

        // Take the main element(s)
        var $elm = $(selector);

        // Select all the parent and descendant elements, including the main element
        var $elementsToShow = $elm.add($elm.parents()).add($elm.find("*"));

        // Hide everything except the elements to show
        $("*").not($elementsToShow).hide();
    }, function(err, result) {
        if (err) {
            return done(err);
        }
        done();
    }, selector);
    return this;
});
```

Then, to show the map we do:

```js
nightmare.hideEverythingExcept(".gm-style");
```

### Google Maps Markers

#### Client side (`preload.js`)

 - Load the page (intercept the Google APIs): override the `addListener` method
 - Collect the markers
 - Once the marker was clicked, the set a boolean variable on the client as `true`.

#### Nightmare Script (`nightmare.js`)

 - Check every 2s if the any marker was clicked for maximum 10 times.

After 10 tries, it declares there are not any markers on the page.

Then it goes to the next url.

## What is Nighmare?

Nightmare is a module based on Electron.

Electron is a platform which is powered by Chromium and you can manipulate the pages inside of the Electron browser using JavaScript.

Chromium is the open-source version of Google Chrome (Chrome has some components which are proprietary).

PhantomJS is another browser, but it's better to use Electron because it contains the latest features in the browser.

## SSIM Algorithm

The SSIM (Structural Similarity) algorithm gets two images and then it splits both in small areas (squares), and compare the average color of each small area with the one from the other image.

```
Happy Face (Image 1)                       Sad Face (Image 2)
+----------------------------------------+ +----------------------------------------+
|                                        | |                                        |
|              OOOOOOOOOO                | |              OOOOOOOOOO                |
|          OOOOOOOOOOOOOOOOOO            | |          OOOOOOOOOOOOOOOOOO            |
|        OOOOO  OOOOOOOO  OOOOO          | |        OOOOO  OOOOOOOO  OOOOO          |
|      OOOOO      OOOO      OOOOO        | |      OOOOO      OOOO      OOOOO        |
|    OOOOOOO  #   OOOO  #   OOOOOOO      | |    OOOOOOO  #   OOOO  #   OOOOOOO      |
|   OOOOOOOOO    OOOOOO    OOOOOOOOO     | |   OOOOOOOOO    OOOOOO    OOOOOOOOO     |
|  OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO    | |  OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO    |
|  OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO    | |  OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO    |
|  OOOO  OOOOOOOOOOOOOOOOOOOOOO  OOOO    | |  OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO    |
|   OOOO OOOOOOOOOOOOOOOOOOOOOO OOOO     | |   OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO     |
|    OOOO   OOOOOOOOOOOOOOOOOO  OOO      | |    OOOOO                    OOOOO      |
|      OOOO   OOOOOOOOOOOOOO   OOO       | |      OOOOOOOOOOOOOOOOOOOOOOOOOOO       |
|        OOOOO   OOOOOOOO   OOOOO        | |        OOOOOOOOOOOOOOOOOOOOOOOO        |
|          OOOOOO        OOOOOO          | |          OOOOOOOOOOOOOOOOOOOO          |
|              OOOOOOOOOOO               | |              OOOOOOOOOOO               |
|                                        | |                                        |
+----------------------------------------+ +----------------------------------------+
```

For example, the SSIM algorithm will split the both images in 90 squares (10 horizontally, 9 vertically) / image.

```
   1    2    3    4    5    6    7    8    9    10         1    2    3    4    5    6    7    8    9    10
  +----|----|----|----|----|----|----|----|----|----+    +----|----|----|----|----|----|----|----|----|----+
 A|    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
  |    |    |    |  OO|OOOO|OOOO|    |    |    |    |    |    |    |    |  OO|OOOO|OOOO|    |    |    |    |
  ---------------------------------------------------    ---------------------------------------------------
 B|    |    |  OO|OOOO|OOOO|OOOO|OOOO|    |    |    |    |    |    |  OO|OOOO|OOOO|OOOO|OOOO|    |    |    |
  |    |    |OOOO|O  O|OOOO|OOO | OOO|OO  |    |    |    |    |    |OOOO|O  O|OOOO|OOO | OOO|OO  |    |    |
  ---------------------------------------------------    ---------------------------------------------------
 C|    |  OO|OOO |    | OOO|O   |   O|OOOO|    |    |    |    |  OO|OOO |    | OOO|O   |   O|OOOO|    |    |
  |    |OOOO|OOO | #  | OOO|O  #|   O|OOOO|OO  |    |    |    |OOOO|OOO | #  | OOO|O  #|   O|OOOO|OO  |    |
  ---------------------------------------------------    ---------------------------------------------------
 D|   O|OOOO|OOOO|    |OOOO|OO  |  OO|OOOO|OOO |    |    |   O|OOOO|OOOO|    |OOOO|OO  |  OO|OOOO|OOO |    |
  |  OO|OOOO|OOOO|OOOO|OOOO|OOOO|OOOO|OOOO|OOOO|    |    |  OO|OOOO|OOOO|OOOO|OOOO|OOOO|OOOO|OOOO|OOOO|    |
  ---------------------------------------------------    ---------------------------------------------------
 E|  OO|OOOO|OOOO|OOOO|OOOO|OOOO|OOOO|OOOO|OOOO|    |    |  OO|OOOO|OOOO|OOOO|OOOO|OOOO|OOOO|OOOO|OOOO|    |
  |  OO|OO  |OOOO|OOOO|OOOO|OOOO|OOOO|OO  |OOOO|    |    |  OO|OOOO|OOOO|OOOO|OOOO|OOOO|OOOO|OOOO|OOOO|    |
  ---------------------------------------------------    ---------------------------------------------------
 F|   O|OOO |OOOO|OOOO|OOOO|OOOO|OOOO|OO O|OOO |    |    |   O|OOOO|OOOO|OOOO|OOOO|OOOO|OOOO|OOOO|OOO |    |
  |    |OOOO|   O|OOOO|OOOO|OOOO|OOOO|O  O|OO  |    |    |    |OOOO|O   |    |    |    |    | OOO|OO  |    |
  ---------------------------------------------------    ---------------------------------------------------
 G|    |  OO|OO  | OOO|OOOO|OOOO|OOO |  OO|O   |    |    |    |  OO|OOOO|OOOO|OOOO|OOOO|OOOO|OOOO|O   |    |
  |    |    |OOOO|O   |OOOO|OOOO|   O|OOOO|    |    |    |    |    |OOOO|OOOO|OOOO|OOOO|OOOO|OOOO|    |    |
  ---------------------------------------------------    ---------------------------------------------------
 H|    |    |  OO|OOOO|    |    |OOOO|OO  |    |    |    |    |    |  OO|OOOO|OOOO|OOOO|OOOO|OO  |    |    |
  |    |    |    |  OO|OOOO|OOOO|O   |    |    |    |    |    |    |    |  OO|OOOO|OOOO|O   |    |    |    |
  ---------------------------------------------------    ---------------------------------------------------
 I|    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
  +----|----|----|----|----|----|----|----|----|----+    +----|----|----|----|----|----|----|----|----|----+
```

Then, it's going to compare each square from the first image with the one having the same coordinates in the second image. For instance, it will start comparing:

 - A1 (from the first image), with the A1 (from the second image). Being both, black, they will be identical: so the score will be pretty high.
 - It will continue with A2, A3, A4, ..., A10, B1, B2, ..., B10, all until now being identical.
 - The first different square is: E2, looking like this:

   ```
   First image  Second image
   +----+       +----+
   |OOOO|       |OOOO|
   |OO  |       |OOOO|
   +----+       +----+
   ```

   Since the algorithm compares the average color, in this case: we have `O` in the second image and *almost* `O` in the first image, but a little bit darker.
   Still, they will be pretty similar, but not identical.

 - Then we have another 5 identical squares (E3-E7).
 - E8 is interpreted in the same way like E2.
 - E9-F1 are identical.
 - F2 has a space, while the second image doesn't. So, they are no identical.
 - F3 is a special case which will be seen identical by the algorithm, but the areas are not the same. That's because the algorithm compares the *average* color. In this case we have:

   ```
   First image  Second image
   +----+       +----+
   |OOOO|       |OOOO|
   |   O|       |O   |
   +----+       +----+
   ```

   The squares are different, but the average color will be the same (because we have the same number of white and black pixels), therefore, the SSIM algorithm will say: they are identical.

 - F4-F8 are not identical. They will low down the final resul of SSIM.
 - F9-G2 are identical.
 - G3, G4, G7 and G8 are different.
 - G5, G6 and G9-H4 are identical.
 - H5 and H6 are different.
 - H7-I10 are identical.

Finally, we should get a high similarity because there are a lot of identical areas (for instance, the top half of the face is identical) and there are just a few very different squares.

In the `img-ssim` module, we extended the SSIM algorithm to accept different sizes. We reorder the images by the width: if the height first image is higher than the second image height, the second one will become the first and the first will be the second.

```
Happy Face (Image 1)                       Sad Face (Image 2)
+----------------------------------------+ +----------------------------------------+
|                                        | |                                        |
|              OOOOOOOOOO                | |              OOOOOOOOOO                |
|          OOOOOOOOOOOOOOOOOO            | |          OOOOOOOOOOOOOOOOOO            |
|        OOOOO  OOOOOOOO  OOOOO          | |        OOOOO  OOOOOOOO  OOOOO          |
|      OOOOO      OOOO      OOOOO        | |      OOOOO      OOOO      OOOOO        |
|    OOOOOOO  #   OOOO  #   OOOOOOO      | |    OOOOOOO  #   OOOO  #   OOOOOOO      |
|   OOOOOOOOO    OOOOOO    OOOOOOOOO     | |   OOOOOOOOO    OOOOOO    OOOOOOOOO     |
|  OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO    | |  OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO    |
|  OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO    | |  OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO    |
|  OOOO  OOOOOOOOOOOOOOOOOOOOOO  OOOO    | |  OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO    |
|   OOOO OOOOOOOOOOOOOOOOOOOOOO OOOO     | |   OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO     |
|    OOOO  OOOOOOOOOOOOOOOOOOO OOOO      | |    OOOOO                    OOOOO      |
|      OOOO   OOOOOOOOOOOOOO   OOO       | |      OOOOOOOOOOOOOOOOOOOOOOOOOOO       |
|        OOOOO   OOOOOOOO   OOOOO        | |        OOOOOOOOOOOOOOOOOOOOOOOO        |
|          OOOOOO        OOOOOO          | |          OOOOOOOOOOOOOOOOOOOO          |
|              OOOOOOOOOOO               | |              OOOOOOOOOOO               |
|  ______           O                    | |                                        |
|  |Stop|           O                    | +----------------------------------------+
|  |____|           O                    |
|     |OOOOOOOOOOOOOOOOOOOOOOOOOOOO      |
|     |             O                    |
|     |             O                    |
|                   O                    |
|                   O                    |
|                  O O                   |
|                 O   O                  |
|                O     O                 |
|                O     O                 |
|                O     O                 |
|                O     O                 |
|                O     O                 |
|              OOO     OOO               |
+----------------------------------------+
```

Then, these two images will be split in the same way:


```
   1   2   3   4   5   6   7   8   9   10  11  12  13  14    1   2   3   4   5   6   7   8   9   10  11  12  13  14
  +---|---|---|---|---|---|---|---|---|---|---|---|---|-+   +---|---|---|---|---|---|---|---|---|---|---|---|---|-+
 A|   |   |   |   |   |   |   |   |   |   |   |   |   | |   |   |   |   |   |   |   |   |   |   |   |   |   |   | |
  |   |   |   |   |  O|OOO|OOO|OOO|   |   |   |   |   | |   |   |   |   |   |  O|OOO|OOO|OOO|   |   |   |   |   | |
  -------------------------------------------------------   -------------------------------------------------------
 B|   |   |   | OO|OOO|OOO|OOO|OOO|OOO|O  |   |   |   | |   |   |   |   | OO|OOO|OOO|OOO|OOO|OOO|O  |   |   |   | |
  |   |   |  O|OOO|O  |OOO|OOO|OO | OO|OOO|   |   |   | |   |   |   |  O|OOO|O  |OOO|OOO|OO | OO|OOO|   |   |   | |
  -------------------------------------------------------   -------------------------------------------------------
 C|   |   |OOO|OO |   |  O|OOO|   |   |OOO|OO |   |   | |   |   |   |OOO|OO |   |  O|OOO|   |   |OOO|OO |   |   | |
  |   | OO|OOO|OO | # |  O|OOO|  #|   |OOO|OOO|O  |   | |   |   | OO|OOO|OO | # |  O|OOO|  #|   |OOO|OOO|O  |   | |
  -------------------------------------------------------   -------------------------------------------------------
 D|   |OOO|OOO|OOO|   | OO|OOO|O  |  O|OOO|OOO|OO |   | |   |   |OOO|OOO|OOO|   | OO|OOO|O  |  O|OOO|OOO|OO |   | |
  |  O|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|   | |   |  O|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|   | |
  -------------------------------------------------------   -------------------------------------------------------
 E|  O|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|   | |   |  O|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|   | |
  |  O|OOO|  O|OOO|OOO|OOO|OOO|OOO|OOO|OOO|  O|OOO|   | |   |  O|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|   | |
  -------------------------------------------------------   -------------------------------------------------------
 F|   |OOO|O O|OOO|OOO|OOO|OOO|OOO|OOO|OOO| OO|OO |   | |   |   |OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OO |   | |
  |   | OO|OO | OO|OOO|OOO|OOO|OOO|OOO|OO |OOO|O  |   | |   |   | OO|OOO|   |   |   |   |   |   |  O|OOO|O  |   | |
  -------------------------------------------------------   -------------------------------------------------------
 G|   |   |OOO|O  | OO|OOO|OOO|OOO|OOO|   |OOO|   |   | |   |   |   |OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|   |   | |
  |   |   |  O|OOO|O  | OO|OOO|OOO|   |OOO|OO |   |   | |   |   |   |  O|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OO |   |   | |
  -------------------------------------------------------   -------------------------------------------------------
 H|   |   |   | OO|OOO|O  |   |   |OOO|OOO|   |   |   | |   |   |   |   | OO|OOO|OOO|OOO|OOO|OOO|OOO|   |   |   | |
  |   |   |   |   |  O|OOO|OOO|OOO|O  |   |   |   |   | |   |   |   |   |   |  O|OOO|OOO|OOO|O  |   |   |   |   | |
  -------------------------------------------------------   -------------------------------------------------------
 I|  _|___|__ |   |   |   | O |   |   |   |   |   |   | |   |   |   |   |   |   |   |   |   |   |   |   |   |   | |
  |  ||Sto|p| |   |   |   | O |   |   |   |   |   |   | |   |   |   |   |   |   |   |   |   |   |   |   |   |   | |
  -------------------------------------------------------   -------------------------------------------------------
 J|  ||___|_| |   |   |   | O |   |   |   |   |   |   | |
  |   |  ||OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|OOO|O  |   | |
  -------------------------------------------------------   -----------------------------------------------------
 K|   |  ||   |   |   |   | O |   |   |   |   |   |   | |
  |   |  ||   |   |   |   | O |   |   |   |   |   |   | |
  -------------------------------------------------------   -----------------------------------------------------
 L|   |   |   |   |   |   | O |   |   |   |   |   |   | |
  |   |   |   |   |   |   | O |   |   |   |   |   |   | |
  -------------------------------------------------------   -----------------------------------------------------
 M|   |   |   |   |   |   |O O|   |   |   |   |   |   | |
  |   |   |   |   |   |  O|   |O  |   |   |   |   |   | |
  -------------------------------------------------------   -----------------------------------------------------
 N|   |   |   |   |   | O |   | O |   |   |   |   |   | |
  |   |   |   |   |   | O |   | O |   |   |   |   |   | |
  -------------------------------------------------------   -----------------------------------------------------
 O|   |   |   |   |   | O |   | O |   |   |   |   |   | |
  |   |   |   |   |   | O |   | O |   |   |   |   |   | |
  -------------------------------------------------------   -----------------------------------------------------
 P|   |   |   |   |   | O |   | O |   |   |   |   |   | |
  |   |   |   |   |  O|OO |   | OO|O  |   |   |   |   | |
  -------------------------------------------------------   -----------------------------------------------------
 Q +---|---|---|---|---|---|---|---|---|---|---|---|---|-+
```

The algorithm will compare the first part (the image), in the same way like we explained above, but it will also consider the missing part from the second image totally different than the part from the left image.

So, starting with the J1 square, the areas will be considered totally different, therefore the SSIM result will be lower.

### `img-ssim` vs `image-ssim`

The main difference between `img-ssim` (the new module) and `image-ssim` is that the `img-ssim` has an option for not enforcing that the images should have the same size. In case they have different sizes, some areas from the second image will be missing. In these cases, the similarity will be totally different, as explained above.

Another option that `img-ssim` has is that we can opt for resizing the images, making them the same size.

#### Taking a look at the code

 1. Parsing the images

    The images are parsed using `image-parser`. This is useful to get the size (width and height) and the pixel colors at specific coordinates (red, green, blue, transparency).

    ```js
    // Create the ImageParser instances
    source = new ImageParser(source);
    target = new ImageParser(target);

    // Parse the two images in parallel
    sameTime([
        done => source.parse(done)
      , done => target.parse(done)
    ], err => {
        // Check if there is an error
        // And continue with the SSIM algorithm after parsing
    });
    ```
 2. Optionally, resize the images (currently skipped)

    In case the `resize` option is `true`, then the images will be resized to the same size. Specifically, the second image will be resized to have the same size with the first.

    ```js
    target.resize(source.width(), source.height(), (err, target) => {
        if (err) { return cb(err); }
        // Then continue to SSIM comparison
    });
    ```

    Currently, we do *not* use this feature, therefore this step will be skipped.

 3. Image iteration

    The both images will be iterated, by two loops: iterating each `N×N` area (where `N` is configurable, by default `8` pixels).

    ```js
    let width = image1.width()
      , height = image1.height()
      ;

    for (let y = 0; y < height; y += windowSize) {
        for (let x = 0; x < width; x += windowSize) {
            // ...
        }
    }
    ```

    Then we get the Luma values for both current areas (from the first and second images). The Luma values represent a list of numbers built from the luminance and chrominance of each pixel from the current area.


    ```js
    // The initialization of the luma values list
    let lumaValues = new Float32Array(new ArrayBuffer(width * height * 4))
    ```

    A luma value is computed by summing the RGB values and multiplying them with the Alpha value (transparency). In case the area doesn't exist (that happens when the images have different sizes), the luma value will be `0`. Therefore, more `0` luma values, will give us a lower average and lower SSIM value in the end.

    The RGB values are taken using the `getPixel` method of the `image-parser` module which outputs an object containing the RGBA values:

    ```js
    // A white pixel would look like this:
    {
        // Red
        r: 255,

        // Green
        g: 255,

        // Blue
        b: 255,

        // Alpha
        a: 1
    }
    ```

    The code snippet below shows how the values are added in the list.

    ```js
    // Pushing new values in the list
    for (let cY = y; cY < maxY; ++cY) {
        for (let cX = x; cX < maxX; ++cX) {
            let cPixel = image.getPixel(cX, cY)
              , res = (
                    cPixel.r * lumVals.r
                  + cPixel.g * lumVals.g
                  + cPixel.b * lumVals.b
                ) * (cPixel.a)
              ;

            // If the `res` value is not a number, that means one of the areas
            // is missing, therefore we will push a `0` value
            lumaValues[counter++] = res === res ? res : 0;
        }
    }
    ```

    After having two lists of Luma values, we will get the average Luma value for both of them. That's simply the arithmetic average of the values from the list. At this point we will have one average Luma value for each area (one for the first image and another one for the second image).

    ```js
    function _averageLuma(lumaValues) {
        let sumLuma = 0.0;
        for (let i = 0; i < lumaValues.length; i++) {
            sumLuma += lumaValues[i];
        }
        return sumLuma / lumaValues.length;
    }
    ```

    The next step receives 4 parameters:

     - the Luma values from the *first* square;
     - the Luma values from the *second* square;
     - the Luma average from the *first* square, and
     - the Luma average from the *second* square

    Based on these values, we calculate the variance and covariance of the values:

    ```js
    // Covariance of X and Y
    let sigxy = 0
        // The variance of X
      , sigsqx = 0
        // The variance of Y
      , sigsqy = 0
      ;

    // Iterate the values
    for (let i = 0; i < lumaValues1.length; i++) {
        // Prepare to calculate the variance
        sigsqx += Math.pow((lumaValues1[i] - averageLumaValue1), 2);
        sigsqy += Math.pow((lumaValues2[i] - averageLumaValue2), 2);

        // Prepare to calculate the covariance
        sigxy += (lumaValues1[i] - averageLumaValue1) * (lumaValues2[i] - averageLumaValue2);
    }

    // Take the averages which will represent the (co)variance values
    let numPixelsInWin = lumaValues1.length - 1;
    sigsqx /= numPixelsInWin;
    sigsqy /= numPixelsInWin;
    sigxy /= numPixelsInWin;
    ```

    Then the [classical formula](https://en.wikipedia.org/wiki/Structural_similarity) for `SSIM(x, y)` is used:

    ```js
    let numerator = (2 * averageLumaValue1 * averageLumaValue2 + c1) * (2 * sigxy + c2);
    let denominator = (
        Math.pow(averageLumaValue1, 2)
      + Math.pow(averageLumaValue2, 2)
      + c1
    ) * (sigsqx + sigsqy + c2);
    ```

    Then, we add the fraction value to the global SSIM value (which at this point is the sum of the SSIM comparisons between each two squares).

    ```js
    mssim += numerator / denominator;
    ```

    Then, the script will continue iterating to the next square and repeat these steps until the end.

 4. Getting the final result

    At this point we have the SSIM sum (sum of the SSIM values for each comparison of square pairs) and the total number of square pairs.

    The final SSIM value will be obained from the average of the SSIM values, therefore we just have to divide the sum to the number of squares.

    ```js
    mssim / numWindows
    ```

    This is the value which is exposed outside of the module, being sent to the script which is using the module.

