# dotnet-demo

Minimal demo of a single page webapp that shows the results of a database query.

Vibe coded with Claude Sonnet 5.

```console
cd /home/bert/Development/dotnet-demo
dotnet new webapp -o TodoApp
cd TodoApp
dotnet add package MySqlConnector
```

Start DB in a podman container

```console
podman pull mariadb:11
podman volume create mariadb-data
podman run -d --name todoappdb -p 3306:3306 -e MARIADB_ROOT_PASSWORD=sekrit -e MARIADB_DATABASE=todo_db -e MARIADB_USER=todo_usr -e MARIADB_PASSWORD=letmeinplz -v mariadb-data:/var/lib/mysql:Z mariadb:11
```

Initialize the database

```console
mariadb -h localhost --port=3306 -utodo_usr -pletmeinplz todo_db << _EOF_
CREATE TABLE IF NOT EXISTS todos (     
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    is_done BOOLEAN NOT NULL DEFAULT FALSE,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);                                                        
_EOF_
```

Start the application with `dotnet run` from the `TodoApp` directory and visit <http://localhost:PORT> in your browser (the port number should be visible in the terminal). You should see a page with a list of todos (which will be empty at first).

## Adding tests

```console
cd /home/bert/Development/dotnet-demo
dotnet new xunit -o TodoApp.Tests
cd TodoApp.Tests
dotnet add reference ../TodoApp/TodoApp.csproj
dotnet add package Microsoft.Extensions.Configuration
dotnet add package Microsoft.Extensions.Configuration.Json
cd ..
dotnet new sln
dotnet sln add TodoApp/TodoApp.csproj TodoApp.Tests/TodoApp.Tests.csproj
```

Create a test database in the MariaDB container:

```console
mariadb -h localhost --port=3306 -uroot -psekrit << '_EOF_'
> CREATE DATABASE IF NOT EXISTS todo_test_db;
GRANT ALL PRIVILEGES ON todo_test_db.* TO 'todo_usr'@'%';
FLUSH PRIVILEGES;
_EOF_
mariadb -h localhost --port=3306 -utodo_usr -pletmeinplz todo_test_db << '_EOF_'
CREATE TABLE IF NOT EXISTS todos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    is_done BOOLEAN NOT NULL DEFAULT FALSE,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);
_EOF_
```

