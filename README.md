# Klaviyo Data Modelling
This dbt package models the Klaviyo data coming from [Daton](https://sarasanalytics.com/daton/). [Daton](https://sarasanalytics.com/daton/) is the Unified Data Platform for Global Commerce with 100+ pre-built connectors and data sets designed for accelerating the eCommerce data and analytics journey by [Saras Analytics](https://sarasanalytics.com).

This package would be performing the following funtions:

- Consolidation - Different geo/regions would have similar tables for some klaviyo customers. Helps in consolidating all the tables into one final stage table 
- Deduplication - Based on primary keys , the tables are deduplicated and the latest records are only loaded into the stage models
- Incremental Load - Models are designed to include incremental load which when scheduled would update the tables regularly
- (Optional) Currency Conversion - Based on the currency input, a couple of currency columns are generated to aid in the currency conversion - (Prerequisite - Exchange Rates connector in Daton needs to be present - Refer [this](https://github.com/saras-daton/currency_exchange_rates))
- (Optional) Time Zone Conversion - Based on the time zone input, the relevant datetime columns are replaced with the converted values

#### Prerequisite 
Daton Connectors for Klaviyo Data - Klaviyo v2, Exchange Rates(Optional)

# Installation & Configuration

## Installation Instructions

If you haven't already, you will need to create a packages.yml file in your project. Include this in your `packages.yml` file

```yaml
packages:
  - package: saras-daton/klaviyo_bigquery
    version: 1.0.0
```

# Configuration 

## Required Variables

This package assumes that you have an existing dbt project with a BigQuery profile connected & tested. Source data is located using the following variables which must be set in your `dbt_project.yml` file.

```yaml
vars:
    raw_projectid: "your_gcp_project"
    raw_dataset: "your_klaviyo_dataset"
```
## Schema Change

 In case, you would like the models to be written to the target schema or a different custom schema, please add the following in the dbt_project.yml file.

```yml
models:
  klaviyo_bigquery:
    +schema: custom_schema_name # leave blank for just the target_schema
```

## Optional Variables

Package offers different configurations which must be set in your `dbt_project.yml` file under the above variables. These variables can be marked as True/False based on your requirements. Details about the variables are given below.

```yaml
vars:
    currency_conversion_flag: False
    timezone_conversion_flag:
        klaviyo: False
    timezone_conversion_hours: 7
    geo_consolidation_flag: False
    geo_name_position: 0
    geo_name:"Canada"
    table_partition_flag: False
```

### Currency Conversion 

To enable currency conversion, which produces two columns - exchange_currency_rate, exchange_currency_code  based on the data from the Exchange Rates Connector from Daton.  please mark the currency conversion flag as True. By default, it is False.

### Timezone Conversion 

To enable timezone conversion, which converts the major date columns according to given timezone, please mark the time zone conversion variable for this pakage as True in the dbt_project.yml file. The klaviyo data is available at UTC timezone and by setting the timezone_conversion_hours variable, it will be offset by the specified number of hours.(Eg: 7,8,-7,-11 etc) By default, it is False.

### Table Partitions

To enable partitioning for the tables, please mark table_partition_flag variable as True. By default, it is False.

### Table Exclusions

Setting these table exclusions will remove the modelling enabled for the below models. By declaring the model names as variables as above and marking them as False, they get disabled. Refer the table below for model details. By default, all tables are created. 

### Geo Consolidation

The name of the Geography/region where the customers are using the klaviyo would be called the Geo in this case. If you are using klaviyo in more than one Geography base , then Geo consolidation can be enabled. Geo name positions (Eg: 0/1/2) gets the Geography name from the integration name based on the location you have given. In case there is only a single Geography base, adding the name in Geo name variable adds the column to the tables. By default, Geo consolidation flag is False and Geo position variable needs to be set.

## Scheduling the Package for refresh

The klaviyo tables that are being generated as part of this package are enabled for incremental refresh and can be scheduled by creating the job in Production Environment. During 'dbt Build', the models get refreshed.

## Models

This package contains models from the Klaviyo API which includes Events, Campaigns, flows data. Please follow this to get more details about [models](https://docs.google.com/spreadsheets/d/1ZPEGUPi9QZSM6PSBSr6PywSqedmzETGnoig2gPiX1sg/edit?usp=sharing). The primary outputs of this package are described below.

| **Model**  | **Description** |
|---------------| ----------------------- |
|[Bounced_Email](models/Klaviyo/klaviyo_bounced_email.sql)  | Details of bounced email events |
|[Clicked_Email](models/Klaviyo/klaviyo_clicked_email.sql)  | Details of clicked email events |
|[Clicked_SMS](models/Klaviyo/klaviyo_clicked_sms.sql)  | Details of clicked sms events |
|[Consented_To_Receive_SMS](models/klaviyo/Klaviyo_consented_to_receive_sms.sql)| Details of events in which customers give permission to receive sms |
|[Dropped_Email](models/Klaviyo/klaviyo_dropped_email.sql)| Details of events in which emails were getting dropped |
|[Failed_To_Deliver_SMS](models/klaviyo/Klaviyo_failed_to_deliver_sms.sql)| Details of events in which sms were failed to get delivered to customers |
|[Flow_Actions](models/Klaviyo/klaviyo_flow_actions.sql)| A list of actions related to flow|
|[Flow_Messages](models/Klaviyo/klaviyo_flow_messages.sql)| Details of messages sent associated with the flows |
|[Flows](models/Klaviyo/klaviyo_flows.sql)| list of the flows associated with the account |
|[Lists](models/Klaviyo/klaviyo_lists.sql)| Details of the lists associated with the account |
|[Marked_Email_As_Spam](models/Klaviyo/klaviyo_marked_email_as_spam.sql)| Details of the events in which emails are marked as spam |
|[Metrics](models/Klaviyo/klaviyo_metrics.sql)| A list of metrics associated with the account |
|[Opened_Email](models/Klaviyo/klaviyo_opened_email.sql)| Details of the opened email events |
|[Opened_Push](models/Klaviyo/klaviyo_opened_push.sql)| Details of the pushed email events|
|[Placed_Order_Extra_Attributes](models/Klaviyo/klaviyo_placed_order_attributes_event_properties_extra.sql)| Additional Details of the placed orders events  |
|[Placed_Order](models/Klaviyo/klaviyo_placed_order.sql)|  Details of the orders placed events
|[Received_Email](models/Klaviyo/klaviyo_received_email.sql)| Details of the received email events |
|[Received_Push](models/Klaviyo/klaviyo_received_push.sql)| Detais of the received push event |
|[Received_SMS](models/Klaviyo/klaviyo_received_sms.sql)| Details of the sms which were received |
|[Segments](models/Klaviyo/klaviyo_segments.sql)| A list of segments associated with the account |
|[Sent_SMS](models/Klaviyo/klaviyo_sent_sms.sql)| Details of the sms which were sent |
|[Subscribed_To_List](models/Klaviyo/klaviyo_subscribed_to_list.sql)| Details of events in which customers subscribed to list |
|[Unsubscribed_from_list](models/Klaviyo/klaviyo_unsubscribed_from_list.sql)| Details of events in which customers unsubscribed from list |
|[Unsubscribed](models/Klaviyo/klaviyo_unsubscribed.sql)| Details of events in which emails were unsubscribed |
## Resources:
- Have questions, feedback, or need [help](https://calendly.com/priyanka-vankadaru/30min)? Schedule a call with our data experts or email us at info@sarasanalytics.com.
- Learn more about Daton [here](https://sarasanalytics.com/daton/).
- Refer [this](https://youtu.be/6zDTbM6OUcs) to know more about how to create a dbt account & connect to Bigquery
