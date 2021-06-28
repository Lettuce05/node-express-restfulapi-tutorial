# Creating RESTful APIs

## Prerequisites
You must have the following installed to follow this tutorial.
- Postman
- Node
- Mongodb Community Server
- Any Text Editory

## Setup
1. Create a new folder with your project name.
2. In a terminal, navigate to the directory of your project and type the command ```npm init```. This will setup you new Node.js project.

## Install Dependencies
1. Install express by typing the command ```npm i express```.
2. Install mongodb and mongoose by typing the command ```npm i mongodb mongoose```.
3. Install babel as a dev dependency by typing the command ```npm i --save-dev babel-cli babel-preset-env babel-preset-stage-0```. Babel allows the newest versions/syntax of javascript to be compiled into legacy javascript that is compatible with most browsers.

## Initial Server Build
### RESTful API Basics
RESTful APIs allow us to interact with a database(backend) through HTTP requests(protocols).
- GET gets the data
- POST add new data
- PUT updates data
- DELETE deletes data

### Initial Server Setup
1. Install ```body-parser``` and ```nodemon``` with the command ```npm i body-parser nodemon```. ```body-parser``` allows us to parse the json http request into our server. ```nodemon``` allows the server to refresh when it detects any changes in our code.

2. Add the following to the scripts value in the package.json file. This line of code will allow us to start our node server and makesure that it compiles the code with babel.
    ```
        // package.json
        "scripts": {
            "start": "nodemon ./index.js --exec babel-node -e js"
        }
    ```

3. In the root of your project directory create an ```index.js``` file with the following code. This will be the base file for our server.
    ```
        import express from "express";

        const app = express();
        const PORT = 4000;

        app.get("/", (req, res) => {
            res.send(`Node and express server running on port ${PORT}`);
        });

        app.listen(PORT, () => {
            console.log(`Your server is running on port ${PORT}`);
        });
    ```
    - ```const app = express();``` creates an instance of our express app/server.
    - ```const PORT = 4000;``` defines the port that our server will use.
    - ```app.get(url, callbackFunction)``` tells our server how to respond when the specified url is requested. For the example above when the base url (root URL) of our application is called with a GET request our application will respond with a webpage saying that our node/express server is running and the specified port it is running on.
    - ```app.listen(PORT, callbackFunction)``` tells our express app to listen for the specified port and how to respond when it is running(starts).

4. In the root of your project directory create an ```.babelrc``` file with the following code. When babel is called to execute by the start script, this file will tell the transpiler what presets to use.
    ```
        //.babelrc
        {
            "presets": [
                "env",
                "stage-0"
            ]
        }
    ```
