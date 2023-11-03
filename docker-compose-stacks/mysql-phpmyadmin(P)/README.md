# MySQL-PHPMyAdmin Docker Compose file

## Notes

- Always set the root password for both database and phpmyadmin
- If changing the root password after deployment, delete the persistant volume then change the password and redeploy.
- Change the database port id there are clashes with port
