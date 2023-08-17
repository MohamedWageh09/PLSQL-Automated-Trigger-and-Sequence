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
