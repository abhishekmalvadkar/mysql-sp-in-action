### Requirement 1: Employee Salary Update

#### Create a stored procedure that:

* Takes an employee_id and a percentage_increase as input.
* Increases the employee's salary by the given percentage.
* Ensures the salary does not exceed a maximum cap of $200,000.
* Returns the updated salary.

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    salary DECIMAL(10,2) NOT NULL
);
```

```sql
INSERT INTO employees (name, salary) VALUES 
('Alice Johnson', 50000),
('Bob Smith', 120000),
('Charlie Davis', 190000);
```

<details>
  <summary>Click to show/hide solution</summary>

```sql
drop procedure if exists sp_update_employee_salary;

delimiter $$

create procedure sp_update_employee_salary
(
	in p_emp_id bigint,
	in p_percentage_increase decimal(5,2)
)

begin
	
	declare v_new_salary decimal(10,2);

	-- Calculate new salary
	select 
		e.salary + (e.salary * p_percentage_increase/100) into v_new_salary 
	from employees e 
	where e.employee_id = p_emp_id;

	-- Ensure salary does not exceed $200,000
	if v_new_salary > 2000000 then
		set v_new_salary = 2000000;
	end if;

	-- Update salary
	update employees e 
	set e.salary = v_new_salary
	where e.employee_id = p_emp_id;

	-- Return updated salary
	select e.salary from employees e where e.employee_id = p_emp_id;
	
	
end $$

delimiter ;
```
</details>

```sql
call sp_update_employee_salary(1, 10);
```

### Logging or tracking salary changes

```sql
CREATE TABLE salary_log (
    log_id INT PRIMARY KEY AUTO_INCREMENT,
    employee_id INT,
    old_salary DECIMAL(10,2),
    new_salary DECIMAL(10,2),
    change_percentage DECIMAL(5,2),
    log_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    error_message VARCHAR(255),
    FOREIGN KEY (employee_id) REFERENCES employees(employee_id)
);
```

### Updated stored procedure

<details>
  <summary>Click to show/hide solution</summary>
	
```sql
drop procedure if exists sp_update_employee_salary;

delimiter $$

create procedure sp_update_employee_salary
(
	in p_emp_id bigint,
	in p_percentage_increase decimal(5,2)
)

begin
	
	declare v_old_salary decimal(10,2);
	declare v_new_salary decimal(10,2);

	-- Find old salary
	select e.salary into v_old_salary from employees e where e.employee_id = p_emp_id;

	-- Calculate new salary
	select 
		e.salary + (e.salary * p_percentage_increase/100) into v_new_salary 
	from employees e 
	where e.employee_id = p_emp_id;

	-- Ensure salary does not exceed $200,000
	if v_new_salary > 2000000 then
		set v_new_salary = 2000000;
	end if;

	-- Update salary
	update employees e 
	set e.salary = v_new_salary
	where e.employee_id = p_emp_id;

	-- Log salary update
	insert into salary_log(employee_id, old_salary, new_salary, change_percentage)
	values (p_emp_id, v_old_salary, v_new_salary, p_percentage_increase);

	-- Return updated salary
	select e.salary from employees e where e.employee_id = p_emp_id;
	
	
end $$

delimiter ;
```
</details>
