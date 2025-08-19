# Snowflake Stored Procedure with Dynamic Object References

## Scenario
Consider a **stored procedure** created with **OWNER’s rights** in Snowflake.  
This procedure accepts **dynamic object names** (e.g., a function and a table) as arguments.

- The **caller** has access to the function and table.  
- The **procedure owner does not**.  

### Question
Can the caller still run the stored procedure successfully?  

👉 **Answer: YES — but not by directly passing object names.**

---

## Solution: Using `SYSTEM$REFERENCE`

Snowflake provides the system function `SYSTEM$REFERENCE` to securely pass object references in the **caller’s context**, instead of passing raw names as strings.  
This ensures that **access checks** happen based on the caller’s privileges, not the procedure owner’s.

---

## Example Walkthrough

1. **Create a custom Data Metric Function (DMF)**  
   ```sql
   create or replace function EMAIL_CHECK(table_name varchar)
   returns number
   as
   $$
     -- logic to check invalid emails
   $$;
