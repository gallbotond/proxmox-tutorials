### Expanded Summary of Issues, Solutions, and File Locations

Go into the docker container for some of the commands below.

```bash
docker exec -it ix-nextcloud-nextcloud-1 bash
```

1. **Maintenance Window Configuration**  
   - **Issue**: No maintenance window start time configured, leading to resource-intensive background jobs running during peak usage hours.
   - **Solution**:  
     Add the following line to the `config.php` file to schedule background jobs during low-usage hours (1 AM - 5 AM UTC):
     ```php
     'maintenance_window_start' => 1,
     ```
     - **File Location**: `/var/www/html/config/config.php`

2. **Mimetype Migrations**  
   - **Issue**: Pending mimetype migrations for database performance optimization.
   - **Solution**:  
     Run the following command inside the Nextcloud container to apply the missing migrations:
     ```bash
     php occ maintenance:repair --include-expensive
     ```
     
     Use this command if outside the container:
     ```bash
     docker exec -u 33 -it ix-nextcloud-nextcloud-1 php occ maintenance:repair --include-expensive
     ```

3. **HTTP Strict Transport Security (HSTS)**  
   - **Issue**: The `Strict-Transport-Security` header is not set, potentially exposing your Nextcloud instance to man-in-the-middle attacks.
   - **Solution**:  
     For **Apache**, add the following to the relevant `VirtualHost` configuration to enforce HSTS:
     ```apache
     <VirtualHost *:443>
       ServerName your-domain.com
       <IfModule mod_headers.c>
         Header always set Strict-Transport-Security "max-age=15552000; includeSubDomains; preload"
       </IfModule>
     </VirtualHost>
     ```
     For **Nginx**, add the following to the `server` block for your Nextcloud instance:
     ```nginx
     server {
       listen 443 ssl;
       server_name your-domain.com;
       add_header Strict-Transport-Security "max-age=15552000; includeSubDomains; preload" always;
     }
     ```
     - **File Locations**:  
       - Apache: `/etc/apache2/sites-available/000-default.conf` (or similar)
       - Nginx: `/etc/nginx/conf.d/default.conf` (or similar)

4. **Missing Optional Indices**  
   - **Issue**: Missing database indices (`fs_storage_path_prefix` in `filecache`, `systag_by_objectid` in `systemtag_object_mapping`) affecting performance.
   - **Solution**:  
     Run the following command inside the Nextcloud container to add the missing indices:
     ```bash
     docker exec -u www-data -it ix-nextcloud-nextcloud-1 php occ db:add-missing-indices
     ```
     You can also check the commands with `--dry-run` before applying.
