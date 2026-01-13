# Recycling-
# GreenCycle Management System (GCMS) — Requirements

## Functional Requirements
1. Users should be able to record incoming waste by selecting category, entering weight, and choosing a supplier.
2. Users should be able to receive a unique Waste Intake ID automatically generated for each new waste intake record.
3. Users should be able to track waste processing stages, including sorting, shredding, and compacting.
4. Staff should be able to update waste processing status in real time from the system.
5. Users should be able to store and view details of processed output such as plastic pellets and metal scraps.
6. Users should be able to manage sales transactions for processed or recycled materials.
7. Users should be able to automatically generate invoices after completing sales transactions.
8. Users should be able to maintain supplier profiles, including contact details and transaction history.
9. Users should be able to view dashboards that show daily, weekly, and monthly operational reports.
10. Users should be able to log in securely and access features based on their assigned role and permissions.
11. Users should be able to export reports in PDF or Excel format for business use.
12. Users should be able to receive system alerts when processed material quantities fall below the defined stock threshold.

## Non-Functional Requirements
1. Performance: System must handle up to 200 concurrent users with response time < 3 seconds.
2. Security: All data transactions must be encrypted using HTTPS/TLS 1.2 or higher.
3. Usability: User interface must follow a consistent layout and be usable with less than 2 hours of training.
4. Scalability: System must support a 2× increase in records without performance degradation.
5. Reliability: System uptime must be at least 99% per month.
6. Backup: Database must automatically back up daily.
