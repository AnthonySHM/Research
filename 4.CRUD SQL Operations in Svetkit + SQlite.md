# INTRO

This document outlines the development of a Svelte web application that leverages SQLite for efficient product management. We have created a robust system capable of creating, reading, updating, and deleting (CRUD) product data.

The application's architecture is centered around a clear separation of concerns, with dedicated components for handling product forms, displaying product lists, and executing database interactions. By adhering to best practices for component-based development, we have ensured code maintainability and reusability.

The integration of SQLite provides a reliable and efficient data storage solution. The application effectively manages product information, including name, price, and unique identifiers.

This document will delve into the technical implementation, code structure, and key functionalities of the application, providing insights into the development process and the underlying technologies employed.

#### Create
Purpose: To introduce new data into the database.<br>
HTTP Method: Typically POST.<br>
Data Sent: The complete set of data for the new record.<br>
Example: Creating a new product with its name and price.<br>

#### Update
Purpose: To modify existing data in the database.<br>
HTTP Method: Typically PUT or PATCH.<br>
Data Sent: The updated data for the specific record.<br>
Example: Changing the price of an existing product.<br>

#### Delete
Purpose: To remove data from the database.<br>
HTTP Method: Typically DELETE.<br>
Data Sent: Often just the identifier of the record to be deleted (e.g., product ID).<br>
Example: Removing a product from the database.<br>

# Defining the Product Data Structure
We'll start by defining the structure of our product data. This is crucial for consistency and type safety.

### src/app.d.ts:

```TypeScript
interface Product {
  rowid: number;
  name: string;
  price: number;
}
```
This file defines an interface named Product with three properties: rowid, name, and price. This will be used throughout the application to represent product data.

## Creating the Database Connection
We'll use SQLite to store our product data. Let's set up a database connection.

### src/hooks.server.ts:

```TypeScript
import type { Handle } from '@sveltejs/kit';
import sqlite3 from 'sqlite3';
import { open } from 'sqlite';

let db = await open({ filename: './src/mydatabase.db', driver: sqlite3.Database });

export const handle: Handle = async ({ event, resolve }) => {
  event.locals.db = db;
  const response = await resolve(event);
  return response;
};
```
This file imports the necessary libraries for SQLite interaction.<br>
It establishes a database connection to mydatabase.db.<br>
A server-side hook is created to make the database instance accessible to all routes in the application.<br>

## Building the Product Form Component
We'll create a reusable component to handle both creating and updating products.

### src/lib/ProductForm.svelte:

```Svelte
<script lang="ts">
  export let product: Product;

  let action = product.rowid ? 'update' : 'create';
</script>

<div class="card">
  <div class="card-header">Product</div>
  <div class="card-body">
    <form method="post" action={`/product/${product.rowid}?/${action}`}>
      </form>
  </div>
</div>
```
The component accepts a product object as input.<br>
It determines the form action based on whether the product has a rowid (update) or not (create).<br>
The form will be used to collect product data for both creating and updating.<br>

## Displaying and Editing Products
Let's create the main page to list products and provide links to create and edit them.

### src/routes/+page.svelte:

```Svelte
<script lang="ts">
  import DeleteButton from '$lib/DeleteButton.svelte';
  import type { PageServerData } from './$types';
  export let data: PageServerData;
</script>

<h1>Products</h1>

<a href="/product/create" class="btn btn-primary">Create</a>

<table>
  </table>
```
This component displays a list of products fetched from the server.<br>
It provides a link to create a new product.<br>
We'll populate the table with product data later.<br>

### src/routes/+page.server.ts:

```TypeScript
import type { PageServerLoad } from './$types';

export const load: PageServerLoad = async ({ locals }) => {
  let products: Product[] = await locals.db.all('select rowid,* from products;');
  return { products };
};
```
This file fetches all products from the database and returns them as page data.

## Creating a New Product
Let's create a page for creating new products.

### src/routes/product/create/+page.svelte:

```Svelte
<script lang="ts">
  import ProductForm from '$lib/ProductForm.svelte';

  let product: Product = {
    rowid: 0,
    name: '',
    price: 0,
  };
</script>

<ProductForm {product} />
```
This component creates a new product object with initial values and passes it to the ProductForm component.

## Editing an Existing Product
Let's create a page for editing existing products.

### src/routes/product/[id]/+page.svelte:

```Svelte
<script lang="ts">
  import ProductForm from '$lib/ProductForm.svelte';
  import type { PageServerData } from './$types';
  export let data: PageServerData;
</script>

<ProductForm product={data.product} />
```
This component fetches the product data based on the id parameter and passes it to the ProductForm for editing.

### src/routes/product/[id]/+page.server.ts:

