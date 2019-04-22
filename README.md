# TickTock

---

TickTock runs scheduled tasks within Docker containers. Each task can be independently configured to run within an existing container or within a container that is automatically created and subsequently removed. TickTock includes built-in support for sending task notification emails via SMTP. It can also be extended to send notifications using a custom service that you provide in the form of a [Node.js](https://nodejs.org/) script.

The interval at which a task is run is defined using natural language with the help of the [Later](https://bunkat.github.io/later/getting-started.html) module. For example, to execute a task every ten minutes you would simply set an `interval` of `every 10 minutes`. Standard [crontab](https://crontab.guru/) intervals are also supported.

TickTock provides a visual front-end (accessible via the browser) through which you can view execution results.

<img src="misc/ticktock.jpg">

## Sample docker-compose.yml

A configuration file (more on that below) must be mounted into the TickTock container at `/config.yml`.
```
#EDITED#
version: '3.4'

services:

  ticktock:
    container_name: "tixxy"
    image: tkambler/ticktock
    
    volumes:
      - ./config.yml:/config.yml
      - ./db.sqlite:/var/ticktock
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - "8080:80"


```
version: '3.4'

services:

  ticktock:
    image: tkambler/ticktock
    volumes:
      - ./example/config.yml:/config.yml
      - ./data:/var/ticktock
      - /var/run/docker.sock:/var/run/docker.sock
```

## Sample Configuration File (/config.yml)

```
# Username / password for web admin panel
admin:
  username: username
  password: password
  
# Mandatory. An array of task descriptions.
  
#EDITED - MUST USE FORMAT FROM SAMPLE FOR IT TO WORK:#

timezone: America/New_York

admin:
  username: username
  password: password
  
tasks:
  - title: Do Something
    description: It does something very important.
    id: do_something
    interval: every 1000 seconds
    type: run
    image: mhart/alpine-node:8.6.0
    command: ["ls", "-al"]
    overlap: false
    enabled: true
    execute_on_start: false
  - title: List Running Processes
    description: It lists running processes.
    id: list_processes
    interval: "* */1 * * *"
    type: run
    image: mhart/alpine-node:8.6.0
    command: ["ps", "aux"]
    overlap: false
    enabled: true
    execute_on_start: false
email:
  smtp:
    from_name: TickTock
    from_email: ticktock@localhost.site
    config:
      host: maildev
      port: 25
      secure: false
      tls:
        secure: false
        ignoreTLS: true
        rejectUnauthorized: false
```

## Executing Tasks on Demand

Create a terminal session within the running TickTock container and run the script as shown below. You will be presented with a list of available tasks. Make a selection, and it will be immediately executed.

```
$ docker-compose exec ticktock sh
$ ./execute
````
## Generating a Task Report

Create a terminal session within the running TickTock container and run the script as shown below. You will be presented with a list of defined tasks, including the previous and next execution times for each task.

```
$ docker-compose exec ticktock sh
$ ./report
````

## Extending TickTock with Custom Notification Recipients

TickTock can be extended to send task notifications to a custom provider of your choosing. To do so, mount a Node application into the container as shown below.

```
version: '3.4'

services:

  ticktock:
    image: tkambler/ticktock
    volumes:
      - ./example/notifications:/opt/ticktock/notifications
      - ./example/config.yml:/config.yml      
      - /var/run/docker.sock:/var/run/docker.sock
```

In this example, we've mounted a `notifications` folder into the container at `/opt/ticktock/notifications`. This folder must contain a Node script named `index.js` that exports a single function, as shown below.

```
/**
 * @param task	- An object describing the task that was executed.
 * @param res 	- An object (or an array of objects, if batching was enabled) that describes the execution's result(s).
 */
module.exports = (task, res) => {
    
	// Forward to Slack, REST endpoint, IRC, etc...
    
};
```
