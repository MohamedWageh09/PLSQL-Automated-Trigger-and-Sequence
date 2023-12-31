``` sql
------------THE VIEW-----------
CREATE OR REPLACE VIEW proj_view AS SELECT u.table_name AS table_name, u.column_name AS column_name
FROM user_tab_columns u
JOIN user_cons_columns uc_cols
ON u.column_name = uc_cols.column_name
JOIN user_constraints u_cons
ON uc_cols.constraint_name = u_cons.constraint_name
WHERE u_cons.constraint_type = 'P'
AND u.data_type = 'NUMBER'
AND u.table_name = uc_cols.table_name
AND u.table_name NOT IN (SELECT table_name FROM user_cons_columns WHERE position > 1);
------ where the column datatype is number --------
SELECT * FROM user_tab_columns u;
------ where the constraint name in user_cons_columns = constraint name in user_constraints
------ where position is < 1
SELECT * FROM user_cons_columns uc_cols;
----- where constraint type is primary key ------
SELECT * FROM user_constraints u_cons;
--------CHECK---------
SELECT * FROM proj_view;
```

```sql
---------The Script ---------

DECLARE
CURSOR tables_cur IS (SELECT * FROM proj_view);
v_seq_counter NUMBER;
seq_name VARCHAR2(30);
v_next_val number;
BEGIN
  FOR table_rec IN tables_cur LOOP
      SELECT COUNT(*) 
      INTO v_seq_counter
      FROM user_sequences
       WHERE UPPER(sequence_name) = UPPER(table_rec.table_name || '_SEQ');
       --drop seq if existed
      IF v_seq_counter > 0 THEN
        EXECUTE IMMEDIATE 'DROP SEQUENCE ' || table_rec.table_name || '_SEQ';
      END IF;
      -- the new seq
        EXECUTE IMMEDIATE 'SELECT MAX(' || table_rec.column_name || ') + 1 FROM ' || table_rec.table_name INTO v_next_val;
        EXECUTE IMMEDIATE 'CREATE SEQUENCE ' || table_rec.table_name || '_SEQ START WITH ' || v_next_val;    
    -- the trigger 
    EXECUTE IMMEDIATE 'CREATE OR REPLACE TRIGGER ' || table_rec.table_name || '_SEQ_TRIG
                         BEFORE INSERT ON ' || table_rec.table_name || '
                         FOR EACH ROW
                         BEGIN
                           :new.' || table_rec.column_name || ' := ' || table_rec.table_name || '_SEQ.nextval;
                         END;';
  END LOOP;
END;
```
