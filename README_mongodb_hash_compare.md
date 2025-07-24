# MongoDB Hash Comparison Script

A Python script that compares hash values between source and destination MongoDB clusters using the `dbHash` command and generates an Excel report highlighting mismatches.

## Overview

This script performs the following operations:
1. Connects to both source and destination MongoDB clusters
2. Runs the `dbHash` command against all non-system databases on both clusters
3. Compares hash values at both database and collection levels
4. Generates a detailed Excel report with color-coded results highlighting mismatches

## Features

- **Comprehensive Comparison**: Compares both database-level MD5 hashes and individual collection hashes
- **Visual Excel Report**: Color-coded Excel output with conditional formatting:
  - 🟢 Green: Matching hashes
  - 🔴 Red: Mismatched hashes
  - 🟠 Orange: Missing databases/collections
- **Detailed Logging**: Comprehensive logging to both file and console
- **Summary Statistics**: Includes a summary sheet with overall comparison statistics
- **Error Handling**: Robust error handling with informative error messages
- **System Database Exclusion**: Automatically excludes system databases (admin, local, config)

## Requirements

- Python 3.7+
- MongoDB clusters accessible via connection strings
- Required Python packages (see requirements.txt):
  - pymongo>=4.0.0
  - pandas>=1.5.0
  - openpyxl>=3.0.0

## Installation

1. Install required packages:
```bash
pip install -r requirements.txt
```

## Usage

### Basic Usage

```bash
python mongodb_hash_compare.py --source "mongodb://source-cluster:27017" --destination "mongodb://dest-cluster:27017"
```

### With Authentication

```bash
python mongodb_hash_compare.py \
  --source "mongodb://username:password@source-cluster:27017/admin" \
  --destination "mongodb://username:password@dest-cluster:27017/admin"
```

### With Custom Output File

```bash
python mongodb_hash_compare.py \
  --source "mongodb://source-cluster:27017" \
  --destination "mongodb://dest-cluster:27017" \
  --output "my_comparison_report.xlsx"
```

### With Verbose Logging

```bash
python mongodb_hash_compare.py \
  --source "mongodb://source-cluster:27017" \
  --destination "mongodb://dest-cluster:27017" \
  --verbose
```

## Command Line Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--source` | Yes | Source MongoDB connection string |
| `--destination` | Yes | Destination MongoDB connection string |
| `--output` | No | Output Excel file path (default: auto-generated with timestamp) |
| `--verbose` | No | Enable verbose logging |

## Connection String Examples

### Standalone MongoDB
```
mongodb://localhost:27017
mongodb://username:password@localhost:27017
```

### Replica Set
```
mongodb://user:pass@host1:27017,host2:27017,host3:27017/?replicaSet=myReplicaSet
```

### MongoDB Atlas
```
mongodb+srv://username:password@cluster.mongodb.net/?retryWrites=true&w=majority
```

### With SSL/TLS
```
mongodb://username:password@host:27017/?ssl=true&ssl_cert_reqs=CERT_NONE
```

## Output Format

The script generates an Excel file with two sheets:

### 1. Hash Comparison Sheet
Contains detailed comparison data with the following columns:
- **Type**: Database or Collection
- **Database**: Database name
- **Collection**: Collection name (empty for database-level rows)
- **Source_Hash**: Hash value from source cluster
- **Destination_Hash**: Hash value from destination cluster
- **Match**: Comparison result (MATCH/MISMATCH/MISSING DB/MISSING COLLECTION)
- **Source_Host**: Source cluster host information
- **Dest_Host**: Destination cluster host information
- **Source_Time_ms**: Time taken for dbHash on source (database level only)
- **Dest_Time_ms**: Time taken for dbHash on destination (database level only)

### 2. Summary Sheet
Contains overall statistics:
- Total databases compared
- Database hash mismatches
- Missing databases
- Total collections compared
- Collection hash mismatches
- Missing collections
- Overall status (PASS/FAIL)

## Color Coding

- 🟢 **Light Green**: Matching hashes between source and destination
- 🔴 **Light Red**: Mismatched hashes between source and destination
- 🟠 **Light Orange**: Missing databases or collections (exists in one cluster but not the other)

## Logging

The script creates a log file `mongodb_hash_compare.log` and also outputs to console. Log levels include:
- INFO: General operation information
- ERROR: Error conditions
- DEBUG: Detailed debugging information (when --verbose is used)

## Important Notes

### Performance Considerations
- The `dbHash` command obtains a shared (S) lock on databases, preventing writes during execution
- Runtime depends on database sizes and number of collections
- Consider running during maintenance windows for production systems

### Limitations
- Not supported on MongoDB Atlas M0, Flex clusters, or serverless instances
- Requires read access to all databases being compared
- Network connectivity required to both clusters simultaneously

### Security
- Connection strings may contain credentials - ensure proper security practices
- Consider using MongoDB connection string options for SSL/TLS encryption
- Avoid hardcoding credentials in scripts

## Example Output

```
2025-07-24 16:20:22 - INFO - Starting MongoDB Hash Comparison
2025-07-24 16:20:22 - INFO - Source cluster: source-cluster:27017
2025-07-24 16:20:22 - INFO - Destination cluster: dest-cluster:27017
2025-07-24 16:20:23 - INFO - Connecting to source cluster...
2025-07-24 16:20:23 - INFO - Source cluster connection successful
2025-07-24 16:20:24 - INFO - Connecting to destination cluster...
2025-07-24 16:20:24 - INFO - Destination cluster connection successful
2025-07-24 16:20:24 - INFO - Found 3 non-system databases: ['testdb', 'inventory', 'analytics']
2025-07-24 16:20:25 - INFO - === COMPARISON SUMMARY ===
2025-07-24 16:20:25 - INFO - Total Databases: 3
2025-07-24 16:20:25 - INFO - Database Mismatches: 0
2025-07-24 16:20:25 - INFO - Missing Databases: 0
2025-07-24 16:20:25 - INFO - Total Collections: 15
2025-07-24 16:20:25 - INFO - Collection Mismatches: 2
2025-07-24 16:20:25 - INFO - Missing Collections: 1
2025-07-24 16:20:25 - INFO - Overall Status: FAIL
2025-07-24 16:20:25 - INFO - Excel report saved to: mongodb_hash_comparison_20250724_162025.xlsx
```

## Troubleshooting

### Connection Issues
- Verify connection strings are correct
- Check network connectivity to both clusters
- Ensure proper authentication credentials
- Verify firewall and security group settings

### Permission Issues
- Ensure the user has read access to all databases
- For Atlas clusters, ensure the database user has appropriate roles

### Performance Issues
- Consider running during low-traffic periods
- Monitor cluster performance during hash computation
- Use connection pooling options if needed

## Author Information

This script was created to facilitate MongoDB cluster comparisons and data validation processes. It's particularly useful for:
- Migration validation
- Replica set consistency checks
- Disaster recovery verification
- Data synchronization validation
