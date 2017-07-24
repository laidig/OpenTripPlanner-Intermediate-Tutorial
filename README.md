# NOTE THIS IS A WORK IN PROGRESS

# OpenTripPlanner-Intermediate-Tutorial

How To Set-Up OpenTripPlanner (OTP) with real-time transit and bike share information.

This tutorial walks through setting up [OpenTripPlanner](http://opentripplanner.readthedocs.io) with data from the San Francisco Bay Area. It includes static and realtime data for the BART train and Bike Share information from Ford GoBike.

## Setup

### Requirements:
* Java 8
* Internet connectivity

### Directories
Create a directory for your OTP files. Lets call the base directory otp, and we're going to add another couple directories inside it for data.

    mkdir otp
    mkdir otp/graphs
    mkdir otp/graphs/sf

### Getting OTP
At this moment (July 2017), GBFS (the bike share data specification) does not exist in an official OTP release, so you need to download the most recent -shaded snapshot of OTP from [here](https://oss.sonatype.org/content/repositories/staging/org/opentripplanner/otp/1.2.0-SNAPSHOT/). It should make it into version 1.2.0. If version 1.2.0 has been released, choose the -shaded version of that JAR.

### Getting Data
There are two data files that you need to download and put into that ```otp/graphs/sf``` directory you just created:
1. The BART Static GTFS from [here](http://www.bart.gov/schedules/developers/gtfs)
1. An extract of OpenStreetMap data for the Bay Area region. Mapzen provides one [here](https://s3.amazonaws.com/metro-extracts.mapzen.com/san-francisco-bay_california.osm.pbf) 


## Getting started

We'll need to tell OpenTripPlanner about sources of real time data. OTP refers to anything that isn't loaded at startup an "updater." These are specified in a file called ```router-config.json```. 

Here's the ```router-config.json``` we're going to use

    {
    "fares": {
        "type": "san-francisco"
    },
    "updaters": [
        {
            "type" : "stop-time-updater",
            "frequencySec": 90,
            "sourceType" : "gtfs-http",
            "url": "http://api.bart.gov/gtfsrt/tripupdate.aspx",
            "feedId": "BART"
        },
        {
          "type": "bike-rental",
          "frequencySec": 300,
          "sourceType": "gbfs",
          "url": "https://gbfs.fordgobike.com/gbfs/gbfs.json"
        }
      ]
    }
    
This tells OTP that to use special fares for San Francisco, set up stop time updates for BART, and query for availabilty for the GoBike bike share. More information on the options here can be found in the [main OTP docs](http://opentripplanner.readthedocs.io/en/latest/Configuration/)

## Building A Graph

First, let's get familiar with the OTP jar. It has a built-in help function that lets you know what the command line options are:

   java -jar otp-1.2.0-(build)-shaded.jar --help.

OTP refers to the compiled data set of streets and transit information as a "Graph." Let's build our first one. 

   java -Xmx4G -server -jar otp-1.2.0-20170713.100504-18-shaded.jar --basePath . --build graphs/sf/

The build should complete: 
    21:33:14.331 INFO (Graph.java:844) Graph written.
    21:33:14.347 INFO (GraphBuilder.java:171) Graph building took 1.1 minutes.
    
OTP can support routing for multiple areas using a feature called (unsurprisingly) routers. We can now tell OTP to run that graph, and pull from a router called ```sf```. 
    java -Xmx4G -server -jar otp-1.2.0-20170713.100504-18-shaded.jar --basePath .  --graphs graphs/ --server --router sf


