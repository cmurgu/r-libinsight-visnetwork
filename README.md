# r-libinsight-visnetwork
A script that visualizes a libinsight export


# load tidy

library(tidyverse)

# load data
# see example for a sense of how we structured our libinsight data. 

data <- read_csv("refined-data.csv")

data

# create node list
# first librarians (instructors)
# second faculty (collaborators)

instructors <- data %>%
  distinct(Instructors) %>%
  rename(label = Instructors)

instructors

faculty <- data %>%
  distinct(Faculty) %>%
  rename(label = Faculty)

faculty 

nodes <- full_join(instructors, faculty, by = "label")
nodes

# add unique ID to nodes

nodes <- nodes %>%
  rowid_to_column("id")

nodes

# add colum groupings and reload node.csv

nodes <- read_csv("node.csv")

# add "title" column with values = to label
nodes <- mutate(nodes, title = label)

nodes

# edges list

per_act <- data %>%
  group_by(Instructors, Faculty) %>%
  summarise(weight = n()) %>%
  ungroup()

per_act

edges <- per_act %>% 
  left_join(nodes, by = c("Instructors" = "label")) %>% 
  rename(from = id)

edges <- edges %>% 
  left_join(nodes, by = c("Faculty" = "label")) %>% 
  rename(to = id)

edges <- select(edges, from, to, weight)

edges <- mutate(edges, title = "Instruction")

edges

# load visNetwork

library(visNetwork)

# simple graph

visNetwork(nodes, edges)

# bells and whistles

network <- visNetwork(nodes, edges, width="600px", main = "Sample Network", submain = "2016 - 2019 Instruction Interactions", footer = "Fig. 1, minimal example") %>%
visLegend() %>%
visGroups(groupname = "Librarian", color = "darkblue", shape = "square") %>%
visGroups(groupname = "F-HUM", color = "red", shape = "triangle") %>%
visOptions(highlightNearest = TRUE, selectedBy = "group", nodesIdSelection = TRUE) %>%
visEdges(arrows = "middle")

network

# save as .html

visSave(network, file = "network.html")


