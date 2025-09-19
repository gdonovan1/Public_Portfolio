# Load R project library ----------------------------------------------
#Always start with a clean session (Ctrl + Shift + F10) so renv libraries can load smoothly--
renv::load(project = "C:/GitHub/Public_Portfolio/visualizations/Process flow diagrams")

# load package libraries
library(tidyverse)
library(visNetwork)
library(rmarkdown)
library(colorspace)
library(DiagrammeR)
library(glue)
library(DiagrammeRsvg)

# Filepaths
path_repo <- "C:/GitHub/Public_Portfolio/visualizations/Process flow diagrams"
path_scripts <- path_data <- file.path(path_repo, "scripts")
path_data <- file.path(path_repo, "data")
path_output <- file.path(path_repo, "output")

# Metadata for VisNetwork diagrams - Run this chunk before rendering markdown
metadata_df0 <- read_csv(file.path(path_data, "ref.mock_table_dependencies.csv"))
unique_datasources <- unique(metadata_df0$data_source)
## To output a single datasource, uncomment line of code below
## And customize as needed
# unique_datasources <- "Hospital_A" 

# VisNetwork Diagram outputs: Loop through each data_source value and render the report
for (source in c(unique_datasources, "ALL")) {
  # Filter the data
  filtered_df <- if (source == "ALL") metadata_df0 else metadata_df0 %>% filter(data_source == source)

  output_filename <- paste0("network_flow_diagram_", source, ".html")
  rmarkdown::render(
    input = file.path(path_scripts, "setup_ref_network_markdown.Rmd"),
    output_file = file.path(path_output, output_filename),
    params = list(datasource_df = filtered_df, datasource = source),
    output_format = "html_document",
    envir = new.env()  # Use a new environment to avoid conflicts
  )
}


# APDE inspired DiagrammeR flow diagram outputs: Loop through each data_source value and render the report
for (source in c(unique_datasources)) {
  # Filter the data
  filtered_df <- metadata_df0 %>% filter(data_source == source)
  
  output_filename <- paste0("flow_diagram_", source, ".svg")  # <-- SVG extension
  image_output <- file.path(path_output, output_filename)
  temp_html <- tempfile(fileext = ".html")
  
  graph <- rmarkdown::render(
    input = file.path(path_scripts, "setup_ref_flow_markdown.Rmd"),
    output_file = temp_html,
    params = list(datasource_df = filtered_df, datasource = source),
    output_format = "html_document",
    envir = new.env()
  )
  
  svg <- export_svg(flow_grviz) 
  writeLines(svg, con = image_output) 
}
