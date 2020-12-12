# API Technologies Stack {#Infrastructure}

<!--  You can label chapter and section titles using `{#label}` after them, e.g., we can reference Chapter \@ref(intro). If you do not manually label them, there will be automatic labels anyway, e.g., Chapter \@ref(methods).-->

<!-- Scraping function in the way they are designed are not portable and can only be executed locally. One way to scale up code is to make it available as API. -->

In order to provide a fast and secure API service to the end user many technologies have been involved. Challenges in scraping as pointed out in section \@ref(challenges) are many and still some remains unsolved. Challenges not only regard scraping per se, but also the way and how many times the service has to interact with users. Some of the main obstacle to tackle in dealing with users are:

- The API has to be executed $n$ given times at fixed date-time daily and it has to store resulting data on a cloud database. This is done with the explicit goal of tracking the evolution of the phenomenon in time.
- The API has to be very fast otherwise data can not be consumed.
- The API has to be deployed so that it can be shared over different stakeholders, without having them to know what it takes. 
- The API maintainer needs to take action be to due to sudden unpredictable immobiliare.it  changes, thus it needs to be continuously maintained and updated.
- On the other code behind the service has to be version controlled and freezed, so that the service can guarantee continuity and prevent failures. 
-  service has to be scalable at need since, due to deployment, when the number of users increases the run time performances should not decrease.
- In addition API inbound traffic has to be managed both in terms traffic and security by securing access only to the ones authorized.

The long list of requirements is met by many DevOps technologies, some of them offer a direct R embedding. As a result the technologies stack is the intersection between a comprehensive documentation available and the requirements listed.
Fo these reasons the recipe is to provide a REST Plumber API with 4 endpoints each of which calls scraping functions in Parallel built in section \@ref(scraping). On top of that a daily Cron Job scheduler exposes one precise API endpoint, which produces and later stores a .csv file in a NOSQL mongoDB Atlas could database. Containerization happens through a Linux OS (Ubuntu distr) Docker container hosted by a free tier AWS EC2 server machine equipped with 30GB max capacity. API endpoints are secured with https protocols, load balanced and protected with authentication by NGINX reverse proxy server. On a second server a Shiny App calls one endpoint gievn specified parameters that returns data from the former infrastructure. A sketch of the infrastructure is represented in figure \@ref(fig:CompleteStructure).

Technologies involved are:

- GitHub version control
- Scheduler cron job, section \@ref(scheduler)
- Plumber REST API, section \@ref(plumberapi)
- Docker containers, section \@ref(docker)
- AWS (Amazon Web Services) EC2 \@ref(aws)
- NGINX reverse proxy, section \@ref(nginx)
- MongoDB Atlas
- Shiny