5. In the terminal run the command ```npm start``` to start the express server on port 4000. ```npm start``` runs the script that we created earlier in the ```package.json``` file.
    - In the terminal you should see something similar to the following image as a response to the command.
    ![initial terminal server response](https://raw.githubusercontent.com/Lettuce05/node-express-restfulapi-tutorial/main/restfulapi_tutorial_pictures/initial-terminal-response.png)

6. In a browser type in the url ```localhost:4000``` and you should be presented with.
![initial server response](https://raw.githubusercontent.com/Lettuce05/node-express-restfulapi-tutorial/main/restfulapi_tutorial_pictures/setting_up_node_server.png)

### Initial Server Files and Folders
In this section we will create the project file structure. Please create the necessary files to match the file structure below.
```
project
│   .babelrc
│   index.js  
│   package-lock.json
│   package.json
│
└───node_modules
│   │   ...
│   
└───src
    └─controllers
      │  crmController.js
    └─models
      │  crmModel.js
    └─routes
      │  crmRoutes.js
```

### Basic routing endpoints
In this section we will create basic routing endpoints for our server.
1. In the ```crmRoutes.js file``` insert the following code to create basic routing endpoints for GET, POST, PUT, and DELETE.
    ```
        // ./src/routes/crmRoutes.js
        const routes = (app) => {
          app
            .route("/contact")
            .get((req, res) => res.send("GET request successful!"))
            .post((req, res) => res.send("POST request successful!"));
        
          app
            .route("/contact/:contactID")
            .put((req, res) => res.send("PUT request successful!"))
            .delete((req, res) => res.send("DELETE request successful!"));
        };
        
        export default routes;
    ```
- ```.route("/route")``` designates the route that the app is listening for 
- ```.get()```,```.post()```,```.put()```, and ```.delete()``` designate the allowed http protocols and a callback function with how the server should respond.

2. Modify index.js to match the following code (Add the indicated lines). In the following code we are importing the routes function that we created and we are calling it with our app (express instance) as the argument.
    ```
        import express from "express";
        import routes from "./src/routes/crmRoutes"; // ***ADD THIS LINE***
        
        const app = express();
        const PORT = 4000;
        
        routes(app); // ***ADD THIS LINE***
        
        app.get("/", (req, res) => {
          res.send(`Node and express server running on port ${PORT}`);
        });
        
        app.listen(PORT, () => {
          console.log(`Your server is running on port ${PORT}`);
        });
    ```
3. In Postman try the following routes (urls) and you should get the appropriate responses that we designated.
- ```http://localhost:4000/contact``` using the ```GET``` and ```POST``` protocols
- ```http://localhost:4000/contact/randomcharacters``` using the ```PUT``` and ```DELETE``` protocols

### Basics of Middleware and Uses
**What is middleware?**
Express functions that have access to the request and response objects and act on them.

In this section we will create a simple example of how to use middleware with our express server.

1. Modify the ```crmRoutes.js``` file to match the following code. The get request method is the only part of the file that has been modified.
    ```
        const routes = (app) => {
          app
            .route("/contact")
            .get(
              (req, res, next) => {
                //middleware
                console.log(`Request from: ${req.originalUrl}`);
                console.log(`Request from: ${req.method}`);
                next();
              },
              (req, res, next) => {
                res.send("GET request successful!");
              }
            )
            .post((req, res) => res.send("POST request successful!"));

          app
            .route("/contact/:contactID")
            .put((req, res) => res.send("PUT request successful!"))
            .delete((req, res) => res.send("DELETE request successful!"));
        };
        
        export default routes;
    ```
    - The next argument allows the first function to call the next function. This allows us to use middleware in the first function to access/modify the http request and response. In this example we are simply getting more information of the url that was requested and the method used in the request. In the second function we are sending the specified response that we want to give.
    
## CRUD Operations
### MongoDB Refresher
- Database with collections
- Collections have documents or objects
- Documents look like JSON objects
- Inside each document you have the data with key-value pairs or arrays
- Our DB has contacts, which are in a contacts collection, and each contact is a document.

### Setting up Robo 3T
1. Install Robo 3T. This application will allow us to visually see our databases and all the collections.
2. Once installed open the application and create a connection to mongodb by following the following steps.
    1. Hit the ```Create``` option in the top left of the MongoDB Connections window.
    ![MongoDB Connections window](https://raw.githubusercontent.com/Lettuce05/node-express-restfulapi-tutorial/main/restfulapi_tutorial_pictures/create-connection.png)
    2. Keep all the default settings in the next window that pops up, and press ```Save``` in the bottom right corner.
    ![Connections Settings Window](https://raw.githubusercontent.com/Lettuce05/node-express-restfulapi-tutorial/main/restfulapi_tutorial_pictures/connection-settings.png)
    3. When you return to the MongoDB Connections screen, click on the connection you just created(it should be already selected), then press ```Connect``` in the bottom right corner.

### Database setup
In this section we will setup our server to connect to our mongoDB database.

1. Modify index.js to match the code below. In this step we are importing ```mongoose``` and ```body-parser```, as well as setting up body-parser and our connection to mongoDB.
    ```
    // ./index.js
    
    import express from "express";
    import routes from "./src/routes/crmRoutes";
    import mongoose from "mongoose"; // ***ADD THIS LINE***
    import bodyParser from "body-parser"; // ***ADD THIS LINE***
    
    const app = express();
    const PORT = 4000;
        
    // ***ADD THIS SECTION***
    // mongoose connection
    mongoose.Promise = global.Promise;
    mongoose.connect("mongodb://localhost/CRMdb", {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
        
    // ***ADD THIS SECTION***
    // body-parser setup
    app.use(bodyParser.urlencoded({ extended: true }));
    app.use(bodyParser.json());
    
    routes(app);
    
    app.get("/", (req, res) => {
      res.send(`Node and express server running on port ${PORT}`);
    });
    
    app.listen(PORT, () => {
      console.log(`Your server is running on port ${PORT}`);
    });
    ```

### Schema Setup
In this section we will create our schema for our contact documents.

1. In the crmModel.js file add the following code.
    ```
    import mongoose from "mongoose";
    
    const Schema = mongoose.Schema;
    
    export const ContactSchema = new Schema({
      firstName: {
        type: String,
        required: "Enter a first name",
      },
      lastName: {
        type: String,
        required: "Enter a last name",
      },
      email: {
        type: String,
      },
      company: {
        type: String,
      },
      phone: {
        type: Number,
      },
      created_date: {
        type: Date,
        default: Date.now,
      },
    });
    ```
    
### Create POST Endpoint
In this section we will create the POST endpoint so that we can add new Contacts to the DB with our API.

1. In the ```crmController.js``` file add the following code. In this code we are importing the Schema that we created and are using it as a model for the database. We are also creating the function that will add the contact to the database or send an error if something went wrong.
    ```
    // ./src/controllers/crmController.js
    
    import mongoose from "mongoose";
    import { ContactSchema } from "../models/crmModel";
    
    const Contact = mongoose.model("Contact", ContactSchema);
    
    export const addNewContact = (req, res) => {
      let newContact = new Contact(req.body);
    
      newContact.save((err, contact) => {
        if (err) {
          res.send(err);
        }
        res.json(contact);
      });
    };
    ```

2. Modify the ```crmRoutes.js``` file to match the following code. In this step we are importing the ```addNewContact``` function that we just created, and we are making it the callback function for the ```POST``` method.
    ```
    // ./src/routes/crmRoutes.js
    
    import { addNewContact } from "../controllers/crmController"; //****ADD THIS LINE****
    
    const routes = (app) => {
      app
        .route("/contact")
        .get(
          (req, res, next) => {
            //middleware
            console.log(`Request from: ${req.originalUrl}`);
            console.log(`Request from: ${req.method}`);
            next();
          },
          (req, res, next) => {
            res.send("GET request successful!");
          }
        )
        .post(addNewContact); //****MODIFY THIS LINE****
    
      app
        .route("/contact/:contactID")
        .put((req, res) => res.send("PUT request successful!"))
        .delete((req, res) => res.send("DELETE request successful!"));
    };
    
    export default routes;
    ```

### Create all items GET endpoint
In this section we will create the GET endpoint that will return all contacts in the Contacts collection.

1. In the ```crmController.js``` file add the following lines of code to the end of the file. The code below is a function that is simply finding all contacts and returning them.
    ```
    // ./src/controllers/crmController.js
    
    export const getContacts = (req, res) => {
      Contact.find({}, (err, contact) => {
        if (err) {
          res.send(err);
        }
        res.json(contact);
      });
    };
    ```
2. Edit the ```crmRoutes.js``` file to match the code below. In this step we are importing the getContacts function and then using it as the second function in the get method.
    ```
    // ./src/routews/crmRoutes.js
    
    import { addNewContact, getContacts } from "../controllers/crmController"; //****MODIFY THIS LINE****

    const routes = (app) => {
      app
        .route("/contact")
        .get((req, res, next) => {
          //middleware
          console.log(`Request from: ${req.originalUrl}`);
          console.log(`Request from: ${req.method}`);
          next();
        }, getContacts) //****MODIFY THIS LINE****
        .post(addNewContact);
    
      app
        .route("/contact/:contactID")
        .put((req, res) => res.send("PUT request successful!"))
        .delete((req, res) => res.send("DELETE request successful!"));
    };
    
    export default routes;
    ```
    
3. In Postman use the GET method for the contact route
    - ```http://localhost:4000/contact```
The response should be all of the contacts in the DB

### Create specific ID GET endpoint
In this section we will add the GET endpoint that will allow us to request a specific contact by its ID.

1. In the ```crmController.js``` file add the following function to the end of the file.
    ```
    // ./src/controllers/crmController.js
    
    export const getContactByID = (req, res) => {
      Contact.findById(req.params.contactID, (err, contact) => {
        if (err) {
          res.send(err);
        }
        res.json(contact);
      });
    };
    ```

2. Modify the ```crmRoutes.js``` file to match the following code. In this step we are importing the ```getContactByID``` function that we just created and we are adding a get method to the second route using this function.
    ```
    // ./src/routews/crmRoutes.js
    
    import {
      addNewContact,
      getContacts,
      getContactByID, 
    } from "../controllers/crmController"; //****MODIFY THIS LINE****

    const routes = (app) => {
          app
            .route("/contact")
            .get((req, res, next) => {
              //middleware
              console.log(`Request from: ${req.originalUrl}`);
              console.log(`Request from: ${req.method}`);
              next();
            }, getContacts)
            .post(addNewContact);
    
      app
        .route("/contact/:contactID")
        .get(getContactByID) //****MODIFY THIS LINE****
        .put((req, res) => res.send("PUT request successful!"))
        .delete((req, res) => res.send("DELETE request successful!"));
    };
    
    export default routes;
    ```
    
3. In Postman use the GET method for the contact route that specifies an ID
    - ```http://localhost:4000/contact/specificID```
The response should be the specific contact requested.

### Create PUT endpoint
In this section we will create the PUT endpoint so that we can update contacts.

1. In the ```crmController.js``` file insert the following function at the end of the file.
    ```
    ./src/controllers/crmController.js
    
    export const updateContact = (req, res) => {
      Contact.findOneAndUpdate(
        { _id: req.params.contactID },
        req.body,
        { new: true, useFindAndModify: false }, //avoids use of deprecated findOneAndUpdate function and specifies to return the new(updated object)
        (err, contact) => {
          if (err) {
            res.send(err);
          }
          res.json(contact);
        }
      );
    };
    ```
    
2. Modify the ```crmRoutes.js``` file to match the following code. In this step we are importing the updateContact function and using it as the callback for the PUT method.
    ```
    ./src/routes/crmRoutes.js
    
    import {
      addNewContact,
      getContacts,
      getContactByID,
      updateContact,
    } from "../controllers/crmController"; //****MODIFY THIS LINE****
    
    const routes = (app) => {
      app
        .route("/contact")
        .get((req, res, next) => {
          //middleware
          console.log(`Request from: ${req.originalUrl}`);
          console.log(`Request from: ${req.method}`);
          next();
        }, getContacts)
        .post(addNewContact);

      app
        .route("/contact/:contactID")
        .get(getContactByID)
        .put(updateContact)  //****MODIFY THIS LINE****
        .delete((req, res) => res.send("DELETE request successful!"));
    };
    
    export default routes;

    ```
    
3. In Postman use the PUT method for the contact route that specifies an ID. In the form-encoded body insert the key-value pairs that you would like to change
    - ```http://localhost:4000/contact/specificID```
The response should be the updated contact.

### Create DELETE Endpoint
In this section we will create the DELETE endpoint so that we can delete contacts from the database.
1. In the ```crmController.js``` file insert the following function at the end of the file.
    ```
    // ./src/controllers/crmController.js
    
    export const deleteContact = (req, res) => {
      Contact.remove({ _id: req.params.contactID }, (err, contact) => {
        if (err) {
          res.send(err);
        }
        res.json({ message: "successfully deleted contact" });
      });
    };
    ```

2. Modify the ```crmRoutes.js``` file to match the following code. In this step we are importing the ```deleteContact``` function and using it as the callback for the DELETE method.
    ```
    // ./src/routes/crmRoutes.js
    
    import {
      addNewContact,
      getContacts,
      getContactByID,
      updateContact,
      deleteContact,
    } from "../controllers/crmController"; //****MODIFY THIS LINE****

    const routes = (app) => {
      app
        .route("/contact")
        // get all contacts
        .get((req, res, next) => {
          //middleware
          console.log(`Request from: ${req.originalUrl}`);
          console.log(`Request from: ${req.method}`);
          next();
        }, getContacts)
        // POST endpoint
        .post(addNewContact);

      app
        .route("/contact/:contactID")
        // get a specific contact
        .get(getContactByID)
        // update a specific contact
        .put(updateContact)
        // delete a specific contact
        .delete(deleteContact); //****MODIFY THIS LINE****
    };
    
    export default routes;
    ```
    
3. In Postman use the DELETE method for the contact route that specifies an ID.
    - ```http://localhost:4000/contact/specificID```
The response should be that the specified contact was successfully deleted.

## Serving Static Files
1. Create a folder named ```public``` in the root of your project. Add any static files to it that you would like to serve.
2. In the ```index.js``` file add the following line of code after the line that contains ```routes(app);```. In this line you are using express' static funtion to serve the static files and you are specifying a directory for the server to look for the static files.
    ```
    app.use(express.static("public"));
    ```



    


    
