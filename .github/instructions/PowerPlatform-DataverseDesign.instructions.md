---
applyTo: "DataverseDesign/**"
---

You are responsible for helping normalize Power Platform Solutions architecture.
You have 2 roles : 
- Helping with documenting and normalizing Dataverse data modelisation
- Helping with documenting and normalizing Dataverse security modelisation

You will use markdown files to document the project.
You will use CONTEXT.md to get technical infos for the project.

In the rest of this file : 
- `ab00` is the Business Solution Code provided in CONTEXT.md
- `prfx_` is the Dataverse Publisher Prefix provided in CONTEXT.md

# 1. Dataverse data modelisation

You create a datamodelisation that can be used In Microsoft Dataverse.
You will create a dataModel.md file in /DataverseDesign folder.
All Description and Display Name will be in Main Language demanded in CONTEXT.md file.

## 1.1 Table

1 paragraph for each table with :
- Display Name  
    - Should be functional but not too long.
    - Singular form
- Schema Name (Like `prfx_ab00_TableName`)
    - In English 
    - Concise
    - No special characters or spaces
    - No link workds like "Of", "And", "The"
    - In CamelCase
- Description
    - Be descriptive enough for LLM Readiness
    - Use your knowledge of requirements.md file if it exists and functional project context in CONTEXT.md file

## 1.2 Columns

In the table paragraph, you will add : 
Table of columns with : 
- Display Name
    - Should be functional but not too long.
- Schema Name (Like `prfx_ColumnName`)
    - In English 
    - Concise
    - No special characters or spaces
    - No link workds like "Of", "and", "the"
    - Do not include business solution code
- Type (of dataverse column)
- Description
    - Be descriptive enough for LLM Readiness
    - Use your knowledge of requirements.md file if it exists and functional project context in CONTEXT.md file

Columns should respect good practices : 
- LookUp suffixed by Id
- Choices suffixed by Code
- Boolean Prefixed by Is or Has

### Important notes

- Include the Dataverse Primary Key Column: Display Name is the Table Display Name + " ID", Schema Name is the table Schema Name + "Id", Type is "GUID", Description is "Primary Key"
Do not include out of the box columns such as createdon, createdby, ....

## 1.3 Global Choices

For each Choices column, you will create a paragraph. 
You will include :
- Display Name (Like `ab00 - Status Choice`)
- Technical Name (Like `prfx_choice_ab00_Status`)

## 1.4 ERD

At the end, a paragraph with a visual representation as an ERD using mermaid

## Important notes

You can rely on a requirements.md file to create the datamodel descriptions.
The user will give you the exact tables to create. Do not invent tables.
The user will give you the exact columns to create. Do not invent columns.

# 2. Dataverse security modelisation

You will create a securityModel.md file.

This file is responsible for explaining the Dataverse security model that will be used to secure access to tables sepcified on the dataModel.md file.

## 2.1 Personas

Youe will create a Dataverse Security Role for each personas, providing access at different depth to privilege of each tables.
Create a Paragraph for each Security Role with : 
- Functional description of the persona and accesses
- Technical Description of how a BU or a Team or a User is used as a Owning Business Unit and/or Owner of a record to provide appropriate accesses.

### Table of permissions as in Securiry role Definition : 

- 1 Row per table
- 1 column for priviledge, keeping it simple with CREATE, READ, WRITE, DELETE (do not mention Assign, Share, Append and Append to)
- Associated Depth using emojis :🟢 ORGANISATION, 🔴 PARENT BU, 🔵 BU, ⚪ USER, ⚫ NONE

## Important notes

Eventually Sharing can be part of the Security Model.
Rely on the dataModel.md file to create the security model.
User will provide you with the exact personas to create. Do not invent personas.
User will provide you with the exact permissions to create. Do not invent permissions.