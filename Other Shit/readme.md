
Introduction

The MuleSoft Data Mapping Repository is a centralized repository that facilitates data integration between different systems within our organization. Currently this repository is managed under MUS - Integration - Data Mapping Repository and the versioning history of changes being applied to the repository is managed under Integration - Data Mapping Repository Change History.

This repository currently holds the mapping between the data objects and data features between the current systems within our organization. However, these pages are manually updated and not user friendly for data management analysis. For each integration the Repository owns a specific excel which will contain the Source and Target Data Structure as well as any Transformation logic that might be applied. 

For example, please check the data mapping between for the Order export from Hybris to ADL Hybris - ADL - Order Export Mapping

  
SOURCE DATA STRUCTURE - Hybris	TARGET DATA STRUCTURE - ADL
Message Type	JSON	Message Type	JSON
Method	POST	Method	POST
Technology	HTTPs REST	Technology	HTTPs REST
Endpoint	{countrycode}/sales/orders/{orderId}	Endpoint	https://c360-ingest-api.eu01.treasuredata.com/v1/db_l0_organic/orders
 Process Type  	Asynchronous	 Process Type  	Asynchonous
WSDL/XSD	n/a	WSDL/XSD	n/a
 File Name  	n/a	 File Name  	n/a




Key features to implement within Integration Repository

Ability to store detailed mapping between specific fields/objects (Length; Data Type; Value)
Search and discovery abilities (search specific fields or objects within the full repository)
Versioning control for mapping
Perform visual representation of the mappings and data flows
Audit trail, role based access control etc 
Support for importing/exporting mappings in various formats
It must have an API integration with Mulesoft to be able to trigger automatic updates based on mapping changes

 Getting Started Guide.pptx (sharepoint.com)




Capabilities	ATLAN	COLLIBRA
Key Features for Mapping Repository Use	
Custom Attributes: Can be used to store detailed mapping information
Lineage Capabilities: Visualize relationships between data across systems
Versioning: Tracks changes to metadata over time
APIs and Webhooks: Enables integration with other tools like MuleSoft
Automation: Supports automated workflows for updating and managing mappings
Search and Discovery: Powerful search capabilities for finding specific mappings
Collaboration Features: Allows teams to work together on mapping definitions
	
Business Glossary: Can be adapted to store detailed mapping definitions
Data Dictionary: Ideal for maintaining field-level mapping information
Lineage Capabilities: Visualize data flow and relationships across systems
Versioning: Tracks changes to metadata and mappings over time
Workflow Engine: Supports approval processes and change management for mappings
APIs and SDKs: Enables integration with other tools, including MuleSoft
Role-Based Access Control: Ensures secure access to mapping information
Reporting and Dashboards: Provides visibility into mapping statuses and metrics

Strengths	
Modern, user-friendly interface
Highly customizable to fit specific mapping needs
Strong in handling complex relationships between data entities
Designed for integration with various data tools and platforms
	
Highly customizable metadata model
Strong in handling complex relationships between data entities
Robust governance features for managing mapping lifecycle
Scalable to handle enterprise-level mapping needs
Used by many Fortune 500 companies across various industries

Integration Capabilities	
Can potentially integrate with MuleSoft through its APIs and webhooks and multiple other API integration (Snowflake, SAP etc)
Allows for real-time synchronization of mapping information

Atlan supported connections: Supported sources – Atlan

Atlan API connector: Manage API assets - Developer (atlan.com)

	
RESTful APIs for programmatic access and updates
Potential for real-time synchronization with integration platforms like MuleSoft
Supports bulk import/export of mapping data

Potential Workflows with MuleSoft

Ability to integrate within Mulesoft API

 
