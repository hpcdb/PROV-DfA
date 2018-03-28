# Examplary Queries for Provenance of Adaptation

## Query 1:
Inspecting parameter tunings ("who", "when", "what"). How many tunings did I do? Which parameters did I change? What were the values when I changed and what values did I change into? When did each adaptation happen?

```sql
SELECT 
  param_tuning.id,
  param_tuning.starttime,
  param_tuning.description,
  param.name,
  param.specification,
  param_tuned.old_value,
  param_tuned.new_value
FROM
  parameter_tuning param_tuning,
  attribute param,
  attribute_tuned param_tuned,
  person pers
WHERE
  param_tuned.parameter_tuning_id = param_tuning.id and
  param_tuned.attribute_id = param.id and
  param_tuning.id = pers.id and  
  param.annotation = 'input parameter of a data transformation' and  
  pers.name = 'Bob'
ORDER BY
  param_tuning.id
```

## Query 2
Inspecting Loop Parameters. What loop parameters did I tune? What were the values when I changed and what values did I change into? When and in which itration did each adaptation happen?
```sql
SELECT 
  loop_tuning.id,
  loop_tuning.starttime,
  loop_tuning.description,
  loop_param.name,
  loop_param.specification,
  loop_param_tuned.old_value,
  loop_param_tuned.new_value,
  out_solver.time_iteration
FROM
  loop_adaptation loop_tuning,
  attribute loop_param,
  attribute_tuned loop_param_tuned,
  person pers,
  execute_data_transformation exec_dt,
  parameter_tuning_exec_data_transformation loop_tuning_exec_dt,
  out_solver_dt out_solver
WHERE
  loop_param_tuned.parameter_tuning_id = loop_tuning.id and
  loop_param_tuned.attribute_id = loop_param.id and
  loop_tuning.id = pers.id and  
  exec_dt.id = loop_tuning_exec_dt.execute_data_transformation_id and
  loop_tuning_exec_dt.parameter_tuning_id = loop_tuning.id and
  out_solver.exec_id = exec_dt.id and
  loop_param.annotation = 'or parameter used for loop control' and  
  pers.name = 'Bob'
ORDER BY
  loop_tuning.id
```


