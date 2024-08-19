#  INTRO
SQLite is a lightweight, serverless database engine. This means it doesn't require a separate server process and is often embedded within applications. It's particularly suitable for smaller datasets and applications where performance and simplicity are crucial.

It is recomended to install both sqlite3 and sqlite in your computer, this is beacause the later has an important open feature in its library that will be used to connect the database to sveltekit<br>

It's important to clarify that SQLite is the database engine itself, while SQLite3 is the command-line tool used to interact with SQLite databases. So, when you install SQLite, you automatically get SQLite3.

#### Downloading SQLite
Visit the official SQLite download page: https://www.sqlite.org/download.html<br>
Download the precompiled binaries for Windows: Look for the file named sqlite-tools-win32-x86-*.zip (replace * with the appropriate version).
#### Extracting and Setting Up
Extract the downloaded ZIP file: Choose a suitable location, such as C:\sqlite.<br>
Add the SQLite directory to your system's PATH: This allows you to run the sqlite3 command from any directory.<br>
Right-click on "This PC" and select "Properties".<br>
Click on "Advanced system settings".<br>
Click on the "Environment Variables" button.<br>
Under "System variables", find the "Path" variable and click "Edit".<br> 
Add the path to the SQLite directory (e.g., C:\sqlite) to the list.<br>
Click "OK" to save the changes.
#### Verifying the Installation
Open a new command prompt.   
Type sqlite3 and press Enter.
If the installation was successful, you should see the SQLite command-line interface.

#### Creating a Database and Tables with SQLite3 Command Line
To create a new SQLite database, simply specify the filename when starting the SQLite3 shell:
```
sqlite3 mydatabase.db
```
similarly, you can use the extension .sqlite for a database, sqlite3 will accept both.

```
sqlite3 mydatbase.sqlite
```
This will create a new database file named mydatabase.db or mydatabase.sqlite in your current directory if it doesn't exist.

#### Creating a Table and Inserting Data
Once you're in the SQLite3 shell, you'll see a sqlite> prompt.<br>
In here you can execute SQL commmands and create a table.<br>
It is important to note the feature that a row called rowid, is automatically created in the table database(To select this row you need to include it in your SQL query), also it might be helpful to use the ".help" command in SQLite3 for further info.<br>
You can also enable table mode for the sake of output format using the command ".mode table" in SQLite3 bash.

# CONNECTING YOUR CREATED SQLITE DATABASE IN SVELTEKIT
Run the code below within the root folder of your sveltekit application to install the necessary dependancies for your project
```
npm install sqlite sqlite3
```
To do this we need to use sveltekit hooks created at 'src/hooks.server.ts'<br>
Hooks are app-wide functions you declare that SvelteKit will call in response to specific events, giving you fine-grained control over the framework's behaviour.<br>
https://kit.svelte.dev/docs/hooks<br>
Within this hook You need to do the following

### Import Statements:
```
import type { Handle } from "@sveltejs/kit";:
```
This line imports the Handle type from the @sveltejs/kit package. This type is used for defining SvelteKit hooks.
```
import sqlite3 from "sqlite3";
```
Imports the sqlite3 module, which is a Node.js built-in module for interacting with SQLite databases.
```
import { open } from "sqlite"; 
```
Imports the open function from the sqlite package. This package provides a promise-based interface for interacting with SQLite databases.

### Database Connection:
```
let db = await open ({ filename: './test.sqlite', driver: sqlite3.Database })
```
This code creates a SQLite database connection.<br>
filename: './test.sqlite': Specifies the path to the SQLite database file, which is located in the same directory as the hook file.<br>
driver: sqlite3.Database: Uses the sqlite3 module as the driver for the database connection.

### SvelteKit Hook:
```
export const handle: Handle = async ({ event, resolve }) => { ... }
```
This line defines a SvelteKit hook named handle with two arguments<br>
event: Contains information about the incoming request.<br>
resolve: A function to handle the request and generate a response.

### Attaching Database to Event Locals:
First you need to add the property called 'db' on type locals<br>
Navigate to your app definition file named 'app.d.ts' in your src folder and add the following interface(Normally you'll find it commented without any code inside) 
```
    interface Locals {
      db: sqlite3.Database;
    }
```
Go back to your hook.
The code snippets below should be inserted within the braces of the hook created named handle.
```
event.locals.db = db
```
Attaches the database connection to the event.locals object. This makes the database accessible to other parts of the SvelteKit application.

### Resolving the Request:
```
const response = await resolve(event);
```
Calls the resolve function to handle the request and generate a response.

Returning the Response:
```
return response;
```
Returns the generated response.

### Overall Functionality
This code sets up a SQLite database connection and attaches it to the event.locals object in a SvelteKit hook. This allows subsequent parts of the application to access the database instance through the event.locals.db property.

### Key Points
The database connection is established using the sqlite and sqlite3 modules.<br>
The database is stored in a file named test.sqlite in the same directory as the hook.<br>
The database connection is attached to the event.locals object for later use.<br>
The hook itself doesn't perform any database operations; it merely establishes the connection and makes it available.<br>
By using this hook, you can access the database from within your SvelteKit routes or pages to perform database operations like querying, inserting, updating, or deleting data.

# HOW TO DISPLAY THE DATA IN YOUR SQLITE DATABASE

