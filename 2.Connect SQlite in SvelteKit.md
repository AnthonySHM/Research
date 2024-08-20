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
let db = await open ({ filename: './src/mydatabase.db', driver: sqlite3.Database })
```
This code creates a SQLite database connection.<br>
filename: './src/mydatabase.db': Specifies the path to the SQLite database file, which is located in the same directory as the hook file.<br>
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
Go to the link below, the loading data documentation in sveltekit and create a file src/routes/+page.server.ts in your project, then try to understand the code there:<br>
https://kit.svelte.dev/docs/load

To Display the data in your database, you will need to create(if not exist) two files in your routes folder named +page.svelte and +page.server.ts<br>
Let us see how to load this data from the server to the client, we'll start with the server code and implement our database connection

## CODE BREAKDOWN: +page.server.ts
### Import Statement
```TypeScript
import type { PageServerLoad } from "./$types";
```

#### import type 
This keyword imports a type definition without importing the actual value. It's used for type safety and code completion.
#### { PageServerLoad } 
Imports the PageServerLoad type from the $types module. This type defines the shape of the load function for server-side rendering in SvelteKit.

### Exporting the Load Function
```TypeScript
export const load: PageServerLoad = async ({ locals }) => {
  // ...
};
```
#### export const load 
Exports a function named load with the const keyword, making it accessible outside the module.
#### :PageServerLoad
Specifies that the load function conforms to the PageServerLoad type, ensuring correct usage within SvelteKit.
#### =async: 
Indicates that the load function is asynchronous, allowing it to perform operations that might take time, such as database queries.
({ locals }) 
Defines the function parameters. The locals object is provided by SvelteKit and contains shared data accessible to server-side code.

### Fetching Products from the Database
Within the load function do the following
```TypeScript
let products: Product[] = await locals.db.all('select rowid,* from products;')
```
#### let products: Product[]
Declares a variable named products as an array of Product objects (assuming Product is a defined type).
#### = await 
Waits for the asynchronous operation to complete before proceeding.
#### locals.db 
Accesses the database connection object stored in the locals object.
#### .all('select rowid,* from products;') 
Executes an SQL query to retrieve all rows and columns from the products table.
The result of the query, an array of objects representing the products, is assigned to the products variable.

### Returning Products
```TypeScript
return { products };
```
Returns an object with a single property, products, containing the fetched product data. This data will be available to the client-side component as part of the page data.

This code defines a server-side load function for a SvelteKit page. It fetches product data from a SQLite database using the locals.db object and returns the fetched products as part of the page data. This data can then be used in the corresponding page component to display the products to the user.

### Key Points:

The code leverages SvelteKit's server-side rendering capabilities.
It utilizes a SQLite database to retrieve product data.
The fetched data is made available to the client-side component.

## CODE BREAKDOWN: +page.svelte

```TypeScript
<script lang="ts">
import type { PageServerData } from "./$types";
export let data : PageServerData;
</script>
```
### Import Statement:

#### import type { PageServerData } from "./$types";
This line imports the PageServerData type from the $types module. This type defines the shape of the data that will be passed to the component from the server.
### Export Statement:
#### export let data : PageServerData; 
This line declares an exported variable named data of type PageServerData. This variable will be populated with the data returned from the +page.server.ts file.
### How It Works
The +page.server.ts file fetches data from the database and returns it as an object.<br>
SvelteKit automatically passes this object to the +page.svelte component as the data prop.<br>
The data prop in the +page.svelte file is then available for use within the component's template.<br>
### Example Usage
```HTML
<h1>Products</h1>
<table class="table table-bodered">
  <thead>
    <tr>
      <th>ID</th>
      <th>Name</th>
      <th>Price</th>
    </tr>
  </thead>
  <tbody>
    {#each data.products as product}
      <tr>
        <td>{product.id}</td>
        <td>{product.name}</td>
        <td>{product.price}</td>
      </tr>
    {/each}
  </tbody>
</table>
```


In this example, the data object contains a products array, which is iterated over to display the product information.

### Key Points
The +page.svelte file receives data from the server through the data prop.<br>
The PageServerData type provides type safety for the data.<br>
The data can be used within the component's template to render dynamic content.<br>
By combining the +page.server.ts and +page.svelte files, you can effectively fetch data from a database and display it in a SvelteKit application.<br>
You will also need to install bootstrap to enable the classes used in the table element



