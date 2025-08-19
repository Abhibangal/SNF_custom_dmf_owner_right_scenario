# Handling Dynamic Object References in Snowflake Stored Procedures

## 1. Scenario
In Snowflake, a stored procedure can be created with **OWNER’s rights**.  
A common design pattern is to make such procedures accept **dynamic object names** as arguments (for example, the name of a function and the name of a table).  

A challenge arises when:  
- The **caller** of the procedure has the required privileges on those objects.  
- The **procedure owner** does **not** have any privileges on them.  

This raises the question:  
**Can the caller still use the stored procedure successfully in this setup?**

---

## 2. Key Consideration
By default, stored procedures created with OWNER’s rights execute under the privileges of the procedure owner.  
If the owner does not have privileges on the objects being passed in dynamically, access issues can occur.

However, the caller is the one who *does* have the necessary access. We need a way to ensure that object resolution happens in the **caller’s security context**, not the owner’s.

---

## 3. Solution: Using `SYSTEM$REFERENCE`
Snowflake provides a built-in system function called **`SYSTEM$REFERENCE`**, which allows objects to be passed securely into stored procedures.  

Instead of passing raw object names as strings, the caller provides **object references** through this function.  
- The reference validates against the caller’s privileges.  
- The stored procedure can then safely use those references without the owner needing direct access.  

This ensures:  
- The procedure remains reusable and secure.  
- The principle of **least privilege** is preserved.  
- Ownership of the procedure does not require unnecessary grants.  

---

## 4. Example Use Case
A practical application of this pattern is in **data quality checks**.  
- A **custom data metric function (DMF)** is created to validate data quality rules (e.g., checking for invalid emails in a table).  
- A **stored procedure** is designed to accept the DMF name, table name, and column name as arguments.  
- The caller, who has access to the DMF and the table, invokes the procedure by passing object references using `SYSTEM$REFERENCE`.  

This way:  
- The procedure owner does not need direct access to the function or the table.  
- The caller’s privileges are respected during execution.  

---

## 5. Benefits of This Approach
- **Security**: Objects are accessed under the caller’s privileges, not the owner’s.  
- **Governance**: Encourages the use of least privilege in production environments.  
- **Flexibility**: Procedures remain reusable across roles and users with different access levels.  
- **Maintainability**: Reduces the need to grant wide privileges to procedure owners.  

---

## 6. Conclusion
When building stored procedures with dynamic object arguments in Snowflake, direct object names cannot always be passed if the procedure owner lacks access.  
The solution is to leverage **`SYSTEM$REFERENCE`**, ensuring that object resolution occurs in the **caller’s context**.  

This design pattern is particularly valuable for **data quality frameworks, governance processes, and reusable analytics utilities** where ownership and access may differ across roles.
