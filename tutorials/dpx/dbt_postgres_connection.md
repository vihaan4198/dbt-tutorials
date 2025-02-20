To configure a PostgreSQL connection in the `profiles.yml` file for dbt, use the following format:

```yaml
your_profile_name:
  target: dev
  outputs:
    dev:
      type: postgres
      host:database-2-instance-1.crc4oy8yyk9n.ap-south-1.rds.amazonaws.com
      port: 5432 
      user: postgres
      pass: >jg5KyiCk[N#-Ro?S9popM:Yp!sN
      dbname: your_database_name
      schema: your_schema_name
      threads: 4  # Adjust based on your needs
      keepalives_idle: 0  # Optional, set TCP keepalive interval
```

### Steps to Configure:
1. Replace `your_profile_name` with the profile name you will use in `dbt_project.yml`.
2. Update `host`, `port`, `user`, `pass`, `dbname`, and `schema` with your actual PostgreSQL connection details.
3. Ensure the `profiles.yml` file is located in `~/.dbt/` (for Unix-based OS) or `%USERPROFILE%\.dbt\` (for Windows).
4. Test the connection using:
   ```sh
   dbt debug
   ```
