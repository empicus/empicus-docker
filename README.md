# Empicus stack built with Docker Compose

A basic Empicus stack environment built using Docker Compose. It consists of the following:

* Data Container (MySQL)
* Client Portal Container (Wildfly)
* Admin Portal Container (Wildfly)
* Core Platform Container (Wildfly)
* Apache Httpd Container (Optional reverse proxy)

### Prerequisites

Download Docker Desktop for Mac or Windows. Docker Compose will be automatically installed. On Linux, make sure you have the latest version of Compose.

### Installing

Clone this repository.

```
git clone git@github.com:empicus/empicus-docker.git
```

Configure the .env file as needed.

```
cd empicus-docker
// modify sample.env as needed
```

Run containers.

```
docker-compose up -d
```

Your Empicus stack is now ready!! You can access it via `http://localhost`.

##  Configuration and Usage

### General Information 
This Docker Stack is build for local development and not for production usage.

### Configuration
This package comes with default configuration options. You can modify them by creating `.env` file in your root directory.
To make it easy, just copy the content from `sample.env` file and update the environment variable values as per your need.

### Configuration Variables
There are following configuration variables available and you can customize them by overwritting in your own `.env` file.


## Authors

* **Edward Banfa** - *Initial work* - [Twitter Profile](https://twitter.com/EdwardBanfa)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
