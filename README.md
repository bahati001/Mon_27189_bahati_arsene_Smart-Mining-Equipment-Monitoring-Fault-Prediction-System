⛏️ Smart Mining Equipment Monitoring & Fault Prediction System

Oracle 21c PL/SQL Capstone Project

Name: BAHATI Arsene
Student ID: …
Group: …

📘 Project Description

This capstone project implements a Smart Mining Equipment Monitoring & Fault Prediction System using Oracle 21c PL/SQL.
The system is designed to enhance mine safety, reduce equipment downtime, and support predictive maintenance by monitoring equipment conditions in real time.

The system captures sensor data such as temperature, vibration, and pressure, automatically detects faults, predicts failure risk, restricts unauthorized modifications during critical periods, and audits all user actions.

This project simulates the backend of a real-world Management Information System (MIS) used in modern mining operations.

🔧 PL/SQL Concepts Used

Table structure modifications (DDL)

Conditional logic in DML

Analytical (window) functions

Functions and packages

Triggers enforcing business rules

Auditing and user activity tracking

Security and access control

🧱 Schema Enhancements – Table Modification (DDL)

To support equipment health evaluation, a new column is added to the Equipment table:

ALTER TABLE Equipment
ADD Health_Status VARCHAR2(20);

📌 Purpose

This column classifies each piece of equipment as:

‘Healthy’

‘Warning’

‘Critical’

based on sensor readings and fault history.
It plays a key role in predictive maintenance decisions.

🛠 Data Manipulation Operations (DML)
✅ Mark Equipment Health Status

Equipment condition is evaluated automatically:

UPDATE Equipment
SET Health_Status = CASE
    WHEN Usage_Hours < 5000 THEN 'Healthy'
    WHEN Usage_Hours BETWEEN 5000 AND 8000 THEN 'Warning'
    ELSE 'Critical'
END;

✅ Delete Incorrect Fault Records

Removes wrongly logged fault data:

DELETE FROM Fault_Log
WHERE Fault_ID = 205;

✅ Insert New Maintenance Record

Logs a completed maintenance activity:

INSERT INTO Maintenance_Records
(Record_ID, Equipment_ID, Maintenance_Date, Cost, Status)
VALUES (401, 'EQ01', SYSDATE, 850000, 'Completed');

📊 Analytical Reporting with Window Functions

To analyze equipment failure risk, an analytical query calculates total faults per equipment:

SELECT 
    e.Equipment_ID,
    e.Equipment_Type,
    COUNT(f.Fault_ID) OVER (PARTITION BY e.Equipment_ID) AS Total_Faults
FROM Equipment e
JOIN Fault_Log f ON e.Equipment_ID = f.Equipment_ID
ORDER BY Total_Faults DESC;

📌 BI Insight

This report helps management identify:

High-risk machines

Equipment requiring urgent replacement

Patterns of repeated failures

📦 Function and Package: MaintenanceTools
✅ Function: Get_Equipment_Health

Returns the health status of an equipment unit:

CREATE OR REPLACE FUNCTION Get_Equipment_Health(p_equipment_id VARCHAR2)
RETURN VARCHAR2 IS
    v_status VARCHAR2(20);
BEGIN
    SELECT Health_Status INTO v_status
    FROM Equipment
    WHERE Equipment_ID = p_equipment_id;
    RETURN v_status;
EXCEPTION
    WHEN NO_DATA_FOUND THEN RETURN 'Unknown';
    WHEN OTHERS THEN RETURN 'Error';
END;
/

✅ Package: MaintenanceTools
CREATE OR REPLACE PACKAGE MaintenanceTools AS
    PROCEDURE Get_Maintenance_History(p_equipment_id IN VARCHAR2);
    FUNCTION Get_Equipment_Health(p_equipment_id VARCHAR2) RETURN VARCHAR2;
END MaintenanceTools;
/

