# Process Flow Diagram Documentation

## Author: Grant Donovan

## Description
This folder contains reproducible code to generate process flow diagrams that outline the flow from inputs to outputs and the processing steps between. 
The folder contains mock metadata and outputs produced by the included code.

## Getting Started

### Repository Contents

This repository includes the following key files and folders:

- **Output/**  
  Folder containing processed SVG flow diagrams used for pipeline documentation. Includes a subfolder with detailed diagrams down to the dashboard tab-level detail.

- **scripts/ref.mock_table_dependencies.csv**  
  CSV reference of mock pipeline table dependency metadata for use in sample flow diagram generation. This serves as an example of how input metadata is organized.

- **setup_ref_flow.R**  
  R script to import dependency metadata and render markdown scripts. Generates both dynamic network diagram and static flow diagram for reference.

- **setup_ref_flow_markdown.Rmd**  
  R Markdown file that transforms metadata into node and edge tables, then generating static SVG flow diagrams.

- **setup_ref_network_markdown.Rmd**  
  R Markdown file that transforms metadata into node and edge tables appropriate for Visnetwork-style visualizations of data processes.

### How to access and use data_pipeline flow diagrams
- **SVG images** - created to be easily embedded in GitHub documentation
  - **Found in** the output subfolder in this repository.
  - To open a zoomable image, right click and press "i" key. Then zoom by pressing ctrl + mousescroll. Pan by centerclicking the scroll button on mouse or by using keyboard arrows.
- **Interactive network diagrams**
  -  **Found in** the output subfolder in this repository.
  - **How to use interactive network diagrams**:
    - This folder has network diagram html files for EACH data source. However, these files need to be downloaded to be viewed. Download flow_diagram_ALL.html to view the combined diagram with ALL datasources.
    - Some definitions:
      - Node: A table or data input, output, product, or report.
      - Edge: The path between nodes, represented by arrows between nodes, usually representing the processing script that outputs one from the other. 
    - Use 'Select by ID' or 'Select by group' filters at top left to highlight subgroups of nodes. Selecting nodes by ID is most useful because it will highlight all input and output nodes directly related to that node.
    - Hover over nodes or edges to get additional details.
    - Optionally can click on edges (arrows) to open linked processing scripts in GitHub if applicable (current examples lead to mock links).
    - **The purpose** of this interactive dashboard is to really scrutinize data pipelines, identifying areas for improvements or to identify dependencies for troubleshooting. 

  ### How to create custom flow diagrams
  1. Edit the template ref.mock_table_dependencies.csv with any data pipeline inputs and outputs.
     - Important notes:
       - Ensure that new rows have a value for every variable.
       - If a new schema is added that previously did not exist in the markdown code, a very unpleasant fillcolor will be added to represent it. Manually hardcode new schema fillcolors in setup_ref_flow_markdown.Rmd to avoid this.
  2. Run setup_ref_flow.R to process any changes in dependency metadata and output new flow diagrams
     - setup_ref_flow is set up to output a flow diagram for EACH datasource present in the metadata AND an overall interconnected diagram. It is possible to run for a single data source by running a select section of code manually in setup_ref_flow.R.
     

