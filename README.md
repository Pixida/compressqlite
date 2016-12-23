# compresSQLite
compresSQLite is a tool to compress a SQLite database for data exchange, thus reducing the data and time consumption of the file transmission.

This tool converts a SQLite databases into a special binary file and vice versa, in order to improve the compression ratio of subsequent compression. The compression improvement is achieved by converting the row-oriented SQLite database to a file with column-oriented layout. Laying out similar data (everything in a database column has a high likelyhood to represent similar data) in a consequitive space on the harddisk. As a result, subsequent compression can do a better job.

The content of the binary file is described as follows:

Database description:

| 1        | 2        |        |
| -------- |--------- | ------ |
| Byte Count header (BC)      | Header |... |

Table description:

|     | 3a        | 3b        | 3c        | 3d1        | 3d2        |     |
| --- | --------- |---------- | --------- | ---------- | ---------- | --- |
| ... | BC create stmt | Create stmt | Number of cols | Col name | Row Count | ... |

Table data description:

|     | 3d3        | [3d4]        | 3d5        |     |
| --- | ---------- |------------- | ---------- | --- |
| ... | BC static payload | [BC dynamic payload] | Payload | ... |

For each table in the database, point 3_ is written to the binary file in a loop. Additionaly, for each column of each table, point 3d_ is also written to the binary file in a loop.

More detailled description of the structure:

1. Byte Count Header (2 bytes): Amount of bytes of the file header. 	
2. Header:
  * compresSQLite MagicNumber (TBD)
  * SQLite version of the original database
  * File checksum
3. Table description:
  * 3a. Byte Count create statement (2 bytes): Amount of bytes of the table creation statement.
  * 3b. Create statement: The table creation statement with column names and types. Also holds information about the primary keys, indices, unique constraints, and other table features like FTS or Spellfix1.
  * 3c. Number of columns (2 bytes): Number of columns in the table.
  * 3d1. Col name (2 bytes): Name of the column
  * 3d2. Row Count (4 bytes): The amount of rows in the table.
  * 3d3. Byte Count static payload (4 bytes): when all entries in the column are of the same length, this field contains the number of bytes of the single payload. If the entries differ in size, all 4 bytes of this entry are 0.
  * 3d4. [Byte Count dynamic payload (4 bytes): Only if the 3d3 field is zero ("0000"), the dynamic payload field is used. Contains the number of bytes of the column data.]
  * 3d5. Payload: The column data.


All defined limits are derived from the SQLite implementation limitations (https://www.sqlite.org/limits.html) and therefore do not add further limitations to the processed database.
