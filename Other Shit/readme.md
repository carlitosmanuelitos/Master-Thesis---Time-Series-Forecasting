### Data Impact Assessment
- **Awareness & Usage**: High awareness across all teams but deemed low value, indicating it's not regularly used or considered relevant to daily operations across teams.
- **Insight**: The deliverable is perceived as non-essential except for specific situations. This might call for a reevaluation of its application or a re-adjustments to align better with business needs.

### Data Lineage
- **Awareness & Usage**: Recognized for its potential value but is held back by maintenance issues (not being up to date, or incomplete data being present) and lack of user-friendliness. 
- **Insight**: Improving the tool’s integration into product workflows and enhancing its user interface could increase its usage and reliability. This could possibly be tackled by the new Atlan integration, but still early to discuss) Review Atlan migration in below confluence link:

### Data Models (Logical, Physical & Mapping)
- **Awareness & Usage**: Valued for onboarding new team members and specific products applications but needs enrichment and less manual maintenance. 
- **Insight**: Automating updates and improvimg content detail could make these models more useful and reduce the workload. Difficulty in automating and generating without manual up keep.

### Data Ownership Matrix
- **Awareness & Usage**: Seen as crucial across product teams for maintaining data integrity, especially with frequent job changes, but lacks consistent updates and clear access methods. Information present is not up-to date. 
- **Insight**: Establishing clearer guidelines for data ownership and ensuring regular audits & updates are scheduled could enhance the utility of this matrix.

### Integration Data Mapping Repository
- **Awareness & Usage**: Valuable but needs to be divulged and better integrate into workflows, with a high need for automation to reduce manual effort into keeping information stored up to date.
- **Insight**: Automating the repository management and enhancing visibility could increase its effectiveness and adoption. Simplifying the interface for better segmentation across systems integration would also be beneficial. 


### PII Data
- **Awareness & Usage**: Recognized for compliance importance but challenged by manual updates and multiplicity of sources. Information not up to date. Not revelant across all product teams.
- **Insight**: Streamlining sources and automating updates could ensure compliance and reduce the burden.


### Reference Data in Confluence
- **Awareness & Usage**: Highly valued across all segments, indicating its positive impact in daily operations. Not applicable to DCX.
- **Insight**: Streamlining its use and addressing inconsistencies between systems could improve operational efficiency and effectiveness. Making sure that the data is consistent and updated could help different product needs.

### Data Remediation
- **Awareness & Usage**: Recognized across Order Orch and F2F as valuable, with no application in CIAM. Well-appreciated and important for ensuring data quality.
- **Insight**: As DCX roles expand, there will be a need for continued improvements to maintain standards. Emphasizing regular updates and exploring automation could sustain high data quality standards.

### Solution Data Architecture
- **Awareness & Usage**: Critical in daily operations for Order Orch and CIAM, less so for F2F where it's underutilized.
- **Insight**: Reducing duplicate efforts and enhancing solution ownership could improve efficiency and relevance, especially in markets where it's currently underutilized. Clarifying roles and responsibilities within the solution architecture team could address the current inefficiencies.


### Data Quality Index
- **Awareness & Usage**: Not optimally designed for all product teams, different standards to be applied across different products.
- **Insight**: There's a need for improvements specifically tailored to DCX to enhance effectiveness and overall adoption. Streamlining and updating the current design to better meet the specific requirements of different teams.


### Data Reconciliation
- **Awareness & Usage**: Acknowledged by with moderate value. This points to varying needs and the importance of customizing the process to fit different operational frameworks.
- **Insight**: Enhancing access for product teams and adapting the process for DCX could provide significant benefits. A review and redesign to make it more adaptable and relevant to all teams, especially DCX, are recommended.


### Data Validation Rules
- **Awareness & Usage**: Recognized importance. These to be current and more closely managed by data teams with concurrent audits for keeping rules updated.
- **Insight**: Ensuring these rules are up-to-date, easily extractable, and managed by data teams with alignment with product teams. Regular revisions to ensure consistency and accuracy.




### Additional Insights
- **Automation**: There is a crucial need for automating processes wherever possible to enhance efficiency and reduce manual workload.
- **Auditing**: Regular audits are essential to ensure that data remains up to standard and meets compliance and business needs.
- **Business Alignment**: There's a need for data requirements to be driven more by business demands rather than IT provisions.
- **Team Alignment**: There needs to be better alignment between the data team & product teams to ensure that data strategies and processes support operational needs effectively.




https://dataladder.com/8-principles-of-data-management/
https://library.si.edu/sites/default/files/pdf/rdm_best_practices.pdf

| Policy/Standard | Scope/Name | Assessment Topic | Evidence of Compliance | Self-Assessment Notes | Next Steps/JIRA Reference |
|-----------------|------------|------------------|------------------------|-----------------------|---------------------------|
| IT-STD-401      | Metadata Management | Data Owner and Data Steward roles, data scope review, metadata management tools usage | Data Owner/Steward listed on SDA, DIA, MDM matrix; CONSGOV contains information; Data Dictionary/Catalog in place | Regular review/update by Data Owner/Steward and IT; Data Models designed/documented | - |
| IT-STD-402      | Data Confidentiality | Define data confidentiality category, implement controls, data classification and logging | Data criticality defined with capability owner and data owner; Data recovery procedures shared | SLAs agreed with data owners per data confidentiality level | - |
| IT-STD-403      | Data Lifecycle Management | Data domain lifecycle definition, data quality assessment, data security mechanisms | CRUD matrix/document signed off by data owner; InfoSec validated security review | Regular data quality assessment; Security review at each solution in scope | - |
| IT-STD-404      | Data Archival, Retention, Disposal | Data Retention period enforcement, Data Archival and Disposal process definition | Proof/evidence of process definition and implementation; Link to audit trail capabilities | Exceptions validated as per IT Risk Management Standard | - |
| IT-STD-405      | Data Quality | Solution Data Architectures documentation, data quality controls and rules | SDA documentation uploaded in Confluence; DQ requirements/control validated with Data Owner/Steward | Data quality rules published and maintained; Critical data assets have DQ rules defined | - |
| IT-STD-406      | Master Data Management and Standardisation | Master data maintenance, MDM authoritative source, As-is To-be analysis | Link to Distributed MDM guidelines; CRUD, SDA for data domain | Periodical quality reviews; Compliance with quality monitoring requirements | - |
| IT-STD-407      | Data Modelling and Architecture | Physical data models for critical data, consistency with solution implementation | Link to PMI PDM; Physical data models describe data structures, relationships, and cardinalities | Data Lineage documentation; Data flows documented at column level | - |
| IT-STD-408      | Data Handling | Data sharing requests logging and approval, security requirements implementation | Link to CONSGOV or JIRA US; Proof/evidence of Legal review and NDA | Data flows for shared data documented; Official Authoritative Sources for data sharing | - |
