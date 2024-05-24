# Distance API call

Tasked with mapping not just a straight line distance, but also the road network distance between millions of locations. Started with 1.3 million different postcodes, to build matrix of all these distances would have resulted in trillions of calculations so I needed to filter down. I started by mappign just postcode areas eliminating any centre point more than 100km apart. Then I did the same with postcode districts for anyhting more than 25km aparts. This filtered down the list to just 8.7 million calcuations. I then needed to convert postcodes to latitude and longitude which I did in my GeoPlanner tool. I then wrote R code using the BING API in order to call the road distances between the latitude and longitude:

# Read in file
w <- read.csv('export.csv')

# Load the required libraries
library(readr)
library(openxlsx)

API <- 'XXX'


calculate_distance <- function(source_lat, source_long, target_lat, target_long, api_key) {
  # Construct the URL for the distance matrix API endpoint
  url <- paste0("https://dev.virtualearth.net/REST/v1/Routes/DistanceMatrix",
                "?origins=", source_lat, ",", source_long,
                "&destinations=", target_lat, ",", target_long,
                "&travelMode=driving",
                "&key=", api_key)
  
  # Make the HTTP GET request
  response <- GET(url)
  
  # Check if request is successful
  if (status_code(response) == 200) {
    # Extract the distance from the JSON response
    distance <- content(response, "parsed")$resourceSets[[1]]$resources[[1]]$results[[1]]$travelDistance
    return(distance)
  } else {
    # Return NA if there's an error
    return(NA)
  }
}

# Decided to split my data file into 9 chucks so if a run failed I would not lose too much.
library(dplyr)
library(httr)

num_rows <- nrow(nw)
chunk_size <- ceiling(num_rows / 9)

# Create a vector of indices to split the data frame
split_indices <- cut(1:num_rows, breaks = seq(1, num_rows, by = chunk_size), labels = FALSE)

# Split the data frame into smaller ones
list_of_dataframes <- split(nw, split_indices)


Distance1 <- list_of_dataframes[[1]]


Distance1Ran <- Distance1 %>%
  mutate(distance = mapply(calculate_distance, SourceLatitude, SourceLongitude, TargetLatitude, TargetLongitude, MoreArgs = list(API)))

# Doing this for all the files took about 7 days
