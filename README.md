# PROV-DfA

This is an online appendix for [IPAW'18](http://provenanceweek2018.org/ipaw/) paper entitled *[Provenance of Dynamic Adaptations in User-steered Dataflows](https://link.springer.com/chapter/10.1007/978-3-319-98379-0_2)*.

The paper introduces PROV-DfA, a dataflow-aware provenance representation of steering actions to model data relationships between user steering actions and their consequences on the intermediate data production and experiment results.

This appendix shows a larger image of PROV-DfA general overview and examplary SQL queries that uses a relational database schema that implements PROV-DfA.

# PROV-DfA general overview
![alt text](https://raw.githubusercontent.com/hpcdb/PROV-DfA/master/PROV-DfA.png "PROV-DfA provenance data model")

# Examplary Queries for Provenance of Adaptation

## Query 1:
Inspecting parameter tunings ("who", "when", "what"). How many tunings did I do? Which parameters did I change? What were the values when I changed and what values did I change into? When did each adaptation happen?

```sql
SELECT
  param_tuning.id,
  param_tuning.starttime,
  param_tuning.description,
  param.name,
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
  param_tuning.person_id = pers.id and  
  param.specification = 'P_I' and  
  pers.name = 'Bob'
ORDER BY
  param_tuning.id
```

## Query 2
Inspecting Loop Parameters. What loop parameters did I tune? What were the values when I changed and what values did I change into? When and in which itration did each adaptation happen?
```sql
SELECT
  loop_adp.id,
  loop_adp.starttime,
  loop_adp.description,
  loop_param.name,
  loop_param_tuned.old_value,
  loop_param_tuned.new_value,
  out_solver.time_iteration
FROM
  loop_adaptation loop_adp,
  attribute loop_param,
  attribute_tuned loop_param_tuned,
  person pers,
  execute_data_transformation exec_dt,
  parameter_tuning_exec_data_transformation loop_tuning_exec_dt,
  solver_out_dtd out_solver
WHERE
  loop_param_tuned.parameter_tuning_id = loop_adp.id and
  loop_param_tuned.attribute_id = loop_param.id and
  loop_adp.person_id = pers.id and  
  exec_dt.id = loop_tuning_exec_dt.execute_data_transformation_id and
  loop_tuning_exec_dt.loop_tuning_id = loop_adp.id and
  out_solver.exec_id = exec_dt.id and
  loop_param.specification = 'L_I' and  
  pers.name = 'Bob'
ORDER BY
  loop_adp.id
```

## Query 3
In parameter tuning 3, how was the output values 10 iterations before and after?
```sql
SELECT
  loop_adp.starttime,
  loop_adp.description,
  out_solver_moment_at_adaptation.time_iteration,
  avg(out_solver_before.final_linear_residual),
  avg(out_solver_after.final_non_linear_residual)  
FROM
  loop_adaptation loop_adp,
  attribute loop_param,
  attribute_tuned loop_param_tuned,
  person pers,
  execute_data_transformation exec_dt,
  parameter_tuning_exec_data_transformation loop_tuning_exec_dt,
  solver_out_dtd out_solver_moment_at_adaptation,
  solver_out_dtd out_solver_before,
  solver_out_dtd out_solver_after
WHERE
  loop_adp.id = 3 and
  loop_adp.id = loop_tuning_exec_dt.loop_tuning_id and
  loop_tuning_exec_dt.execute_data_transformation_id = exec_dt.id and
  exec_dt.id = out_solver_moment_at_adaptation.exec_id and
  out_solver_before.time_iteration < out_solver_moment_at_adaptation.time_iteration and
  out_solver_after.time_iteration > (out_solver_moment_at_adaptation.time_iteration - 10) and
  out_solver_after.time_iteration > out_solver_moment_at_adaptation.time_iteration and
  out_solver_after.time_iteration < (out_solver_moment_at_adaptation.time_iteration + 10) and
  loop_adp.person_id = pers.id and  
  pers.name = 'Bob'
GROUP BY
  loop_adp.starttime, loop_adp.description, out_solver_moment_at_adaptation.time_iteration
```

## Query 4
On average, how long iterations were lasting before and after I reduced input files from the input data? Which files were affected?
```sql
SELECT
  data_red.id,
  in_solver.file_path,
  out_solver_moment_at_adaptation.time_iteration,
  avg(exec_data_before.endtime-exec_data_before.starttime),
  avg(exec_data_after.endtime-exec_data_after.starttime)
FROM
  data_reduction data_red,
  deleted_data_element del_element,
  person pers,
  solver_files_in_dtd in_solver,
  solver_out_dtd out_solver_moment_at_adaptation,
  execute_data_transformation exec_dt_time_loop, --  during execution, there is always a dt_time_loop running in a time loop simulation
  execute_data_transformation exec_data_before,
  execute_data_transformation exec_data_after
WHERE
  exec_dt_time_loop.dt_id = 3 and -- loop evaluate DT
  data_red.id = data_red_exec_dt.data_reduction_id and
  data_red_exec_dt.execute_data_transformation_id = exec_dt_time_loop.id and
  exec_dt_time_loop.id = out_solver_moment_at_adaptation.exec_id and
  data_red.id = del_element.data_reduction_id and
  del_element.data_element_id = in_solver.id and  
  exec_data_before.id < exec_dt_time_loop.id and
  exec_data_after.id > exec_dt_time_loop.id and
  data_red.person_id = pers.id and  
  pers.name = 'Bob'
GROUP BY
  data_red.id, in_solver.file_path, out_solver_moment_at_adaptation.time_iteration
```
