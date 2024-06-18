---
title: React Router
order: 6
---



### Don't Do Case

#### SubComponent with GraphQL Query
In this case, the `SubComponent` handles the GraphQL query directly, which should be avoided:

```javascript
// SubComponent.js
import { gql, useQuery } from '@apollo/client';

const GET_USERS = gql`
    query GetUsers {
        users {
            id
            name
            email
        }
    }
`;

function SubComponent() {
    const { data, loading, error } = useQuery(GET_USERS);

    if (loading) return <p>Loading...</p>;
    if (error) return <p>Error: {error.message}</p>;

    return (
        <div>
            <h1>Users</h1>
            <ul>
                {data.users.map(user => (
                    <li key={user.id}>
                        {user.name} ({user.email})
                    </li>
                ))}
            </ul>
        </div>
    );
}

export default SubComponent;
```

#### Main Page Importing SubComponent
The main page simply imports and uses `SubComponent`, without handling the data fetching properly.

```javascript
// mainPage.js
import SubComponent from './SubComponent';

export default function MainPage() {
    return (
        <div>
            <SubComponent />
        </div>
    );
}
```

### Do Case

#### Step-by-Step Refactor

1. **Define the GraphQL Query in the Main Page**

```javascript
// mainPage.js
import { gql } from '@apollo/client';

const GET_USERS = gql`
    query GetUsers {
        users {
            id
            name
            email
        }
    }
`;
```

2. **Set Up the Loader Function**

```javascript
// mainPage.js
import { json, LoaderFunction } from '@remix-run/node';
import { useLoaderData } from '@remix-run/react';
import { initializeApollo } from '~/utils/apolloClient'; // Utility to initialize Apollo Client
import SubComponent from './SubComponent';

// NOTE: Instead below you can also do graphql query directly here with fetch policy instead of writing loader as described here 
export let loader: LoaderFunction = async () => {
    const client = initializeApollo();
    try {
        const { data } = await client.query({ query: GET_USERS });
        return json(data);
    } catch (error) {
        console.error('Error fetching users:', error);
        throw new Response('Error fetching users', { status: 500 });
    }
};
```

3. **Create the SubComponent as a Pure Component**

```javascript
// SubComponent.js
function SubComponent({ users }) {
    return (
        <div>
            <h1>Users</h1>
            <ul>
                {users.map(user => (
                    <li key={user.id}>
                        {user.name} ({user.email})
                    </li>
                ))}
            </ul>
        </div>
    );
}

export default SubComponent;
```

4. **Use the SubComponent in the Main Page**

```javascript
// mainPage.js
import { useLoaderData } from '@remix-run/react';
import SubComponent from './SubComponent';

export default function MainPage() {
    const { users } = useLoaderData();

    return (
        <div>
            <SubComponent users={users} />
        </div>
    );
}
```

### Summary

**Don't Do Case:**
- Placing GraphQL queries inside subcomponents makes it harder to manage data flow and breaks SSR benefits.
- Subcomponents are not pure and are tightly coupled with data-fetching logic.

**Do Case:**
- GraphQL queries are centralized in the main routed page's loader.
- Data fetching is handled during SSR, improving performance.
- Subcomponents are kept pure and receive data via props, making them more reusable and easier to test.
