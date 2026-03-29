###1. What is a Trigger?

A trigger is a block of code that defines an action to be executed automatically when a specific event occurs in the database, provided a certain condition is met.

### 2. Trigger Components (The ECA Model)

A trigger is defined by three main parts:

- **Event:** The SQL instruction that activates the trigger, such as `INSERT`, `UPDATE`, or `DELETE`.
    
- **Condition:** An optional `WHEN` clause that must be true for the trigger's action to run.
    
- **Action:** The block of PL/SQL code that executes when the event occurs and the condition is met.
    

### 3. Types of Triggers

Triggers are categorized by their timing and the level at which they operate:

**Timing (Before vs. After)**

- **BEFORE:** Runs _before_ the SQL statement is executed. It is ideal for validating or modifying data before it is saved.
    
- **AFTER:** Runs _after_ the SQL statement is executed. It is used for actions that depend on the data being successfully saved, like updating audit logs.
    

**Level (Statement vs. Row)**

- **Statement-level:** Executes exactly once for the SQL statement, regardless of how many rows are affected.
    
- **Row-level (`FOR EACH ROW`):** Executes once for every single row affected by the SQL statement. This level allows you to use specific variables:
    
    - `:OLD`: Refers to the data _before_ the change.
        
    - `:NEW`: Refers to the data _after_ the change.
        

### 4. Special Triggers: `INSTEAD OF`

An `INSTEAD OF` trigger is used specifically on **Views**. It allows you to update a complex view (like one involving multiple tables) that the database wouldn't normally allow you to modify directly. Instead of the database trying to update the view, it runs your trigger code to manually update the underlying base tables.

### 5. Practical Example

A common use for a trigger is enforcing complex business rules, such as ensuring an employee's salary stays within the minimum and maximum range defined for their grade. If a user tries to `UPDATE` a salary to an invalid amount, the trigger catches the error and can stop the transaction using `raise_application_error`.



### Example :

### 1. BEFORE vs. AFTER (Timing)

The timing determines if the code runs before or after the database change is committed.

- **BEFORE Example (Data Validation):** Imagine a rule where a hotel room's `TarifNuitée` (nightly rate) cannot be less than 5,000 DA. A **BEFORE** trigger can check the value and block the update if it's too low.
    ```sql
    CREATE OR REPLACE TRIGGER Check_Min_Price
    BEFORE INSERT OR UPDATE OF TarifNuitée ON Chambre
    FOR EACH ROW
    BEGIN
        IF :NEW.TarifNuitée < 5000 THEN
            RAISE_APPLICATION_ERROR(-20001, 'Price too low for this hotel.');
        END IF;
    END;
    ```
    
- **AFTER Example (Auditing):** After a reservation is successfully made, you might want to log the action in a separate `Audit_Log` table for security.
    ```sql
    CREATE OR REPLACE TRIGGER Log_New_Reservation
    AFTER INSERT ON Reservation
    FOR EACH ROW
    BEGIN
        INSERT INTO Audit_Log (Action, TableName, ActionDate)
        VALUES ('INSERT', 'Reservation', SYSDATE);
    END;
    ```


### 2. Statement-Level vs. Row-Level

This defines how many times the trigger code executes.

- **Statement-Level (Security Check):** If you want to prevent _any_ deletions from the `Hotel` table during the weekend, you use a statement trigger. It runs once, whether you try to delete 1 hotel or 100.
```sql
CREATE OR REPLACE TRIGGER Block_Weekend_Delete
    BEFORE DELETE ON Hotel
    BEGIN
        IF TO_CHAR(SYSDATE, 'DY') IN ('SAT', 'SUN') THEN
            RAISE_APPLICATION_ERROR(-20002, 'Deletions not allowed on weekends.');
        END IF;
    END;
```
- **Row-Level (Calculating Values):** Using `FOR EACH ROW`, you can use `:OLD` and `:NEW` variables to react to specific data changes in every row.
    
    - **Example:** Automatically updating a client's `NbCptE` (number of savings accounts) in the `CLIENT` table whenever a new record is added to the `COMPTE` table.
        

### 3. INSTEAD OF Triggers (For Views)

These are used specifically to handle updates on complex views that contain joins or aggregations, which the database cannot update automatically.

- **Example:** Recall the `ChambeHotel` view that joined `Chambre` and `Hotel`. If you want to allow a user to "update" the hotel name through that view, you write an `INSTEAD OF` trigger that manually directs the update to the correct physical `Hotel` table.
    

```sql    CREATE OR REPLACE TRIGGER Update_Hotel_Via_View
    INSTEAD OF UPDATE ON ChambeHotel
    FOR EACH ROW
    BEGIN
        UPDATE Hotel 
        SET NomHotel = :NEW.NomHotel 
        WHERE IdHotel = :OLD.IdHotel;
    END;
```

### 4. Using Conditions (WHEN Clause)

You can add a `WHEN` clause so the trigger action only runs if a specific condition is met, saving system resources.

- **Example:** Only trigger a "High Spending Alert" if the `MontantTotal` of a reservation exceeds 100,000 DA.

```sql
CREATE OR REPLACE TRIGGER High_Value_Alert
    AFTER INSERT ON Reservation
    FOR EACH ROW
    WHEN (NEW.MontantTotal > 100000)
    BEGIN
        -- Code to send an alert or email
        DBMS_OUTPUT.PUT_LINE('High value reservation detected!');
    END;
```


[[2026-03-29_12-45_1.png]]
[[2026-03-29_12-45.png]]