![(#fig:CompleteStructure)complete infrastructure, author's source](images/tot_infra.jpg)

As a side note each single part of this thesis has been made according to some of the API inspiring criteria of sharing and self containerization. RMarkdown [@rmarkdown1] documents (book's chapters) are compiled and then converted into .html files. Through Bookdown [@bookdown2] the resulting documents are put together according to general .yml instruction file and are readble as gitbook.
Files are then pushed to a [Github repository](https://github.com/NiccoloSalvini/thesis). By a simple trick with GH pages, .html files are dispalyed into a Github subdomain hosted at [link](https://niccolosalvini.github.io/thesis/).  The resulting  deployed gitbook can also produce a .pdf version output through a Xelatex engine. Xelatex compiles .Rmd documents according to a .tex template which formatting rules are contained in a further .yml file. The pdf version of the thesis can be obtained by clicking the download button, then choosing pdf output version in the upper banner. For further references on the topic @bookdown2

Some of the main technologies implied will be viewed singularly, nonetheless for brevity reasons the rest needs to be skipped.


## Scheduler{#scheduler}

\BeginKnitrBlock{definition}\iffalse{-91-83-99-104-101-100-117-108-101-114-93-}\fi{}<div class="definition"><span class="definition" id="def:scheduler"><strong>(\#def:scheduler)  \iffalse (Scheduler) \fi{} </strong></span>A Scheduler in a process is a component on a OS that allows the computer to decide which activity is going to be executed. In the context of multi-programming it is thought as a tool to keep CPU occupied as much as possible.</div>\EndKnitrBlock{definition}

As an example it can trigger a process while some other is still waiting to finish. There are many type of scheduler and they are based on the frequency of times they are executed considering a certain closed time neighbor.

- Short term scheduler: it can trigger and queue the "ready to go" tasks
  - with pre-emption 
  - without pre-emption

The ST scheduler selects the process and It gains control of the CPU by the dispatcher. In this context we can define latency as the time needed to stop a process and to start a new one. 

- Medium term scheduler 
- Long term scheduler

for some other useful but beyond the scope references, such as the scheduling algorithm the reader can refer to [@wiki:scheduler].

### Cron Jobs
\BeginKnitrBlock{definition}\iffalse{-91-67-114-111-110-106-111-98-93-}\fi{}<div class="definition"><span class="definition" id="def:cronjob"><strong>(\#def:cronjob)  \iffalse (Cronjob) \fi{} </strong></span>Cron job is a software utility which acts as a time-based job scheduler in Unix-like OS. Linux users that set up and maintain software environments exploit cron to schedule their day-to-day routines to run periodically at fixed times, dates, or intervals. It typically automates system maintenance but its usage is very flexible to whichever needed. It is lightweight and it is widely used since it is a common option for Linux users.</div>\EndKnitrBlock{definition}
The tasks by cron are driven by a crontab file, which is a configuration file that specifies a set of commands to run periodically on a given schedule. The crontab files are stored where the lists of jobs and other instructions to the cron daemon are kept.
Each line of a crontab file represents a job, and the composition follows the syntax in figure \@ref(fig:crontab)

![(#fig:crontab)Crontab Scheduling Syntax](images/crontab.PNG)

Each line of a crontab file represents a job. This example runs a shell named scheduler.sh at 23:45 (11:45 PM) every Saturday. .sh commands can update mails and other minor routines.

45 23 * * 6 /home/oracle/scripts/scheduler.sh

Some rather unusual scheduling definitions for crontabs can be found in this reference [@wiki:cronjob]. Crontab's syntax completion can be made easier through [this](https://crontab.guru/) GUI.  

The cron job needs to be ran on scraping fucntions at 11:30 PM every single day. The get_data.R script first sources an endpoint function, then it applies the function with fixed parameters. Parameters describe the url specification, so that each time the scheduler runs the get_data.R collects data from the same source. Day after day .json files are generated and then stored into a NOSQL *mongoDB* database whose credentials are public. Data are collected on a daily basis with the explicit aim to track day-by-day changes both in the new entries an goners in rental market, and to investigate the evolution of price differentials over time. Spatio-Temporal modeling is still quite unexplored, data is saved for future used. Crontab configuration for daily 11:30 PM schedules has this appearance:

30 11 * * * /home/oracle/scripts/get_data.R

To a certain extent what it has been already presented since now might fit for personal use. A scheduler can daily execute the scraping script and can generate a .csv file. Later the same .csv file can be parsed into an application and analysis can be locally reported. The solution proposed is totally _not feasible_ in a production environment, since in order to be executed a vast number files has to be sourced and a number of functions should be routinely called. For these reasons the present architecture can not be shared. The solution adopted tries to minimize the analyst/scientist involvement into scraping procedures by offering a compact and fast (due to Parallel execution) service that manages all the processes without having to know how scraping under the hood is working. 

## REST API 

\BeginKnitrBlock{definition}\iffalse{-91-65-80-73-93-}\fi{}<div class="definition"><span class="definition" id="def:api"><strong>(\#def:api)  \iffalse (API) \fi{} </strong></span>API stands for application programming interface and it is a set of definitions and protocols for building and integrating application software. Most importantly APIs let a product or a service communicate with other products and services without having to know how they’re implemented.</div>\EndKnitrBlock{definition}
APIs are thought of as contracts, with documentation that represents an general agreement between parties.
There are many types of APIs that exploit different media and architectures to communicate with apps or services.
\BeginKnitrBlock{definition}\iffalse{-91-82-69-83-84-93-}\fi{}<div class="definition"><span class="definition" id="def:rest"><strong>(\#def:rest)  \iffalse (REST) \fi{} </strong></span>The specification REST stands for **RE**presentational **S**tate **T**ransfer and is a set of _architectural principles_. </div>\EndKnitrBlock{definition}
When a request is made through a REST API it transfers a representation of the state to the requester. This representation, is submitted in one out of the many available formats via HTTP: JSON (Javascript Object Notation), HTML, XLT, TXT. JSON is the most popular because it is language agnostic [@what_is_a_rest_api], as well as it is more comfortable to be read and parsed.
In order for an API to be considered REST, it has to conform to these criteria:

- A client-server architecture made up of clients, servers, and resources, with requests managed through HTTP.
- Stateless client-server communication, meaning no client information is stored between requests and each request is separate and unconnected.
- Cacheable data that streamlines client-server interactions.
- A uniform interface between components so that information is transferred in a standard form. This requires that:
  - resources requested are identifiable and separate from the representations sent to the client.
  - resources can be manipulated by the client via the representation they receive because the representation contains enough information to do so.
  - self-descriptive messages returned to the client have enough information to describe how the client should process it.
  - hypermedia, meaning that after accessing a resource the client should be able to use hyperlinks to find all other currently available actions they can take.
- A layered system that organizes each type of server (those responsible for security, load-balancing, etc.) involved the retrieval of requested information into hierarchies, invisible to the client.

REST API accepts http requests as input and elaborates them through end points. An end point identifies the operation through traditional http methods (e.g. /GET /POST) that the API caller wants to perform. Further documentation and differences between HTTP and REST API can be found to this [reference](https://docs.aws.amazon.com/it_it/apigateway/latest/developerguide/http-api-vs-rest.html).

open REST API examples: 
- BigQuery API: A data platform for customers to create, manage, share and query data.
- YouTube Data API v3: The YouTube Data API v3 is an API that provides access to YouTube data, such as videos, playlists, and channels.
- Cloud Natural Language API: Provides natural language understanding technologies, such as sentiment analysis, entity recognition, entity sentiment analysis, and other text annotations, to developers.
- Skyscanner Flight Search API: The Skyscanner API lets you search for flights & get flight prices from Skyscanner's database of prices, as well as get live quotes directly from ticketing agencies.
- Openweathermap API: current weather data for any location on Earth including over 200,000 cities.

![API general functioning functioning](images/Rest-API.png)

### Plumber REST API{#plumberapi}

Plumber allows the user to create a REST API by adding decoration comments to the existing R code, in this case to scraping code. Decorations are a special type of comments that suggests to Plumber where and when the API specifications parts are. Below a simple example extracted by the documentation:


```r
# plumber.R

#* Echo back the input
#* @param msg The message to echo
#* @get /echo
function(msg="") {
  list(msg = paste0("The message is: '", msg, "'"))
}

#* Plot a histogram
#* @serializer png
#* @get /plot
function() {
  rand = rnorm(100)
  hist(rand)
}

#* Return the sum of two numbers
#* @param a The first number to add
#* @param b The second number to add
#* @post /sum
function(a, b) {
  as.numeric(a) + as.numeric(b)
}
```

three endpoints associated to 2 /GET and 1 /POST requests are made available. Functions are made clear without names so that whenever the endpoint is called functions are directly executed.
Decorations are marked as this `#*` and they are followed by specific keywords denoted with `@`.
- the `@params` keyword refers to parameter that specifies the corpus of the HTTP request, i.e. the inputs with respect to the expected output. If default parameters are inputted then the API response is the elaboration of the functions with default parameters. As opposite endpoint function elaborates the provided parameters and returns a response.
- `#* @serializer` specifies the extension of the output file when needed.
- `#* @get`  specifies the method of HTTP request sent.
- `/echo` is the end point name.
- `@filter` decorations activates a filter layer which are used to track logs and to parse request before passing the argbody to the end points.

Many more options are available to customize plumber API but are beyond the scope, a valuable resource for further insights can be found in the dedicated package website [@an_api_generator_for_r]

### Immobiliare.it _Parallel_ REST API

(Sanitization, antidossing, logs tracking)

The API service is composed by 4 endpoints */scrape* , */links*, */complete* and */get_data*:

- */scrape performs a fast Parallel scraping of the website that leverages a rooted tree shortest path to get to data (250 X 5 predictors in $\approx 10.91^{''}$). This comes at the cost of the number of available covariates to scrape which are:  title, price, number of rooms, sqmeter, primarykey. By default the end point scrape data from Milan real estate rents. It is a superficial and does not contain geospatial, however it might fit for some basic regression settings. The macrozone parameter allows to specify the NIL (Nucleo Identità Locale), targeting very detailed zones in some of the cities for which is available (Roma, Firenze, Milano, Torino). 

- */links: extracts the list of each single advertisement link belonging to each of the npages parameter specified, recall section \@ref(webstructure). It displays sufficient performances in terms of run time. It is strictly needed to apply the following endpoint. .thesis options secures a pre combined url with the data wanted for thesis analysis. The option takes care to decompose the website structure of the url supplied with the aim to apply scraping function in the /complete endpoint.

- */complete:  both the function all.links and complete are sourced. The former with the aim to grab each single links and store it into an object. The latter to actually iterate Parallel scraping on each of the extracted link. 

- */get_data: it triggers the data extraction by sourcing the /complete endpoint and then storing .json file into the NOSQL mongoDB ATLAS


![Swagger UI screenshot, author's source](images/swagger.PNG)

### REST API documentation{#APIdocs}


- Get FAST data, it covers 5 covariates:
```
      GET */scrape
      @param city [chr string] the city you are interested in (e.g. "roma", "milano", "firenze"--> lowercase, without accent)
      @param npages [positive integer] number of pages to scrape, default = 10, min  = 2, max = 300
      @param type [chr string] "affitto" = rents, "vendita"  = sell 
      @param macrozone [chr string] avail: Roma, Firenze, Milano, Torino; e.g. "fiera", "centro", "bellariva", "parioli" 
      content-type: application/json 
```
- Get all the links 

```
      GET */link
      @param city [chr string] the city you are interested to extract data (lowercase without accent)
      @param npages [positive integer] number of pages to scrape default = 10, min  = 2, max = 300
      @param type [chr string] "affitto" = rents, "vendita"  = sell 
      @param .thesis [logical] data used for master thesis
      content-type: application/json 
```   
      
-  Get the complete set of covariates (52) from each single links, takes a while

```
      GET */complete
      @param city [chr string] the city you are interested to extract data (lowercase without accent)
      @param npages [positive integer] number of pages to scrape default = 10, min  = 2, max = 300
      @param type [chr string] "affitto" = rents, "vendita"  = sell 
      @param .thesis [logical] data used for master thesis
      content-type: application/json
```

Up to this point the API can smoothly run in local and potentially can be deployed to share results without requesting scraping process knowledge. However the API software for now is not portable and is very heavy. In addition it can also run into failures for many reasons, one among the others is package versione incompatibility due to updates. In the end it also fully relies on the laptop computational power that can be heavily stressed when a number of API calls are executed, especially for single threaded programming languages as R.
The approach followed proposes a dedicated lightweight software environment that minimizes dependencies both improving performances and enabling the _cloud computing_ coverage. A fast growing techonlogy is what fits the need.

## Docker{#docker}

\BeginKnitrBlock{definition}\iffalse{-91-68-111-99-107-101-114-93-}\fi{}<div class="definition"><span class="definition" id="def:docker"><strong>(\#def:docker)  \iffalse (Docker) \fi{} </strong></span>_Docker_ [@docker] is a software tool to create and deploy applications using containers.
_Docker containers_ are a standard unit of software (i.e. software boxes) where everything needed for applications, such as libraries or dependencies can be run reliably and quickly. Containers are also portable, in the sense that they can be taken from one computing environment to the following without further adaptations.</div>\EndKnitrBlock{definition}

Containers can be thought as an abstraction that groups code and dependencies together. One major advantage of containers is that multiple containers can run on the same machine with the same OS with their specific dependencies. Each container can run its own isolated process in the user space, so that each task/application is complementary to the other. The fact that containers are treated singularly enables a collaborative framework that it also simplifies bugs isolation. 

<!-- Containers are lightweight and take up less space than Virtual Machines (container images are files which can take up typically tens of MBs in size), can handle more applications and require fewer Virtual Machines and OS. The structure differences are figrued in \@ref(fig:(#fig:dockervsvm)). -->


<!-- ![(#fig:dockervsvm)Docker containers versus Virtual Machines, ](images/dockerVSvirtualmachines.PNG) -->

When images are built _Docker container_ are created and can be open sourced through Docker Hub.
_Docker Hub_ is a web service provided by Docker for searching and sharing container images with other teams or developers in the community. Docker Hub can connect with GitHub behind authorization entailing an image version control tool. Once the connection is established  changes that are pushed with git to the GitHub repository are passed to Docker Hub. The push command automatically triggers the image building. Then docker image can be tagged (salvini/api-immobiliare:latest)so that on one hand it is recognizable and on the other can be reused in the future. Once the building stage is completed the DH repository can be pulled and then run locally on machine or cloud, see section \@ref(aws).
Docker building and testing images can be very time consuming. R packages can take a long time to install because code has to be compiled, especially if using R on a Linux server or in a Docker container. 
Rstudio [package manager](https://packagemanager.rstudio.com/client/#/) includes beta support for pre-compiled R packages that can be installed faster. This dramatically reduces packages time installation [@nolis_2020].
In addition to that an open source project [rocker](https://www.rocker-project.org/images/) has narrowed the path for developers by building custom R docker images for a wide range of usages. What can be read from their own website about the project is: "The rocker project provides a collection of containers suited for different needs. find a base image to extend or images with popular software and optimized libraries pre-installed. Get the latest version or a reproducible fixed environment". 

### Why Docker

[Indeed](https://it.indeed.com/), an employment-related search engine, released an article on 2019 displaying changing trends from 2015 to 2019 in Technology Job market, a summary of those changes is in figure \@ref(fig:indeedstats). Many changes are relevant in key technologies. Two among the others technologies (i.e. docker and Azure, arrow pointed) have experienced a huge growth and both refer to the certain demand input: _containers_ and _cloud computing_.
The landscape of Data Science is changing from reporting to application building:
In 2015 - Businesses reports drive better decisions.
In 2020 - Businesses need apps to empower better decision making at all levels.

![(#fig:indeedstats)Indeed top skills for 2019 in percent changes, @top_tech_skills source](images/Inkedindeed_jobs_LI.jpg)

For all the things said what docker is bringing to business are [@red_hat_customer_portal]:

- _Speed application deployment_ : containers include the minimal run time requirements of the application, reducing their size and allowing them to be deployed quickly.
- _Portability across machines_ : an application and all its dependencies can be bundled into a single container that is independent from the host version of Linux kernel, platform distribution, or deployment model. This container can be transferred to another machine that runs Docker, and executed there without compatibility issues.
- _Version control and component reuse_ : you can track successive versions of a container, inspect differences, or roll-back to previous versions. Containers reuse components from the preceding layers, which makes them noticeably lightweight. In addition due to Docker Hub it is possible to establish a connection between Git and DockerHub. Vesion
- _Sharing_ : you can use a remote repository to share your container with others. It is also possible to configure a private repository hosted on Docker Hub.
- _Lightweight footprint and minimal overhead_ : Docker images are typically very small, which facilitates rapid delivery and reduces the time to deploy new application containers.
- _Fault isolation_ :Docker reduces effort and risk of problems with application dependencies. Docker also freezes the environment to the preferred packages version so that it guarantees continuity in deployment and isolate the container from system fails coming from package version updates.

The way to tell docker which system requirements are needed in the newly born software is a _Dockerfile_.

### Dockerfile{#dockerfile}

Docker can build images automatically by reading instructions from a Dockerfile. A Dockerfile is a text document that contains all the commands/rules a generic user could call on the CLI to assemble an image. Executing the command `docker build` from shell the user can trigger the image building. That executes sequentially several command-line instructions. For thesis purposes a Dockerfile is written with the specific instructions and then the file is pushed to GitHub repository. Once pushed DockerHub automatically parses the repository looking for a plain text file whose name is "Dockerfile". When It is matched then it triggers the building of the image.

The Dockerfile used to trigger the building of the docker container has the following sequential set of instructions in figure \@ref(fig:dockerfile)) :

![(#fig:dockerfile)Example of a Dockerfile from Docker Hub, author's source](images/dockerfile.PNG)

where the instructions are:

- `FROM rocker/tidyverse:latest` : The command imports a pre-built image by the rocker team that contains the latest (tag latest) version of base-R along with the tidyverse packages.


- `MAINTAINER Niccolo Salvini "niccolo.salvini27@gmail.com"` : The command tags the maintainer and its e-mail contact information.


- `RUN apt-get update && apt-get install -y \ libxml2-dev \ libudunits2-dev` :The command update and install Linux dependencies needed for running R packages. `rvest` requires libxml2-dev and `magrittr` needs libudunits2-dev. If they are not installed then associated libraries can not be loaded. Linux dependencies needed have been found by trial and error while building containers. Building  logs messages print errors and suggest which dependency is mandatory.


- `RUN R -e "install.packages(c('plumber','tibble','...',dependencies=TRUE)` : the command install all the packages required to execute the files (R files) containerized for the scraping. Since all the packages have their direct R dependencies the option `dependencies=TRUE` is needed. 


- `RUN R -e "install.packages('https://cran.r-project.org/.../iterators, type='source')`
`RUN R -e "install.packages('https://cran.r-project.org/.../foreach/, type='source')`
`RUN R -e "install.packages('https://cran.r-project.org/.../doParallel, type='source')`
DoParallel was not available in package manager for R version later than 4.0.0. For this reason the choice was to install a previous source version by the online repository, as well as its dependencies.


- `COPY \\` The command tells Docker copies all the files in the container.


- `EXPOSE 8000` :  the commands instructs Docker that the container listens on the specified network ports 8000 at runtime. It is possible to specify whether the port exposed listens on UDP or TCP, the default is TCP (this part needs a previous set up previous installing, for further online documentation It is recommended [@docker_documentation_2020] )

- `ENTRYPOINT ["Rscript", "main.R"]` : the command tells docker to execute the file main.R within the container that triggers the API start. In main.R it are pecified both the port and the host where API expects to be exposed (in this case port 8000). 

In order to make the system stand-alone and make the service available to a wider range of subjects a choice has to be made. The service has to have both the characteristics to be run on demand and to specify query parameters. 


## AWS EC2 instance{#aws}

Exporting the API on a server allows to make scraping available to a various number of services thorough multitude of subjects. Since it can not be specified a-priori how many times and users are going to enjoy the service a scalable solutio might fill the needs. Scalable infrastructure through a flexible cloud provider combined with nginx load balancing can offer a stable and reliable infrastructure for a relatively cheap price.
AWS offers a wide range of services each of which for a wide range of budgets and integration. Free tier servers can be rent up to a certain amount of storage and computation that nearly 0s the total bill. The cloud provider also has a dedicated webpage to configure the service needed with respect to the usage named [amazon cost manager](https://aws.amazon.com/en/aws-cost-management/).

\BeginKnitrBlock{definition}\iffalse{-91-65-87-83-32-69-67-50-93-}\fi{}<div class="definition"><span class="definition" id="def:aws"><strong>(\#def:aws)  \iffalse (AWS EC2) \fi{} </strong></span>Amazon Elastic Compute Cloud (EC2) is a web service that contributes to a secure, flexible computing capacity in the AWS cloud. EC2 allows to rent as many virtual servers as needed with customized capacity, security and storage.
</div>\EndKnitrBlock{definition}
[few words still on EC2]

### Launch an EC2 instance

The preliminary step is to pick up an AMI (Amazon Machine Image). AWS AMI are already pre set-up machines with standardized specifications built with the purpose to speed up choosing the a customed machine. Since the project is planned to be nearly 0-cost a “Free Tier Eligible” Linux server is chosen. By checking the Free Tier box all the available free tiers machines are displayed. The machine selected has this specification: t2.micro with 1 CPU and 1GB RAM and runs on a Ubuntu distribution OS. First set up settings needs to be left as-is, networking and VPC can always be updated when needed. In the "add storage" step 30 GB storage are selected, moreover 30 represent the upper limit since the server can be considered free tier. Tags windows are beyond the scope. Secondly configuration needs to account security and a new entry below SSH connection (port 22) has to be set in. New security configuration has to have TCP specification and should be associated to port 8000. Port 8000, as in dockerfile section \@ref(dockerfile), has been exposed and needs to be linked to the security port opened. 

![aws_dashboard](images/aws.PNG)

At this point instance is prepared to run and in a few minutes is deployed. Key pairs, if never done before, are generated and a .pem file is saved and securely stored. Key pairs are a mandatory step to log into the Ubuntu server via SSH. SSH connection in Windows OS can be handled with [PuTTY](https://www.putty.org/), which is a SSH and telnet client designed for Windows. At first PuTTYgen, a PuTTY extensions, has to convert the key pair .pem  file into a .ppk extension (otherwise PuTTY can not read it). Once .ppk is converted is immediately sourced in the authorization panel. If everything works and authentication is verified then the Ubuntu server CLI appears and interaction with the server is made possible. 
Once the CLI pops out some Linux libraries to check file structure ("tree") and Docker are installed. Then a connection with Docker hub is established providing user login credentials. From the Hub repository the container image is pulled on the machine and is then executed with the docker RUN command. 
AWS automatically assign to the server a unique Public DNS address which is going to be the REST API url to call. 
the Public DNS has the following form:

`ec2-15-161-94-121.eu-south-1.compute.amazonaws.com`


## NGINX reverse proxy server{#nginx}

For analysis purposes NGINX is open source software for reverse proxying and load balancing.
Proxying is typically used to distribute the load among several servers, seamlessly show content from different websites, or pass requests for processing to application servers over protocols other than HTTP.
[...]

When NGINX proxies a request, it sends the request to a specified proxied server, fetches the response, and sends it back to the client. It is possible to proxy requests to an HTTP server (another NGINX server or any other server) or a non-HTTP server (which can run an application developed with a specific framework, such as PHP or Python) using a specified protocol. Supported protocols include FastCGI, uwsgi, SCGI, and memcached.
[...]


.conf file and installation on Linux server. Security and Authentication. 


## Software development Workflow 


![(#fig:sfmap)Sofwtare development workflow, author's source](images/SoftwareDevWF.jpg)

## Further Integrations
From a software point of view a more robust code can be obtained embedding recent R software development frameworks as `Golem` @colin_fay_2020 into the existing code. The framework stimulates the usage of modules According to the latest literature APIs (as well as Shiny) should be treated as R packages (@plungr aligns with that) as argued in section 4.2 @colin_fay_2020 and in the [comment](https://deanattali.com/2015/04/21/r-package-shiny-app/) by Dean Attali. As a consequence of that *TDD* (i.e.Test-Driven Development @TDD_2004) through the tools of @usethis and @testthat during package building can make software more robust, organized and production graded. Loadtest @loadtest can help figure out 
CI/CT
It can be missed to cite a popular API development integration  service and automate testing tool [Postman](https://www.postman.com/), which does the best when POST requests endpoints are served, since for the moment they are not required it is not used.

Pins is an r packages [this link](https://rstudio.com/resources/rstudioconf-2020/deploying-end-to-end-data-science-with-shiny-plumber-and-pins/?mkt_tok=eyJpIjoiTmprNU1USXhPVEprWXpNMSIsInQiOiJtTUhKVzlvSjVIV2hKc0NRNVU1NTRQYSsrRGd5MWMyemlTazQ5b1lHRGJXNVBLcnpScjZRaWVcL2JGUjBPNGIwV3pwY1dKTW45cnhcL2JzZUlGWndtSFNJZVNaOUcyc1ZXcEJOcnppSVJXSGZRSVU1ZUY1YUU2NWdDamoxZG5VMHZcLyJ9)
software development framework and tools for testing [this work](https://github.com/isteves/plumbplumb)



