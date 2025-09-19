# ETL pipeline email alert --------------------------------------------
#
# Purpose: General alert to monitor ETL pipeline run logs for errors & monitor update logs for out-of-date tables.
# This code refers to an update log, a batch run schedule, and run logs, to flag any
# tables in key pipelines that are not up to date, and to flag any errors or novel warnings
# that appear in run logs. This process then emails interested parties with a conditional report.
#
# Author: Grant Donovan, adapted from team template
# Note that this script is heavily adapted from a template alert 
# written by Public Health - Seattle & King County CD-EPI A&I Data Systems branch team members, notably Lawrence Lee
# The script is adapted to review update logs and code logs of multiple ETL pipelines
# Referencing unique update schedules for each pipeline using a process I produced.
# Sections with the heaviest modifications in this version include 
# - The log reading process
# - The error flagging process
# - The warning flagging process and warning exclusions
# - Check if all datasets were updated
#

start <- Sys.time()
# Setup --------------------------------------

# Load libraries and odbc connection
library(tidyverse)
library(lubridate)
library(blastula)
library(DBI)
library(odbc)
library(purrr)
library(knitr)

# Set connection to SQL server
con <- dbConnect(odbc()
                 , "SQL Main")

# Specify email recipients
email_recipients <- "mock@mockmail.com"

# email_recipients <- c("gdonovan@kingcounty.gov")

# Specify log paths
path_log_dp <- "//mockpath/Code_logs"

