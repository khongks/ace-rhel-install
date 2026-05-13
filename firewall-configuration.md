## Open Firewall ports on the Linux machine for App Connect Enterprise

1. Check if Firewall Daemon is running 
   ```
   sudo firewall-cmd --state
   ```

1. If it is running, you need to check the rules for the ports.

   - List ports
   ```
   sudo firewall-cmd --list-ports
   ```

   - List all configurations
   ```
   sudo firewall-cmd --list-all
   ```

   - Check permanent configuration
   ```
   firewall-cmd --permanent --list-all
   ```

1. Add ports for App Connect, 4414 for Console, 7800-7805 for HTTP, 7843-7848 for HTTPS, 7750-7755 for MCP 
   ```
   sudo firewall-cmd --permanent --add-port=4414/tcp
   sudo firewall-cmd --permanent --add-port=7800-7805/tcp
   sudo firewall-cmd --permanent --add-port=7843-7848/tcp
   sudo firewall-cmd --permanent --add-port=7750-7755/tcp
   ```

1. Reload the firewall
   ```
   sudo firewall-cmd --reload
   ```
