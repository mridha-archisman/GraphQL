# GraphQL

## Getting Started

To start a express-graphql server, in the terminal, in an empty folder enter the following commands :

```zsh

    mkdir backend
    cd backend
    npm init -y
    npm install --save express graphql express-graphql nodemon cors
    touch schema.js
    mkdir Schema
    touch Schema/graphql.js
```

We first make a simple express server :

```js

    const express = require('express');
    const bodyParser = require('body-parser');
    const cors = require('cors');

    const app = express( );

    app.use( bodyParser.json( ) );
    app.use( cors( ) );

    app.listen( 4000, ( ) => console.log('Server started') );
```

## Creating a GraphQL Schema :

A graphql-schema stores information about how the data is connected, and queries and mutations used for
reading/writing into that data.

```js

    const UserType = new GraphQLObjectType(

        {
            name: 'User',
            fields: {

                id: {

                    type: GraphQLString
                },

                name: {

                    type: GraphQLString
                }
            }
        }
    );

    const RootQuery = new GraphQLObjectType(

        {
            name: 'RootQuery',
            fields: {

                user: {

                    // type and args are the return type and parameter schema

                    type: UsertType,
                    args: {

                        id: { type: GraphQLString }
                    },

                    resolve ( parent, args ) {

                        // find an user from database and return it
                    }
                }
            }
        }
    );

    module.exports = new GraphQLSchema(

        {
            query: RootQuery
        }
    );
```

In the server.js file import this graphql-schema configuration and use it as a middleware.

```js

    app.use( '/graphql', graphQLHttp(

        {
            graphiql: true,
            schema
        }
    ));
```

## Nested Queries

Along with the users, now let us introduce the field of companies and we are going to relate the users
with the companies.

```js

    const CompanyType = new GraphQLObjectType(

        {
            name: 'Comapany',
            fields: {

                id: {

                    type: GraphQLString
                },

                name: {

                    type: GraphQLString
                },

                description: {

                    type: GraphQLString
                }
            }
        }
    );
```

In the UserType, we introduce the company field: 

```js

    company: {

        type: CompanyType,

        resolve ( parent, arg ) {

            // fetch the company detais, from the company-id stored in parent.company field
        }
    }
```

Let's add another query, to fetch company details using an id

```js

    company: {

        type: CompanyType,
        args: {

            id: GraphQLString
        },

        resolve ( parent, args ) {

            // search from database and return the company details
        }
    }
```

## Bidirectional Relations

Lets now introduce users-list field for each company and create a bidirectional relation.

```js

    users: {

        type: new GraphQLList( UserType ),

        resolve ( parent, args ) {

            // fetch the users using the company id stored in parent.id field
        }
    }
```

But by introducing bi-directional relation, there is going to be an error. Suppose our code structure
is like this :

```js

    const CompanyType = ...;

    const UserType =  ...;
```

Since, UserType is defied after CompanyType, but in the definition of CompanyType, we are using that
constant UserType, we will run into an error that 'UserType' is not defined. So we are going to create
a function which returns the new GraphQLObject({ ... }) object. By doing this, the function will be
executed after all the variables are intialized in the file, due to which it doesn't run into the error
we were previously getting !

```js

    const CompanyType = ( ) => {

        return new GraphQLObjectType(

            {
                ...
            }
        );
    };
```

## Query Fragments

Suppose, now we want to query for company details. Queries are written like this :

```

    query {

        company ( id: "1" ) {

            id
            name
            description
        }
    }
```

What if, we want to perform 2 queries at the same tim for the same query name like this :

```

    query {

        company ( id: "1" ) { ... }
        company ( id: "2" ) { ... }
    }
```

We will get an error stating that, it cannot perform multiple queries with the same name ! So the
solution is :

```

    query {

        apple: company ( id: "1" ) {

            description
        }

        google: company ( id: "2" ) {

            description
        }
    }
```

We can shorten this code up, by using query fragments like this :

```

    query {

        apple: company ( id: "1" ) {

            ...CompanyDetails
        }

        google: company ( id: "2" ) {

            ...CompanyDetails
        }

        fragment CompanyDetails on company {

            description
        }
    }
```

* The difference between queries and mutations is, mutations are used for creation or deletion of data
( as called mutation ) and queries are used for querying data.

## Client Side GraphQL

Install the following libraries in your react app.

```zsh

    npm install --save react-apollo apollo-client graphql-tag
```

Now wrap your exisiting application with the apollo-provider :

```jsx

    import ApolloClient from 'apollo-client';
    import { ApolloProvider } from 'react-apollo';

    export default function App ( ) {

        const client = ApolloClient({});

        return (

            <ApolloProvider client = { client }>
            
                <Layout />
            </ApolloProvider>
        )
    }
```

We can write queries in the client side in this manner :

```js

    import gql from 'graphql-tag';

    const query = gql`
    
        query {

            company ( id: "${ id }") {

                name
                description
            }
        }
    `;
```

After writing the query, we need to bond that with the component, in which we are going to need the
query. This means the query will be automatically sent to our graphql-server, when the component is
renderred. The response data from the graphql-server is passed as props.data.<query-name> to the
component.

```js

    import { graphql } from 'react-apollo';

    export default graphql( query )(

        function Home ( props ) {

            console.log( props.data.company );

            return (

                <div className = 'Home'></div>
            )
        }
    );
```

## Conditional Renderring

this.props.data.loading represents that whether the query/mutation result is loading or not.

```js

    export default graphql( query )(

        function Home ( props ) {

            if ( props.data.loading ) {

                return <div>Loading...</div>;
            } else {

                return (

                    <div className = 'Home'></div>
                );
            }
        }
    );
```

## Clientside mutations

We bind the mutations with the component in the same manner :

```js

    const mutation = gql`
    
        mutation {

            addUser {

                id
            }
        }
    `;

    export default graphql( mutation )( AddUser );
```

Now to execute that mutation query in the component, based on any event, we can do this :

```js

    fuunction submitHandler ( event ) {

        this.props.mutate(

            {
                variables: {

                    title: state.title
                }
            }
        ).then(

            function ( ) {

                // perform some operation on successful mutation
            }
        ).catch(

            function ( ) {

                // on mutation failure
            }
        );
    }
```

The mutate( ) function is passed as props to the component. When we execute the function, the mutation
query is sent to the backend. Wecan also mention refetchQueries ( queries that will be executed after
the mutation was successful ).

```jsx

    this.props.mutate(

        {

            variables: {

                title: state.title
            },

            refetchQieries: [

                { query: query }
            ]
        }
    ).then( ).catch( );
```

## Query variables

Let our query is defined in another file. And we imported it in our component file. Then, we can pass
the query variables in this way.

```js

    import { fetchSong } from '../Queries';

    export default graphql( fetchSong, {

        options: function ( props ) {

            return {

                variables: { id: props.params.id }
            };
        }
    })( SearchSong );
```

## Error Handling

Let us consider, user authenticated. More specifically user signup. Users can run into some errors,
like signing up with an existing email or enetering wrong password. How do we handle those errors !

```js

    function signup ( parent, args, request ) {

        return User.findOne({ email }).then(

            existingUser => {

                if ( existingUser ) {

                    throw new Error('Email in use');
                }

                else {

                    return user.save( );
                }
            }
        );
    }
```

In the client-side, we can access this error:

```js

    this.props.mutate({

        ...
    }).catch(

        response => {

            const errors = response.graphQLErrors.map(

                // error.message is the error that we threw
                error => error.message
            );
        }
    );
```