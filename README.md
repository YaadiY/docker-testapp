# docker-testapp

## Start MongoDB and mongo-express

Create the Docker network if it does not exist:

```powershell
docker network create mongo-network
```

Start MongoDB, replacing the example username and password with your own:

```powershell
docker run -d -p 27017:27017 --name mongo --network mongo-network -e MONGO_INITDB_ROOT_USERNAME=your-mongo-user -e MONGO_INITDB_ROOT_PASSWORD=your-mongo-password mongo
```

Start mongo-express with the same credentials:

```powershell
docker run -d -p 8081:8081 --name mongo-express --network mongo-network -e ME_CONFIG_MONGODB_ADMINUSERNAME=your-mongo-user -e ME_CONFIG_MONGODB_ADMINPASSWORD=your-mongo-password -e ME_CONFIG_MONGODB_URL="mongodb://your-mongo-user:your-mongo-password@mongo:27017" mongo-express
```

Open `http://localhost:8081` to access mongo-express. To inspect its logs:

```powershell
docker logs mongo-express
```

## Start the Node.js server

From the project folder, run:

```powershell
node .\server.js
```

The server listens on port **5050**.

## How the services work together

```text
Your browser
  ├── http://localhost:5050/         → Express app in server.js
  │                                      └── MongoDB driver → MongoDB
  └── http://localhost:8081/         → mongo-express → MongoDB
```

- **Docker** runs programs in separate containers and lets those containers communicate over a Docker network. A container's `localhost` refers to that same container, not another one.
- **MongoDB** stores documents in databases and collections. In this app, `server.js` accesses the `users` collection in `apnacollege-db`. MongoDB normally listens on port **27017**.
- **mongo-express** is an optional web-based admin interface for viewing and managing MongoDB. The Express app connects directly to MongoDB; it does not go through mongo-express.
- **server.js** is the Node.js/Express app. It serves files from `public`, listens on port **5050**, and provides `GET /getUsers` to read users and `POST /addUser` to insert a user.

If MongoDB runs in Docker while `server.js` runs directly on Windows, the app can connect through `localhost:27017` if that port is published. If both run in Docker on the same network, the app should connect using the MongoDB container name or network alias (for example, `mongo`), not `localhost`. The `-p` option publishes a container port to your computer.

Both database routes in `server.js` currently call `client.connect(URL)`, but `URL` is not defined. Use `await client.connect();` instead; the connection URL is already supplied when creating the client. The `POST /addUser` route also needs to send a response after inserting, or the requester may wait indefinitely.

MongoDB is required for the database routes to work. mongo-express is only a convenient way to inspect the database.
