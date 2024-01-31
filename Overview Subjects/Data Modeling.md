# Data Modeling Principles :
The currrent document gives an overview regarding data modeling techniques and principles that can help building easily maintainable Data Warehouses.  
The main purpoe of data modeling is to give a deep comprehension of the data ecosysteme we are buiding and the relationships linking all of it's components. It is similar to a blueprint in the industriel fields done before starting to build planes for example (It could be done after if we want to understand an already build system using REVERSE ENGINEERING).  
## Data Modeling concepts :
- Data Subjects : commonly called "Entities" or "Classes" that has "Objects". it represents a business subject such as "Employee", "Sales" etc. Some business component that we can define and describe (in terms of relational Databases, it is a table)
- Data Attributes : it is the detailed description of a subject like for an employee we have age, job, date of birth etc (represented with columns).
- Relationships : self explanatory, it is answering what links an entity to another like an employee and a department? An employee works at a department. in terms of relationships naming we can indicate one direction naming like "An employee works at a department" or also bidirectional naming refering also to "Department contain employees"
- Business rules for data : this is where we need to understand how the business handels the relationships between subjects. cardinalities, if a relation is mandatory or not, attributes allowed to be nulls, data change dynamics (referential integrity ex : if a row is deleted in a table the corresponding in other tables must be deleted etc).
## Modeling architectural options :
Two major types:  
- Data Modeling-specific (the focus in this document):
    - Classic ER (Chen 1976) : Entity relashionchip using rectangles, oval and so on.
    - Post Classic ER (Crows Foot) : different notation enhanced version of calssic ER.
- Systems modeling:
    - Semantic (IDEF modeling)
    - UML (object oriented system)
## Data Modeling life cycle:
three major milestones:  
- Conceptual Modeling :  
  ![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5f91bd5b-8132-4c8c-ae46-f072223687eb)
- Logical Modeling :  
  ![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/19477e1d-7f69-4265-b38b-c7e50300bd02 "Logical Model")
- Physical Modeling :  
![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/11196f3c-9c26-4ee1-a00b-8cc6e6b44ccc)
## Two major calsses of systems data modeling is used in:
- OLTP type (transactionnal databases): transactionnal systems.
    - the conceptual level for this type represents the real wolrd process
    - the logical level would focus on data normalization rules etc
- OLAP type (Datawarehouses, data marts ...): analyical systems.
    - the conceptual model for this type is the dimentional representation.
    - the logical model is the fact-dimension relationnal representation of the DW.
## Blocks of data modeling:
- Entity:
  represented by a rectangle. tow types of entities are represented by different corners :
  - Normal Entity : Squared-off corners
  - Weak Entity : round corners  
  in the classic notation the entity is a rectangle with it's name inside.  
![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/2e4fa116-ffd1-44d1-84c7-c0764dbf04e2)

  in the crow's foot notation the entity is a rectangle with it's name at the top and the attributes listed under with a lign seperating the two.  
![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/8e6e4a01-d099-45bd-b9da-3a6065450632)  
  or the name outside, the identifying attributes at first and then the remaing attributes  
 ![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5d8703b1-aaa9-4c18-b970-fc44c7cd0610)
- Attributes :  
  set of data descibing the Entity.
  in the classic notation, it is represented by an oval shape linked to the entity.
  ![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/bcb24ec6-36ef-42ff-8035-c28ab3e1a8e3)

  Attributes must not be complexe like name_address_country. It should be name addriss country.
  exploring the data before modeling and doing a data profiling is so useful to help determining the cacarteristics of the attributes.
    - Multivalued attributes: it is a type of attributes where the same observation (student for example) can have different values for the same attribute (email for example)
      in the classic notation, the attribute is represented with a double oval (cercle) around:
      
      ![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/b1f8a689-b494-4d4c-b30b-a0450252d6ab)  
      in the crow's foot notation, the attribute is put between braces:
      
      ![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/1e752af8-2888-4ba8-bcfa-b8be3d08851d)


  