# Check log for errors --------------------------------------
# Import table_update_log with update_times and join expected update times and log_filename
table_update_log <- dbGetQuery(con
                               , paste0("select 
                                            a.*, 
                                            b.run_days_expected, 
                                            b.run_time_expected,
                                            b.log_filename,
                                            b.schedule_active,
                                            b.doh_vpn_yn
                                    		from staging.table_update_log_view as a
                                    		left join ref.key_bat_schedule as b
                                    		on a.data_source = b.data_source_key
                                    		where data_source IS NOT NULL
                                    				AND run_time_expected IS NOT NULL
                                        AND b.schedule_active = 'y'")
)

# Read in logs
log_filenames <- unique(table_update_log$log_filename)
logs <- map(log_filenames, ~{
  log_path_dp <- file.path(path_log_dp, .x)
  
  # Check if the file exists in path_log_dp
  if (file.exists(log_path_dp)) {
    readLines(log_path_dp)
  } else {
    stop(paste("File not found:", .x))
  }
})

# Flag errors within each log
log_errors_list <- map2(logs, log_filenames, function(log, file) {
  log_errors <- data.frame(line = grep('Error', log),
                           message = log[grep('Error', log)]) %>% 
    mutate(filename = file, .before = line)
  return(log_errors)
})

log_errors <- bind_rows(log_errors_list) %>% 
  # Remove benign errors
  filter(!message %in% c("Error in gzfile(file, \"wb\") : cannot open the connection"
                         , "Headers Error"
                         , "Error in save.image(name) : "))
if (nrow(log_errors) == 0) {
  log_errors <- data.frame()
}

# Import warning exclusions
warning_exclusions <- dbGetQuery(con, 'select * from ref.log_warning_exclusions')
# Flag warnings within each log
log_warnings_list <- map2(logs, log_filenames, function(log, file) {
  
  warning_lines <- grep('Warning', log)
  
  log_warnings <- data.frame(line = warning_lines,
                           message = sapply(warning_lines, function(line) paste(log[line:(line+1)], collapse = "\n"))) %>% 
    mutate(filename = file, .before = line) 
  return(log_warnings)
})
log_warnings <- bind_rows(log_warnings_list) %>% 
  filter(!grepl("In gzfile|was built under R version", message),
         !any(filename %in% warning_exclusions$filename &
                sapply(warning_exclusions$message, grepl, message))
  ) 

# Check if all datasets were updated --------------------------------------
# Calculate datetime of most recent expected run and flag if update_time not up-to-date
table_update_log <- table_update_log %>%
  mutate(update_time = force_tz(update_time, tz = "America/Los_Angeles"),
         run_days_expected = lapply(strsplit(table_update_log$run_days_expected, ","), as.numeric)) %>% 
  rowwise() %>%
  mutate(run_dt_expected = {
    current_datetime <-  Sys.time()
    current_date <-  as.Date(current_datetime)
    past_week_dates <- current_date - (1:7) # excludes today for now
    past_week_dates <- as.character(past_week_dates[order(wday(past_week_dates))]) # puts in index order
    expected_dates <- past_week_dates[run_days_expected] # list of dates expected in last week
    if (wday(current_date) %in% run_days_expected) {
      expected_dates <- c(expected_dates, as.character(current_date)) # Add today's date if expected
    }
    datetime_strings <- paste(expected_dates, run_time_expected) #converts expected dates to datetime
    datetime_objects <- ymd_hms(datetime_strings, tz = "America/Los_Angeles")
    max_datetime <- max(datetime_objects[datetime_objects < current_datetime]) #most recent expected run datetime
    max_datetime
  }, .after = script) %>% 
  select(-c(run_days_expected, run_time_expected, log_filename)) %>%
  arrange(data_source, script)

table_update_log_not_updated <- table_update_log %>%
  filter(update_time < run_dt_expected)

table_update_log_updated <- table_update_log %>% 
  anti_join(table_update_log_not_updated, by = c("sql_schema", "sql_table"))


# Generate alert email ----------------------------------------------------

# Intro text and email subject
if(nrow(log_errors > 0)
   | nrow(table_update_log_not_updated > 0)) {
  email_subject = paste0("Data Pipelines - ERROR")

  email_text_intro <- "There are issues that need review in one or more Data_Pipeline process this morning"
} else {
  email_subject = paste0("Data Pipelines - Success")

  email_text_intro <- "There were no major issues flagged in Data_Pipeline processes this morning"
}

## Text for errors found in log
if (nrow(log_errors > 0)) {
  log_errors_text <- "There were error messages flagged within run logs. Please review the following: "
  log_errors_table <- kable(log_errors
                            , format = "html")
  
} else {
  log_errors_text <- "No error messages were flagged within run logs "
  log_errors_table <- ""
}

## Text for warnings found in log
if (nrow(log_warnings > 0)) {
  log_warnings_text <- "There were warning messages flagged within run logs. Please review the following and add benign warnings to the "
  exclusions_link <- "[warning exclusions list](https://github.com/mockorg/Data_Pipeline/blob/main/Scripts%20-%20Setup/ref.warning_exclusions.csv): "
  log_warnings_table <- kable(log_warnings
                            , format = "html")
} else {
  log_warnings_text <- "No warning messages were identified within run logs"
  exclusions_link <- ""
  log_warnings_table <- ""
}

## Text indicating whether all datasets were updated
if (nrow(table_update_log_not_updated > 0)) {
  table_update_log_text <- "The following tables were not updated:"
  table_update_log_table_not_updated <- kable(table_update_log_not_updated
                                              , format = "html")
  
} else {
  table_update_log_text <- ""
  table_update_log_table_not_updated <- ""
}

table_update_log_table <- kable(table_update_log_updated
                                , format = "html")


email <- compose_email(
  body = md(
    c(
      email_text_intro
      , " "
      , log_errors_text
      , log_errors_table
      , " "
      , log_warnings_text
      , exclusions_link
      , log_warnings_table
      , " "
      , table_update_log_text
      , table_update_log_table_not_updated
      , " "
      , "The following tables were successfully updated: "
      , table_update_log_table
    )))


smtp_send(
  email = email
  , subject = email_subject
  , to = email_recipients
  , from = creds_key("outlook_creds")$user
  , credentials = creds_key("outlook_creds")
)

