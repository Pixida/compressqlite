# compresSQLite
compresSQLite is a tool to compress a SQLite database for data exchange, thus reducing the data and time consumption of the file transmission.

This tool converts a SQLite databases into a special binary file and vice versa, in order to improve the compression ratio of subsequent compression. The compression improvement is achieved by converting the row-oriented SQLite database to a file with column-oriented layout. Laying out similar data (everything in a database column has a high likelyhood to represent similar data) in a consequitive space on the harddisk. As a result, subsequent compression can do a better job.

The content of the binary file is described as follows:

Database description:

| 1      | 2      |
| ------ | ------ |
| Header | SQLite database description |

Table description for each table:

| 2a     | 2b     | 2c     | 3      | 4      |
| ------ | ------ | ------ | ------ | ------ |
| BC CREATE stmt | CREATE stmt | Number of cols | Table col descriptions | Table index descriptions |

Table column description for each column:

| 3a     | 3b     | 3c     | 3d     | 3e     |
| ------ | ------ | ------ | ------ | ------ |
| BC col payload | BC col name | Col name | Row Count | Col data |

Column data description:

| 3e1    | [3e2] 3e3    |
| ------ | ------------ |
| BC data constant | [BC data dynamic] data  |

Table index description:

| 4a     | 4b     | 4c     |
| ------ | ------ | ------ |
| BC index CREATE stmt | Index CREATE stmt | End of Index marker |

Table index description:

For each table in the database, point 3_ is written to the binary file in a loop. Additionaly, for each column of each table, point 3d_ is also written to the binary file in a loop.

More detailled description of the structure:

1. Header (32 bytes):
  * compresSQLite header string (13 bytes): "COMPRESSQLITE"
  * compresSQLite version (2 bytes): compresSQLite version number (e.g. 1)
  * SQLite version of the original database (4 bytes): SQLite version number (e.g. 3016002 for SQLite 3.16.2)
  * Reserved header space (13 bytes): Currently not used
2. SQLite database description: Contains table descriptions for all tables in the SQLite database.
  * 2a.    BC CREATE stmt (2 bytes): Byte count of table CREATE statement.
  * 2b.    CREATE stmt (variable): The table CREATE statement with column names and types. Also holds information about the primary keys, unique constraints and other table features like FTS or Spellfix1.
  * 2c.    Number of cols (2 bytes): Number of columns in the table.
  * 3\.    Desciption of all table columns.
  * 4\.    Desciption of all table indices.
3. Table column descriptions
  * 3a.    BC col payload (4 bytes): Byte count of total column payload. Includes byte length of 3b, 3c, 3d and 3e.
  * 3b.    BC col name (2 bytes): Byte count of the column name.
  * 3c.    Col name (variable): Name of the column.
  * 3d.    Row Count (4 bytes): The amount of rows in the table.
  * 3e.    Col data (variable): Data contained in the table column including byte count.
  * 3e1.   BC data comnstant (4 bytes): Byte count of each column entry, if the data length of each entry is constant. Otherwise set to zero ("0000").
  * [3e2]. BC data dynamic (4 bytes)[optional]: Byte count of the following column entry, if the data length of each cell is variable. Only used when 3e1 is set to zero ("0000").
  * 3e3.   data: The data of each column entry. If 3e1 is set to zero ("0000"), each entry is preceeded by its byte count.
4. Table index descriptions:
  * 4a. BC index CREATE stmt (2 bytes): Byte count of index CREATE statement.
  * 4b. Index CREATE stmt (Variable): The index CREATE statement.
  * 4c. End of Index marker (2 bytes): Marks end of index description. Set to zero ("00").

All defined limits are derived from the SQLite implementation limitations (https://www.sqlite.org/limits.html) and therefore do not add further limitations to the processed database.
