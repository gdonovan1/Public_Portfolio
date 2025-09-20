# Process Flow Diagram Documentation

## Description
This folder contains reproducible code to generate process flow diagrams that outline the flow from inputs to outputs and the processing steps between. 
The folder contains mock metadata and outputs produced by the included code.

## Getting Started

### Repository Contents

This repository includes the following key files and folders:

- **Output/**  
  Folder containing processed SVG flow diagrams used for pipeline documentation. Includes a subfolder with detailed diagrams down to the dashboard tab-level detail.

- **scripts/ref.mock_table_dependencies.csv**  
  CSV reference of mock pipeline table dependency metadata for use in sample flow diagram generation. This is the primary table to customize as needed for diagram inputs.

- **setup_ref_flow.R**  
  R script to process dependency data and generate formatted flow data for visualization. Generates both dynamic network diagram and static flow diagram for reference.

- **setup_ref_flow_markdown.Rmd**  
  R Markdown file to generate static PNG flow diagrams from dependency tables.

- **setup_ref_network_markdown.Rmd**  
  R Markdown file for network-style visualizations of data dependencies.

### How to access and use data_pipeline flow diagrams
- **SVG images** - created to be embedded in GitHub documentation
  - **Found in** the output subfolder in this repository.
  - To open a zoomable image, right click and press "i" key. Then zoom by pressing ctrl + mousescroll. Pan by centerclicking the scroll button on mouse or by using keyboard arrows.
- **Interactive network diagrams**
  -  **Found in** CIFFS drive here: 📂
  - **How to use interactive network diagrams**:
    - This folder has network diagram html files for EACH data source. Open flow_diagram_ALL.html to view the combined diagram with ALL datasources.
    - Some definitions:
      - Node: A table or data input, output, product, or report.
      - Edge: The path between nodes, represented by arrows between nodes, usually representing the processing script that outputs one from the other. Can also refer to the Tableau workbook outputting particular reports. 
    - Use 'Select by ID' or 'Select by group' filters at top left to highlight subgroups of nodes. Selecting nodes by ID is most useful because it will highlight all input and output nodes directly related to that node.
    - Hover over nodes or edges to get additional details.
    - Click on edges (arrows) to open linked processing scripts in GitHub if applicable.
    - **The purpose** of this interactive dashboard is to really scrutinize data pipelines, identifying areas for improvements or to identify dependencies for troubleshooting. 

  ### How to update data_pipeline flow diagrams
  1. Update ref.mock_table_dependencies.csv and ref.production_table_dependencies.csv with any changes to data pipeline inputs or outputs or routine dashboards that use data_pipeline products.
     - Important notes:
       - Ensure that new rows have a value for every variable.
       - If a new schema is added that previously did not exist, a very unpleasant fillcolor will be added to represent it. Manually hardcode new schema fillcolors in setup_ref_flow_markdown.Rmd to avoid this.
  2. Run setup_ref_flow.R to process any changes in dependency metadata and output new flow diagrams
     - You can optionally change what metadata is read in to output a functional flow diagram; however, the markdown code is optimalized for the data_pipeline metadata.
     - By default, the code will output diagrams that limit the number of nodes for Tableau dashboards. To produce flow diagrams with more detail, change the tableau_detail parameter to TRUE.
       These will save to a subfolder.
     - setup_ref_flow is set up to output a flow diagram for EACH datasource present in the metadata AND an overall interconnected diagram.
     

