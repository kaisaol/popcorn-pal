# Popcorn Pal 🍿
This is a clone of the original repository.

## Table of contents

1. [Popcorn Pal](##popcorn-pal)

2. [Developer Information](##developer-information)

3. [Design Choices](##design-choises)

4. [Technologies](##technologies)

5. [Testing](##testing)

6. [How To Run](##how-to-run)

7. [How To Test](##how-to-test)

8. [Common Pitfalls](#common-pitfalls)

## Popcorn Pal

Popcorn Pal is a website designed to help you discover the perfect movie or series to watch. It showcases IMDb rankings and top-rated films, making it easy to explore quality options. Whether you're searching for something new or just need a bit of inspiration, Popcorn Pal is here to guide your decision.

### The application and its different pages are explained in the following walkthrough

#### Homepage:

The homepage features a movie carousel showcasing a selection of highlighted films. Below this, there is an overview of the top 10 rated movies, making it easy for users to discover popular titles. Additionally, a navigation section provides quick links to various parts of the site, facilitating straightforward exploration.

#### DiscoverPage

The Discover page provides users with a tool to find a movie by allowing them to sort films based on ranking, duration, title, and release year. Users can also apply filters by categories, making it simple to narrow down options to specific genres or types of films.

#### MoviePage

By clicking on "Read More," you will be directed to a dedicated movie page that displays detailed information about the selected film. This movie page includes the title, a brief description, and the IMDb ranking. Users also have the option to like the movie, add it to their watchlist, and leave a comment in the comment section. The comment section encourages engagement by allowing users to reply to others' comments and vote on comments with upvotes and downvotes, fostering lively discussions and debates

#### WatchlistPage

The Watchlist page offers similar functionality to the Discover page but focuses exclusively on the movies you’ve added to your watchlist. It provides an organized overview of your selected films, allowing you to easily manage your viewing preferences. Additionally, you can filter your watchlist based on your viewing status, such as "Watched," "Want to Watch," and "Currently Watching."

#### FavoriteMoviePage

This section displays an overview of the movies you have liked. It allows you to easily access and revisit your favorite films, providing a quick reference to your preferred choice

#### Profile Page

The profile page provides an overview of a user, displaying their follower count and the number of people they follow. It also includes a link to their watchlist, allowing others to explore their selections for inspiration.

#### For You Page

This page presents results tailored to your interests. As you follow more users, your feed becomes more personalized, recommending both movies and users that match your tastes. This feature helps you discover new content and connect with like-minded individuals in the community.

#### Sign In and Sign Up Pages

The Sign In and Sign Up pages allow users to create an account by providing their name, email address, and creating a password. Once registered, users can easily log in using their email and password.

## Developer Information

Developed by:

- Brage Baugerød
- Bengt Andreas Rotheim
- Kaisa Øyre Larsen
- Brinje Marie Haugli

## Design Choices

### Choice of data

We chose to use movie data for our project. This type of data is very versatile and allows us to showcase advanced components with stuff like images, title, genres and many other subfields. We decided to utilize IMDb for retrieving movie data. This is done in a way that doesn't abuse their services, by only fetching data and populating our own database when a search result falls below a given search score we have defined.

### Choices related to search, filtering and sorting

#### Search

We have two different options for searching: movie-search and user-search. Both options supply real time suggestions and results. To avoid searching too many times, we decided to use a debounce function that only allows fetching results every 300ms.

When searching for a movie, we showcase the result in a paginated manner. Using a trigram algorithm for our movie search, we don't limit the search results only to full matches. We felt this makes the search experience much better, especially since it allows for searching an infinite\* result set.

For the user search we use a modified full match algorithm. A score is given for the results based on some relevancy paramaters, like if the match occured early in the name, or if a fullmatch has occurred.

#### Filtering

We provide filtering options on the pages [DiscoverPage](####DiscoverPage) and [WatchlistPage](####WatchlistPage). On both pages the dataset is filtered on the backend and not locally on the frontend. This means that it filters on the entire dataset instead of just what is loaded in. All filtering options are also stored in session storage, making the user experience much smoother.

#### Sorting

We also provide sorting options on the pages [DiscoverPage](####DiscoverPage) and [WatchlistPage](####WatchlistPage). You can sort on different kinds of fields, and you are also able to order the dataset by ascending or descending order. Here we also sort based on the entire dataset and not only what is available locally. All sorting options are stored in query parameters. The reason for using query parameteres for this functionality over something like local or session storage, is because it then preserves sorting options in the URL. Additionally it provides a way of "sharing" the current state to other users by them being able to share their current URL.

### Choices related to sustainability

Our application relies heavily on images for displaying movie data. This is something we felt we couldn't work around. However, we have taken some measures related to sustainability. Firstly we downscale all images before we request them. This is done on the frontend by setting a width and a height in the image url that closely matches the size required by the component. This is all made possible by IMDb having a very flexible CDN setup for images. This solution drastically improves performance as many of the images are downscaled by 10x.

We also try to leverage the apollo local state as efficiently as possible. This is in order to reduce the amount of requests we need to send to the backend when data is mutated. This means that a page, unless needing to fetch additional data, will only send a couple of requests.

### Choices related to accessibility

We heavily make use of Radix Primitives for building blocks for our components. This library provides a wide range of base components that follows the WAI-AREA guidelines for accessibility. It takes care of many details related to accessibility, like including `role` and `aria` attributes, focus management, and keyboard navigation.

In components that are not built using Radix Primitives, we provide accessible features by providing the attributes ourselves. By utilizing a strict linter like biome, we could easily spot and fix accessibility issues.

### Choices related to global state management

We use apollo local state for global state management. Apollo client provides `InMemoryCache` which is a powerful state management tool. We use their merge functions where possible and then manually update state where it makes sense. This results in no extra requests being made for fetching invalidated data after mutations, except for mutations in `WatchlistPage`.

In `WatchlistPage` all data is made invalid when a mutation change related to adding/removing watchlist items happens. This is because the data simply doesn't make sense to update locally as it would require way to complicated logic to order the mutated object locally according to our sorting and filtering options. This means that the invididual watchlist items are managed by the state, but the paginated result is not. We also stumbled upon a limitation in apollo local state here. There is no way (that we could find) to get objects from the apollo state based on a partial key. This means that we had to develop our own utility function for iterating over the state and invalidating partial keyed objects.

### Choices related to reusable code

Our components are heavily inspired by the "building block" philosophy that Radix Primitives and shadcn uses. This means that almost all our components are easy and intuitive to use, while also providing flexibility.

Our component philosophy is to always try to provide components that are built by our own building blocks. In order to easily distinguish between the role of a component, and how it would be used, we decided to use an atomic component folder structure. This folder structure allows us to put smaller and more primitive components in "atoms", components that usually uses a couple atoms in while being a bit more complicated in "molecules", and the most complex components in organisms. Typical components that fall into the organisms category are components like the navbar, larger sections and complex data components like `EditProfileModal` and `FilterableMovieSection`.

We rarely implement larger sections of a page directly in the page component. This is to make our code as reusable as possible, while giving individual components more responsibility for itself. We do however often keep state and use data fetching hooks in the "root" component. This is to have one "controlling" component that manages many smaller or bigger components. These larger data fetching implementations can therefore often be seen in the page components. This way of doing things also makes the code easier to read for the entire team as you don't have to look through all components in order to understand how data fetching works in the component chain.

For data querying and mutations we provide custom hooks. This is because many of the operations require state management after it has finished executing. This logic is common for all implementations, and therefore providing them in hooks makes our code as reusable as possible while still being easy to read.

## Technologies

### Frontend

- `React` and `Typescript`
  - We use React with typescript for this application. React enables dynamic, component-based interfaces that are efficient and scalable. TypeScript adds type safety, which reduces bugs and improves reliability in the code.
- `Radix Primitives`
  - We use Radix Primitives for accessible base components. More about this library and our usage of it can be read in the [Choices related to accessibility](###Choices related to accessibility) section.
- `React Router`
  - We use React Router for multiple page support in the single page app. By using the `Link`-component that the library provides, we also achieve extremely smooth routing between pages.
- `Tailwind CSS` and `Radix Colors`
  - Tailwind CSS is used in our entire frontend for styling components and elements. Their easy to use and quick to write philosophy not only provides a great developer experience, but it also provides multiple browser support out of the box. The only thing we decided to swap out was the default tailwind colors. We opted to use a better color scale which we felt like Radix Colors provided.
- `Apollo Client` (graphql)
  - We use Apollo Client in order to query and mutate data on the backend using graphql. It also provides a state management system that we have heavily utilized.
- `aHooks`
  - For a couple of additional react hooks we use a library called aHooks. This provides powerful hooks like `useResponsive()`, `useSessionStorageState()` and many more.
- `nuqs`
  - We use nuqs which is a library for keeping state in query parameters. Their implementation provides type safe functionality that are easy to read and work with.
- `react-hook-form` and `zod`
  - We use react hook form for form-control and zod for schema validation. These two libraries used in conjuction with each other makes the developer experience very smooth.
- `react-toastify`
  - We use react-toastify for displaying messages to the end-user. We opted to use this library instead of creating our own solution as theirs are very well tested as well as it having a great design.
- `Storybook`
  - Although it isn't a technology that directly impacts the user experience, we wanted to mention our use of storybook. By creating stories for each component (where it makes sense), we can develop and test the component in a controlled manner without having to launch the frontend. Their easy to use web-interface also provides a kind of "component documentation" which is very useful when working in a team.

### Backend

Our backend is made using `Apollo Server` and `express.js` as the foundation. We also use a variety of other technologies:

- `PostgreSQL` and `kysely`
  - We use PostgresSQL as our database. This relational SQL database is perfect for our relational needs, as well as it scales very well for larger applications. In order to write type safe sql queries, we opted to use `kysely`. This library provides a thin abstraction layer over SQL which was perfect for our application. It also provides migration-functionality, and in conjunction with our self-developed migration CLI, we can make changes to the database schema in a controlled manner.
- `Minio`
  - We use minio in order to store images on our VM. Minio is a s3 compatible storage solution which provides a great developer experience while being secure and efficient.
- `Zod`
  - We also use Zod on the backend. Zod allows us to create easy to use type schemas and validation functionality. A lot of our resolvers utilize zod in order to validate the input data efficiently.

## Testing

**Cypress**

Cypress is used for end-to-end testing to validate key features like search, navigation, filtering, and dark/light mode toggling. It simulates user interactions, helping to identify issues and ensure a reliable user experience.

**Unit tests**

We employ unit testing to verify the functionality of individual components in our applications. This practice helps identify bugs early, improves code quality, and ensures that changes do not introduce new issues.

## How To Run

This project is split into two separate subprojects, `frontend` and `backend`. These are individual packages that must be run from their respective folders. Follow their README's for more instructions on how to install and run each of them.

**NOTE**
The actual environment variables needed to run the application can be found in our VM at `/home/bragebau/.env`. If you want to copy these to be used locally, please put the `.env` file in the `/backend` folder for everything to work.

- [Frontend Instructions](./frontend/README.md)
- [Backend Instructions](./backend/README.md)

## How To Test

### End to end tests

**NOTE:** The backend must be running locally for end-to-end tests to work. Follow the guide in the [backend README](./backend/README.md) on how to start the backend locally.  
(The above note was added after the submission date. Hopefully that is okay)

Open a new terminal

navigate to the frontend directory

```
cd frontend
```

run cypress:

```
npm run cy:run
```

### Components tests
To run all tests for components, make sure you are currently in the frontend folder, and run:
```
npm run test
```
## Common pitfalls

- **Unresponsive page**
  - If the page looks to be unresponsive, make sure that you are connected to the NTNU network either through eduroam or using the NTNU VPN.
  - Another issue causing the unresponsiveness can be that the backend isn't running properly. If this might be the case, please follow `Running the application on our VM`-guide from the [backend readme](./backend/README.md) on how to restart the backend on our VM.
- **Missing environment variables**
  - Make sure that you are using the correct environment variables. You can read more about this [here](##how-to-run)
