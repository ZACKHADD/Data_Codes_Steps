## This file is a recap of some of the main things to master in dbt to prepare for the exam

- Configuration at the schema or database level can be override at the object level such as table : eg freshness can be set at the schema level and override at the table level to change the loaded at column for example !
- We can use CODEGEN_PACKAGE to generate the source.yml file if we have hundreds of models
- In dbt studio we can generate the downstreams model as simple select with source refrence using the button generate model.
- We can implement tests for models we create but also for sources we use !
- run tests only on sources : dbt test --select source:*
- If we have a big doc to create we use markdown files that may contain several doc blocks and then we reference that in the description of a filed or table : {{ doc('order_status') }}
- We can also document macros by adding a yml file for that macro
- in dbt we can set a protection on a project to control the access to thoe models if called in a package : PUBLIC, PROTECTED  and PRIVATE
- indirect selection modes :
  - Eager mode runs all tests regardless of dependencies
  - Buildable mode runs tests with dependencies in selected nodes
  - Empty mode ignores all test dependencies
  - Cautious mode restricts tests to exclusively selected node dependencies
