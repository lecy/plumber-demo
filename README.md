# plumbeR package demo

Tutorial on creating a REST API using the **plumbeR** and **plumberDeploy** packages in R. 

See: 

* https://appsilon.com/r-rest-api/
* https://www.rplumber.io/


## Creating the API with plumbeR 

The following script (from the package demo page) is stored in the file: [**plumber/plumber.R**](https://github.com/lecy/plumber-demo/blob/main/plumber/plumber.R).

It creates three API endpoints named **/echo**, **/plot**, and **/sum**. 
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
  rand <- rnorm(100)
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


**BMF EXAMPLE**

```r
library( plumber )
library( dplyr )

data.url <- "https://raw.githubusercontent.com/lecy/plumber-demo/main/data/tinybmf.csv"
bmf <- read.csv( data.url )

#* Returns nonprofits in the specified subsector
#* @param ntee
#* @get /countries
function( ntee ) {
  bmf %>%
    filter( NTEE1 == ntee )
}
```


## Local API Deployment 

The following code will deploy a virtual API endpoint on your machine for testing purposes. 

```r
library( plumber )

# 'plumber.R' is the location of the API file
pr("plumber/plumber.R") %>%
  pr_run(port=8000)
```

It will not, however, create a working IP address or URL for the endpoint. For that you will need to use plumberDeploy to host the API via DigitalOcean or other cloud server. 



## Testing the Endpoint

You can test the endpoints by typing these API calls into your local browser: 

```
http://localhost:8000/echo?msg=hello

http://localhost:8000/plot

http://localhost:8000/sum?a=10&b=5
```



## Getting Data from Endpoints Using R Scripts

Typically we will want to leverage API endpoints as part of a workflow, so we would like to send requests and receive data from R. 

In order to send requests and process the information that is returned from the server we need to use packages designed to work with internet protocol transfers. The most general and commonly-used are **curl** and **httr** as well as **jsonlite** for parsing results that are packaged as json strings. 

```r
library( curl ) 
library( httr )
library( jsonlite )  # processing 
```

```r
## DEMO OF DATA GRAB FROM API USING R 


# Deploy using the following code in a separate R instance:
# 
# plumber::pr("plumber/plumber.R") %>% plumber::pr_run(port=8000)


curl::curl( "http://localhost:8000/echo?msg=hello" )
# A connection with                                                  
# description "http://localhost:8000/echo?msg=hello"
# class       "curl"                                
# mode        "r"                                   
# text        "text"                                
# opened      "closed"                              
# can read    "yes"                                 
# can write   "no"    


api.data.raw <- httr::GET( "http://localhost:8000/echo?msg=hello" )
# Response [http://localhost:8000/echo?msg=hello]
#   Date: 2022-04-06 20:42
#   Status: 200
#   Content-Type: application/json
#   Size: 35 B


data.as.json <- httr::content( api.data.raw, as = "text" )
# [1] "{\"msg\":[\"The message is: 'hello'\"]}"


jsonlite::fromJSON( data.as.json )
# $msg
# [1] "The message is: 'hello'"
```



## Deploying an API Using plumberDeploy 

See: 

* https://github.com/meztez/plumberDeploy
* http://cran.nexr.com/web/packages/analogsea/vignettes/doapi.html

Working with Amazon EC2 instances: 

* https://medium.com/tmobile-tech/using-docker-to-deploy-an-r-plumber-api-863ccf91516d
* https://mblukac.github.io/posts/2021/05/plumber_AWSEC2/
* https://datawookie.dev/blog/2017/09/hosting-a-plumber-api-on-aws/
