# GitHub Repository React-Three-Fiber Visualizations!

To use Apollo Client for fetching data from GitHub's GraphQL API, you'll need to first install Apollo Client and its dependencies:

```bash
npm install @apollo/client graphql
```

Now you can modify the `InteractiveModel` component to fetch all repositories along with their name, description, stars, forks, and URL.

Here's how you can integrate Apollo Client to fetch the data:

```tsx
import React, { useEffect, useState, useRef } from 'react';
import { Canvas, useFrame, useThree } from '@react-three/fiber';
import { ApolloClient, InMemoryCache, gql, useQuery } from '@apollo/client';
import { Html } from '@react-three/drei';

// Initialize Apollo Client
const client = new ApolloClient({
  uri: 'https://api.github.com/graphql',
  headers: {
    Authorization: `bearer YOUR_GITHUB_ACCESS_TOKEN_HERE`,
  },
  cache: new InMemoryCache(),
});

// GraphQL Query to fetch repository data
const GET_REPOS = gql`
  query {
    viewer {
      repositories(first: 10) {
        nodes {
          name
          description
          stargazerCount
          forkCount
          url
        }
      }
    }
  }
`;

const InteractiveModel: React.FC = () => {
  const { loading, error, data } = useQuery(GET_REPOS, { client });
  const [isClicked, setIsClicked] = useState(false);
  const modelRef = useRef<any>(null);
  const { camera } = useThree();

  // Animate the model
  useFrame(() => {
    if (modelRef.current) {
      modelRef.current.rotation.y += 0.01;
      if (isClicked) {
        modelRef.current.scale.x = Math.sin(camera.position.z * 0.1) + 1;
        modelRef.current.scale.y = Math.sin(camera.position.z * 0.1) + 1;
        modelRef.current.scale.z = Math.sin(camera.position.z * 0.1) + 1;
      }
    }
  });

  // Handle click event to animate or break apart the model
  const handleClick = () => {
    setIsClicked(true);
  };

  // Assuming the first repository's primary language is JavaScript for demo purposes
  const glowColor = 'blue';

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error :(</p>;

  return (
    <>
      {data.viewer.repositories.nodes.map((repo: any) => (
        <mesh key={repo.name} ref={modelRef} onClick={handleClick}>
          <boxBufferGeometry attach="geometry" />
          <meshStandardMaterial attach="material" color={glowColor} emissiveIntensity={isClicked ? 0 : 1} />
          {isClicked && (
            <Html position={[0, 1, 0]}>
              {/* Display metrics like stars, forks, etc. as HTML elements */}
              <div style={{ color: 'white' }}>
                <p>Name: {repo.name}</p>
                <p>Stars: {repo.stargazerCount}</p>
                <p>Forks: {repo.forkCount}</p>
                <p><a href={repo.url} target="_blank" rel="noreferrer">Go to Repository</a></p>
              </div>
            </Html>
          )}
        </mesh>
      ))}
    </>
  );
};

const App: React.FC = () => {
  return (
    <Canvas>
      <ambientLight />
      <pointLight position={[10, 10, 10]} />
      <InteractiveModel />
    </Canvas>
  );
};

export default App;
```

Please note that you'll need to replace `YOUR_GITHUB_ACCESS_TOKEN_HERE` with your actual GitHub access token.

This component now fetches the list of repositories using Apollo Client and GraphQL. Each repository is represented by a 3D box that rotates. When clicked, the box will "break apart" by changing its scale, and a HUD will appear showing the repository's name, star count, fork count, and a link to the repository.

# To use the `InteractiveModel` component in a React.js and Node project, you'll need to set up a few things

### 1. Setting Up Node Backend (Optional)

If you're planning to use Node.js only as a backend, you can create a simple server using Express or another framework of your choice. This is optional if you don't need any server-side logic.

Create a new directory for your project, navigate into it, and initialize a new Node.js application:

```bash
mkdir my-react-node-app
cd my-react-node-app
npm init -y
```

Install Express:

```bash
npm install express
```

Create an `index.js` for your server:

```javascript
const express = require('express');
const app = express();
const port = 3001; // Make sure this is different from your React app's port

app.get('/api/hello', (req, res) => {
  res.send('Hello World!');
});

app.listen(port, () => {
  console.log(`Server running at http://localhost:${port}/`);
});
```

### 2. Setting Up React Frontend

You can create a new React application using Create React App:

```bash
npx create-react-app client
```

Navigate to your `client` directory:

```bash
cd client
```

Install the necessary packages:

```bash
npm install @apollo/client graphql three @react-three/fiber @react-three/drei
```

Now you can place the `InteractiveModel` component code into a new file, say `InteractiveModel.tsx`, in your `src` directory.

### 3. Run Both Servers

- Start your Node.js server:

  ```bash
  node index.js
  ```

- In a new terminal, navigate to the `client` folder and start your React app:

  ```bash
  npm start
  ```

Your React app should now be running on `http://localhost:3000` and your Node.js server on `http://localhost:3001`.

### 4. Use the Component

In your React app, you can use the `InteractiveModel` component inside your main component file (usually `App.js` or `App.tsx`).

```jsx
// App.tsx

import React from 'react';
import { Canvas } from '@react-three/fiber';
import InteractiveModel from './InteractiveModel';  // Import the component

function App() {
  return (
    <div className="App">
      <Canvas>
        <ambientLight />
        <pointLight position={[10, 10, 10]} />
        <InteractiveModel />  {/* Use the component */}
      </Canvas>
    </div>
  );
}

export default App;
```

Remember to replace `YOUR_GITHUB_ACCESS_TOKEN_HERE` in the `InteractiveModel` component with your actual GitHub access token.

That's the basic setup. Now you have a React.js frontend with a 3D interactive component, running alongside a Node.js backend. You can expand this to fit your specific requirements.


### Side Quest for Passive Income:

Once you have this component polished, you can package it into a library or service that helps developers showcase their GitHub repositories in a unique, interactive way. You could monetize through a subscription model, offering basic features for free and more advanced customization and interactivity at a premium.

