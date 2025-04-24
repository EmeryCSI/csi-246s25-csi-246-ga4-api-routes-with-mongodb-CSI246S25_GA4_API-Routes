# Renton Technical College CSI-246

<div align="center">
    <img src="logo.jpg" alt="Logo">
    <h3 align="center">Guided Activity 4: NextJS API Routes with MongoDB Atlas Integration</h3>
</div>

# Guided Activity: Building Next.js API Routes with MongoDB Atlas Integration

## Overview

In this guided activity, you will create API routes in a Next.js application that connects to MongoDB Atlas using Mongoose. You will build a simple album management system that allows creating, reading, updating, and deleting album records through API endpoints that can be tested with Postman.

## Learning Objectives

By completing this activity, you will learn:
- How to set up and configure a Next.js 15 project with TypeScript
- How to create and connect to a MongoDB Atlas database
- How to implement and use Mongoose models and schemas
- How to build API routes in Next.js
- How to implement CRUD (Create, Read, Update, Delete) operations
- How to test API endpoints using Postman

## Initial Setup and Configuration

### Step 1: Creating a New Next.js Project

Create a new Next.js project with TypeScript:

```bash
npx create-next-app@latest album-api
```

When prompted, configure your project with these settings:
- Use TypeScript: Yes
- Use ESLint: Yes
- Use Tailwind CSS: No (We don't need it for an API-only project)
- Use `src/` directory: No
- Use App Router: Yes
- Customize default import alias: No

### Step 2: Installing Mongoose

Install Mongoose to work with MongoDB:

```bash
cd album-api
npm install mongoose
```

### Step 3: Project Structure Setup

Create a simplified folder structure focused on API routes:

```bash
mkdir -p app/api/albums
mkdir -p "app/api/albums/[id]"
mkdir -p lib/models
```

## Setting Up MongoDB Atlas and Database Connection

### Step 1: Creating a MongoDB Atlas Account

1. Visit [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) and create an account or sign in
2. Create a new cluster (or use an existing one)
3. Set up a database user with password authentication
4. Configure network access (Add your IP or allow all IP addresses)
5. Get your connection string

### Step 2: Environment Variable Setup

Create a `.env.local` file in your project root:

```
MONGODB_URI=your_connection_string_here
```

Make sure to replace the placeholder with your actual MongoDB connection string.

### Step 3: Database Connection Module

Create the file `lib/db.ts`:

```typescript
import mongoose from 'mongoose';

// Track the connection status
let isConnected = false;

export const connectToDatabase = async () => {
    // If we're already connected, use the existing connection
    if (isConnected) {
        console.log('Using existing database connection');
        return;
    }

    try {
        // Check if we have a MONGODB_URI environment variable
        if (!process.env.MONGODB_URI) {
            throw new Error(
                'Please define the MONGODB_URI environment variable inside .env.local'
            );
        }

        // Connect to the database
        await mongoose.connect(process.env.MONGODB_URI, {
            // These options help with connection stability
            bufferCommands: false,
        });

        // Update connection status
        isConnected = true;
        console.log('New database connection established');
    } catch (error) {
        console.error('Error connecting to database:', error);
        throw error;
    }
};
```

### Step 4: Creating the Album Model

Create the file `lib/models/Album.ts`:

```typescript
import mongoose from 'mongoose';

// Define the interface for our Album document
interface IAlbum extends mongoose.Document {
    title: string;
    artist: string;
    year: number;
    genre: string;
    createdAt: Date;
    updatedAt: Date;
}

// Check if the model already exists before creating a new one
// This is important in Next.js development due to hot reloading
const Album = mongoose.models.Album || mongoose.model<IAlbum>('Album', new mongoose.Schema({
    title: {
        type: String,
        required: [true, 'Please provide an album title'],
        maxlength: [100, 'Title cannot be more than 100 characters']
    },
    artist: {
        type: String,
        required: [true, 'Please provide an artist name'],
        maxlength: [100, 'Artist name cannot be more than 100 characters']
    },
    year: {
        type: Number,
        required: [true, 'Please provide a release year'],
        min: [1900, 'Year must be after 1900'],
        max: [new Date().getFullYear(), 'Year cannot be in the future']
    },
    genre: {
        type: String,
        required: [true, 'Please provide a genre'],
        maxlength: [50, 'Genre cannot be more than 50 characters']
    }
}, {
    timestamps: true  // This automatically adds createdAt and updatedAt fields
}));

export default Album;
```

## Understanding Next.js API Routes

### What Are API Routes?

API routes in Next.js allow you to create server-side endpoints as part of your Next.js application. They provide a straightforward way to build your API without needing a separate backend server. These routes live within the app directory (using App Router) and are created using the `route.ts` or `route.js` convention.

### How API Routes Work in Next.js 15

#### File-Based Routing

Next.js uses a file-based routing system for API routes:

- Base routes are defined in `app/api/[route-name]/route.ts`
- Each route file can export functions corresponding to HTTP methods:
  ```typescript
  export async function GET() {...}
  export async function POST(request: Request) {...}
  export async function PUT(request: Request) {...}
  export async function DELETE() {...}
  ```

#### Route Handlers

Route handlers receive and respond to HTTP requests:

- They receive a `Request` object that contains the incoming request details
- They return a `Response` object or, more commonly, a `NextResponse` which extends the standard Response object
- For example:
  ```typescript
  import { NextResponse } from 'next/server';
  
  export async function GET() {
    return NextResponse.json({ message: 'Hello World' }, { status: 200 });
  }
  ```

#### Dynamic Routes

You can create dynamic segments in routes using brackets:

- For example, `app/api/products/[id]/route.ts` will match `/api/products/1`, `/api/products/abc`, etc.
- The dynamic parameters are passed in the `params` object:
  ```typescript
  export async function GET(
    request: Request,
    { params }: { params: { id: string } }
  ) {
    const { id } = params;
    // Use the id parameter
  }
  ```

### Key Limitations and Rules

1. **One Handler Per HTTP Method:** You can only have one handler function per HTTP method in a given route file. You cannot have multiple GET functions in the same route.ts file.

2. **No File Mixing:** You cannot mix `page.tsx` and `route.ts` in the same folder. A folder must either be a page or an API route, not both.

3. **No Middleware in Route Handlers:** Route handlers do not support middleware directly. You need to use the app router middleware instead.

4. **Response Types:** Route handlers must return a Response or NextResponse object. You cannot return plain objects or JSX.

5. **Caching Behavior:** By default, Route Handlers are cached when using the GET method with Response objects. You can opt out of caching using:
   ```typescript
   export const dynamic = 'force-dynamic';
   ```

6. **Client Components:** API routes cannot be used inside Client Components directly. You need to call them using fetch or a similar method.

### When to Use Next.js API Routes vs. Separate API Servers

#### Advantages of Next.js API Routes

1. **Unified Development:** Keep frontend and backend in the same codebase, simplifying development and deployment.

2. **Serverless by Default:** API routes run as serverless functions, providing good scalability without managing servers.

3. **TypeScript Integration:** Seamless integration with TypeScript for type safety across your API and UI.

4. **Simplified Deployment:** Deploy your entire application (UI and API) together with a single command.

5. **Easy Access to Database Connections:** Share database connections between API routes, making database access more efficient.

6. **Reduced Context Switching:** Developers can work on frontend and backend using the same tooling and environment.

#### When to Consider a Separate API Server

1. **Large, Complex APIs:** If your API is very large or complex, a dedicated API server might provide better organization and separation of concerns.

2. **Team Organization:** If you have separate frontend and backend teams, dedicated servers allow teams to work independently.

3. **Different Scaling Needs:** When your API and frontend have significantly different scaling requirements or resource needs.

4. **Microservices Architecture:** If you're building a microservices architecture where different services need to be deployed and scaled independently.

5. **Language or Framework Requirements:** If you need to use a different language or framework for your API than what Next.js supports.

6. **Existing API Infrastructure:** If you already have an existing API infrastructure that you want to maintain separately.

### Best Practices for Next.js API Routes

1. **Organize by Resource:** Group your API routes by resource (e.g., `/api/users`, `/api/products`) to maintain a clean structure.

2. **Error Handling:** Implement consistent error handling patterns across all API routes.

3. **Input Validation:** Validate request inputs before processing to prevent security issues and data problems.

4. **Authentication & Authorization:** Use middleware or helper functions to handle authentication and authorization consistently.

5. **Rate Limiting:** Implement rate limiting to protect your API from abuse.

6. **Separation of Concerns:** Keep your route handlers focused on request/response handling, with business logic in separate service modules.

7. **Consistent Response Format:** Maintain a consistent response format across all endpoints for easier client integration.

### Alternative to API Routes: Server Actions

For simpler use cases, especially when you only need the API to support your own UI, Next.js Server Actions provide an alternative approach:

- Server Actions allow you to run server-side code directly from your components
- They eliminate the need to create API endpoints for many common scenarios
- They're simpler to implement as they don't require setting up request/response handling
- Example usage:
  ```typescript
  'use server';
  
  export async function createAlbum(formData: FormData) {
    // Connect to the database and create an album
    // No need to handle request/response objects
  }
  ```

When choosing between API Routes and Server Actions, consider:
- API Routes are better for external API consumers and more complex APIs
- Server Actions are simpler for internal usage when the API only supports your own UI

## Creating API Routes

### Step 1: Create Base API Route

Create `app/api/albums/route.ts`:

```typescript
import { NextResponse } from 'next/server';
import { connectToDatabase } from '@/lib/db';
import Album from '@/lib/models/Album';

// GET handler - Retrieve all albums
export async function GET() {
    try {
        // Connect to the database
        await connectToDatabase();

        // Fetch all albums, sorted by creation date (newest first)
        const albums = await Album.find({})
            .sort({ createdAt: -1 });

        // Return the albums with a 200 status code
        return NextResponse.json({ albums }, { status: 200 });
    } catch (error) {
        // Log the error for debugging
        console.error('Error fetching albums:', error);
        
        // Return a generic error message to the client
        return NextResponse.json(
            { error: 'Failed to fetch albums' },
            { status: 500 }
        );
    }
}

// POST handler - Create a new album
export async function POST(request: Request) {
    try {
        // Connect to the database
        await connectToDatabase();

        // Parse the request body
        const data = await request.json();

        // Create a new album with the provided data
        const newAlbum = await Album.create(data);

        // Return the created album with a 201 status code
        return NextResponse.json({ album: newAlbum }, { status: 201 });
    } catch (error) {
        console.error('Error creating album:', error);
        return NextResponse.json(
            { error: 'Failed to create album' },
            { status: 500 }
        );
    }
}
```

### Step 2: Create Dynamic Route Handlers

Create `app/api/albums/[id]/route.ts`:

```typescript
import { NextResponse } from 'next/server';
import { connectToDatabase } from '@/lib/db';
import Album from '@/lib/models/Album';

// Helper function to check if an ID is valid
function isValidObjectId(id: string) {
    return /^[0-9a-fA-F]{24}$/.test(id);
}

// GET handler - Retrieve a specific album
export async function GET(
    request: Request,
    { params }: { params: { id: string } }
) {
    try {
        const { id } = params;

        // Validate the ID format
        if (!isValidObjectId(id)) {
            return NextResponse.json(
                { error: 'Invalid album ID' },
                { status: 400 }
            );
        }

        await connectToDatabase();
        
        // Find the album by ID
        const album = await Album.findById(id);

        // Check if album exists
        if (!album) {
            return NextResponse.json(
                { error: 'Album not found' },
                { status: 404 }
            );
        }

        return NextResponse.json({ album }, { status: 200 });
    } catch (error) {
        console.error('Error fetching album:', error);
        return NextResponse.json(
            { error: 'Failed to fetch album' },
            { status: 500 }
        );
    }
}

// PUT handler - Update a specific album
export async function PUT(
    request: Request,
    { params }: { params: { id: string } }
) {
    try {
        const { id } = params;

        // Validate the ID format
        if (!isValidObjectId(id)) {
            return NextResponse.json(
                { error: 'Invalid album ID' },
                { status: 400 }
            );
        }

        await connectToDatabase();

        // Get the update data from the request body
        const updateData = await request.json();

        // Find and update the album
        const updatedAlbum = await Album.findByIdAndUpdate(
            id,
            updateData,
            { new: true, runValidators: true }
        );

        // Check if album exists
        if (!updatedAlbum) {
            return NextResponse.json(
                { error: 'Album not found' },
                { status: 404 }
            );
        }

        return NextResponse.json({ album: updatedAlbum }, { status: 200 });
    } catch (error) {
        console.error('Error updating album:', error);
        return NextResponse.json(
            { error: 'Failed to update album' },
            { status: 500 }
        );
    }
}

// DELETE handler - Delete a specific album
export async function DELETE(
    request: Request,
    { params }: { params: { id: string } }
) {
    try {
        const { id } = params;

        // Validate the ID format
        if (!isValidObjectId(id)) {
            return NextResponse.json(
                { error: 'Invalid album ID' },
                { status: 400 }
            );
        }

        await connectToDatabase();

        // Find and delete the album
        const deletedAlbum = await Album.findByIdAndDelete(id);

        // Check if album exists
        if (!deletedAlbum) {
            return NextResponse.json(
                { error: 'Album not found' },
                { status: 404 }
            );
        }

        return NextResponse.json(
            { message: 'Album deleted successfully' },
            { status: 200 }
        );
    } catch (error) {
        console.error('Error deleting album:', error);
        return NextResponse.json(
            { error: 'Failed to delete album' },
            { status: 500 }
        );
    }
}
```

## Testing API Endpoints with Postman

### Step 1: Start your Next.js application

```bash
npm run dev
```

Your API will be available at `http://localhost:3000/api/albums`.

### Step 2: Testing with Postman

1. Download and install [Postman](https://www.postman.com/downloads/) if you don't have it already

2. Create a new collection called "Album API"

3. Test the following endpoints:

#### CREATE a new album
- Method: POST
- URL: http://localhost:3000/api/albums
- Headers: Content-Type: application/json
- Body (raw JSON):
```json
{
  "title": "Thriller",
  "artist": "Michael Jackson",
  "year": 1982,
  "genre": "Pop"
}
```
- Expected response: JSON object with the created album

#### GET all albums
- Method: GET
- URL: http://localhost:3000/api/albums
- Expected response: JSON array of albums

#### GET one album
- Method: GET
- URL: http://localhost:3000/api/albums/[album_id]
  (Replace [album_id] with an actual ID from your GET all response)
- Expected response: JSON object with the requested album

#### UPDATE an album
- Method: PUT
- URL: http://localhost:3000/api/albums/[album_id]
- Headers: Content-Type: application/json
- Body (raw JSON):
```json
{
  "genre": "Pop/R&B"
}
```
- Expected response: JSON object with the updated album

#### DELETE an album
- Method: DELETE
- URL: http://localhost:3000/api/albums/[album_id]
- Expected response: Success message

### Step 3: Verify in MongoDB Atlas

1. Log in to MongoDB Atlas
2. Navigate to your cluster
3. Click "Browse Collections"
4. Verify that your data changes are reflected in the database

## Submission

After completing the assignment, commit your code to GitHub:

```bash
git add .
git commit -m "Completed"
git push
```
