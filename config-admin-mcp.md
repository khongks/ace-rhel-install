# Configure administrative MCP tools

1. For integration node, edit node.conf.yaml. 
   ```
   vi /var/mqsi/components/INODE01/overrides/node.conf.yaml
   ```
   or edit server.conf.yaml (standalone server).
   ```
   vi server.conf.yaml
   ```

1. Add this section in the file. You can see readOnly to `false` or `true`.
   
   Integration node.
   ```
   MCP:
     Admin:
       enabled: true
       port: 4450
       readOnly: false
   ```
   Standalone Integration server
   ```
   MCP:
     Admin:
       enabled: true
       port: 7650
       readOnly: false
   ```

1. Available tools are listed [here](https://www.ibm.com/docs/en/app-connect/13.0.x?topic=overview-built-in-admin-mcp-tools).

   - info
   - list_integrations
   - list_application_needs
   - list_policies
   - list_credentials
   - list_integration_servers
   - describe_integration_server
   - describe_message_flow

1. Go to WebAdmin UI, click on `configure admin MCP` in the top right.

1. Configure Admin MCP with port number 4450.

1. Select Admin Tools

1. Connect to the MCP endpoint using Streamable HTTP as the transport protocol.
   ```
   https://hostname:4450/mcp
   ```