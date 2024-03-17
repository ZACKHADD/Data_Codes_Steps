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
- Data Modeling-specific (the focus in this document) Entity-Relationship:
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
      the multivalued attributes notation is the most element diffrentiating between conceptual and logical model.
  - Relationships :
    It represents the link between two entities.
    In the classic ER, it is represented by dimon shape with a name inside or outside of it:
    ![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/69b1f00f-5a7c-4862-9cee-5542fa2e80b8)
    In the crow's foot it is just a line with a name obove it.

- Cardinalities:
  They refer to the real wolrd business rules in terms of relationship between entities. for example between classroom and teachers, can a teacher teach 0 calssrooms or at least 1. Can a classroom be teached by several teachers or at max 1 etc. So from this analysis we can enrich the relationship saying it is 0 to many, many to many many to 1 ...
## Hierarchies for the entities:
  Hierarchies are special types of relationchips that gives some differenciation between types of an entity. for example if we take the entity "Teacher" it can be a a full time teacher or a part time teacher in the faculty. Each of these two types has it's own detailed attributes.  
  The objective of hierarchies is to represent the most all the constraints that should be applied on attributes and objects in the phisical model.
  in classic ER, hierarchies are represented as follows :  
  ![WhatsApp Image 2024-02-02 at 15 40 04](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5fa7c49a-3c75-4960-9109-6207a5fd993c)  
  The principal entity is called supertype.  
  ![WhatsApp Image 2024-02-02 at 15 40 03 (1)](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/4f770420-41e5-4ad2-869a-06d8b998230d)  
The principal entity is called childtype.  
The relation can be either exclusive (X inside) meaning only one the object can be only one type of the childrentypes or inclusive (I inside) meaning it can be both.  
  ![WhatsApp Image 2024-02-02 at 15 40 03](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/c099c3d7-814a-42e6-8d0f-62e773835042)  
In the crow's foot notation, hierarchies are represented as follows:  
  ![WhatsApp Image 2024-02-02 at 15 40 04](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/fbf4fbe0-2720-44ff-862d-87be6feef982)

## Constraints on Attributes :
Constraints on attributes is a powerful tool to contrôle data in the data model. it allows to define how these attributes should be built. For example : we can put some conditions on data type and size of the attribute, whether the NULL values are allowed or not and the permissible values (age > 18 for example) etc.  
## Types of entities :
We have two major types of entities : Strong & Weak Entities.  
- Strong entity : an entity that does not need another entity to be defined or to exist.
- Weak entity : an entity that depends on another entity to be identified or to exist.
in terms of classic ER notation, they are represented with a double rectangle as follows :  
![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/7e7c811e-f22e-4cf3-af80-6a7c47a4805d)  
the double diamon here represents the weak relationchip between a strong and weak entity.  
in crow's foot notation, the weak entity is a rounded corner rectangle with no identifier (Primary key) (only at the conceptual level).
## Types of relationchips :
We can have mainly three relationchips situations :
- Multiple relationchips between two entities: for example teacher that teaches students but also can advise other students that he does not teach.
- Recursive relationchips where an entity has a relation with itself : like when an entity called persons can have the father and son so the relation would be X son of Y and the relation goes out of the entity and return to it.
- Ternary relationchips involving three entities (at conceptual level) :  
  ![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/12a5e608-571e-4060-9134-f08c0d679962)  
- Relationchips like entities "Gerunds" : a relationchip that also acts like an entity. for example :  
  ![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/9c2fb99f-f24f-4508-9b0d-2ea5f94df49f)

## Cardinalities :
simply the number of instances in both sides of a relationchip.  
We represent in both sides maximum and minimum cardinality.  
the maximum cardinality can be either a specific number (the exact maximum allowed) or M meaning many.  
the minimum can be 0 or one (optional vs mandatory) or an explicit number (so rare) of min cardinality.  
The cardinalities must be explicit in words like : the entity must or can have 0 or n instance from another entity.  
In classic ER notation, cardinalities are represented as follows : 1-M ----- 0-M.
In Crows foot the min and max cardinalities are represented as follows:  
![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/3d225719-d702-4f83-af54-80b61e292c57)  
## Normalization when working with transactional DB:
Normalisation helps modeling (the logical level) data for relational databases. 3 main forms of normalization are the most used :
- 1st Normal form : every ro (tuple in modling language) must be unique. No repeating groups, each table cell should contain only a single value, and each column should have a unique name. The first normal form helps to eliminate duplicate data and simplify queries.
- 2nd Normal form : eliminates redundant data by requiring that **each non-key attribute be dependent on the primary key (as a whole and not partially)**. This means that each column should be directly related to the composite primary key as a combination and not to a part of it.
  Example : if we have a composite primary key of 3 identifiers, all the non-key columns shoud not be dependent only on one identifier but on the combination the 3 identifiers.
- 3rd Normal form : builds on 2NF by requiring that **all non-key attributes are independent of each other**. This means that each column should be directly related to the primary key, and not to any other columns in the same table.  
 **NB: for performance reasons, in OLAP DB we denormalize (with an acceptable level in dimensions) to avoid multiple joins in queries**
## Many to many relationship solution (mainly in transactional BD):
To adress the issue of many to many relationships, we add a **bridge entity** as follows :  
![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/f7ae64bf-6ac4-4465-bc25-c0d49ad2df41)  
**Moving from conceptual to logical model, we add some constraints, foreing keys ..** 
**From Logical to Physical model**
The physical model will contain technical names, all constraints (PK, FK, Admissible values, NULL or not, size ..), indexes and all elements that will be technicaly implemented depending on the tool chosen.  
We also, depending on the objective of our data, denormalize, create agregates, create materialized views ...  

## Some data modeling tools:
- VISIO
- DRAWIO
- CA ERwin
- ER/Studio Data Architect
- Redshift (online)
- ...


  
