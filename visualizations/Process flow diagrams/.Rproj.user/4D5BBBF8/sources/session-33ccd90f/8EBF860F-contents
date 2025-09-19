# Header --------------------------------------------
# Grant Donovan Custom Utility functions
# Purpose:
# Create utilities that can be used across different scripts
#
# Author:
# - Grant Donovan





# General Purpose --------------------------------------------

# ut_download_data
# - Downloads queried data from sql server, 
#   optionally searches for existing table if depend = TRUE
ut_download_data <- function(data, sql_schema, sql_table, vars = NULL, nrows = NULL, depend=FALSE, test = FALSE){
  if(test == TRUE) {
    test_prefix <- 'zztest_'
  } else {
    test_prefix <- ''
  }
  # Clean up the vars input
  if (!is.null(vars)) {
    vars <- paste(vars, collapse = ", ")
  } else{
    vars <- "*"
  }
  # add nrows if not NULL
  if (!is.null(nrows)) {
    nrows_limit <- paste0("TOP (",as.integer(nrows),") ")
  } else{
    nrows_limit <- ""
  }
  # Create the SQL query
  query <- paste0("SELECT ", nrows_limit, vars, " FROM ", sql_schema, ".", test_prefix, sql_table)
  # Execute the query or return the data
  if (!exists(data) | depend==FALSE){
    return(dbGetQuery(con, query))
  } else {
    return(get(data))
  }
} #END OF FUNCTION



# Documentation --------------------------------------------

# ut_data_dict
# - Generate data dictionary from SQL tables and optionally incorporates any imported description dataframe 
#   or manually updated column descriptions
ut_data_dict <- function(sql_schema, sql_table, description_df = NULL, overwrite = FALSE){
  # designate data dictionary filepath
  path_dd <- paste0(path_repo_local, "data_dictionaries/", sql_schema,".",sql_table,"_dd.csv") 
  
  sql_dictionary <- dbGetQuery(con, 
                               paste0("SELECT TABLE_SCHEMA as sql_schema, TABLE_NAME as sql_table, COLUMN_NAME as [column], 
                    ORDINAL_POSITION as ordinal_position, IS_NULLABLE as is_nullable, 
                    DATA_TYPE as data_type, CHARACTER_MAXIMUM_LENGTH as char_max_length, 
                    COALESCE(NUMERIC_PRECISION, DATETIME_PRECISION) as precision
                    FROM INFORMATION_SCHEMA.columns
                    WHERE TABLE_SCHEMA = '",sql_schema,"' and TABLE_NAME = '",sql_table,"' ", 
                                      "order by TABLE_SCHEMA, TABLE_NAME, ORDINAL_POSITION;"))
  
  if (file.exists(path_dd) & !overwrite == TRUE) {
    descriptions <- as.data.frame(read_csv(path_dd, show_col_types = FALSE)) %>% 
      select(sql_schema, sql_table, column, description)
    
    if(nrow(anti_join(sql_dictionary,descriptions, by = c('sql_schema','sql_table','column')))>0|
       nrow(anti_join(descriptions, sql_dictionary, by = c('sql_schema','sql_table','column')))>0){
      warning(paste0("Existing variables in ",sql_schema,".",sql_table," do not match ",
                     "those currently described in the data dictionary file: ",path_dd,
                     ". Please review and revise descriptions file."))
    }
    
    dd_final <- left_join(sql_dictionary, descriptions, 
                          by = c('sql_schema','sql_table','column'))
  }else{
    dd_final <- sql_dictionary %>% 
      mutate(description = "")
  }
  
  if (!is.null(description_df)) {
    dd_final <- dd_final %>% 
      left_join(description_df, by = c('sql_table','column')) %>% 
      mutate(description = coalesce(description.x, description.y)) %>% 
      select(-c(description.x, description.y))
  }
  
  write.csv(dd_final, path_dd, row.names = FALSE)
  
  print(paste0(sql_schema,".",sql_table," data dictionary written to ",
               path_dd))
} # END OF FUNCTION

# ut_run_time 
# - calculate run time from start to end
ut_run_time <- function(start, end) {
  dsec <- as.numeric(difftime(end, start, unit = "secs"))
  hours <- floor(dsec / 3600)
  minutes <- floor((dsec - 3600 * hours) / 60)
  seconds <- round(dsec - 3600*hours - 60*minutes, digits = 2)
  cat("Total time to completion:", hours, "hr,", minutes, "min,", seconds, "seconds\n")
} # END OF FUNCTION