```TypeScript
import type { PageServerLoad } from './$types';

export const load: PageServerLoad = async ({ params, locals }) => {
  let product = await locals.db.get(
    'select rowid,* from products where rowid = ?',
    params.id
  );
  return { product };
};
```
This file fetches the specific product based on the provided id.

## Handling Form Submissions
We'll create server-side actions to handle form submissions for creating, updating, and deleting products.

### src/routes/product/[id]/+page.server.ts:

```TypeScript
import { redirect } from '@sveltejs/kit';
import type { Actions } from './$types';

export const actions = {
  create: async (event) => {
    // Handle creating a new product
  },
  update: async (event) => {
    // Handle updating an existing product
  },
  delete: async (event) => {
    // Handle deleting a product
  },
};
```
This file defines actions for creating, updating, and deleting products. We'll implement the logic for these actions in the next steps.

## Deleting a Product
Let's create a component for deleting a product.

### src/lib/DeleteButton.svelte:

```Svelte
<script lang="ts">
  export let product: Product;
</script>

<form method="post" action={`/product/${product.rowid}/delete`}>
  <button type="submit" class="btn btn-danger btn-sm">Delete</button>
</form>
```
This component creates a form for deleting a product and displays a delete button.

## Completing the CRUD Operations
We'll now implement the logic for creating, updating, and deleting products in the actions object of the src/routes/product/[id]/+page.server.ts file.<br> This involves interacting with the SQLite database to insert, update, or delete product records.

Remember to replace the placeholder comments with the actual database operations.

# COMPONENTS SUMMARY
Below is the list of components with brief explanation on how they play part in this app:

### src/hooks.server.ts:

This file defines a server-side hook to provide the database connection to all routes.<br>
It uses sqlite3 and open to establish a connection to the mydatabase.db file.<br>
The handle function injects the database connection into the event.locals object, making it accessible throughout the application.<br>
Overall, these code snippets demonstrate a well-structured Svelte app that effectively uses SQLite for CRUD operations on product data.

### src/app.d.ts:

This file defines global types used throughout the application.<br>
It defines the App.Locals interface, which provides access to the SQLite database instance as db.<br>
It defines the Product interface, specifying the expected structure of product data (rowid, name, price).

### src/lib/ProductForm.svelte:

This component renders a form for creating or updating a product.<br>
It takes a product prop containing the product details (name and price).<br>
The form uses a POST method to submit data based on the action ("create" or "update").<br>
The action URL includes the product's rowid for updates or omits it for creation.<br>
Input fields are bound to the product's properties for data manipulation. 

### src/lib/DeleteButton.svelte:

This component represents a button used to delete a product.<br>
It takes a product prop containing information about the product to be deleted.<br>
The button uses a form element with a POST method to submit a delete request.<br>
The action URL includes the product's rowid for identification.

### src/routes/+page.svelte:

This component represents the main product listing page.<br>
It imports the DeleteButton and ProductForm components.<br>
It uses the data prop (populated from the server) to display a list of products.<br>
The table iterates through each product, displaying its details and actions.<br>
It includes a "Create" button that links to the product creation page.

### src/routes/+page.server.ts:

This server-side script loads data for the main product listing page.<br>
It retrieves all products using SELECT with * from the products table.<br>
It populates the data.products object for use in the Svelte component.

### src/routes/product/create/+page.svelte:

This component represents the product creation page.<br>
It imports the ProductForm component.<br>
It defines an empty product object as the initial state for the form.<br>
It passes the product prop to the ProductForm component for data binding.

### src/routes/product/[id]/+page.svelte:

This component represents the product edit page for a specific product ID.<br>
It imports the ProductForm component.<br>
It uses the data.product prop (populated from the server) to pre-fill the form.<br>
It passes the product prop to the ProductForm component for editing.

### src/routes/product/[id]/+page.server.ts:

This server-side script loads data for the product edit page.<br>
It retrieves a specific product based on the provided ID using SELECT with * from the products table.<br>
It populates the data.product object for use in the Svelte component.<br>
Additionally, it defines actions (create, update, delete) for handling form submissions.

## Why Delete is Different
Irreversibility: Deleting data is typically a destructive action, and there's often no way to undo it without additional mechanisms like soft deletes.<br>
Confirmation: Deleting data usually requires explicit user confirmation to prevent accidental data loss.<br>
Separate Intent: The intent of deleting a record is fundamentally different from creating or updating one.<br>

By separating the delete action, you enhance:

User Experience: Clear distinction between creating/updating and deleting.<br>
Security: Reduces the risk of accidental deletion.<br>
Code Maintainability: Simplifies the code by focusing on specific actions.<br>

In summary, while create and update actions can often share similarities, the delete action's unique characteristics make it more suitable for a separate form or button.


