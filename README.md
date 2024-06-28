# Table Schema
![Table Diagram](https://github.com/parthbagda211/database-tasks/blob/branch/tables.png)


# Tables 

```-- Create Clinic table
CREATE TABLE Clinic (
  clinicId INT PRIMARY KEY,
  clinicName VARCHAR(255),
  clinicAddress VARCHAR(255),
  clinicContact VARCHAR(20)
);

INSERT INTO Clinic (clinicId, clinicName, clinicAddress, clinicContact) VALUES
  (1, 'Acme Clinic', '123 Main St, Anytown', '555-1234'),
  (2, 'Apex Clinic', '456 Oak Rd, Somewhere', '555-5678'),
  (3, 'Zenith Clinic', '789 Elm St, Elsewhere', '555-9012');

-- Create Doctor table  
CREATE TABLE Doctor (
  doctorId INT PRIMARY KEY,
  doctorName VARCHAR(255),
  doctorSpecialization VARCHAR(255),
  clinicId INT,
  FOREIGN KEY (clinicId) REFERENCES Clinic(clinicId)
);

INSERT INTO Doctor (doctorId, doctorName, doctorSpecialization, clinicId) VALUES
  (1, 'Dr. Jane Doe', 'Family Medicine', 1),
  (2, 'Dr. John Smith', 'Cardiology', 1),
  (3, 'Dr. Sarah Lee', 'Pediatrics', 2),
  (4, 'Dr. Michael Kim', 'Dermatology', 2),
  (5, 'Dr. Emily Chen', 'Orthopedics', 3);

-- Create Patient table
CREATE TABLE Patient (
  patientId INT PRIMARY KEY,
  patientName VARCHAR(255),
  patientEmail VARCHAR(255),
  patientPhone VARCHAR(20),
  patientDOB DATE
);

INSERT INTO Patient (patientId, patientName, patientEmail, patientPhone, patientDOB) VALUES
  (1, 'Alice Johnson', 'alice@example.com', '555-1111', '1985-03-15'),
  (2, 'Bob Williams', 'bob@example.com', '555-2222', '1990-07-20'),
  (3, 'Charlie Davis', 'charlie@example.com', '555-3333', '1992-11-05'),
  (4, 'David Lee', 'david@example.com', '555-4444', '1978-06-30'),
  (5, 'Emily Chen', 'emily@example.com', '555-5555', '1995-02-12');

-- Create Appointment table
CREATE TABLE Appointment (
  appointmentId INT PRIMARY KEY,
  appointmentDateTime DATETIME,
  patientId INT,
  doctorId INT,
  clinicId INT,
  appointmentStatus VARCHAR(20),
  FOREIGN KEY (patientId) REFERENCES Patient(patientId),
  FOREIGN KEY (doctorId) REFERENCES Doctor(doctorId),
  FOREIGN KEY (clinicId) REFERENCES Clinic(clinicId)
);

INSERT INTO Appointment (appointmentId, appointmentDateTime, patientId, doctorId, clinicId, appointmentStatus) VALUES
  (1, '2024-06-28 10:00:00', 1, 1, 1, 'Booked'),
  (2, '2024-06-28 11:30:00', 2, 1, 1, 'Booked'),
  (3, '2024-06-29 14:00:00', 3, 3, 2, 'Booked'),
  (4, '2024-06-29 15:30:00', 4, 4, 2, 'Booked'),
  (5, '2024-06-30 09:00:00', 5, 5, 3, 'Booked'),
  (6, '2024-06-27 13:00:00', 1, 2, 1, 'Cancelled'),
  (7, '2024-06-26 16:30:00', 2, 3, 2, 'Completed'),
  (8, '2024-06-25 11:00:00', 3, 4, 2, 'Cancelled'),
  (9, '2024-06-24 14:45:00', 4, 5, 3, 'Completed'),
  (10, '2024-06-23 09:30:00', 5, 1, 1, 'Cancelled');
```






# Quries:

1. **All appointments booked in the last 7 days for a doctor**:

```sql
SELECT * 
FROM Appointment
WHERE doctorId = 1
  AND appointmentDateTime >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)
  AND appointmentStatus = 'Booked'
ORDER BY appointmentDateTime DESC;
```

Output:
| appointmentId | appointmentDateTime | patientId | doctorId | clinicId | appointmentStatus |
|---------------|---------------------|-----------|----------|----------|-------------------|
| 2             | 2024-06-28 11:30:00 | 2         | 1        | 1        | Booked            |
| 1             | 2024-06-28 10:00:00 | 1         | 1        | 1        | Booked            |

2. **All appointments booked in the last 2 days and scheduled within the next 5 hours for a doctor**:

```sql
SELECT *
FROM Appointment
WHERE doctorId = 1
  AND appointmentDateTime >= DATE_SUB(CURRENT_DATE(), INTERVAL 2 DAY)
  AND appointmentDateTime <= DATE_ADD(CURRENT_TIMESTAMP(), INTERVAL 5 HOUR)
  AND appointmentStatus = 'Booked'
ORDER BY appointmentDateTime ASC;
```

Output:
| appointmentId | appointmentDateTime | patientId | doctorId | clinicId | appointmentStatus |
|---------------|---------------------|-----------|----------|----------|-------------------|
| 1             | 2024-06-28 10:00:00 | 1         | 1        | 1        | Booked            |
| 2             | 2024-06-28 11:30:00 | 2         | 1        | 1        | Booked            |

3. **Users who have at least 1 appointment and have their birthday coming in the next 5 days**:

```sql
SELECT p.patientId, p.patientName, p.patientDOB
FROM Patient p
INNER JOIN Appointment a ON p.patientId = a.patientId  
WHERE DATE_ADD(p.patientDOB, INTERVAL YEAR(CURRENT_DATE) - YEAR(p.patientDOB) + 
              (CASE WHEN DATE_FORMAT(CURRENT_DATE, '%m-%d') >= DATE_FORMAT(p.patientDOB, '%m-%d') THEN 1 ELSE 0 END) YEAR)
              BETWEEN CURRENT_DATE AND DATE_ADD(CURRENT_DATE, INTERVAL 5 DAY)
GROUP BY p.patientId
HAVING COUNT(a.appointmentId) >= 1;
```

Output:
| patientId | patientName | patientDOB |
|-----------|-------------|------------|
| 2         | Bob Williams| 1990-07-20 |

4. **Appointments for a particular patient in the last 7 days**:

```sql
SELECT *
FROM Appointment
WHERE patientId = 1
  AND appointmentDateTime >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)
ORDER BY appointmentDateTime DESC;
```

Output:
| appointmentId | appointmentDateTime | patientId | doctorId | clinicId | appointmentStatus |
|---------------|---------------------|-----------|----------|----------|-------------------|
| 1             | 2024-06-28 10:00:00 | 1         | 1        | 1        | Booked            |
| 6             | 2024-06-27 13:00:00 | 1         | 2        | 1        | Cancelled         |

5. **Appointment cancellation percentage for a doctor by clinic**:

```sql
SELECT 
  c.clinicName,
  d.doctorName,
  ROUND(100.0 * SUM(CASE WHEN a.appointmentStatus = 'Cancelled' THEN 1 ELSE 0 END) / COUNT(*), 2) AS cancellationPercentage
FROM Appointment a
JOIN Doctor d ON a.doctorId = d.doctorId
JOIN Clinic c ON a.clinicId = c.clinicId
WHERE d.doctorId = 1  
GROUP BY c.clinicName, d.doctorName;
```

Output:
| clinicName | doctorName | cancellationPercentage |
|------------|------------|------------------------|
| Acme Clinic| Dr. Jane Doe| 50.00                  |
| Acme Clinic| Dr. John Smith| 0.00                 |