✅ Package Body
CREATE OR REPLACE PACKAGE BODY MaintenanceTools AS

    PROCEDURE Get_Maintenance_History(p_equipment_id IN VARCHAR2) IS
    BEGIN
        FOR rec IN (
            SELECT Maintenance_Date, Cost, Status
            FROM Maintenance_Records
            WHERE Equipment_ID = p_equipment_id
        ) LOOP
            DBMS_OUTPUT.PUT_LINE(
                'Date: ' || rec.Maintenance_Date ||
                ', Cost: ' || rec.Cost ||
                ', Status: ' || rec.Status
            );
        END LOOP;
    END;

    FUNCTION Get_Equipment_Health(p_equipment_id VARCHAR2)
    RETURN VARCHAR2 IS
        v_status VARCHAR2(20);
    BEGIN
        SELECT Health_Status INTO v_status
        FROM Equipment
        WHERE Equipment_ID = p_equipment_id;
        RETURN v_status;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN RETURN 'Unknown';
    END;

END MaintenanceTools;
/

📅 Restricted Maintenance Periods (Safety Control)
✅ Step 1: Create Restricted_Dates Table
CREATE TABLE Restricted_Dates (
    Restricted_Date DATE PRIMARY KEY,
    Reason VARCHAR2(100)
);

INSERT INTO Restricted_Dates
VALUES (TO_DATE('2025-06-01','YYYY-MM-DD'), 'Safety Inspection Day');

✅ Step 2: Trigger to Restrict DML During Critical Periods
CREATE OR REPLACE TRIGGER Restrict_Critical_Period_Changes
BEFORE INSERT OR UPDATE OR DELETE ON Maintenance_Records
DECLARE
    v_count NUMBER;
BEGIN
    SELECT COUNT(*) INTO v_count
    FROM Restricted_Dates
    WHERE Restricted_Date = TRUNC(SYSDATE);

    IF v_count > 0 THEN
        RAISE_APPLICATION_ERROR(
            -20010,
            'Maintenance updates are restricted during safety inspection periods.'
        );
    END IF;
END;
/

🔍 Audit Trail and User Action Logging
✅ Audit Log Table
CREATE TABLE Equipment_Audit_Log (
    Log_ID NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    User_Name VARCHAR2(50),
    Action_Type VARCHAR2(10),
    Table_Name VARCHAR2(50),
    Action_Date DATE,
    Status VARCHAR2(20),
    Description VARCHAR2(200)
);

✅ Audit Trigger
CREATE OR REPLACE TRIGGER Audit_Equipment_Changes
AFTER INSERT OR UPDATE OR DELETE ON Equipment
FOR EACH ROW
BEGIN
    INSERT INTO Equipment_Audit_Log
    (User_Name, Action_Type, Table_Name, Action_Date, Status, Description)
    VALUES (
        SYS_CONTEXT('USERENV','SESSION_USER'),
        CASE
            WHEN INSERTING THEN 'INSERT'
            WHEN UPDATING THEN 'UPDATE'
            ELSE 'DELETE'
        END,
        'Equipment',
        SYSDATE,
        'Logged',
        'Equipment data modified'
    );
END;
/

🧪 Testing and Validation
✅ Check Restricted Dates
SELECT * FROM Restricted_Dates
WHERE Restricted_Date = TRUNC(SYSDATE);

✅ Disable Trigger (Testing)
ALTER TRIGGER Restrict_Critical_Period_Changes DISABLE;

✅ Insert Test Data
INSERT INTO Equipment
VALUES ('EQ99', 'Drill Machine', 9500, 'Critical');

✅ Enable Trigger Again
ALTER TRIGGER Restrict_Critical_Period_Changes ENABLE;

✅ View Audit Log
SELECT * FROM Equipment_Audit_Log
ORDER BY Action_Date DESC;

✅ Conclusion

The Smart Mining Equipment Monitoring & Fault Prediction System demonstrates a powerful PL/SQL-based MIS solution that improves mining operations by:

Automating fault detection

Supporting predictive maintenance

Enforcing safety-based access control

Providing reliable audit trails

Enabling BI-driven decisions

It effectively showcases advanced Oracle PL/SQL capabilities and aligns with real-world industrial systems used to improve safety and operational efficiency in mining environments.
